# Linux Systems, Kernel & Embedded Security Learning Plan

## Objective

Build a strong practical knowledge base in Linux by treating Linux as a system that can be:

- built from source
- configured and deployed
- inspected and instrumented
- extended with services
- modified at the kernel level
- connected to real hardware through device drivers
- hardened and attacked from a cybersecurity perspective

The long-term goal is to develop a project that combines:

**C/C++ → Linux → kernel → drivers → hardware → networking → embedded Linux → cybersecurity**

This path deliberately avoids forcing an early career decision between embedded systems, systems engineering, kernel development, and cybersecurity. The projects should provide enough exposure to discover which areas are most compelling.

---

# Overall Roadmap

```text
                    Linux fundamentals
                           │
                           ▼
              ┌────────────────────────┐
              │ Build Linux yourself   │
              │ boot → init → rootfs   │
              └────────────┬───────────┘
                           │
                           ▼
                  Linux internals
             processes / memory / IPC
             filesystems / networking
                           │
                           ▼
                 Kernel development
             modules → character driver
             → device model → interrupts
                           │
                           ▼
                  Embedded Linux
            cross compilation / DT / U-Boot
            Buildroot / Yocto / real hardware
                           │
                           ▼
                   Security layer
             attack surface / permissions
             secure boot / isolation / IPC
                           │
                           ▼
                 Capstone project
           embedded Linux + custom driver
              + service + security model
```

**Expected duration:** approximately 6–9 months at 6–8 hours/week, with the emphasis on genuine understanding rather than simply completing tutorials.

The key principle is:

> Every phase should produce something tangible.

---

# Phase 0 — Build the Laboratory

**Duration:** ~1 week

## Goal

Create a Linux development environment suitable for experimentation without immediately depending on physical embedded hardware.

## Recommended tools

- Linux VM or dedicated Linux machine
- Git
- GCC / Clang
- GDB
- QEMU
- Linux kernel source
- `make`
- CMake
- `strace`
- `ltrace`
- `perf`
- `ftrace`
- `bpftrace` later
- Wireshark
- serial-console tools when physical hardware is introduced

## Commands to become comfortable with

```bash
ps
top
lsmod
modprobe
dmesg
journalctl
systemctl
mount
lsblk
ip
ss
lsof
strace
readelf
objdump
nm
file
gdb
```

Do not learn these merely as commands to memorize.

For example:

```bash
strace ./program
```

should lead to the question:

> "What is happening at the boundary between this process and the kernel?"

That mental model is more valuable than memorizing command options.

---

# Phase 1 — Build Linux From the Ground Up

**Duration:** 3–4 weeks

## Project 1 — Build a Minimal Linux System

The first major objective is:

> Boot Linux in QEMU and get it to execute a program that I wrote.

Start with:

```text
Bootloader
    ↓
Linux kernel
    ↓
initramfs
    ↓
/init
    ↓
your program
```

Then progressively add:

- `/proc`
- `/sys`
- `/dev`
- BusyBox
- networking
- SSH
- your own init
- logging
- a service

The important realization is:

> Linux is not Ubuntu.

A Linux system is essentially:

```text
kernel + userspace + filesystem + init + configuration + applications
```

## Resources

### Linux From Scratch

Use Linux From Scratch selectively to understand how the components of a Linux system are assembled.

https://www.linuxfromscratch.org/lfs/read.html

Do not treat the goal as simply completing an LFS installation. The objective is understanding the pieces.

### Bootlin Embedded Linux

Bootlin has excellent free material covering:

- embedded Linux
- cross-compilation
- U-Boot
- kernel configuration/building
- root filesystems
- embedded build systems
- QEMU labs

https://bootlin.com/doc/training/embedded-linux/

---

# Phase 2 — Understand What the Kernel Actually Does

**Duration:** 4–6 weeks

Move beyond Linux commands and focus on operating-system concepts.

## Processes

Understand:

```text
fork()
exec()
wait()
clone()
exit()
```

and:

- PID
- process states
- scheduling
- context switching
- virtual memory
- signals
- file descriptors

## Project 2 — Miniature Process Supervisor

Write a small C/C++ program that:

- launches child processes
- monitors them
- restarts them
- captures stdout/stderr
- handles signals
- reports status

Then compare the concepts with:

- systemd
- supervisord
- containers

The purpose is to understand what a service manager actually does.

---

# Phase 3 — Linux Services and IPC

**Duration:** ~3 weeks

Study:

- Unix domain sockets
- pipes
- shared memory
- signals
- POSIX message queues
- sockets
- `epoll`
- systemd services
- permissions
- capabilities

## Project 3 — Multi-Process Sensor System

Build a small architecture such as:

```text
                ┌───────────────┐
                │ sensor daemon │
                └───────┬───────┘
                        │
                     IPC/socket
                        │
                ┌───────▼───────┐
                │ control daemon│
                └───────┬───────┘
                        │
                    REST/CLI
                        │
                ┌───────▼───────┐
                │     client    │
                └───────────────┘
```

Initially the sensor can be simulated.

Later, replace it with a real kernel device.

This transition is important because it connects userspace software with kernel and hardware concepts.

---

# Phase 4 — Kernel Development

**Duration:** 6–8 weeks

The official Linux kernel documentation should become one of the primary references.

https://docs.kernel.org/

## 1. Kernel Modules

Build:

```text
hello.ko
```

Learn:

```c
module_init()
module_exit()
```

Then study:

- module parameters
- `printk`
- kernel logs
- exported symbols
- Kbuild
- module dependencies

## 2. Kernel Memory

Understand:

```text
kmalloc
kzalloc
vmalloc
copy_to_user
copy_from_user
```

Key concept:

> Kernel memory and user memory are fundamentally different environments.

## 3. Concurrency

Spend serious time here.

Study:

- mutexes
- spinlocks
- atomic operations
- wait queues
- completions
- RCU
- workqueues
- interrupt context
- process context

A crucial question to understand:

> Why is sleeping sometimes illegal inside the kernel?

---

# Phase 5 — Build Your First Device Driver

This is a major milestone.

## Project 4 — Character Device Driver

Create something such as:

```text
/dev/mydevice
```

with support for:

```text
open()
read()
write()
ioctl()
poll()
close()
```

and a userspace client:

```text
mydevice-cli
```

Architecture:

```text
             USER SPACE

       ┌──────────────────┐
       │ mydevice-cli     │
       └────────┬─────────┘
                │
         read/write/ioctl
                │
─────────────── boundary ───────────────
                │
                ▼
       ┌──────────────────┐
       │ mydevice driver  │
       └────────┬─────────┘
                │
                ▼
          kernel subsystem
```

This should become a strong portfolio project because it demonstrates understanding beyond simply writing Linux applications.

---

# Phase 6 — Move to Real Hardware

Only after completing substantial QEMU/kernel work should physical embedded hardware become central.

Prefer platforms with strong Linux support rather than obscure development boards.

Possible platforms include:

- BeagleBone Black
- BeaglePlay
- STM32MP1/MP2 boards
- NXP i.MX boards

Bootlin's kernel material uses platforms such as BeagleBone Black, BeaglePlay and NXP i.MX93, and covers kernel configuration, the device model, Device Tree, character devices and hardware drivers.

https://bootlin.com/doc/training/linux-kernel/

---

# Phase 7 — Device Tree + Real Driver

This is where embedded Linux starts becoming especially relevant.

Understand the relationship between hardware and Linux:

```text
CPU
 │
 ├── MMU
 │
 ├── interrupt controller
 │
 ├── UART
 │
 ├── SPI
 │
 ├── I²C
 │
 └── GPIO
```

Study:

- Device Tree
- platform devices
- platform drivers
- Linux device model
- sysfs
- GPIO
- I²C
- SPI
- interrupts
- DMA eventually

## Project 5 — Real Sensor Driver

Use a simple I²C sensor such as:

- temperature sensor
- IMU
- environmental sensor

Target architecture:

```text
                Hardware
                    │
                   I²C
                    │
                    ▼
             Linux I²C subsystem
                    │
                    ▼
              your driver
                    │
                    ▼
               /dev/...
                    │
                    ▼
             userspace daemon
                    │
                    ▼
                 network
```

This is a real embedded Linux project rather than a simulation.

---

# Phase 8 — Build Your Own Embedded Linux Distribution

## First: Buildroot

Learn:

```text
toolchain
   ↓
bootloader
   ↓
kernel
   ↓
root filesystem
   ↓
applications
```

Buildroot is useful because it exposes these components without immediately introducing the complexity of Yocto.

Bootlin's Buildroot material:

https://bootlin.com/training/buildroot/

## Then: Yocto

Move to Yocto after understanding Buildroot.

The goal is to understand what Yocto's abstractions are actually producing rather than simply following recipes mechanically.

---

# Phase 9 — Add Cybersecurity

Do not make cybersecurity the first phase.

First make Linux itself your security laboratory.

## Linux Security Fundamentals

Study:

```text
UID/GID
permissions
capabilities
sudo
namespaces
cgroups
seccomp
LSM
AppArmor / SELinux
```

Then progressively attack your own system.

## Project 6 — Harden the Embedded System

Take the previous architecture:

```text
sensor
  ↓
kernel driver
  ↓
daemon
  ↓
network
```

Ask:

> What happens if the daemon is compromised?

Implement:

- non-root services
- minimal capabilities
- restricted filesystems
- seccomp
- read-only filesystems
- isolated namespaces
- firewall rules
- signed updates
- encrypted communication
- logging/auditing

This provides a much stronger cybersecurity exercise than simply completing isolated CTF challenges.

---

# Phase 10 — Capstone Project

Eventually the system should resemble:

```text
                     ┌──────────────────┐
                     │ Remote client    │
                     └────────┬─────────┘
                              │ TLS
                              ▼
                     ┌──────────────────┐
                     │ Embedded Linux  │
                     │ service         │
                     └────────┬─────────┘
                              │ IPC
                              ▼
                     ┌──────────────────┐
                     │ sensor daemon    │
                     └────────┬─────────┘
                              │
                         kernel API
                              │
                     ┌────────▼─────────┐
                     │ custom driver    │
                     └────────┬─────────┘
                              │
                            I²C/SPI
                              │
                     ┌────────▼─────────┐
                     │ physical sensor │
                     └──────────────────┘

Security:
──────────────────────────────────────────
secure boot
least privilege
capabilities
seccomp
encrypted communication
signed updates
logging / auditing
```

At this point the project demonstrates:

**C/C++ → Linux → kernel → drivers → hardware → networking → embedded Linux → security**

That is a coherent technical profile spanning embedded systems, systems programming and cybersecurity.

---

# Core Resources

Rather than collecting dozens of books and tutorials, use a small number of strong primary resources.

## 1. Linux Kernel Documentation

https://docs.kernel.org/

Use this continuously rather than reading it cover-to-cover.

It is the reference for current kernel APIs, subsystems, internals, tracing, testing, locking and driver development.

## 2. Bootlin

https://bootlin.com/docs/

This should be one of the main learning resources.

Relevant areas:

- Embedded Linux
- Linux kernel and driver development
- Buildroot
- Yocto
- debugging
- networking
- real-time Linux
- security

## 3. Linux From Scratch

https://www.linuxfromscratch.org/lfs/read.html

Use it to understand how a Linux userspace is assembled.

## 4. Linux Kernel Development Process

https://www.kernel.org/doc/html/latest/process/howto.html

Eventually learn how Linux kernel development and upstream contribution actually work.

## 5. Bootlin Embedded Linux Security

https://bootlin.com/training/security/

Useful later for:

- filesystem encryption
- CVEs
- SBOMs
- secure updates
- measured boot
- TPM
- IMA/EVM

---

# Recommended Learning Strategy

Do not follow the conventional path:

```text
Linux commands
      ↓
Linux administration
      ↓
Linux certification
      ↓
kernel
```

For an experienced C/C++ and embedded developer, a more efficient route is:

```text
             existing C/C++ knowledge
                         │
                         ▼
                   Linux internals
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          userspace              kernel
              │                     │
              └──────────┬──────────┘
                         ▼
                    embedded Linux
                         │
                         ▼
                       driver
                         │
                         ▼
                     hardware
                         │
                         ▼
                      security
```

The aim is to learn Linux by building things rather than spending months on introductory administration material.

---

# First 4-Week Milestone

The first concrete objective should be:

> **Build a minimal Linux system in QEMU from source, boot it with a custom kernel, create your own init process, add BusyBox, write a small C service, and interact with the system through `/proc` and `/sys`.**

Then deliberately break things and debug them using:

- `dmesg`
- `strace`
- GDB
- kernel tracing

If this milestone is completed properly, it should provide a surprisingly strong foundation before introducing physical embedded hardware.

---

# Guiding Principle

The project should continuously evolve rather than being discarded after each phase.

```text
Minimal Linux
      ↓
+ userspace service
      ↓
+ IPC
      ↓
+ kernel module
      ↓
+ character driver
      ↓
+ Device Tree
      ↓
+ physical sensor
      ↓
+ embedded Linux build system
      ↓
+ network interface
      ↓
+ security hardening
      ↓
CAPSTONE
```

The resulting repository should document not only the final project, but the experiments and failures encountered along the way.

That history is part of the learning.
