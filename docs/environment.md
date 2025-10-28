# Environment variables {#environment}

<!--
  This page groups all the environment variables used to configure QuEST.    
  
  @author Tyson Jones
  @author Vasco Ferreira (refactoring)
-->

> **TOC**:
> - [Quest variables](#environment_quest)
> - [Standard variables](#environment_standard)
> - [Test variables](#environment_test)

-----------------

## Quest variables {#environment_quest}

| Variable | (Default) Values | Notes |
| -------- | ---------------- | ----- |
| [`DEFAULT_VALIDATION_EPSILON`](https://quest-kit.github.io/QuEST/group__modes.html#ga55810d6f3d23de810cd9b12a2bbb8cc2) | (single: 1E-5, double: 1E-12, quad: 1E-13), positive `qreal` | Determines epsilon used for numerical validation. |
| [`PERMIT_NODES_TO_SHARE_GPU`](https://quest-kit.github.io/QuEST/group__modes.html#ga7e12922138caa68ddaa6221e40f62dda) | (0), 1| When distributed, determines whether more than one MPI process will be able to access a single GPU. |

-----------------

## Standard variables {#environment_standard}

| Variable | Notes | Reference |
| -------- | ----- | --------- |
| `OMP_NUM_THREADS` | Number of threads to use for multithreading. It is prudent to choose as many threads as the number of cores of (each of) your CPU(s).|[OMP_NUM_THREADS](https://www.openmp.org/spec-html/5.0/openmpse50.html)|
| `ROCR_VISIBLE_DEVICES` | Set which AMD GPUs to expose to QuEST. | [AMD environment variables](https://rocm.docs.amd.com/projects/HIP/en/docs-develop/reference/env_variables.html)
| `CUDA_VISIBLE_DEVICES` | Set which NVIDIA GPUs to expose to QuEST.| [NVIDIA environment variables](https://docs.nvidia.com/cuda/cuda-c-programming-guide/#cuda-environment-variables)
| `CUDA_DEVICE_ORDER` | Set which NVIDIA GPUs to use preferentially.| [NVIDIA environment variables](https://docs.nvidia.com/cuda/cuda-c-programming-guide/#cuda-environment-variables) 

-----------------

## Test variables {#environment_test}

The `v4` unit tests make use of the below, optional environment variables to control their rigour and runtime.

| Environment variable  | Default | Description |
| -------- | ------- | ------- |
| `TEST_NUM_QUBITS_IN_QUREG` | `6` | The number of qubits in the Qureg(s) undergoing unit testing. In addition to operation upon larger Quregs being exponentially slower, beware that more qubits permit more variations and permutations of input parameters like target qubits, factorially increasing the number of tests per operation. |
| `TEST_MAX_NUM_QUBIT_PERMUTATIONS`  | `0` | The maximum number of control and target qubit permutations under which to unit test each function. Set to `0` (default) to test all permutations, or to a positive integer (e.g. `50`) to accelerate the unit tests. See more info [here](https://quest-kit.github.io/QuEST/group__testutilsconfig.html#gac5adcc10bd26c56f20344f5ae3d9ba41). |
| `TEST_MAX_NUM_SUPEROP_TARGETS` | `4` | The maximum number of superoperator targets for which to unit test functions `mixKrausMap()` and `mixSuperOp()`. These are computationally equivalent to simulating unitaries with double the number of targets upon a density matrix. Set to `0` to test all sizes which is likely prohibitively slow, or to a positive integer (e.g. the default of `4`) to accelerate the unit tests. |
| `NUM_MIXED_DEPLOYMENT_REPETITIONS` | `10` | The number of times (minimum of `1`) to repeat each random mixed-deployment unit test for each deployment combination. |
| `TEST_ALL_DEPLOYMENTS` | `1` | Whether unit tests will be run using all possible deployment combinations (i.e. OpenMP, CUDA, MPI) in-turn (`=1`), or only once using all available deployments simultaneously (`=0`). |
