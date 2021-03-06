# HPC Performance Analysis - Memory

This repository will be used for experiments with memory performance.

Modern CPU architectures have several levels of cache, each with different semantics.

## Experiments

### Day 1: 2015-01-12

#### 0. Setup and clone

Create a GitHub account, clone this repository, and configure your Git environment

    git clone https://yourname@github.com/CUBoulder-HPCPerfAnalysis/memory.git
    git config --global user.name 'Your Name'
    git config --global user.email your.email@colorado.edu

Feel free to use SSH instead on HTTPS.
If you use bash, I recommend downloading and sourcing [git-prompt](https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh) and [git-completion.bash](https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash).
Git resources:

* [Official Git Documentation](http://git-scm.com/documentation)
* [Git for Computer Scientists](http://eagain.net/articles/git-for-computer-scientists/)
* [Learn Git Branching (interactive)](https://pcottle.github.io/learnGitBranching/)
* https://try.github.io/
* [Interactive Cheat Sheet](http://ndpsoftware.com/git-cheatsheet.html)

#### 1. Compile and run the included [STREAM benchmark](http://www.cs.virginia.edu/stream/).

    make CC=gcc CFLAGS='-O3 -march=native' stream
    ./stream

#### 2. Add Dot to the results

Implement a "dot product" test (x ⋅ y = sum_i x_i y_i) and commit your changes.

#### 3. Create a CSV file at `results/yourname.csv` with the format

```
    # Username, Machinename, CPU name, CPU GHz, CPU Cores, CPU Cores used, Private cache/core (MB), Shared cache (MB), Array Length (MB), Peak MB/s, Copy MB/s, Scale MB/s, Add MB/s, Triad MB/s, Dot MB/s
    jed, batura, i7-4500U, 1.8, 2, 1, 0.256, 4.096, 76.3, 25600, 19555, 12784, 13483, 13677, ?
    ...
```

Report the "best rate".
On Linux systems, look in `/proc/cpuinfo`.
For theoretical peak bandwidth, try http://ark.intel.com.
Leave a `?` for missing data (but try not to have missing data).
Commit this new file:

    git add results/yourname.csv
    git commit

Use the commit message to explain your workflow creating the results file.
We're going to add more tests in the future, so automation would be good here (collaboration encouraged).

#### 4. Submit your changes as a pull request

Make a GitHub Fork [this repository](https://github.com/CUBoulder-HPCPerfAnalysis/memory/fork) on GitHub, push changes to your fork, and submit a pull request to the main repository.

#### 5. Open issues
There may be ambiguities in the specification.
If you spot any, [open an issue](https://github.com/CUBoulder-HPCPerfAnalysis/memory/issues).
Also open issues for ways to improve the workflow and provenance for running experiments and reporting data.

### Day 2: 2015-01-14

#### 0. Postmortem from Day 1

* How machine-readable is our data?
* Is all our data accurate?
* Are these results reproducible?
  * Did everyone use the same compiler?
  * Same compilation flags?  (Do compilation flags matter?)
  * Was anything else running?
* How well do humans follow instructions?
  * How much can we automate?

#### 1. Visualizing data

It is useful to have a modern statistical environment for analyzing and plotting data.
The [R Project](http://www.r-project.org/) is a widely used open source statistical package that compares well with commercial packages and has a great user repository (new statistical methods tend to show up here first).
Unfortunately, the R language has some shortcomings and is not general purpose.
[Pandas](http://pandas.pydata.org/) is an up-and-coming Python package that provides a "data frame", a suite of common statistical tools, and plotting similar to R.
I recommend Pandas for this class, but welcome you to use any package you feel comfortable with.
Experiment with plotting interesting relationships using the `stream-analyze.py` script.
The [Pandas visualization documentation](http://pandas.pydata.org/pandas-docs/stable/visualization.html) may be useful, as is the [IPython interpreter](http://ipython.org/).

#### 2. Effect of non-contiguous access

Prefetchers like to follow contiguous memory streams.
What happens to performance if we interleave threads?
The block-cyclic mapping of the range `0,1,...,N-1` defined by `j(i) = (i*b)%N + (i*b)//N` may be useful.
What happens if many threads try to write to the same cache line?
Can you measure the effect of [false sharing](https://en.wikipedia.org/wiki/False_sharing) ([longer article](http://simplygenius.net/Article/FalseSharing)), sometimes called "cache line contention".

Design an experiment to test cache behavior with multiple threads, run it to produce data, and make a plot using Pandas, R, or another plotting system.
Commit the source code changes, your data, the plotting script, and any figures you produce.
Describe what your experiment is testing and your interpretation of the data and figures in your commit message and submit as a pull request.
Plan to present these results (~5 minutes each) next class period (Wednesday, 2015-01-21).

### Day 3: 2015-01-21

#### 0. Postmortem and presentations

* Effect of array sizes
* Effect of interlacing/striding
* Relative cost of moving a cache line into memory multiple times versus having multiple cores write to it concurrently
* Practical issues

#### 1. Introduction to stencil operations

* Modeling reusable and non-reusable memory accesses
* Arithmetic Intensity
* Multiplicative (Gauss-Seidel) versus additive (Jacobi)
* Boundary conditions

### Day 4: 2015-01-26

#### 0. Experiences with stencil.c

* What experiments did you run?
* Did you try simplifying/modifying the code?
* Are boundary conditions a problem?
* Were divisions hoisted out of the inner loop?

#### 1. Introduction to profiling tools

* Gprof (comes with binutils, gcc support)

  * A simple tool with compiled-in function-level instrumentation.
  * Usage: compile with `-pg`, run application (which now writes `gmon.out`), and use `gprof executable gmon.out`
  * Good performance, accurate measure of function calls.
  * What was inlined?
  * Ways to prevent inlining.
  * [Gprof2Dot](https://code.google.com/p/jrfonseca/wiki/Gprof2Dot)

* [Valgrind](http://valgrind.org) (start with Callgrind)

  * Simulator -- accurate, reproducible, fine-grained, very slooooow
  * Usage: `valgrind --tool=callgrind ./program -args`
  * Nice visualizations with [KCachegrind](http://kcachegrind.sourceforge.net/html/Home.html)
  * Annotated source and assembly (`--dump-instr=yes`)

* [Linux Perf](https://perf.wiki.kernel.org/index.php/Main_Page)

  * Interrupt-based profiler, does not need special compilation.
  * [Brendan Gregg's Examples](http://www.brendangregg.com/perf.html) -- best place to start
  * Usage: `perf record ./program -args`, then `perf report`
  * Annotates assembly
  * Good for drilling into system issues.

* Other instrumentation systems

  * [Scalasca](http://www.scalasca.org/) -- open source, parallel support
  * [TAU](https://www.cs.uoregon.edu/research/tau/about.php) -- open source, parallel support
  * [Intel VTune](https://software.intel.com/en-us/intel-vtune-amplifier-xe/)
  * Sun Studio used to be freely distributed and include a profiler.

### Day 5: 2015-01-28

#### 0. Discussion

* Divisions not hoisted out of loop
* Can streamline conditionals
* Use attributes for fine-grained control of inlining
* Used Perf to check assembly

#### 1. How to optimize further

* Performance model is not in the ballpark for memory bandwidth
* Only smaller problem sizes can benefit much
* Vectorization should be able to speed up small sizes

### Day 6: 2015-02-02

#### 0. Experiments with stencils

#### 1. Performance models

* Express resource constraints as a linear program
* Start with floating point and memory bandwidth, related by arithmetic intensity
* Multiple phases
* Amdahl's Law

#### 2. Intro to iterative solvers

* (At least) two phases: matrix application and orthogonalization
* Preconditioning
* Scalability of local methods

#### 3. PETSc and Janus

* Install locally (if convenient). http://mcs.anl.gov/petsc
* Request an account and OTP device. https://rc.colorado.edu/support/userguide/accountrequest

## References

* [STREAM Benchmark](http://www.cs.virginia.edu/stream/)
* [Ulrich Drepper, *What Every Programmer Should Know About Memory*, 2007](http://www.akkadia.org/drepper/cpumemory.pdf)
* [Gustavo Duartes, *Cache: A Place for Concealment and Safekeeping*, 2009](http://duartes.org/gustavo/blog/post/intel-cpu-caches/)
* [Gustavo Duartes, *Getting Physical With Memory*, 2009](http://duartes.org/gustavo/blog/post/getting-physical-with-memory/)
* [John McCalpin's blog](http://sites.utexas.edu/jdm4372/) (mostly about memory performance)
* [Datta et al, *Optimization and Performance Modeling of Stencil Computations on Modern Microprocessors*, 2009](http://epubs.siam.org/doi/abs/10.1137/070693199)
* [Malas et al, *Optimizing the performance of streaming numerical kernels on the IBM Blue Gene/P PowerPC 450 processor*, 2012](http://dx.doi.org/10.1177/1094342012444795) ([arXiv](http://arxiv.org/pdf/1201.3496.pdf))

## Tools

* [hwloc](http://www.open-mpi.org/projects/hwloc/): Portable Hardware Locality
* [likwid](https://code.google.com/p/likwid/): x86 performance tools
* [Intel Intrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)
