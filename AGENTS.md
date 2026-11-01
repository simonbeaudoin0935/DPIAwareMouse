# AGENTS.md

Guidance for AI agents (and humans) working on this repository.

## Project

**DPIAwareMouse** is a DPI-aware mouse crossing tool for Linux: it makes the
mouse pointer cross between monitors with different DPI scaling factors at the
matching physical position instead of jumping. The project is in its early
skeleton stage; `src/main.cpp` is currently a stub that prints the version.

- Language: C++17, built with CMake (minimum 3.22).
- License: GPL-3.0-or-later. New source files get an SPDX header
  (`// SPDX-License-Identifier: GPL-3.0-or-later`) plus a copyright line.
- Long-term goal: become a proper first-class Debian package.

## Layout

| Path                | Purpose                                             |
| ------------------- | --------------------------------------------------- |
| `CMakeLists.txt`    | CMake project; also owns the version number         |
| `src/`              | C++ sources                                         |
| `man/`              | Manual pages (section 1), installed by CMake        |
| `debian/`           | Debian packaging metadata (debhelper compat 13)     |
| `.vscode/tasks.json`| VS Code build tasks                                 |

## Building

VS Code tasks (preferred):

- **CMake: Configure** — `cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON`
- **CMake: Build** — default build task, compiles into `build/`
- **Debian: Build package** — `dpkg-buildpackage --unsigned-source
  --unsigned-changes --build=binary`; artifacts land in the parent directory

Equivalent CLI commands work from the repository root.

### Compiler policy

- GCC or Clang is required. Warnings are errors, including format,
  conversion, signedness, shadowing and unsafe C++ constructs. Linker
  warnings are fatal too. Lightweight libstdc++ assertions stay enabled
  even in Release builds.
- RelWithDebInfo (the default) retains debug symbols with `-O3` and
  IPO/LTO. Release also uses `-O3` and IPO/LTO.
  MinSizeRel keeps size optimization and LTO. Do not use `-Ofast`,
  `-ffast-math` or `-march=native`: correctness and portable packages matter.
- All builds require stack canaries on every function, frame pointers,
  PIE, full RELRO, immediate symbol binding and a non-executable stack.
- Stack-clash protection, control-flow protection and zero-initialized
  automatic variables are enabled when supported; configuration warns
  explicitly if a compiler/architecture lacks one of these features.
- Optimized builds require libc fortification (level 3 preferred, with
  an explicit warning when falling back to level 2).
- Debug builds intentionally omit optimization, LTO and project-added
  fortification for effective debugging. Other hardening remains enabled.
- Debian builds enable all dpkg hardening features and use RelWithDebInfo;
  `DEB_BUILD_OPTIONS=noopt` selects Debug instead.
- Validate both Release and Debug builds after compiler-policy changes,
  inspect ELF protections, and run the full Debian build plus lintian.

## Debian packaging rules

The packaging quality bar is **zero lintian output** for both the source and
binary package. After any packaging change, verify with:

```sh
dpkg-buildpackage --unsigned-source --unsigned-changes
lintian ../dpiawaremouse_<version>_amd64.changes
```

Conventions:

- Source format is `3.0 (native)` while upstream and packaging share this
  repository. Switch to `3.0 (quilt)` when targeting the Debian archive.
- `Standards-Version` is pinned to the newest version the locally installed
  lintian knows (currently 4.6.2); bump it alongside a tooling upgrade or
  archive submission.
- `debian/source/options` excludes `build/`, `obj-*/` and `.vscode/` from the
  source tarball; extend it if new generated or editor files appear.
- Every binary installed into `/usr/bin` must ship a manual page (Debian
  Policy 12.1); add it under `man/` and install it from `CMakeLists.txt`.
- The version lives in two places that must stay in sync: the `project()`
  version in `CMakeLists.txt` and the latest `debian/changelog` entry.
- New changelog entries target `UNRELEASED` until an actual release is cut.

## Commit conventions

The commit history is meant to be impeccable — suitable for upstream review
and Debian sponsorship:

- Imperative, capitalized subject line (no trailing period), max ~50 chars;
  no conventional-commit prefixes.
- A body explaining *what* and *why*, wrapped at 72 columns.
- Each commit is self-contained: the project configures, builds and stays
  lintian-clean at every commit.
- Never commit build artifacts; `.gitignore` covers `build/`, `obj-*/` and
  debhelper droppings.
- History that has not been pushed may be rebased to keep it clean; pushed
  history is never rewritten.
