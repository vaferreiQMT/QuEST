# 🚀  Launching

<!--
  Instructions for running QuEST with different parallelisations
  (this comment must be under the title for valid doxygen rendering)

  @author Tyson Jones
-->

Launching your [compiled](compile.md) QuEST application can be as straightforward as running any other executable, though some additional steps are needed to make use of hardware acceleration. This page how to launch your own QuEST applications on different platforms, how to run the examples and unit tests, how to make use of multithreading, GPU-acceleration, distribution and supercomputer job schedulers, and monitor the hardware utilisation.


> **TOC**:
> - [Examples](#launch_examples)
> - [Tests](#launch_tests)
> - [Multithreading](#launch_multithreading)
>    * [Monitoring utilisation](#launch_monitoring-utilisation)
>    * [Improving performance](#launch_improving-performance)
> - [GPU-acceleration](#launch_gpu-acceleration)
>    * [Launching](#launch_launching)
>    * [Monitoring](#launch_monitoring)
>    * [Benchmarking](#launch_benchmarking)
> - [Distribution](#launch_distribution)
>    * [Launching](#launch_launching-1)
>    * [Benchmarking](#launch_benchmarking-1)
> - [Multi-GPU](#launch_multi-gpu)
> - [Supercomputers](#launch_supercomputers)
>    * [SLURM](#launch_slurm)
>    * [PBS](#launch_pbs)




> [!NOTE]
> This page assumes you are working in a `build` directory into which all executables have been compiled.



---------------------


## Examples {#launch_examples}

> See [<code>compile.md</code>](#compile_examples) for instructions on compiling the examples.


The example source codes are located in [`examples/`](/examples/) with structure
```
examples/
    isolated/
        complex_arithmetic.c
        complex_arithmetic.cpp
        ...
    extended/
        dynamics.c
        dynamics.cpp
        ...
    tutorials/
        min_example.c
        ...
```
where `file.c` and `file.cpp` respectively demo QuEST's `C11` and `C++14` interfaces. Files are divided between subdirectories:
 - `isolated/` which contains demos of _one_ function or task, typically showcasing all the different syntaxes and interfaces available.
 - `extended/` which contains longer, standalone examples performing common tasks and algorithms in quantum computing. 
 - `tutorials/` which contains guides with step-by-step explanations.


These files are [compiled](#compile_examples) into executables of the same name, respectively suffixed with `_c` or `_cpp`, and (by default) are saved in subdirectories of `build` which mimic the structure of `examples/`. E.g.
```
build/
    examples/
        isolated/
            complex_arithmetic_c
            complex_arithmetic_cpp
            ...
        extended/
            dynamics_c
            dynamics_cpp
            ...
        tutorials/
            min_example_c
```

> [!NOTE]  
> On Windows, the executables are located in `\Release\` subdirectories, assuming the parameter 
> `--config Release` was specified during compilation (see [compiled](#compile_optimising)).

Most of these executables can be run directly from within `build`, e.g.
```bash
./examples/extended/dynamics_c
```
while others require command-line arguments:
```bash
./examples/extended/reporting_environments_cpp

# output
Must pass single cmd-line argument:
  1 = serial
  2 = multithreaded
  3 = GPU-accelerated
  4 = distributed
  5 = all
  6 = auto
```

> [!NOTE]  
> On Windows, the executables are `.exe` files and can be run with e.g.
> ```
> \examples\isolated\Release\initialising_paulis_c.exe
> ```


---------------------


## Tests {#launch_tests}

> See [<code>compile.md</code>](#compile_tests) for instructions on compiling the unit tests.


QuEST's unit and integration tests are compiled into executable `tests` within the `tests/` subdirectory, and can be directly run from within the `build` folder via
```bash
./tests/tests
```
which should, after some time, output something like
```
QuEST execution environment:
  precision:       2
  multithreaded:   1
  distributed:     1
  GPU-accelerated: 1
  cuQuantum:       1
  num nodes:       16
  num qubits:      6
  num qubit perms: 10

Tested Qureg deployments:
  GPU + MPI

Randomness seeded to: 144665856
===============================================================================
All tests passed (74214 assertions in 240 test cases)
```

This `tests` binary accepts all of the [Catch2 CLI arguments](https://github.com/catchorg/Catch2/blob/devel/docs/command-line.md), for example to run specific tests
```bash
./tests/tests applyHadamard
```
or all tests within certain groups
```bash
./tests/tests "[qureg],[matrices]"
```
or specific test sections and subsections:
```bash
./tests/tests -c "validation" -c "matrix uninitialised"
```

If the tests were compiled with [distribution enabled](#compile_distribution), they can distributed via
```bash
mpirun -np 8 ./tests/tests
```

Alternatively, the tests can be run through [CTest](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Testing%20With%20CMake%20and%20CTest.html) within the `build` directory via either
```bash
ctest
```
```bash
make test
```
which will log each passing test live, outputting something like
```
Test project /build
        Start   1: calcExpecPauliStr
  1/240 Test   #1: calcExpecPauliStr .............................   Passed   14.03 sec
        Start   2: calcExpecPauliStrSum
  2/240 Test   #2: calcExpecPauliStrSum ..........................   Passed   10.06 sec
        Start   3: calcExpecNonHermitianPauliStrSum
  3/240 Test   #3: calcExpecNonHermitianPauliStrSum ..............   Passed   10.34 sec
        Start   4: calcProbOfBasisState
  4/240 Test   #4: calcProbOfBasisState ..........................   Passed    0.33 sec
        Start   5: calcProbOfQubitOutcome
  5/240 Test   #5: calcProbOfQubitOutcome ........................   Passed    0.12 sec
        Start   6: calcProbOfMultiQubitOutcome
  6/240 Test   #6: calcProbOfMultiQubitOutcome ...................   Passed   15.07 sec
        Start   7: calcProbsOfAllMultiQubitOutcomes
...
```
Alas tests launched in this way cannot be deployed with distribution.


---------------------


## Multithreading {#launch_multithreading}

> [!NOTE]
> Parallelising QuEST over multiple cores and CPUs requires first compiling with 
> multithreading enabled, as detailed in [<code>compile.md</code>](#compile_multithreading). 

### Monitoring utilisation {#launch_monitoring-utilisation}


The availability of multithreaded deployment can also be checked at runtime using [`reportQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#ga08bf98478c4bf21b0759fa7cd4a97496), which outputs something like:
```bash
  [compilation]
    isOmpCompiled...........1
  [deployment]
    isOmpEnabled............1
```
where `Omp` signifies OpenMP and the two `1` respectively indicate it has been compiled and runtime enabled.

Like all programs, the CPU utilisation of a running QuEST program can be viewed using

| OS    | Program | Method |
| -------- | ------- | ---- |
| Linux  | [HTOP](https://htop.dev/) | Run `htop` in terminal  |
| MacOS |  [Activity Monitor](https://support.apple.com/en-gb/guide/activity-monitor/welcome/mac) | Place on dock > right click icon > `Monitors` > `Show CPU usage` (see [here](https://stackoverflow.com/questions/50260592/getting-each-of-the-cpu-cores-usage-via-terminal-in-macos)) |
| Windows  |  [Task Manager](https://learn.microsoft.com/en-us/shows/inside/task-manager) |`Performance` > `CPU` > right click graph > `Change graph to` > `Logical processors` (see [here](https://superuser.com/questions/1398696/how-to-see-usage-of-each-core-in-windows-10)) |


Note however that QuEST will not leverage multithreading at runtime when either:
- Your `Qureg` created with [`createQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#gab3a231fba4fd34ed95a330c91fcb03b3) was too small to invoke automatic multithreading.
- You called [`initCustomQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#ga485268e52f838743357e7a4c8c241e57) and disabled multithreading for all subsequently-created `Qureg`.

Usage of multithreading can be (inadvisably) forced using [`createForcedQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#ga619bbba1cbc2f7f9bbf3d3b86b3f02be) or [`createCustomQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#ga849971f43e246d103da1731d0901f2e6).


### Improving performance {#launch_improving-performance}

Performance may be improved by setting other [OpenMP variables](https://www.openmp.org/spec-html/5.0/openmpch6.html). Keep in mind that for large `Qureg`, QuEST's runtime is dominated by the costs of modifying large memory structures during long, uninterrupted loops: namely the updating of statevector amplitudes. Some sensible settings include

- [`OMP_DYNAMIC`](https://www.openmp.org/spec-html/5.0/openmpse51.html) `=false` to disable the costly runtime migration of threads between cores.
- [`OMP_PROC_BIND`](https://www.openmp.org/spec-html/5.0/openmpse52.html) `=spread` to (attemptedly) give threads their own caches (see [here](https://developer.arm.com/documentation/102580/0100/Control-the-placement-of-OpenMP-threads)).
  > Replace this with [`KMP_AFFINITY`](https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2023-0/thread-affinity-interface.html) on Intel compilers.
- [`OMP_PLACES`](https://www.openmp.org/spec-html/5.0/openmpse53.html) `=threads` to allocate each spawned thread to a CPU hardware thread.
  > Alternatively set `=cores` to assign one thread per core, helpful when the hardware threads interfere (e.g. due to caching conflicts).


OpenMP experts may further benefit from knowing that QuEST's multithreaded source code, confined to [`cpu_subroutines.cpp`](/quest/src/cpu/cpu_subroutines.cpp), is almost exclusively code similar to
```cpp
#pragma omp parallel for if(qureg.isMultithreaded)
for (qindex n=0; n<numIts; n++)
```
```cpp
#pragma omp parallel for reduction(+:val)
for (qindex n=0; n<numIts; n++)
    val += 
```
and never specifies [`schedule`](https://rookiehpc.org/openmp/docs/schedule/index.html) nor invokes setters in the [runtime library routines](https://www.openmp.org/spec-html/5.0/openmpch3.html). As such, all behaviour can be strongly controlled using environment variables, for example by:

- [`OMP_SCHEDULE`](https://www.openmp.org/spec-html/5.0/openmpse49.html)
- [`OMP_DEFAULT_DEVICE`](https://www.openmp.org/spec-html/5.0/openmpse63.html#x302-20790006.15)
- [`OMP_THREAD_LIMIT`](https://www.openmp.org/spec-html/5.0/openmpse58.html#x297-20700006.10)



> [!TIP]
> Sometimes the memory bandwidth between different sockets of a machine is poor, and it is substantially better to exchange memory in bulk between their NUMA nodes, rather than through repeated random access. In such settings, it can be worthwhile to hybridise multithreading and distribution, even upon a single machine, partitioning same-socket threads into their own MPI node. This forces inter-socket communication to happen in-batch, via message-passing, at the expense of using _double_ total memory (to store buffers). See the [distributed](#launch_distribution) section.



---------------------


## GPU-acceleration {#launch_gpu-acceleration}

> [!NOTE]
> Using GPU-acceleration requires first compiling QuEST with `CUDA` or `HIP` enabled (to utilise NVIDIA and AMD GPUs respectively) as detailed in [<code>compile.md</code>](#compile_gpu-acceleration).


### Launching {#launch_launching}

The compiled executable is launched like any other, via
```bash
./myexec
```

Using _multiple_ available GPUs, regardless of whether they are local or distributed, is done through additionally enabling [distribution](#launch_multi-gpu).


### Monitoring {#launch_monitoring}


To runtime check whether GPU-acceleration was compiled and is being actively utilised, call [`reportQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#ga08bf98478c4bf21b0759fa7cd4a97496).
This will display a subsection like
```
  [compilation]
    isGpuCompiled...........1
  [deployment]
    isGpuEnabled............1
```
where the `1` indicate GPU-acceleration was respectively compiled and is runtime available (i.e. QuEST has found suitable GPUs). When this is the case, another section will be displayed detailing the discovered hardware properties, e.g.
```
  [gpu]
    numGpus...........2
    gpuDirect.........0
    gpuMemPools.......1
    gpuMemory.........15.9 GiB per gpu
    gpuMemoryFree.....15.6 GiB per gpu
    gpuCache..........0 bytes per gpu
```

Utilisation can also be externally monitored using third-party tools:


| GPU  | Type | Name |
| -------- | ------- | ---- |
| NVIDIA | CLI | [`nvidia-smi`](https://docs.nvidia.com/deploy/nvidia-smi/index.html) |
| NVIDIA | GUI | [Nsight](https://developer.nvidia.com/nsight-systems) |
| AMD | CLI, GUI | [`amdgpu_top`](https://github.com/Umio-Yasuno/amdgpu_top) |



Note however that GPU-acceleration might not be leveraged at runtime when either:
- Your `Qureg` created with [`createQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#gab3a231fba4fd34ed95a330c91fcb03b3) was too small to invoke automatic GPU-acceleration.
- You called [`initCustomQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#ga485268e52f838743357e7a4c8c241e57) and disabled GPU-acceleration for all subsequently-created `Qureg`.

Usage of GPU-acceleration can be (inadvisably) forced using [`createForcedQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#ga619bbba1cbc2f7f9bbf3d3b86b3f02be) or [`createCustomQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#ga849971f43e246d103da1731d0901f2e6).






### Benchmarking {#launch_benchmarking}

Beware that the CPU dispatches tasks to the GPU _asynchronously_. Control flow returns immediately to the CPU, which will proceed to other duties (like dispatching the next several quantum operation's worth of instructions to the GPU) while the GPU undergoes independent computation (goes _brrrrr_).
This has no consequence to the user who uses only the QuEST API, which will automatically synchronise the CPU and GPU when necessary (like inside functions [`calcTotalProb()`](https://quest-kit.github.io/QuEST/group__calc__properties.html#gab082910d33473ec29e1d5852943de468)).

However, it _does_ mean codes which seeks to benchmark QuEST must be careful to _wait for the GPU to be ready_ before beginning the stopwatch, and _wait for the GPU to finish_ before stopping the stopwatch. This can be done with the [`syncQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#gaaa19c3112f1ecd80e3296df5c0ed058d), which incidentally also ensures nodes are synchronised when distributed.


---------------------


## Distribution {#launch_distribution}


> [!NOTE]
> Distributing QuEST over multiple machines requires first compiling with 
> distribution enabled, as detailed in [<code>compile.md</code>](#compile_distribution). 

> [!IMPORTANT]
> Simultaneously using distribution _and_ GPU-acceleration introduces additional considerations detailed in the [proceeding section](#launch_multi-gpu).


### Launching {#launch_launching-1}

A distributed QuEST executable called `myexec` can be launched and distributed over (e.g.) `32` nodes using [`mpirun`](https://www.open-mpi.org/doc/v4.1/man1/mpirun.1.php) with the 
```bash
mpirun -np 32 ./myexec
```
or on some platforms (such as with Intel and Microsoft MPI):
```bash
mpiexec -n 32 myexec.exe
```

Some supercomputing facilities however may require custom or additional commands, like [SLURM](https://slurm.schedmd.com/documentation.html)'s [`srun`](https://slurm.schedmd.com/srun.html) command. See an excellent guide [here](https://docs.lumi-supercomputer.eu/runjobs/scheduled-jobs/distribution-binding/#distribution), and the job submission guide [below](#launch_supercomputers).
```bash
srun --nodes=8 --ntasks-per-node=4 --distribution=block:block
```



> [!IMPORTANT]
> QuEST can only be distributed with a _power of `2`_ number of nodes, i.e. `1`, `2`, `4`, `8`, `16`, ...

> [!NOTE]
> When [multithreading](#launch_multithreading) is also enabled, the environment variable `OMP_NUM_THREADS` 
> will determine how many threads are used by _each node_ (i.e. each MPI process). Ergo optimally
> deploying to `8` machines, each with `64` CPUs (a total of `512` CPUs), might resemble:
> ```bash
> OMP_NUM_THREADS=64 mpirun -np 8 ./myexec
> ```



It is sometimes convenient (mostly for testing) to deploy QuEST across more nodes than there are available machines and sockets, inducing a gratuitous slowdown. Some MPI compilers like [OpenMPI](https://www.open-mpi.org/) forbid this by default, requiring additional commands to permit [oversubscription](https://docs.open-mpi.org/en/main/launching-apps/scheduling.html).
```bash
mpirun -np 1024 --oversubscribe ./mytests
```



### Benchmarking {#launch_benchmarking-1}

QuEST strives to reduce inter-node communication when performing distributed simulation, which can otherwise dominate runtime. Between these rare communications, nodes work in complete independence and are likely to desynchronise, especially when performing operations with non-uniform loads. In fact, many-controlled quantum gates are skipped by non-participating nodes which would otherwise wait idly!

Nodes will only synchronise when forced by the user (with [`syncQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#gaaa19c3112f1ecd80e3296df5c0ed058d)), or when awaiting necessary communication (due to functions like [`calcTotalProb()`](https://quest-kit.github.io/QuEST/group__calc__properties.html#gab082910d33473ec29e1d5852943de468)). Furthermore, `Qureg` created with [`createQureg()`](https://quest-kit.github.io/QuEST/group__qureg__create.html#gab3a231fba4fd34ed95a330c91fcb03b3) will automatically disable distribution (and be harmlessly cloned upon every node) when they are too small to outweigh the performance overheads.

This can make monitoring difficult; CPU loads on different nodes can correspond to different stages of execution, and memory loads may fail to distinguish whether a large `Qureg` is distributed or a small `Qureg` is duplicated! Further, a node reaching the end of the program and terminating does not indicate the simulation has finished - other desynchronised nodes may still be working.

It is ergo always prudent to explicitly call [`syncQuESTEnv()`](https://quest-kit.github.io/QuEST/group__environment.html#gaaa19c3112f1ecd80e3296df5c0ed058d) immediately before starting and ending a performance timer. This way, the recorded runtime should reflect that of the slowest node (and ergo, the full calculation) rather than that of the node which happened to have its timer output logged.


---------------------


## Multi-GPU {#launch_multi-gpu}


> TODO:
> - explain usecases (multi local GPU, multi remote GPU, hybrid)
> - explain GPUDirect
> - explain CUDA-aware MPI
> - explain UCX
> - detail controlling local vs distributed gpus with device visibility



> helpful ARCHER2 snippet:
> ```bash
> # Compute the raw process ID for binding to GPU and NIC
> lrank=$((SLURM_PROCID % SLURM_NTASKS_PER_NODE))
> 
> # Bind the process to the correct GPU and NIC
> export CUDA_VISIBLE_DEVICES=${lrank}
> export UCX_NET_DEVICES=mlx5_${lrank}:1
> ```


---------------------


## Supercomputers {#launch_supercomputers}

A QuEST executable is launched like any other in supercomputing settings, including when distributed.
For convenience however, we offer some example [SLURM](https://slurm.schedmd.com) and [PBS](https://www.openpbs.org/) job submission scripts to deploy QuEST in various configurations. These examples assume QuEST and the user source have already been compiled, as guided in [<code>compile.md</code>](compile.md).


> [!NOTE]
> These submission scripts are only illustrative. It is likely the necessary configuration and commands on
> your own supercomputing facility differs!


### SLURM {#launch_slurm}

4 machines each with 8 CPUs:
```bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
OMP_NUM_THREADS=8 srun ./myexec
```

1 machine with 4 local GPUs:
```bash
#SBATCH --nodes=1
#SBATCH --tasks-per-node=4
#SBATCH --gres=gpu:4
#SBATCH --distribution=block:block
#SBATCH --hint=nomultithread
srun ./myexec
```

1024 machines with 16 local GPUs (divides `Qureg` between 16384 partitions):
```bash
#SBATCH --nodes=1024
#SBATCH --tasks-per-node=16
#SBATCH --gres=gpu:16
#SBATCH --distribution=block:block
#SBATCH --hint=nomultithread
srun ./myexec
```


### PBS {#launch_pbs}

4 machines each with 8 CPUs:
```bash
#PBS -l select=4:ncpus=8
OMP_NUM_THREADS=8 aprun -n 4 -d 8 -cc numa_node ./myexec
```
