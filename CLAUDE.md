# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of OS course projects based on OSTEP (Operating Systems: Three Easy Pieces) from UW-Madison. Projects fall into two categories:
- **C/Linux projects**: Standard C programs compiled with gcc and run on Linux/macOS
- **xv6 kernel projects**: Modifications to the xv6 teaching OS, run under QEMU emulation

## Building

Each project is self-contained. The typical build for a C/Linux project:
```sh
gcc -o <program> <program>.c -Wall
```

For projects with shared libraries (e.g., `filesystems-distributed-ufs`):
```sh
gcc -shared -fPIC -o libmfs.so mfs.c
```

For xv6 projects, from inside the cloned `xv6-public` directory:
```sh
make TOOLPREFIX=i386-elf- qemu-nox   # macOS with MacPorts toolchain
make qemu-nox                          # if TOOLPREFIX is set in Makefile
```
Quit QEMU with `C-a x`.

## Running Tests

Each project has a `test-<name>.sh` script that invokes the shared test runner:

```sh
# Run all tests for a project (from within the project directory)
./test-wcat.sh

# Run a single test
./test-wcat.sh -t 3

# Continue after failures
./test-wcat.sh -c

# Verbose output (shows what each test runs)
./test-wcat.sh -v
```

The test runner (`tester/run-tests.sh`) executes test cases from `tests/` and compares stdout, stderr, and exit code against expected files. Output lands in `tests-out/`.

Test case files use a numbered format:
- `N.run` — shell command to execute
- `N.out` / `N.err` / `N.rc` — expected stdout, stderr, exit code
- `N.desc` — human-readable description
- `N.pre` / `N.post` — optional setup/teardown

To debug a failing test:
```sh
diff tests/3.out tests-out/3.out
cat tests/3.run   # see what was executed
```

## Project Architecture

### filesystems-distributed-ufs (most architecturally complex)
A UDP-based distributed file server with three components:
- **`ufs.h`** — on-disk format: `super_t`, `inode_t`, `dir_ent_t`. Block size is 4096 bytes, inodes have 30 direct block pointers, directory entries are 32 bytes (28-byte name + 4-byte inum).
- **`mfs.h`** — client library API (`MFS_Init`, `MFS_Lookup`, `MFS_Stat`, `MFS_Read`, `MFS_Write`, `MFS_Creat`, `MFS_Unlink`, `MFS_Shutdown`)
- **`mkfs.c`** — creates a disk image: `./mkfs <image> <num_inodes> <num_data_blocks>`
- **Server** (to implement): listens on UDP, reads/writes the image file, calls `fsync()` before every success reply (idempotency requirement)
- **`libmfs.so`** (to implement): client library that uses `select()` with a 5-second timeout and retries on no response

Disk layout (all in 4KB blocks): superblock → inode bitmap → data bitmap → inode table → data region.

### processes-shell
Implement `wish` (Wisconsin Shell). Supports built-ins (`exit`, `cd`, `path`), I/O redirection (`>`), and parallel commands (`&`). Tests are in `tests/`.

### concurrency-webserver
Multi-threaded HTTP server in `src/`. Key files: `wserver.c` (main), `request.c` (HTTP handling), `io_helper.c` (I/O wrappers).

### concurrency-mapreduce
Implement the MapReduce API defined in `mapreduce.h` using pthreads.

### xv6 Projects
All xv6 modifications are made inside a separately cloned `xv6-public` repo. The projects here provide specs and tests; actual kernel code lives in xv6-public. See `INSTALL-xv6.md` for toolchain setup (requires `qemu` and `i386-elf-gcc` via MacPorts on macOS).

## xv6 Setup (macOS)

```sh
sudo port install qemu i386-elf-gcc gdb
git clone https://github.com/mit-pdos/xv6-public
cd xv6-public
# Edit Makefile: set TOOLPREFIX = i386-elf-  and  CPUS := 1
make qemu-nox
```
