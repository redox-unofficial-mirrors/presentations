# Redox OS Script

This script covers the first online presentation of Redox OS, I plan to follow the [FOSDEM](https://fosdem.org/) model for presentations because it brings more interest from the public.

**Each section will be a page**

### Prologue

On this presentation we will explain:

- What Is Redox OS?
- Why Rust?
- Memory Safety
- Compiler Enforcement
- Why a New OS?
- Design
- Microkernel
- Device Drivers
- Filesystems
- System Calls
- Bug Isolation
- Rare Kernel Panics
- Single System API
- Sandbox
- Influences
- Minix
- Plan 9
- BSD
- Linux
- seL4
- How It Works?
- How System Interfaces Are Implemented?
- Hardware Support 
- Program Compatibility
- libredox
- relibc
- What It Runs?
- Future Goals
- Project Goals

### What Is Redox OS?

- Redox OS is a general purpose operating system written in Rust. Our aim is to provide a fully functioning Unix-like microkernel-based operating system, that is secure, reliable and free

### Why Rust?

- A modern, ergonomic programming language with performance similar to C
- Memory safe - no buffer overflows, null pointers or use-after-free
- Excellent concurrency support
### Why a New OS?

- Writing an OS in a safe, performant, ergonomic language was the primary goal
- A microkernel architecture makes the OS more resilient in the face of hardware failures and software errors
- It was a chance to improve several system components, and rewrite them in Rust
  - RedoxFS, Orbital Window Manager, relibc (libc written in Rust) and many Rust versions of system utilities
### Design

- Redox follow a microkernel-based architecture and a single system API called "Scheme"

### Microkernel

- In a microkernel design, most system components are user-space daemons with inter-process communication (like Minix)
- Drivers and filesystems, for example

Why?

- This kernel model improve the system security and reliability a lot

### Device Drivers

- Drivers are user-space daemons
- Being a common Unix process they can have their own namespace with restricted schemes
- A driver on Redox can't damage other system interfaces

Why?

- Drivers on monolithic kernels can't be isolated from the kernel
- In a monolithic kernel, a bad driver could wipe your data, because the driver run on the same address space of the filesystem (thus same privilege level)

### Filesystems

- A filesystem on Redox is an user-space daemon

Why?

- Bad drivers or kernel bugs can't damage your data
- The filesystem can be restarted without a system shutdown

### System Calls

- The Redox system call interface is Unix-like
- We have open, pipe, pipe2, lseek, read, write, brk and execv, for example
- Currently, we support the most common Linux system calls

Why?

- More program compatibility without modifications.
### An emulated system call example - epoll

- Linux's epoll provides a way to watch for data to become available on multiple file descriptors
- The application provides a list of file descriptors to watch, some callback data, and an optional timeout
- Linux's epoll is implemented in relibc using Redox's event scheme
  - The application calls `epoll_create1`- internally, relibc calls `open` on the "event:" scheme, and returns the event_fd file descriptor
  - The application calls `epoll_pwait` using the event_fd and a list of file descriptors to wait on - relibc does the following
  - If a timeout is being used, the "time:" scheme is opened, and a message is written to the time_fd requesting a message in n milliseconds
  - A message is written to the event_fd to watch the time_fd
  - The list of file descriptors provided by the application is also written to the event_fd
  - relibc then does a blocking read of the event_fd, waiting for activity
  - The "event:" scheme, which is in the kernel, monitors activity on other schemes, and detects when a file descriptor is ready
  - Since the "time:" scheme is implemented as file service, the timeout is just another file descriptor to watch
  - When a monitored file descriptor becomes ready, the "read" of the event_fd unblocks and the list of active fds is read
  - If there was a timeout event, it is deleted from the list of events before returning (this is transparent to the application)
  - The application reads the list of active fds, as it would if it was running on Linux
### Bug Isolation

- Most system components are isolated from other components on memory (different address spaces like normal programs), thus bugs on some system component can't spread to other components

### Rare Kernel Panics

- The kernel code size is very small, as 95% of the system components runs on user-space
- The number of bugs can be easily close to zero (KISS principle)

### Single System API

- All system resources are files (like Plan 9)
- Each component implement schemes for its resources

Why?

- Reduce the complexity of the system
- More flexibility between components

### Sandbox

- Each process has a namespace (like Plan 9)

Why?

- Sandbox by default in all programs (resource isolation)
- Hardened against malwares.

### Influences

- Redox was influenced by many Unix-like operating systems

### Minix

- The microkernel architecture and IPC model

### Plan 9

- The Scheme API for system communication and namespaces

### BSD

- The capability-based security system from Capsicum (FreeBSD)

### Linux

- System calls and performance optimizations

### seL4

- The microkernel design and component separation

### How It Works?

- System components use system calls instead of internal kernel APIs
- There are user-space schemes and kernel schemes for many use cases

### How System Interfaces Are Implemented?

- Schemes are implemented by each system daemon or the kernel

### Hardware Support

- IDE, SATA and NVMe (HDD/SSD devices)
- PCI (most motherboard-based devices, like dedicated GPUs)
- Limited PCI Express support
- Ethernet (Intel/Realtek)
- Sound (Intel/Realtek)
- VGA/GOP (all computer displays)
- USB and Wi-Fi soon

### Program Compatibility

- Programs using Rust Standard Library (libstd)
- Programs using ported Rust crates
- Partial compatibility with POSIX programs
- Source compatible with Linux/BSD programs and libraries

### libredox

- A system library for Rust crates

### relibc

- A C/C++ standard library written in Rust.
- Redox uses it to offer C/C++ support for programs and libraries

### What It Runs?

- NetSurf browser
- A graphical terminal emulator (orbterm)
- Music (with rodioplay or ffplay from ffmpeg)
- Video (ffplay from ffmpeg)
- Some games like Doom (SDL1/SDL2 games)
- Some emulators
- Some tools like Git and Vim
- More soon

### Future Goals

- Wayland support (port the COSMIC compositor)
- USB and Wi-Fi support
- ACPI support
- More ported programs and libraries

### Project Goals

- Be an alternative for Linux, BSD, MacOSX, Windows and Android
- Bring the full Rust potential for the Unix world
- Bring more safety and stability for computing