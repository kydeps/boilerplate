# Introduction

There is some amount of boilerplate associated with setting up the repo for a new project. For me this includes:

1. Adding a number of standard files which I want to be the same across all my projects:
   * `.gitignore`
   * `.clang-format`
   * `.clang-tidy`
   * `.cmake-lint`
2. Adding sample starting points for some important files:
   * `README.md` and `LICENSE`
   * CircleCI configuration in `.circleci/config.yml`
   * Support for `cmake --isntall ...` which includes `cmake/config.cmake.in`
   * A program and/or a test to make sure the repo is setup properly for build and test:
      * `/src/test.cc`
      * `/src/CMakeLists.txt`

[KyDeps Boilerplate](https://github.com/kydeps/boilerplate) is my way of automating the tediousness of the above process away. 

Tha provided automation mechanism also facilitates the reuse of the files in group (1) above between projects.

Finally, the mechanism is relatively simple and allows for new boilerplate specific to your projects. Fork away!

# Minimal Example

The simplest way to experience all of the above involves two steps.

1. Create a new directory with a single `CMakeLists.txt` file which reads as follows:

   ```cmake
   cmake_minimum_required(VERSION 3.22)
   project(hello_world VERSION 0.1)

   set(KYDEPS_BOILERPLATE_CLANG_TIDY OFF)
   include(FetchContent)
   FetchContent_Declare(
       kydeps_boilerplate
       GIT_REPOSITORY https://github.com/kydeps/boilerplate.git
       GIT_TAG main)
   FetchContent_MakeAvailable(kydeps_boilerplate)
   KyDepsAddBoilerplate()

   add_subdirectory(src)
   ```

2. From within the directory execute the follwing:

   ```bash
   mkdir -p build && \
   	cd build && \
   	cmake -S .. -B . && \
   	cmake --build . && \
   	ctest
   ```


The result should look something like this:

```bash
% mkdir -p build && cd build && cmake -S .. -B . && cmake --build . && ctest
-- The C compiler identification ...
...
-- creating /Users/kamen/work/t/.gitignore
-- creating /Users/kamen/work/t/.clang-format
-- creating /Users/kamen/work/t/.cmake-lint
-- creating /Users/kamen/work/t/.vscode/tasks.json
-- creating sample code in /Users/kamen/work/t/src...
-- KyDepsPackageTarget : test0
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/kamen/work/t/build
[ 50%] Building CXX object src/CMakeFiles/test0.dir/test.cc.o
[100%] Linking CXX executable bin/test0
[100%] Built target test0
Test project /Users/kamen/work/t/build
    Start 1: test0
1/1 Test #1: test0 ............................   Passed    0.33 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.34 sec
```

You now have a fully functional project structure to build on! Let's examine it:

```bash
% tree -a -I build -I .git
.
├── .circleci
│   └── config.yml
├── .clang-format -> /Users/kamen/work/t/build/_deps/kydeps_boilerplate-src/.clang-format
├── .cmake-lint -> /Users/kamen/work/t/build/_deps/kydeps_boilerplate-src/.cmake-lint
├── .gitignore -> /Users/kamen/work/t/build/_deps/kydeps_boilerplate-src/.gitignore.copy
├── .vscode
│   └── tasks.json -> /Users/kamen/work/t/build/_deps/kydeps_boilerplate-src/.vscode/tasks.json
├── CMakeLists.txt
├── LICENSE
├── README.md
├── cmake
│   └── config.cmake.in
└── src
    ├── CMakeLists.txt
    └── test.cc

```

As we can see the files from group (1) are symlinks to dynamic content... they will not be checked in into the repo for the new project.

The rest of the files constitute a functional repo which configures, builds, and tests out of the box.

We also provide a [template repo](https://github.com/kydeps/template), which one can use to get started quickly!

# Feature Detail

All boilerplate features can be individually enabled and disabled by setting apropriate options. Except when noted otherwise, all the features are enabled by default.

## Transient, Cross-Repo Configruation Features

This section describes the boilerplate features which are dynamically added at configuration time and are not checked in as part of the main project. This technique allows for controlling them from (the chose branch of) the boilerplate repo, and therefore having consistent configuration across many projects.

Note that the branch of the boilerplate repo is configured in the main project's `CMakeLists.txt` and by default it is `main`. We recommend using a git revision SHA for a slight speedup during repeated configure invocations.

###  `.clang-format` and `.clang-tidy`

The boilerplate provides default .clang-formant and .clang-tidy definitions. In order to make use of them, one needs to have the corresponding tools installed and on the execution path. As of this writing, pre-combiled binaries of LLVM can be found at its [GitHub releases page](https://github.com/llvm/llvm-project/releases).

The creation of the `.clang-format` symlink is controlled by the `KYDEPS_BOILERPLATE_CLANG_FORMAT` option. 

The creation of the `.clang-tidy` symlink is controlled by the `KYDEPS_BOILERPLATE_CLANG_TIDY` option. This option also controls initialization of the `clang-tidy` environment in CMake (e.g. setting `CMAKE_CXX_CLANG_TIDY` appropriately). Since `clang-tidy` is quite expensive to run, this option is explicitly disabled in the root `CMakeLists.txt` which is used to bootstrap the whole process. It can be flipped back and forth as needed to perform `clang-tidy` checks.

### `.cmake-lint` and associated `.vscode/tasks.json`

This boilerplate provides default `.cmake-lint` configuration for the `cmake-format` and `cmake-lint` tools. In order to make use of the feature, one has to have the distribution of the tools installed on their system as described in their [GitHub README](https://github.com/cheshirekow/cmake_format). 

Visual Studio Code support for execution of the tools against the currently open file is supported through the  `Tasks: Run Task` command.

This feature is controlled by the `KYDEPS_BOILERPLATE_CMAKE_LINT` option.

### `.gitignore`

A suitable `.gitignore` file is provided and is controlled by the `KYDEPS_BOILERPLATE_GITIGNORE` option.

As of this writing the default content of the file is:

```
/.gitignore
/.clang-format
/.clang-tidy
/.cmake-lint
/.vscode
/.vscode/**
/build
/build/**
```

Which effectively allows for not checking in any of the cross-repo configuration files discussed so far, as well as the project build directory.

## Bootstrap Features

The following features are used to bootstrap the repo for the main project and generate files with are checked in into the main project.

### `README.md`

A sample `README.md` file is created when one does not exist and the `KYDEPS_BOILERPLATE_README` option is `ON`.

### `LICENSE`

A sample LICENSE file is created when one does not exist and the `KYDEPS_BOILERPLATE_LICENSE` option is `ON`.

As of this writing, the boilerplate repo uses the MIT License. Feel free to fork the repo and override the license with what you use by default.

### `.circleci/config.yml`

A sample configuration for CircleCI. Note the project would still need to be enabled in CircleCI through the CircleCI interface, but with this sample configuration it is ready to be enabled out of the box. This feature is controlled by the `KYDEPS_BOILERPLATE_CIRCLECI` option.

### `/src/test.cc` and `/src/CMakeLists.txt`

A sample test and associated binary is added if the `KYDEPS_BOILERPLATE_SRC` is `ON`. This allows for the repo to be configurable, buildable, and testable out of the box.

### `cmake/config.cmake.in` and support for `cmake --install ...`

Perhaps one of the boilerplate features I am most excited about is the ability to make the main project play nicely with `cmake --install ...`. Properly configuring a CMake project to support installation is not obvious. Most of the support which this boilerplate provides has been adapted from [this excellent blog post](https://unclejimbo.github.io/2018/06/08/Modern-CMake-for-Library-Developers/).

The TL;DR: version is that a one-liner needs to be added after any CMake target in the main project which is supposed to be exported for installation. The example boilerplate includes `KyDepsPackageTarget(test0)` in the provided `src/CMakeLists.txt` which makes the `test0` target part of the installed package.

Installation can by tested using the following example command from the `build` folder of the main project:

```
cmake --install . --prefix testing123
```

And then examining the `build/testing123` directory. 

Note that exporting library targets to be installed requires the `KyDepsPackageLibraryTarget(...)` one-liner and a fully working example is available at https://github.com/kydeps/ky_temp_path.





