require 'yaml'
require 'logger'
require 'fileutils'

PARTICIPANTS_FILE = File.expand_path('../participants.yml', __FILE__)
LEADER_BOARD_FILE = File.expand_path('../leader_board.md', __FILE__)
DAYS = (1..25).map { |day| format('%02d', day) }

participants = []
benchmark_results = {}
all_time_results = {}

def input(day, part)
  file = File.expand_path("../input/#{day}/#{part}", __FILE__)
  File.read(file)
end

def expected_output(day, part)
  file = File.expand_path("../expected_output/#{day}/#{part}", __FILE__)
  File.read(file)
end

def benchmark(executable, part, input)
  started = Time.now

  result = IO.popen([executable, part.to_s], 'w+') do |program|
    program.print(input)
    program.close_write
    program.read
  end

  [Time.now - started, result]
end

LOGGER = Logger.new(STDERR)

# rubocop:disable BlockLength

File.open(PARTICIPANTS_FILE) do |f|
  YAML.safe_load(f).each do |participant, options|
    participants << participant
    repo = options['repo']
    language = options['lang']
    id = "#{participant} [#{language}]"

    namespace participant do
      base_path = File.expand_path("../tmp/#{participant}", __FILE__)

      desc 'run benchmarks for participant'
      task run_benchmarks: [:build] do
        DAYS.each do |day|
          benchmark_results[day] ||= {}

          executable = "#{base_path}/bin/#{day}"
          next unless File.exist?(executable)

          [1, 2].each do |part|
            benchmark_results[day][part] ||= {}

            time, output = benchmark(executable, part, input(day, part))
            if output != expected_output(day, part)
              LOGGER.warn(participant) do
                "Day #{day} part #{part} output differs from expected output " \
                '- Skipping'
              end

              next
            end

            all_time_results[id] ||= []
            all_time_results[id] << time
            benchmark_results[day][part][id] = time
          end
        end
      end

      task build: :fetch do
        system('make', '-C', base_path)
      end

      task fetch: [base_path] do
        FileUtils.cd(base_path) do
          system('git', 'fetch')
          system('git', 'reset', '--hard', 'origin/master')
        end
      end

      file base_path do
        system('git', 'clone', repo, base_path)
      end
    end
  end
end

desc 'Run benchmarks for all participants and display results'
task run_benchmarks: participants.map { |p| "#{p}:run_benchmarks" } do
  File.open(LEADER_BOARD_FILE, 'w+') do |f|
    f.puts '# Leader Board'

    benchmark_results.each do |day, results_for_day|
      next if results_for_day.empty?

      f.puts "## Day #{day}"
      f.puts

      results_for_day.each do |part, results_for_part|
        next if results_for_part.empty?

        f.puts "### Part #{part}"
        f.puts

        results_for_part.sort.each_with_index do |(id, result), index|
          rank = index + 1
          millis = (result * 1000).round(1)
          f.puts "#{rank}. #{id} (#{millis} ms)"
        end

        f.puts
      end
    end

    f.puts '## All Time'
    f.puts

    all_results = all_time_results.sort_by do |_, results|
      results.reduce(:+) / results.size**2
    end

    all_results.each_with_index do |(participant, results), index|
      rank = index + 1
      days = results.size / 2
      f.puts "#{rank}. #{participant} (#{days} days submitted)"
    end

    f.puts
  end
end
