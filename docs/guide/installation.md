---
title: Installation
---

# Installation

FoXy builds with fpm, FoBiS or CMake. All you need is a Fortran 2008 compiler: gfortran 14 or newer, ifx 2025 or
nvfortran 26 are tested.

## One-line install

```bash
curl -fsSL https://github.com/szaghi/FoXy/releases/latest/download/install.sh | bash
```

It installs the latest release into `~/.local` with the build system you choose among those FoXy ships
(`--build fobis|fpm|cmake`: a menu asks when you do not say), the compiler you name (`--fc`, `--gnu`, `--intel`,
`--nvidia`), one install per compiler (`lib/foxy/<compiler>-<version>/`, `include/foxy/<compiler>-<version>/`), and
makes it usable through pkg-config (`pkg-config --cflags --libs foxy`, with `PKG_CONFIG_PATH=$HOME/.local/lib/pkgconfig`).
The release carries FoXy's dependencies (BeFoR64, FACE, PENF, StringiFor), so CMake needs no network. The fobis build
has a GNU library mode only: with ifx or nvfortran use `--build fpm` or `--build cmake`. Pass options after
`bash -s --`, e.g. `... | bash -s -- --build cmake --prefix /opt/foxy`; `--help` lists them all, `--uninstall`
removes the install. The installer is FoBiS's ([One-line installer](https://szaghi.github.io/FoBiS/cookbook/installer)).

## fpm

Add FoXy to the dependencies of your `fpm.toml`; fpm fetches it, and its dependencies, when you build:

```toml
[dependencies]
FoXy = { git = "https://github.com/szaghi/FoXy" }
```

## FoBiS

[FoBiS](https://github.com/szaghi/FoBiS) is the build tool of FoXy's development. Clone the repository and fetch the
dependencies (declared in `fobos`, cloned into `src/third_party/`):

```bash
git clone https://github.com/szaghi/FoXy
cd FoXy
fobis fetch
```

Build the library, `lib/libfoxy.a` with its modules in `lib/mod`, and link your program to it:

```bash
fobis build --mode foxy-static-gnu
gfortran -I lib/mod my_program.f90 lib/libfoxy.a -o my_program
```

The library holds the dependencies too: one archive, one module directory.

To run the tests (each test is a program in `src/tests`, built in `exe/`):

```bash
fobis build --mode tests-gnu && bash scripts/run_tests.sh          # also tests-gnu-debug
fobis build --mode tests-intel && bash scripts/run_tests.sh        # ifx, also tests-intel-debug
fobis build --mode tests-nvidia && bash scripts/run_tests.sh       # nvfortran, also tests-nvidia-debug
```

`fobis build --lmodes` lists every mode.

## CMake

CMake uses the dependencies in `src/third_party/`: fetch them first.

```bash
git clone https://github.com/szaghi/FoXy
cd FoXy
fobis fetch
cmake -S . -B build
cmake --build build
cmake --install build
```

CMake installs package config files: `find_package(FoXy)` finds it in your project.

## Compilers

With your own flags:

- **ifx**: compile with `-heap-arrays`; ifx puts large temporaries (e.g. the content of a big tag) on the stack, which
  overflows on large documents.
- **nvfortran**: preprocess with `-Mpreprocess` and define `_NVF` (`-D_NVF`), which turns on StringiFor's nvfortran
  workarounds.

## Dependencies

| Library | Purpose |
|---|---|
| [PENF](https://github.com/szaghi/PENF) | portable kinds (`I1P`…`I8P`, `R4P`, `R8P`), number to string |
| [StringiFor](https://github.com/szaghi/StringiFor) | the string type used by the parser |
| [BeFoR64](https://github.com/szaghi/BeFoR64) | base64, used by StringiFor |
| [FACE](https://github.com/szaghi/FACE) | ANSI colours, used by StringiFor |

`fobis fetch` clones them at their default branches and records the revisions in `src/third_party/fobos.lock`;
`fobis fetch --update` updates them.

Next: [the tutorial](/manual/).
