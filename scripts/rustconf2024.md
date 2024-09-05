# RustConf 2024

- Explain what Redox is

Redox OS is a microkernel-based general-purpose operating system written in Rust that aims to be a complete Linux/BSD alternative.

- Explain the main features

```
- Microkernel-based

Most system components are common user processes that runs on user-space with root privileges and bugs don't change the memory of other system components causing data corruptions.

- Rust-written

Most memory allocations are verified by the compiler to prevent bugs, concurrency is immune to data races and NULLs can't cause undefined behavior.

- Unified system interface

Like Plan 9, all system resources are files (not RedoxFS file resources) and use the same interfaces for all tasks, it makes the system communication less complex and confusing.

Each system component provide "schemes" that provide resources, the scheme path format is `/scheme/scheme-name/resource-name`

- Copy-On-Write filesystem written in Rust

RedoxFS is the main Redox filesystem inspired by ZFS, it has the following features:

- Compatible with Redox and Linux (FUSE)
- Copy-on-write
- Data/metadata checksums
- Transparent encryption
- Standard Unix file attributes
- File/directory size limit up to 193TiB (212TB)
- File/directory quantity limit up to 4 billion per 193TiB (2^32 - 1 = 4294967295)
- Disk encryption fully supported by the Redox bootloader, letting it load the kernel off an encrypted partition.
- MIT licensed

Being MIT licensed, RedoxFS can be bundled on GPL-licensed operating systems (Linux, for example).
```

- Explain the [microkernel benefits](https://doc.redox-os.org/book/ch04-01-microkernels.html)

```
- More Stable

As most system components run on user-space with separated memory address spaces, a crash in one of them don't crash the whole system.

Also non-storage drivers can't corrupt your data because they can't arbitrarily access the filesystem memory. While in monolithic kernels drivers and filesystems share the same memory address space (same privilege), thus a buggy driver can overwrite (corrupt) the filesystem data.

- Better Security

Most security bugs can't spread to the whole system due to isolated system components, each system component has a unique namespace that discard acquired resources.

- Better Debugging

As most system components are common user processes their crashes can be debugged without a kernel panic.
```

- Explain the [Rust benefits](https://doc.redox-os.org/book/ch01-06-why-rust.html)

```
- Memory-Safety

All memory allocations are verified by the compiler to prevent bugs.

- Thread-Safety

Concurrent code is immune to data races.

- NULL-Safety

NULLs can't make undefined behavior.
```

- Explain the drivers

All Redox drivers are written in Rust, it avoid them to have memory errors and bad pointers that are very common bugs on drivers written in C.

- Explain relibc

relibc (Redox C Library) is a C standard library written in Rust, it also contain the Redox scheme interface, POSIX translation layer and stable ABI.

- Explain libredox

libredox (Redox System Library) is our library to provide a stable ABI for system components and programs.

- Explain Contain

Contain is the Redox sandbox driver to manage namespaces and their resources with capabilities, it was inspired by the FreeBSD's Capsicum.

- Explain Orbital

Orbital is the Redox display server and window manager written in Rust, it does CPU rendering and support some GUI frontends.

- Explain what [GUI backends support Orbital](https://doc.redox-os.org/book/ch04-09-graphics-windowing.html#libraries)

```
Currently Orbital supports the following GUI frontends:

- softbuffer
- winit
- Slint (through softbuffer and winit)
- Iced (through softbuffer and winit)
- egui (winit or SDL2 can be used)
- SDL1.2
- SDL2
- Mesa3D's OSMesa
```

- Explain [notable programs](https://doc.redox-os.org/book/ch01-10-important-programs.html) that Redox can run

```
- GCC
- LLVM
- FFMPEG
- OpenSSL
- Mesa3D
- SDL2
- Git
- RustPython
- GNU Bash
```

- Explain the [currently supported hardware](https://doc.redox-os.org/book/ch01-09-hardware-support.html)
- Run common Unix tools (ls, cd, rm, cat, grep, mkdir, cp, free -h)
- Show all installed programs (`ls /usr/bin`)
- Download `bootstrap.sh` with curl
- Print the `bootstrap.sh` file
- Download the `acid` repository
- Run `tokei` on the `acid` repository
- Play a music (ffplay)
- Play a video (ffplay)
- Do a directory processing benchmark (`time ls /usr/bin`)
- Do a Git benchmark
- Do a RAM benchmark using `dd`
- Do a I/O benchmark using `dd`