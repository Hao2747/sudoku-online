## Tdoku: A fast Sudoku Solver and Generator

#### Overview
This project contains an optimized Sudoku solver and puzzle generator for conventional 9x9 puzzles (as well as Sukaku
"pencilmark" puzzles with clues given as negative instead of positive literals). It also contains two 
other solvers with several variations exploring different ideas for optimization visited during
development. Lastly, it contains a benchmarking framework for comparing performance among these 
solvers and among several third party solvers of current or historical interest using a variety of
datasets. 

See [https://t-dillon.github.io/tdoku](https://t-dillon.github.io/tdoku) for a long discussion of the thinking behind these solvers.

This project's solvers include:

Solver | Description
-------|------------
[src/solver_dpll_triad_simd.cc](https://github.com/t-dillon/tdoku/blob/master/src/solver_dpll_triad_simd.cc) | The fast solver, optimized with particular focus on speed of solving hard Sudoku instances, and by this measure the fastest of any solver I'm aware of. I'll refer to this one as "Tdoku".
[src/solver_dpll_triad_scc.cc](https://github.com/t-dillon/tdoku/blob/master/src/solver_dpll_triad_scc.cc) | A DPLL-based solver for exploring how the puzzle representation can be optimized and strongly connected components can be exploited to reduce backtracking.
[src/solver_basic.cc](https://github.com/t-dillon/tdoku/blob/master/src/solver_basic.cc) | A very simple solver. Fast as simple solvers go, but mainly here to illustrate how poorly such solvers handle hard puzzles.

See [here](https://github.com/t-dillon/tdoku/blob/master/other/README.md) for details on third party solvers
supported for benchmarking. For a glimpse of comparative performance at opposite ends of the difficulty 
spectrum, here are results for a range of benchmarked solvers on a dataset of ~49,000 very
difficult puzzles with Sudoku Explainer ratings of 11 or higher:

<pre>
|data/puzzles4_forum_hardest_1905_11+  |  puzzles/sec|  usec/puzzle|   %no_guess|  guesses/puzzle|
|--------------------------------------|------------:|------------:|-----------:|---------------:|
|minisat_augmented          S/shrc+./m+|       518.1 |     1,930.3 |       0.0% |         104.38 |
|fast_solv_9r2              E/sh..../m.|     2,291.5 |       436.4 |       0.0% |         171.55 |
|kudoku                     E/sh..../m.|     2,280.3 |       438.5 |       0.0% |         142.29 |
|norvig                     C/sh..../m.|       398.3 |     2,510.7 |       0.0% |         178.70 |
|bb_sudoku                  C/shrc../m.|     5,709.9 |       175.1 |       0.0% |         200.41 |
|fsss                       C/shrc../m.|     6,490.3 |       154.1 |       0.0% |         117.52 |
|jsolve                     C/shrc../m.|     7,245.6 |       138.0 |       0.0% |         100.21 |
|fsss2                      D/sh..../m.|    10,045.0 |        99.6 |       0.0% |         139.23 |
|jczsolve                   B/shr.../m.|    11,741.6 |        85.2 |       0.0% |         171.20 |
|sk_bforce2                 B/shrc-./m+|    13,084.3 |        76.4 |       0.0% |         122.64 |
|<b>tdoku                      T/shrc+./m+|    23,592.4 |        42.4 |       0.0% |          64.95 </b>|
</pre>

And here are results on the well-known and commonly-benchmarked dataset of ~49,000 generally very easy 17-clue puzzles:

<pre>
|data/puzzles1_17_clue                 |  puzzles/sec|  usec/puzzle|   %no_guess|  guesses/puzzle|
|--------------------------------------|------------:|------------:|-----------:|---------------:|
|minisat_augmented          S/shrc+./m+|     5,059.6 |       197.6 |      76.0% |           1.06 |
|fast_solv_9r2              E/sh..../m.|    37,871.7 |        26.4 |      44.6% |           4.47 |
|kudoku                     E/sh..../m.|    38,060.5 |        26.3 |      44.6% |           4.57 |
|norvig                     C/sh..../m.|     8,523.6 |       117.3 |      44.6% |           4.84 |
|bb_sudoku                  C/shrc../m.|   132,124.8 |         7.6 |      76.0% |           1.55 |
|fsss                       C/shrc../m.|   190,709.8 |         5.2 |      76.0% |           0.94 |
|jsolve                     C/shrc../m.|   188,429.3 |         5.3 |      76.0% |           0.77 |
|fsss2                      D/sh..../m.|   185,972.9 |         5.4 |      44.6% |           4.46 |
|jczsolve                   B/shr.../m.|   272,192.5 |         3.7 |      70.5% |           1.76 |
|sk_bforce2                 B/shrc-./m+|   345,550.1 |         2.9 |      74.1% |           1.02 |
|<b>tdoku                      T/shrc+./m+|   364,370.8 |         2.7 |      78.7% |           0.61 </b>|
</pre>

For configuration and full details of the runs used for this comparison, [see here](https://github.com/t-dillon/tdoku/tree/master/benchmarks/results_i7-1065G7/i7-1065G7_clang-8_O3_native).

Here is a chart comparing a narrower set of the fastest solvers on a wider range of datasets 
ordered roughly from easiest to hardest, and for each solver using the results from its most 
favorable tested compiler and compiler options:

![](https://docs.google.com/spreadsheets/d/e/2PACX-1vRrWT05pUsB0LRS8ZR-j7WNvoUIpX6TDHBGeWhJnd7bRedgNn-a60TLVIRYO9A51yUZuXo-ugWx-ibK/pubchart?oid=1741583019&format=image)


#### Building and Running

Build this project's solvers and run them through benchmarks as follows:

```bash
unzip data.zip
./BUILD.sh
./build/run_tests
./build/run_benchmark data/*
```
Building the project also produces a library containing the fast simd solver.  You can build a 
simple test program that reads Sudoku (or 729-character pencilmark Sudoku) from stdin and displays 
the solution count and solution (if unique) like so:

```bash
gcc example/solve.c build/libtdoku.a -O3 -o solve -lstdc++ -lm
# count solutions:
./solve < data/puzzles0_kaggle
# find single solution:
./solve 1 < data/puzzles0_kaggle
```

#### Benchmarking Other Solvers

This project is set up to facilitate benchmarking against a number of the fastest known solvers, as
well as some solvers of historical interest. Also included is a solver based on minisat and able to
test several different SAT encodings. 
Follow the [instructions here](https://github.com/t-dillon/tdoku/blob/master/other/README.md) to find
and set up sources where necessary.

With sources set up, the benchmarks [found here](https://github.com/t-dillon/tdoku/tree/master/benchmarks) were run 
using [BENCH.sh](https://github.com/t-dillon/tdoku/blob/master/BENCH.sh) by specifying an architecture, taskset CPU mask, and list of compiler/flag specifications as
in the following example:

```bash
./BENCH.sh i7-4930k 0x8 gcc-6_O3_native clang-8_O3_native clang-8_O3_native_pgo ...
```

See CMakeLists.txt for the set of -D options to pass to BUILD.sh if you want to build and benchmark
all supported solvers. See the benchmark program's help for details of how to run, solvers that
have been built, and build info:

```
$ build/run_benchmark -h
usage: run_benchmark <options> puzzle_file_1 [...] 
options:
  -c [0|1]            // output csv instead of table [default 0]
  -e <seed>           // random seed [default random_device{}()]
  -n <size>           // test set size [default 2500000]
  -r [0|1]            // randomly permute puzzles [default 1]
  -s solver_1,...     // which solvers to run [default all]
  -t <secs>           // target test time [default 20]
  -v [0|1]            // validate during warmup [default 1]
  -w <secs>           // target warmup time [default 10]
solvers: 
 _tdev_basic _tdev_basic_heuristic minisat_minimal_01 minisat_natural_01 minisat_complete_01 
 minisat_augmented_01 _tdev_dpll_triad _tdev_dpll_triad_scc_i _tdev_dpll_triad_scc_h 
 _tdev_dpll_triad_scc_ih norvig fast_solv_9r2 kudoku bb_sudoku jsolve fsss2 fsss2_locked jczsolve 
 sk_bforce2 rust_sudoku tdoku gss gss[1-8]
build info: Clang 8.0.1 -O3 -march=native
```

If you'd like to add another solver to the benchmark suite, see [this commit](https://github.com/t-dillon/tdoku/commit/98b599074a00f15b7a13761053b984e237b8511a) for an example of
how to do so. If you have another interesting comparison I'd love to hear from you!

#### Generating Puzzles

Tdoku includes a program for generating both conventional and pencilmark Sudoku. The process is
governed by a customizable loss function that can drive towards low or high clue puzzles, hard
or easy puzzles, or some combination of goals.
```
$ build/generate -h
usage: generate <options> <pattern_file>

options:
  -c <clue_weight>    scoring weight for number of clues
  -g <guess weight>   scoring weight(exponent) for geo mean guesses
  -r <random weight>  scoring weight for uniform noise
  -d <drop>           number of clues to drop before re-completing
  -e <num_evals>      number of permutations to eval for guess estimate
  -n <pool size>      number of top scored puzzles to keep in pool
  -a [0|1]            display all puzzles (not just top scored)
  -m [0|1]            use minisat instead of tdoku for eval
  -p [0|1]            generate pencilmark puzzles
  -h                  display this help message
```
Examples:
```bash
# starting from 100 random conventional Sudoku puzzle seeds perform {-1+?} generation driving 
# towards hard puzzles (according to minisat evaluation of 50 random puzzle permutations). 
$ build/generate -p0 -c0 -g1 -d1 -n100 -e50

# starting from 100 random pencilmark Sudoku puzzle seeds perform {-2+?} generation driving  
# towards low-clue puzzles with a secondary emphasis on puzzle difficulty (according to minisat 
# evaluation of 5 random puzzle permutations). 
build/generate -c1 -g0.01 -r0 -d2 -n100 -e5
```
