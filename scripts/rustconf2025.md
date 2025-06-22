# RustConf 2025

### What Redox is?

Redox OS is a microkernel-based general-purpose operating system written in Rust that aims to be a complete Linux/BSD alternative.

### Features

- Microkernel Architecture

Most system components are common user processes which runs on userspace with high privileges, bugs can't corrupt the memory of other system components due to virtual memory isolation.

- Written in Rust

Most memory allocations are verified by the compiler to prevent bugs, multi-threading is reliable and NULLs can't cause undefined behavior.

- Unified System Interface

Like Plan 9, all system resources are files (not filesystem resources) and use the same interfaces for all tasks, it makes the system communication less complex and confusing.

Each system component provide "schemes" which provide resources, the scheme path format is: "/scheme/scheme-name/resource-name"

- Copy-On-Write Filesystem Written in Rust

RedoxFS is the main Redox filesystem inspired by ZFS, which contains the following features:

- Copy-on-write
- Data/metadata checksums
- Transparent encryption
- Standard Unix file attributes
- File/directory size limit up to 193TiB (212TB)
- File/directory quantity limit up to 4 billion per 193TiB (2^32 - 1 = 4294967295)
- Disk encryption fully supported by the Redox bootloader, letting it load the kernel from an encrypted partition.
- FUSE support

Being MIT licensed, RedoxFS can be bundled on GPL-licensed operating systems (Linux, for example).

### Microkernel Benefits

- More Stable

As most system components run on userspace with separated memory address spaces, a crash in one of them can't crash the whole system.

Also non-storage drivers can't corrupt your data because they can't arbitrarily access the filesystem memory. While in monolithic kernels drivers and filesystems share the same memory address space (same privilege), thus a buggy driver can overwrite (corrupt) the filesystem data.

- Better Security

Many security bugs can't spread to the whole system due to isolated system components, each system component has an unique namespace that discard acquired resources.

- Better Debugging

As most system components are common user processes their crashes can be debugged without a kernel panic.

### Rust Benefits

- Memory-Safety

All memory allocations are verified by the compiler to prevent bugs.

- Thread-Safety

Multi-threaded code is reliable to be used, preventing obscure bugs.

- NULL-Safety

NULLs can't make undefined behavior.

### Drivers

All drivers are written in Rust which avoid them to have memory errors and bad pointers which are very common bugs on drivers written in C.

### relibc

relibc is a C standard library written in Rust, it also contain the scheme interface, POSIX compatibility layer and stable ABI.

### libredox

libredox (Redox System Library) is our library to provide a stable ABI for system components and programs.

### Contain

Contain is the Redox experimental sandbox driver to manage namespaces and their resources with capabilities, it was inspired by the FreeBSD's Capsicum.

It currently allow namespace filtering to remove not allowed system resources, similar to Flatpak permissions and OpenBSD pledge.

### Orbital

Orbital is the Redox display server and window manager written in Rust, it does CPU rendering and support some GUI frontends.

### Supported GUI Frontends

Currently Orbital supports the following GUI frontends:

- softbuffer
- winit
- Slint (through softbuffer and winit)
- Iced (through softbuffer and winit)
- egui (winit or SDL2 can be used)
- SDL1.2
- SDL2
- Mesa3D's OSMesa
- X11

### Notable Programs

Notable programs that Redox can run:

- GCC
- LLVM
- FFMPEG
- OpenSSL
- Mesa3D
- X11
- SDL2
- Git
- RustPython
- GNU Bash

### Supported Hardware

- Intel/AMD (32/64-bit) and ARM 64-bit CPUs
- ACPI, PCI and USB hardware interfaces
- IDE (PATA), SATA (AHCI) and NVMe storage interfaces
- BIOS VESA and UEFI GOP video
- Intel and Realtek sound chipsets
- Intel Gigabit/10 Gigabit and Realtek ethernet
- PS/2 and USB keyboards, mouse and touchpad

## Demo

- Start QEMU with the "make qemu uefi=yes disk=nvme" command for better performance and screen resolution
- Run common Unix tools (ls, cd, rm, cat, grep, mkdir, cp)
- "cat Welcome.txt"
- "grep "Redox" Welcome.txt"
- "cp Welcome.txt test"
- "ls"
- "pwd"
- "mkdir test"
- "cd test"
- "rm -r test"
- "free -h"
- "df -h"
- Show all installed programs ("ls /usr/bin")
- Download "podman_bootstrap.sh" with curl ("curl -sf https://gitlab.redox-os.org/redox-os/redox/raw/master/bootstrap.sh -o bootstrap.sh
")
- Print the "podman_bootstrap.sh" file ("cat bootstrap.sh")
- Download the "acid" repository with benchmark ("time git clone https://gitlab.redox-os.org/redox-os/acid")
- Run "tokei" on the "acid" repository ("tokei acid")
- Play a music (ffplay)
- Play a video (ffplay)
- Do a directory processing benchmark ("time ls /usr/bin")
- Do a RAM benchmark using "dd" ("dd bs=1M count=1024 if=/scheme/zero of=/scheme/null
")
- Do a filesystem read benchmark using "dd" ("dd bs=1M count=256 if=/usr/games/neverball/neverball of=/scheme/null conv=fdatasync
")
- Do a filesystem write benchmark using "dd" ("dd bs=1M count=256 if=/usr/games/neverball/neverball of=fs_write_speed_bench conv=fdatasync
")
