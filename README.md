# Benchmarks for Advent of Code 2017

[Leaderboard](./leader_board.md)

## Participating

1. Check your code into git and push it to a publicly accessible remote
2. Make sure your repository contains a makefile which produces executable
   files named after the zero padded day of the challenge in a subdir called
   `bin` e.g. `bin/01`, `bin/02`, `bin/03` etc.
3. Make sure your executables take the puzzle input from stdin, take one
   argument `part no.` and produce the output for the given input and part of
   the challenge .
4. Fork this repo, add yourself to the [`participants.yml`](./participants.yml)
   file and make sure `rake <your_name>:run_benchmarks` runs without errors
   (you will have to have ruby installed for running `rake`)
5. Commit your changes, push and make a pull request with information about
   possibly needed dependencies for successful builds/runs.
