# OS-multi-container-runtime

## 1. Team Information

| Name | SRN |
|---|---|
| Anirudh Dharane | PES1UG24CS555 |
| Chathurth R | PES1UG24CS563 |

---

## 2. Project Overview

This project implements a lightweight multi-container runtime in C with a Linux kernel memory-monitor module. The system demonstrates core Operating Systems concepts including namespaces, process lifecycle management, IPC, synchronization, scheduling, and kernel/user boundary enforcement.

### Main Components
- **engine.c** — user-space runtime, supervisor daemon, CLI commands, metadata management, logging, container lifecycle.
- **monitor.c** — Linux Kernel Module (LKM) that monitors container memory usage and enforces limits.
- **monitor_ioctl.h** — shared ioctl interface between user space and kernel space.
- **memory_hog.c** — workload that grows memory usage.
- **cpu_hog.c** — CPU-intensive workload for scheduler experiments.
- **io_pulse.c** — I/O activity workload.
- **Makefile** — single-command build flow.

---

## 3. Features

- Multi-container supervision from one long-running supervisor
- CLI-based runtime control (`start`, `run`, `ps`, `logs`, `stop`)
- PID / UTS / mount namespace isolation
- Filesystem isolation using `chroot()`
- Per-container metadata tracking
- Per-container log capture for stdout/stderr
- UNIX domain socket control-plane IPC
- Kernel-space RSS monitoring with soft/hard thresholds
- Scheduling experiments using `nice`
- Clean teardown with reaping and no zombie processes

---

## 4. Build, Load, and Run Instructions

## Prerequisites
- Ubuntu 22.04 / 24.04 VM
- GCC / Make
- Linux headers installed
- Root privileges (`sudo`)

## Build Everything

```bash
make
```

## Prepare Root Filesystems

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

## Load Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

## Start Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

## Start Containers (new terminal)

```bash
sudo ./engine start alpha ./rootfs-alpha "sleep 60"
sudo ./engine start beta ./rootfs-beta "sleep 60"
```

## Run Foreground Container

```bash
sudo ./engine run gamma ./rootfs-alpha "exit 7"
```

## View Metadata

```bash
sudo ./engine ps
```

## View Logs

```bash
sudo ./engine logs alpha
```

## Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

## Inspect Kernel Logs

```bash
sudo dmesg | tail -n 30
```

## Unload Module

```bash
sudo rmmod monitor
```

---

## 5. CLI Contract

```text
engine supervisor <base-rootfs>
engine start <id> <container-rootfs> <command> [--soft-mib N] [--hard-mib N] [--nice N]
engine run   <id> <container-rootfs> <command> [--soft-mib N] [--hard-mib N] [--nice N]
engine ps
engine logs <id>
engine stop <id>
```

---

## 6. Demo with Screenshots

### 1. Multi-container supervision
Two containers managed by one supervisor.

![Screenshot 4](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/4.png)

### 2. Metadata tracking
Container IDs, PIDs, and states shown by `ps`.

![Screenshot 5](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/5.png)

### 3. Bounded-buffer logging
stdout and stderr captured to per-container logs.

![Screenshot 6](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/6.png)
![Screenshot 7](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/7.png)

### 4. CLI and IPC
CLI request sent and supervisor responded.

![Screenshot 8](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/8.png)

### 5. Soft-limit warning
Kernel warns when soft threshold exceeded.

![Screenshot 11](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/11%20monitor.png)

### 6. Hard-limit enforcement
Kernel kills container after exceeding hard threshold.

![Screenshot 11](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/11%20monitor.png)

### 7. Scheduling experiment
Different runtimes under different priorities.

![Screenshot 12](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/12.png)

### 8. Clean teardown
No supervisor and no zombie processes remain.

![Screenshot 14](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/14.png)

---|---|---|
| Multi-container supervision | [Screenshot 4](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/4.png) | Two containers managed by one supervisor |
| Metadata tracking | [Screenshot 5](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/5.png) | Container IDs, PIDs, and states shown by `ps` |
| Bounded-buffer logging | [Screenshot 6](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/6.png), [Screenshot 7](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/7.png) | stdout and stderr captured to per-container logs |
| CLI and IPC | [Screenshot 8](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/8.png) | CLI request sent, supervisor responds |
| Soft-limit warning | [Screenshot 11](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/11%20monitor.png) | Kernel warns when soft threshold exceeded |
| Hard-limit enforcement | [Screenshot 11](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/11%20monitor.png) | Kernel kills container after hard threshold |
| Scheduling experiment | [Screenshot 12](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/12.png) | Different runtimes under different priorities |
| Clean teardown | [Screenshot 14](https://github.com/anirudhdharane1/OS-Jackfruit/blob/e50d143433d9cbdb0765f81754bc2564f44aad9a/images/14.png) | No supervisor and no zombie processes remain |

---

## 7. Engineering Analysis

## A. Isolation Mechanisms

The runtime uses Linux namespaces to provide lightweight isolation while sharing the same host kernel.

- **PID namespace** gives processes inside the container an independent process tree. A process can appear as PID 1 inside the container while having a different host PID externally.
- **UTS namespace** isolates hostname/domain name, allowing each container to set its own hostname.
- **Mount namespace** isolates mount changes such as mounting `/proc` inside the container.
- **`chroot()`** changes the visible filesystem root for the process, preventing direct traversal of the host filesystem from inside the container.
- A production runtime may prefer `pivot_root()` for stronger mount-root isolation, but `chroot()` was chosen here for implementation simplicity.

All containers still share the host kernel, scheduler, physical memory, device drivers, and global kernel security policies.

## B. Supervisor and Process Lifecycle

A long-running supervisor centralizes container management.

Benefits:
- Single control point for all CLI commands
- Parent process can reap dead children with `waitpid()`
- Tracks metadata such as PID, state, start time, exit code
- Routes logs and handles stop requests via signals
- Avoids orphaned processes and zombie accumulation

Without a supervisor, each container would need independent management logic, making lifecycle control more complex.

## C. IPC, Threads, and Synchronization

Two IPC paths are used:

1. **Control Plane:** UNIX domain socket between CLI client and supervisor.
   - Used for `start`, `run`, `ps`, `logs`, `stop`
   - Chosen because it is local, efficient, and supports request/response semantics.

2. **Logging Plane:** pipes from container stdout/stderr into supervisor.
   - Output is processed and stored in log files.

### Shared Data Structures and Races
- Container metadata list can be accessed by multiple commands and reaper logic.
- Logging queue can be accessed by producers/consumers concurrently.
- Kernel monitor list is accessed by ioctl path and timer callback.

### Synchronization Choices
- **Mutexes** protect metadata and kernel linked-list operations because code paths may sleep.
- In kernel designs, synchronization primitives must match execution context because interrupt or timer paths cannot always sleep.
- **Condition variables** (bounded buffer design) coordinate producers and consumers efficiently.
- **Spinlocks** were not preferred because long critical sections or sleeping operations may occur.

## D. Memory Management and Enforcement

**RSS (Resident Set Size)** measures the amount of physical memory currently mapped into RAM for a process. It does **not** fully represent swap usage, shared-memory ownership semantics, or total virtual address space size.

### Why Two Limits?
- **Soft limit:** warning threshold for observability and early action.
- **Hard limit:** enforcement threshold that kills the offending process.

This mirrors real systems where monitoring and enforcement are separate policies.

### Why Kernel Space?
The kernel has authoritative visibility into process memory state and signal delivery. A user-space monitor can be delayed, bypassed, or race with process execution. Kernel-space enforcement is more reliable and immediate.

## E. Scheduling Behavior

Linux scheduling balances fairness, responsiveness, and throughput. Linux commonly uses the Completely Fair Scheduler (CFS), which tracks virtual runtime (vruntime) to distribute CPU time across runnable tasks. In our experiment, the lower-priority task (`nice +10`) completed slower than the default-priority task.

This shows:
- higher nice value reduces CPU preference
- scheduler still guarantees progress (fairness)
- priority influences throughput under contention

---

## 8. Design Decisions and Tradeoffs

| Subsystem | Design Choice | Tradeoff | Justification |
|---|---|---|---|
| Isolation | Namespaces + chroot | Shares host kernel | Lightweight compared to full VMs |
| Supervisor | Single long-running daemon | Single point of failure | Simplifies lifecycle management |
| IPC | UNIX socket + pipes | More components to manage | Clean separation of control vs logs |
| Kernel Monitor | Timer-based polling | Periodic checks, not continuous hooks | Simpler and sufficient for coursework |
| Scheduler Demo | `nice` priority comparison | Small timing differences possible | Directly demonstrates Linux scheduling |
| Filesystem Root Isolation | `chroot()` instead of `pivot_root()` | Weaker than full root pivoting | Simpler and reliable for coursework prototype |

---

## 9. Scheduler Experiment Results

| Workload | Priority | Completion Time |
|---|---|---|
| fast | default | 9.118s |
| slow | nice +10 | 10.003s |

### Interpretation
The lower-priority process took longer, indicating reduced scheduling preference. Linux maintained fairness while biasing CPU time toward the default-priority workload.

---

## 10. Validation Summary

| Requirement | Covered? | Best Screenshot |
|---|---|---|
| 1 Multi-container supervision | ✅ | Screenshot 4 |
| 2 Metadata tracking | ✅ | 1 / 4 / 5 |
| 3 Bounded-buffer logging | ✅ | 6 / 7 |
| 4 CLI and IPC | ✅ | 8 |
| 5 Soft-limit warning | ✅ | 11 |
| 6 Hard-limit enforcement | ✅ | 11 |
| 7 Scheduling experiment | ✅ | 12 |
| 8 Clean teardown | ✅ | 14 |

---

## 11. Cleanup

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
sudo rmmod monitor
```

---

## 12. Conclusion

This project demonstrates how core operating system mechanisms combine to build a practical container runtime. By integrating namespaces, process supervision, IPC, synchronization, scheduling control, and kernel memory enforcement, we implemented a compact but realistic system that mirrors the architecture of production runtimes at educational scale.

