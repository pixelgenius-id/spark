
# Spark
[![Latest Release](https://img.shields.io/github/v/release/pixelgenius-id/spark?color=blue)](https://github.com/pixelgenius-id/spark/releases)
[![Contributors](https://img.shields.io/github/contributors/pixelgenius-id/spark)](https://github.com/pixelgenius-id/spark/graphs/contributors)
[![Forks](https://img.shields.io/github/forks/pixelgenius-id/spark?style=social)](https://github.com/pixelgenius-id/spark/network/members)
[![Stars](https://img.shields.io/github/stars/pixelgenius-id/spark?style=social)](https://github.com/pixelgenius-id/spark/stargazers)

> [!IMPORTANT]
> Spark is a fork of [Leap](https://github.com/AntelopeIO/leap) maintained by [PixelGenius](https://github.com/pixelgenius-id/spark) to support [VexChain](https://vexchain.io), while preserving upstream compatibility with the Antelope protocol.

## Table of Contents
1. [Branches](#branches)  
2. [Supported Operating Systems](#supported-operating-systems)  
3. [Binary Installation](#binary-installation)  
4. [Build and Install from Source](#build-and-install-from-source)  
   - [Prerequisites](#prerequisites)  
   - [Step 1 - Clone Spark](#step-1---clone)  
   - [Step 2 - Checkout Release](#step-2---checkout-release-tag-or-branch)  
   - [Step 3 - Build](#step-3---build)  
   - [Step 4 - Test](#step-4---test)  
   - [Step 5 - Install](#step-5---install)  
5. [Bash Autocomplete](#bash-autocomplete)

---

Spark is a C++ implementation of the [Antelope](https://github.com/AntelopeIO) protocol, forked from Leap.  
It contains blockchain node software and supporting tools for developers and node operators, with enhancements for VexChain.

---

## Branches
The `main` branch is the development branch. Do not use it for production.  
For production deployments, please use **tagged releases**.  

Upstream source (Leap) is kept as reference:  
👉 https://github.com/AntelopeIO/leap

---

## Supported Operating Systems
Spark officially supports:
- Ubuntu 22.04 Jammy
- Ubuntu 20.04 Focal

Other Linux variants (Debian, Mint, Arch, etc.) may work on a best-effort basis.

---

## Binary Installation
From the [releases](https://github.com/pixelgenius-id/spark/releases) page, download the `*.deb` package for your version of Ubuntu.  

Install with:
```bash
sudo apt-get update
sudo apt-get install -y ~/Downloads/spark*.deb
```

Verify installation:

```
nodeos --full-version
```

----------

## **Build and Install from Source**

  

### **Prerequisites**

-   C++20 compiler and standard library
    
-   CMake 3.16+
    
-   LLVM 7–11 (Linux only)
    
-   libcurl 7.40.0+
    
-   git
    
-   GMP
    
-   Python 3 + numpy
    
-   zlib
    

  


### Step 1 - Clone
If you don't have the Leap repo cloned to your computer yet, [open a terminal](https://itsfoss.com/open-terminal-ubuntu) and navigate to the folder where you want to clone the Leap repository:
```bash
cd ~/Downloads
```
Clone Leap using either HTTPS...
```bash
git clone --recursive https://github.com/pixelgenius-id/spark.git
```
...or SSH:
```bash
git clone --recursive git@github.com:pixelgenius-id/spark.git
```

> ℹ️ **HTTPS vs. SSH Clone** ℹ️  
Both an HTTPS or SSH git clone will yield the same result - a folder named `leap` containing our source code. It doesn't matter which type you use.

Navigate into that folder:
```bash
cd spark
``````


### Step 2 - Checkout Release Tag or Branch
Choose which [release](https://github.com/AntelopeIO/leap/releases) or [branch](#branches) you would like to build, then check it out. If you are not sure, use the [latest release](https://github.com/AntelopeIO/leap/releases/latest). For example, if you want to build release 3.1.2 then you would check it out using its tag, `v3.1.2`. In the example below, replace `v0.0.0` with your selected release tag accordingly:
```bash
git fetch --all --tags
git checkout v0.0.0
```

Once you are on the branch or release tag you want to build, make sure everything is up-to-date:
```bash
git pull
git submodule update --init --recursive
```


### Step 3 - Build
Select build instructions below for a [pinned build](#pinned-build) (preferred) or an [unpinned build](#unpinned-build).

> ℹ️ **Pinned vs. Unpinned Build** ℹ️  
We have two types of builds for Spark: "pinned" and "unpinned." A pinned build is a reproducible build with the build environment and dependency versions fixed by the development team. In contrast, unpinned builds use the dependency versions provided by the build platform. Unpinned builds tend to be quicker because the pinned build environment must be built from scratch. Pinned builds, in addition to being reproducible, ensure the compiler remains the same between builds of different Spark major versions. Spark requires the compiler version to remain the same, otherwise its state might need to be recovered from a portable snapshot or the chain needs to be replayed.

> ⚠️ **A Warning On Parallel Compilation Jobs (`-j` flag)** ⚠️  
When building C/C++ software, often the build is performed in parallel via a command such as `make -j "$(nproc)"` which uses all available CPU threads. However, be aware that some compilation units (`*.cpp` files) in Spark will consume nearly 4GB of memory. Failures due to memory exhaustion will typically, but not always, manifest as compiler crashes. Using all available CPU threads may also prevent you from doing other things on your computer during compilation. For these reasons, consider reducing this value.

> 🐋 **Docker and `sudo`** 🐋  
If you are in an Ubuntu docker container, omit `sudo` from all commands because you run as `root` by default. Most other docker containers also exclude `sudo`, especially Debian-family containers. If your shell prompt is a hash tag (`#`), omit `sudo`.

#### Pinned Reproducible Build
The pinned reproducible build requires Docker. Make sure you are in the root of the `spark` repo and then run
```bash
DOCKER_BUILDKIT=1 docker build -f tools/reproducible.Dockerfile -o . .
```
This command will take a substantial amount of time because a toolchain is built from scratch. Upon completion, the current directory will contain a built `.deb` and `.tar.gz` (you can change the `-o .` argument to place the output in a different directory). If needing to reduce the number of parallel jobs as warned above, run the command as,
```bash
DOCKER_BUILDKIT=1 docker build --build-arg SPARK_BUILD_JOBS=4 -f tools/reproducible.Dockerfile -o . .
```

#### Unpinned Build
The following instructions are valid for this branch. Other release branches may have different requirements, so ensure you follow the directions in the branch or release you intend to build. If you are in an Ubuntu docker container, omit `sudo` because you run as `root` by default.

Install dependencies:
```bash
sudo apt-get update
sudo apt-get install -y \
        build-essential \
        cmake \
        git \
        libcurl4-openssl-dev \
        libgmp-dev \
        llvm-11-dev \
        python3-numpy \
        file \
        zlib1g-dev
```

On Ubuntu 20.04, install gcc-10 which has C++20 support:
```bash
sudo apt-get install -y g++-10
```

To build, make sure you are in the root of the `spark` repo, then run the following command:
```bash
mkdir -p build
cd build

## on Ubuntu 20, specify the gcc-10 compiler
cmake -DCMAKE_C_COMPILER=gcc-10 -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr/lib/llvm-11 ..

## on Ubuntu 22, the default gcc version is 11, using the default compiler is fine
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr/lib/llvm-11 ..

make -j "$(nproc)" package
```

Now you can optionally [test](#step-4---test) your build, or [install](#step-5---install) the `*.deb` binary packages, which will be in the root of your build directory.


### **Step 4 - Test**

Spark inherits Leap’s test structure, with additional tests for VexChain.

| Test Suite                | Test Type              | Size           | Notes |
|----------------------------|------------------------|----------------|-------|
| Parallelizable tests       | Unit tests             | Small          | Run with `ctest -j "$(nproc)" -LE _tests` |
| WASM spec tests            | Unit tests             | Small          | Ensures WASM compliance, `ctest -j "$(nproc)" -L wasm_spec_tests` |
| Serial tests               | Component/Integration  | Medium         | Uses specific ports/files, `ctest -L "nonparallelizable_tests"` |
| Long-running tests         | Integration            | Medium–Large   | Stress tests, `ctest -L "long_running_tests"` |
| VexChain integration tests | Chain tests            | Medium         | Specific to Spark/VexChain, `ctest -L "vexchain_tests"` |

---

#### **Parallelizable Tests**
When building from source, we recommend running at least the parallelizable tests.  
This test suite consists of tests that do not require shared resources (file descriptors, specific folders, or ports).  
They can safely run concurrently across multiple threads. These are mostly small and fast unit tests.  

Run with:
```bash
ctest -j "$(nproc)" -LE _tests
```

#### **WASM Spec Tests**

  

The WASM spec tests verify that the WASM execution engine is compliant with the WebAssembly standard.

They are small, CPU-intensive, and numerous (over 1000). While they run quickly, on shared hosts (e.g., CI/CD), performance may degrade.

Disabling hyperthreading on the host often resolves these issues.

  

Run with:

```
ctest -j "$(nproc)" -L wasm_spec_tests
```

#### **Serial Tests**

  

The serial test suite includes medium component or integration tests that  **cannot**  run concurrently.

They rely on specific resources like process names, ports, or filesystem paths, and may interfere with other  nodeosinstances.

Execution time is moderate, but these tests are strongly recommended.

  

Run with:

```
ctest -L "nonparallelizable_tests"
```

#### **Long-Running Tests**

  

These are medium-to-large integration tests that stress the system and rely heavily on shared resources.

They can take a long time to complete and are intended for stability and stress validation.

  

Run with:

```
ctest -L "long_running_tests"
```

#### **VexChain Integration Tests**

  

Additional integration tests specific to Spark and VexChain.

  

Run with:

```
ctest -L "vexchain_tests"
```
----------

### **Step 5 - Install**

  

From your build directory:

```
sudo apt-get update
sudo apt-get install -y ./spark[-_][0-9]*.deb
```

Or via make install:

```
sudo make install
```

----------

## **Bash Autocomplete**

  

cleos  and  leap-util  provide many commands and options. Bash autocompletion can make them easier to use.

  

For  .deb  packages:

```
sudo apt-get install bash-completion
```

If building from source, install the following files into your bash-completion directory:

-   build/programs/cleos/bash-completion/completions/cleos
    
-   build/programs/leap-util/bash-completion/completions/leap-util
    

  

Refer to  [bash-completion documentation](https://github.com/scop/bash-completion#faq)  for install locations.

----------