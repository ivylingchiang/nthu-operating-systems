# Operating Systems — NTHU CS

Kernel-level programming assignments from NTHU's Operating Systems course (CS342301, Fall 2024): system calls, paging, CPU scheduling, and a file system built on NachOS, plus a multithreaded Pthreads pipeline.

## Overview

[NachOS 4.0](https://homes.cs.washington.edu/~tom/nachos/) is a teaching operating system written in C++. It runs as a user process on Linux and executes user programs on a simulated MIPS machine. Each machine problem (MP1–MP4) extends the same kernel: system calls, then multiprogramming with paging, then CPU scheduling, then the file system. A separate Pthreads assignment covers the producer–consumer problem with real POSIX threads. The work was done in a team of two. Each folder has a written report (in Chinese) that traces the relevant kernel code paths and explains the implementation.

## Assignments

| Assignment | Topic | What we implemented | Key files |
|---|---|---|---|
| [MP1](NachOS-4.0_MP1) | System calls | Added the `Open`, `Read`, `Write` and `Close` file I/O system calls, from the user-level stubs in `start.S` through the `ExceptionHandler` to the kernel. Open files are tracked in an `OpenFileTable` (at most 20), and invalid IDs and failed opens return errors. Traced how `Halt`, `Create` and `PrintInt` move from user mode to kernel mode. | `code/userprog/exception.cc`, `code/userprog/ksyscall.h`, `code/filesys/filesys.h`, `code/test/start.S` |
| [MP2](NachOS-4.0_MP2) | Multiprogramming and memory | Added per-process page tables so that several user programs can run in memory at once. The kernel keeps a table of used physical frames and loads each page into any free frame. Added a new `MemoryLimitException` for when memory runs out, and frames are freed when a thread exits. | `code/userprog/addrspace.cc`, `code/threads/kernel.{h,cc}`, `code/machine/machine.h` |
| [MP3](NachOS-4.0_MP3) | CPU scheduling | Replaced the default round-robin scheduler with a 3-level multilevel feedback queue: L1 is preemptive SJF using exponentially averaged burst-time estimates, L2 is non-preemptive priority, and L3 is round-robin with a 100-tick quantum. Added aging, which raises a thread's priority by 10 after it waits more than 1500 ticks. Also added an `-ep` flag to set each process's starting priority and a `z` debug flag that logs scheduling events. | `code/threads/scheduler.{h,cc}`, `code/threads/thread.{h,cc}`, `code/threads/alarm.cc`, `code/threads/kernel.cc` |
| [MP4](NachOS-4.0_MP4) | File system | Connected the MP1 system calls to the real NachOS file system. Raised the maximum file size above 4 KB by chaining file headers in a linked list. Added subdirectories with absolute paths, up to 64 entries per directory, and the `-mkdir`, `-l` and `-lr` (recursive list) commands. Bonus work: enlarged the simulated disk from 128 KB to 64 MB to allow 64 MB files, and made header size grow with file size. | `code/filesys/filehdr.{h,cc}`, `code/filesys/filesys.{h,cc}`, `code/filesys/directory.{h,cc}`, `code/threads/main.cc` |
| [Pthreads](NTHU-OS-Pthreads) | Multithreaded programming | Built a reader → producers → consumers → writer pipeline connected by thread-safe queues (mutex and condition variables). A controller thread adds or removes consumer threads as the worker queue fills up or empties. Measured how the controller's check period, thresholds and queue sizes affect run time. | `ts_queue.hpp`, `producer.hpp`, `consumer.hpp`, `consumer_controller.hpp`, `writer.hpp`, `main.cpp` |

## Tech stack

- **Language:** C++ (NachOS kernel, Pthreads assignment), MIPS assembly (system call stubs)
- **Platform:** Linux; NachOS runs user programs on a simulated MIPS machine
- **Toolchain:** `g++`, GNU Make, and the MIPS cross-compiler included in each MP folder (`usr/local/nachos`) for building user programs
- **Concurrency:** POSIX threads (`pthread_mutex`, `pthread_cond`)
- **Utilities:** Python scripts for generating test input and checking output (Pthreads), Docker Compose (optional Pthreads build)

## Build & run

NachOS (shown for MP1; MP2–MP4 work the same way):

```bash
cd NachOS-4.0_MP1/code/build.linux
make clean && make            # builds the nachos kernel
cd ../test
make halt                     # cross-compiles a user program
../build.linux/nachos -e halt
```

Examples from the later MPs:

```bash
# MP2: run two user programs at once
../build.linux/nachos -e consoleIO_test1 -e consoleIO_test2

# MP3: start processes with initial priorities and print scheduler events
../build.linux/nachos -ep hw3t1 40 -ep hw3t2 80 -d z

# MP4: format the disk, create a directory, copy a file in, list recursively
../build.linux/nachos -f
../build.linux/nachos -mkdir /dir1
../build.linux/nachos -cp num_1000.txt /dir1/num1000
../build.linux/nachos -lr /
```

Pthreads:

```bash
cd NTHU-OS-Pthreads
make                          # or: make docker-build
./main <n> <input_file> <output_file>
```

See each folder's README and spec PDF for the exact test commands.

## Repository structure

```
.
├── NachOS-4.0_MP1/        # System calls
├── NachOS-4.0_MP2/        # Multiprogramming / paging
├── NachOS-4.0_MP3/        # CPU scheduling (MLFQ)
├── NachOS-4.0_MP4/        # File system
│   ├── code/              # NachOS kernel source (threads, userprog, filesys, machine, test, ...)
│   ├── usr/local/nachos/  # MIPS cross-compiler toolchain
│   ├── *_Spec*.pdf        # Assignment spec (course material)
│   └── MP*_report_*.pdf   # Our report
└── NTHU-OS-Pthreads/      # Producer–consumer with Pthreads
```

## Notes

The assignment specs, the base NachOS source, and the test programs belong to the NTHU CS342301 course staff and the original NachOS authors (see `COPYRIGHT` in each NachOS folder). They are included here only for reference. My contribution is the implementation work and the reports described above.
