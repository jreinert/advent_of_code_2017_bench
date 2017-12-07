require 'yaml'
require 'logger'
require 'fileutils'

PARTICIPANTS_FILE = File.expand_path('../participants.yml', __FILE__)
LEADER_BOARD_FILE = File.expand_path('../leader_board.md', __FILE__)
EXPECTED_OUTPUT_FILE = File.expand_path('../expected_output.yml', __FILE__)
INPUT_FILES = Dir[File.expand_path('../input/*/*', __FILE__)].sort
EXPECTED_OUTPUT = File.open(EXPECTED_OUTPUT_FILE, &YAML.method(:safe_load))
LOGGER = Logger.new(STDERR)

participants = []
benchmark_results = {}
all_time_results = {}

# rubocop:disable BlockLength

File.open(PARTICIPANTS_FILE) do |f|
  YAML.safe_load(f).each do |participant, options|
    participants << participant
    repo = options['repo']
    language = options['lang']
    id = "[#{participant}](#{repo}) [#{language}]"

    namespace participant do
      base_path = File.expand_path("../tmp/#{participant}", __FILE__)

      desc 'run benchmarks for participant'
      task run_benchmarks: [:build] do
        IO.popen(["#{base_path}/bin/benchmark", *INPUT_FILES]) do |process|
          process.each do |line|
            day, part, result, usecs = line.split(/\s+/)
            day = Integer(day)
            part = Integer(part)

            benchmark_results[day] ||= {}
            benchmark_results[day][part] ||= {}
            if result != EXPECTED_OUTPUT[day - 1][part - 1].to_s
              LOGGER.warn(participant) do
                "Day #{day} part #{part} output differs from expected output " \
                '- Skipping'
              end
              next
            end

            millis = Integer(usecs) / 1000.0
            all_time_results[id] ||= []
            all_time_results[id] << millis
            benchmark_results[day][part][id] = millis
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

        sorted_results = results_for_part.sort_by { |_, result| result }
        sorted_results.each_with_index do |(id, result), index|
          rank = index + 1
          f.print "#{rank}. #{id} (#{result.round(2)} ms"
          if index > 0
            factor = (result / sorted_results[0][1]).round(2)
            f.print " - #{factor} × slower"
          end

          f.puts ')'
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
      f.puts "#{rank}. #{participant} (#{results.size} parts submitted)"
    end

    f.puts
  end
end
