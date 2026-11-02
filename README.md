# Pet Elevator (COP4610 Project 2)

**Group 8**

> **Slack day:** We are using 1 slack day for this project.

A Linux kernel project in three parts. Part 1 traces system calls with `strace`. Part 2 is a `my_timer` kernel module that reports the current time and the time since the last read through `/proc/timer`. Part 3 adds three system calls to Linux 6.16.6 and implements a pet elevator kernel module: a kthread moves the elevator between five floors, a mutex protects the shared floor and elevator lists, and `/proc/elevator` shows the elevator's state.

## Table of Contents
  - [Group Members](#group-members)
  - [Division of Labor](#division-of-labor)
    - [Part 1: System Call Tracing](#part-1-system-call-tracing)
    - [Part 2: Timer Kernel Module](#part-2-timer-kernel-module)
    - [Part 3a: Adding System Calls](#part-3a-adding-system-calls)
    - [Part 3b: Kernel Compilation](#part-3b-kernel-compilation)
    - [Part 3c: Threads](#part-3c-threads)
    - [Part 3d: Linked List](#part-3d-linked-list)
    - [Part 3e: Mutexes](#part-3e-mutexes)
    - [Part 3f: Scheduling Algorithm](#part-3f-scheduling-algorithm)
    - [README/Documentation](#readmedocumentation)
  - [File Listing](#file-listing)
  - [How to Compile and Run](#how-to-compile-and-run)
    - [Part 1](#part-1)
    - [Part 2](#part-2)
    - [Part 3](#part-3)
  - [Features Implemented](#features-implemented)
  - [Implementation Notes](#implementation-notes)
  - [Known Bugs and Limitations](#known-bugs-and-limitations)
  - [Development Log](#development-log)
    - [Jenny Jiang](#jenny-jiang)
    - [Linh Nguyen](#linh-nguyen)
    - [Sid Nguyen](#sid-nguyen)
  - [Group Meetings](#group-meetings)

## Group Members
| Name | FSU Email | GitHub ID |
|------|-----------|-----------|
| Jenny Jiang | jjj24d@fsu.edu | specificShark |
| Linh Nguyen | ltn23@fsu.edu | LinhNguyen2901 |
| Sid Nguyen | hn24c@fsu.edu | hoangdung-nguyen |

## Division of Labor

### Part 1: System Call Tracing
- **Description**: Write `empty.c` and `part1.c` with exactly five more system calls, and record both with `strace`.
- **Completed by**: Sid Nguyen

### Part 2: Timer Kernel Module
- **Description**: Write the `my_timer` module that uses `ktime_get_real_ts64()`, creates `/proc/timer` on load, prints the current and elapsed time on each read, and removes the entry on unload.
- **Completed by**: Linh Nguyen

### Part 3a: Adding System Calls
- **Description**: Add `start_elevator` (548), `issue_request` (549) and `stop_elevator` (550) to the kernel: `syscalls/syscalls.c`, `syscalls/Makefile`, `syscall_64.tbl`, `include/linux/syscalls.h` and the top-level kernel `Makefile`.
- **Completed by**: Sid Nguyen

### Part 3b: Kernel Compilation
- **Description**: Configure, compile and install Linux 6.16.6 with the new system calls, and verify them with `uname -r` and the provided `test-syscalls` test.
- **Completed by**: Sid Nguyen

### Part 3c: Threads
- **Description**: Control the elevator from a kthread that starts with `start_elevator` and stops when the module is removed.
- **Completed by**: Linh Nguyen

### Part 3d: Linked List
- **Description**: Keep the pets waiting on each floor and the pets in the elevator in `linux/list.h` lists, with each pet allocated by `kmalloc`.
- **Completed by**: Sid Nguyen

### Part 3e: Mutexes
- **Description**: Protect all shared elevator and floor data with one mutex (kthread, system calls, `/proc/elevator` read, module cleanup) without holding it while sleeping.
- **Completed by**: Jenny Jiang

### Part 3f: Scheduling Algorithm
- **Description**: Decide the elevator's direction, board pets in FIFO order within the 5-pet / 50 lb limits, and wait 1 second when loading and 2 seconds when moving between floors.
- **Completed by**: Linh Nguyen

### README/Documentation
- **Description**: Write this README.
- **Completed by**: Jenny Jiang

## File Listing
```
pet-elevator/
|-- part1/
|   |-- empty.c             Empty C program (baseline for strace)
|   |-- empty.trace         strace output of empty
|   |-- part1.c             Same program with exactly five extra system calls
|   |-- part1.trace         strace output of part1
|   `-- Makefile
|-- part2/
|   |-- src/
|   |   `-- my_timer.c      Timer module: /proc/timer with current and elapsed time
|   `-- Makefile            Builds my_timer.ko
|-- part3/
|   |-- src/
|   |   |-- elevator_mod.c  Module init/exit, /proc/elevator, issue_request
|   |   |-- elevator.c      kthread loop, scheduling, loading and unloading
|   |   |-- elevator.h      struct elevator, states, function prototypes
|   |   |-- pet.c           Pet and floor list helpers
|   |   `-- pet.h           struct pet, struct floor, pet types and weights
|   |-- syscalls.c          Kernel system call stubs (548-550)
|   |-- tests/              Provided tests (system calls, producer/consumer)
|   `-- Makefile            Builds elevator.ko
|-- Makefile
`-- README.md               This file
```

## How to Compile and Run
**Requirements:** Linux 6.16.6 compiled and installed with system calls 548-550 (see Part 3), `gcc`, `make`, `build-essential`, `strace`, and `sudo` to load and remove kernel modules. We used the lab VM on the CS Proxmox cluster.

### Part 1
```bash
cd part1
make
strace -o empty.trace ./empty
strace -o part1.trace ./part1
```
`part1.trace` has exactly five more system calls than `empty.trace`.

### Part 2
```bash
cd part2
make                     # builds my_timer.ko
sudo insmod my_timer.ko
cat /proc/timer          # current time
sleep 1
cat /proc/timer          # current time and elapsed time
sudo rmmod my_timer
```

### Part 3
**Kernel (once):**
```bash
cd /usr/src/linux-6.16.6
make menuconfig
make -j$(nproc)
sudo make modules_install
sudo make install
sudo reboot
uname -r                 # 6.16.6
```

**Check the system calls:**
```bash
cd part3/tests/system-calls-test
make
sudo insmod syscheck.ko
./test-syscalls          # all three system calls should exist
sudo rmmod syscheck
```

**Build and run the elevator:**
```bash
cd part3
make                     # builds elevator.ko
sudo insmod elevator.ko
cd tests/elevator-test
make
./consumer --start
./producer 10
watch -n 1 cat /proc/elevator
./consumer --stop
sudo rmmod elevator
```

## Features Implemented
| Part | Feature | Where |
|------|---------|-------|
| 1 | Five extra system calls traced with strace | `part1/` |
| 2 | `/proc/timer` with current and elapsed time | `part2/src/my_timer.c` |
| 3a | System calls `start_elevator` (548), `issue_request` (549), `stop_elevator` (550) | `part3/syscalls.c`, kernel tree |
| 3b | Kernel 6.16.6 compiled and installed | lab VM |
| 3c | Elevator controlled by a kthread | `elevator.c` (`elevator_thread`) |
| 3d | Pets on floors and in the elevator kept in `linux/list.h` lists, allocated with `kmalloc` | `pet.c`, `elevator.c` |
| 3e | One mutex around all shared data | `elevator.c`, `elevator_mod.c` |
| 3f | Scheduling: direction, strict FIFO boarding, LOADING 1 s, moving 2 s | `elevator.c` |
| 3 | `/proc/elevator` output | `elevator_mod.c` (`elevator_read`) |

## Implementation Notes
**Mutex (3e).** One mutex in `struct elevator` protects the floor lists, the elevator list, the counters and the state. It is taken by the kthread for each step, by `issue_request` when adding a pet, by `start_elevator`/`stop_elevator` when changing the state, by the `/proc/elevator` read, and when freeing pets on unload. To keep the lock short and avoid deadlocks:
- `kmalloc` (which may sleep) and `copy_to_user` happen outside the lock.
- The kthread sleeps (`msleep`) only after unlocking, so `cat /proc/elevator` and the producer never wait for a whole floor move.
- `kthread_stop()` is never called while holding the lock, because the kthread may be waiting for it.

**Stopping.** `stop_elevator` only sets a `stopping` flag. The kthread keeps unloading pets but stops loading new ones, and switches to OFFLINE when the elevator is empty. It returns 1 if the elevator is already stopping.

**Unloading the module.** The system call pointers are cleared first, then the kthread is stopped, the proc entry removed, and finally every pet on the floors and in the elevator is freed.

**Proc buffer.** The 10,000-byte output buffer is allocated with `kmalloc` instead of on the kernel stack, which is only 16 KB.

## Known Bugs and Limitations
- None known at the time of submission. [Update after final testing]
- The project assumes pets never go to the floor they start on, and that the proc output is at most 10,000 characters, as allowed by the assignment.

## Development Log

### Jenny Jiang
| Date | Work done |
|------|-----------|
| 2026-10-14 | Collected everyone's FSU email and CS username and sent the lab VM request to the TA |
| 2026-10-21 | Set up access to the group VM (Proxmox console, SSH through shell.cs.fsu.edu) |
| 2026-10-30 | Added mutex locking to `issue_request`, start/stop and the proc read; added `mutex_init` |
| 2026-10-30 | Implemented stopping with a `stopping` flag (unload everyone, then OFFLINE) and the module cleanup order |
| 2026-10-30 | Debugged a module stuck during unload, caused by a list head passed by value (infinite loop while holding the lock) |
| 2026-11-01 | Fixed a double mutex unlock and the idle busy loop in the kthread; moved the proc buffer from the stack to the heap (PR #1) |
| 2026-11-02 | Wrote the README |

### Linh Nguyen
| Date | Work done |
|------|-----------|
| 2026-10-09 | Submitted the division of labor |
| 2026-10-16 | Created the GitHub repository |
| 2026-10-21 | Finished and tested Part 2 (`my_timer`); proposed `struct pet` and `struct elevator` |
| 2026-10-22 | Started the elevator kthread (3c) |
| 2026-11-01 | Updated elevator timing; strict FIFO boarding; fixed the direction at the top and bottom floors (PR #2, #3) |
| 2026-11-01 | Fixed the system call test build (PR #4) |
| 2026-11-02 | Fixed elevator startup recovery and cleaned the project for submission (PR #5) |

### Sid Nguyen
| Date | Work done |
|------|-----------|
| 2026-10-19 | Set up the lab VM account; added the starter files to the repository |
| 2026-10-20 | Added the system calls, compiled and installed the 6.16.6 kernel, and tested it with the provided files |
| 2026-10-21 | Part 1: `empty.c`, `part1.c` and both traces |
| 2026-10-30 | Imported the project from the VM into GitHub; first working version of the elevator; fixed scheduling |
| 2026-11-01 | Added and updated the Makefiles; set the top floor to 5; updated the Part 3 output and moved the proc buffer to the heap |

## Group Meetings
All three members took part in every meeting. Between meetings we kept in touch through a group chat.

**Meeting 1: early October 2026**
We read through the assignment and divided the work. Following the recommendation, we listed two members for each part, but agreed that each of us would lead our own parts. Linh submitted the division of labor on Canvas by October 9. We also decided that we needed a lab VM, so that a broken kernel could be rolled back instead of breaking our own computers.

**Meeting 2: October 14, 2026 (group chat)**
We prepared the VM request. Jenny collected everyone's FSU email and CS username and sent the request to the TA.

**Meeting 3: October 19-20, 2026**
Once the VM was ready, Sid set up the shared account and started compiling the kernel. Because the build took several hours, he asked the rest of us to log in and keep it running when he could not leave his laptop on. On October 20 Sid reported that the kernel was compiled, installed and tested with the provided files. Linh pointed out that the parts depend on each other (the elevator needs the system calls, and the mutex and proc output need the data structures), so we agreed to meet in person.

**Meeting 4: Wednesday, October 21, 2026**
We met in a study room to plan Part 3 and catch up on Parts 1 and 2.
- Sid finished Part 1 during the meeting.
- We walked through Part 2 together: how a kernel module is loaded and removed, how a proc file is created, and how reading `/proc/timer` calls our function. Linh finished and tested it that evening.
- We agreed on the elevator data structures. Linh proposed `struct pet` (type, start floor, destination floor, weight, list node) and `struct elevator` (current floor, weight, pet count, state, passenger list, waiting lists, mutex), plus the main functions (add a request, load, unload, move).
- We decided the order of work: system calls and kernel first, then the kthread, then the mutex and scheduling. Since we share one VM, we agreed to tell each other before loading a module or rebuilding the kernel.