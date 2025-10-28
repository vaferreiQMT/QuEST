# ❤️  Contributing

<!--
  How to contribute
  (this comment must be under the title for valid doxygen rendering)
  
  @author Tyson Jones
-->

> [!IMPORTANT]  
> This page is under construction! In the meantime, feel free to open an issue, a discussion or a pull request, or reach out to `tyson.jones.input@gmail.com`.


> **TOC**:
> - [Architecture](#contributing_architecture)
> - [Style guide](#contributing_style-guide)
> - [Documentation](#contributing_documentation)
> - [Testing](#contributing_testing)

---------------



## Architecture {#contributing_architecture}

All user-visible API signatures are contained in `include/`, divided into semantic submodules (like `calculations.h` and `qureg.h`), but all exposed by `quest.h`. They are all strictly `C` _and_ `C++` compatible, hence their `.h` file extension.

The source code within `src/` is divided between five subdirectories, listed below in order of increasing control flow depth. All code is parsed strictly by `C++`, hence all files have `.cpp` and `.hpp` extensions.
- `api/` 
  > contains definitions of the API, directly callable by users (in `C` _or_ `C++`, so it contains de-mangling guards). Functions are divided between files therein similarly to `include/`, and they call only `core/` functions.
- `core/`
  > contains internal non-simulation functions like user-input validaters, non-accelerated maths, and dispatchers to hardware-accelerated backends.
- `comm/` 
  > contains functions needed for exchanging data between distributed nodes, as invoked by the `core/` layer before hardware-accelerated backends.
- `cpu/` 
  > constitutes the multithreaded CPU backend, containing OpenMP-accelerated subroutines and (potentially) AVX intrinsics.
- `gpu/` 
  > constitutes the GPU backend, containing CUDA-accelerated subroutines, GPU hardware queriers, interfaces to CUDA libraries (like Thrust and cuQuantum), and wrappers for AMD/HIP compatibility. Note that some files therein have suffix `.cpp` in lieu of `.cu`, because they are permittedly parsed by non-CUDA compilers when disabling GPU-acceleration at compile-time.

The control flow from the user interface (`quest.h`) to the hardware-accelerated simulation subroutines is mostly captured by:

- `include/quest.h`
- `api/*`
- `core/validation`
  - `core/memory`
  - `gpu/gpu_config`
  - `comm/comm_config`
- `core/utilities`
- `core/localiser`
  - `comm/comm_routines`
- `core/accelerator`
  - `cpu/cpu_subroutines`
  - `gpu/gpu_subroutines`
    - `gpu/gpu_kernels`
    - `gpu/gpu_thrust`
    - `gpu/gpu_cuquantum`


Every API function first validates the user given inputs via `core/validation.cpp`, which in-turn may consult hardware facilities like available RAM (via `core/memory.cpp`) and VRAM (via `gpu/gpu_config.cpp`), or the distributed configuration (via `comm/comm_config.cpp`). Simulation API functions will then invoke `core/localiser.cpp` which checks whether distributed data exchange is necessary in order for all subsequently needed data to become locally available. If so, it invokes functions of `comm/comm_routines.cpp`, then proceeds. Stage `core/accelerator.cpp` chooses whether to process the (potentially just received) data using the CPU (accelerating with `OpenMP`) or the GPU (accelerating with `CUDA`). GPU-acceleration can involve dispatching to custom kernels (in `gpu/cpu_kernels.cpp`), or Thrust routines (in `gpu/gpu_thrust.hpp`), or alternatively to a cuQuantum routine (in `gpu/gpu_cuquantum.hpp`) if optionally compiled. At any call depth, functions within `core/errors.cpp` will be called to confirm preconditions.

> [!TIP]
> See [PR #615](https://github.com/QuEST-Kit/QuEST/pull/615) for an illustration of integrating 
> new functions into the QuEST software architecture.


---------------


##  Style guide {#contributing_style-guide}

Don't agonise about style - write your code as you see fit and we can address major issues in review/PR.
Some encouraged conventions include:

- use `camelCase` for everything except:
  - constants which use `CAPITALS_AND_UNDERSCORES`
  - related function prefixes, like `prefix_someFunction()`
- favour clarity over concision, for example
  ```cpp
  qcomp elem = state[ind][ind];
  qreal prob = std::real(elem);
  return prob;
  ```
  over
  ```cpp
  return std::real(state[ind][ind]);
  ```
- never ever do:
  ```cpp
  using namespace std;
  ```
  but _do_ shorten common containers like `vector`:
  ```cpp
  using std::vector;

  vector<int> mylist;
  ```
- whitespace is free; use it wherever it can improve clarity, like to separate subroutines.
  ```cpp
  // i000 = nth local index where all suffix bits are 0
  qindex i000 = insertThreeZeroBits(n, braQb1, ketQb2, ketQb1);
  qindex i0b0 = setBit(i000, ketQb2, braBit2);
  qindex i1b1 = flipTwoBits(i0b0, braQb1, ketQb1);

  // j = nth received amp in buffer
  qindex j = n + offset;

  // mix pair of amps using buffer
  qcomp amp0b0 = qureg.cpuAmps[i0b0];
  qcomp amp1b1 = qureg.cpuAmps[i1b1];

  qureg.cpuAmps[i0b0] = c1*amp0b0 + c2*(amp1b1 + qureg.cpuCommBuffer[j]);
  qureg.cpuAmps[i1b1] = c1*amp1b1 + c2*(amp0b0 + qureg.cpuCommBuffer[j]);
  ```
- use `auto` where it improves readability, discretionarily. Obviously it is better than massive, unimportant types of objects or heavily templated collections, but sometimes knowing the precise type of a primitive is helpful
- It is permissable to avoid superfluous braces around single-line branches:
  ```cpp
  if (cond)
      return x;
  ```
- always prefix calls to mathematical functions like `abs()` with the `std` namespace, i.e. `std::abs()`. This avoids ambiguity with `C` overloads like `abs(int)` which can cause insidious bugs! The full list of functions to prefix are:
  - `abs`
  - `real`
  - `imag`
  - `conj`
  - `norm`
  - `sin`
  - `cos`
  - `log`
  - `log2`
  - `exp`
  - `pow`
  - `sqrt`
  - `floor`
  - `ceil`
  - `atan2`
  - `min`
  - `max`


---------------

# Documentation {#contributing_documentation}

TODO



---------------

# Testing {#contributing_testing}

TODO
