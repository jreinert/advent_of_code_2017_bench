# Benchmarks for Advent of Code 2017

[Leaderboard](./leader_board.md)

## Participating

1. Check your code into git and push it to a publicly accessible remote
1. Make sure your repository contains a makefile which produces an executable
   file named `benchmark` inside a subdir called `bin`
1. Make sure your executable takes a list of files as arguments. Each file
   contains the puzzle input for a day and part e.g.

   ```
   bin/benchmark day_01_pt_01 day_01_pt_02 day_02_pt_01 day_02_pt_02 ...
   ```

   The executable should then produce the benchmark output in the following
   format:

   ```
   %<day>d %<part>d %<result>s %<time>d\n
   ```

   __NOTE__: the unit for `time` is microseconds. You can skip a part by
   returning `skip` as a result.
1. Fork this repo, add yourself to the [`participants.yml`](./participants.yml)
   file and make sure `rake <your_name>:run_benchmarks` runs without errors
   (you will have to have ruby installed for running `rake`)
1. Commit your changes, push and make a pull request with information about
   possibly needed dependencies for successful builds/runs.
