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

- **CMake: Configure** — `cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON`
- **CMake: Build** — default build task, compiles into `build/`
- **Debian: Build package** — `dpkg-buildpackage --unsigned-source
  --unsigned-changes --build=binary`; artifacts land in the parent directory

Equivalent CLI commands work from the repository root.

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
