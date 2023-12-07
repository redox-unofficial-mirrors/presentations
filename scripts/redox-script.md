# Redox OS Script

This script covers the first online presentation of Redox OS, I plan to follow the [FOSDEM](https://fosdem.org/) model for presentations because it brings more interest from the public.

**Each section will be a page**

### Prologue

- What Is Redox OS?
- Redox Basics Demo
- Influences
- Design
- Schemes
- An emulated system call example - epoll
- Current Status
- Development Plans
- Cosmic Editor Demo

### What Is Redox OS?

Redox OS is a general purpose operating system.
- The goal is a more secure, more reliable alternative to Linux
- Written in Rust
- Microkernel based
- Server, Desktop and Minimal versions
- MIT Licensed

### Origin Story

In 2015 the Rust programming language became stable (version 1.0), Redox OS was one of the first operating systems to exist on this language, it started as an unikernel and gathered the attention of many Rust developers.

They contributed and decided that the OS should be focused on security and reliability, thus the system migrated to a microkernel architecture and a single system API for resources.

Minix and Plan 9 were the main inspiration.

### Why Rust?

- A modern, ergonomic programming language with performance similar to C
- Memory safe - no buffer overflows, null pointers or use-after-free
- Excellent concurrency support

### Why a New OS?

- Writing an OS in a safe, performant, ergonomic language was the primary goal
- A microkernel architecture makes the OS more resilient in the face of hardware failures and software errors
- It was a chance to improve several system components, and rewrite them in Rust
  - RedoxFS, Orbital Window Manager, relibc (libc written in Rust) and Rust versions of many system utilities

### Design

- Redox uses a microkernel-based architecture, with system services provided as "Schemes" (a communication API)
- In a microkernel design, the kernel is kept to a minimum, just providing coordination between applications, services and resources
- Most system services are user-space daemons using system calls for inter-process communication

Why?

- This kernel model improve the system security and reliability a lot

### Influences

- Redox was influenced by many Unix-like operating systems
- Minix - The microkernel architecture and IPC model
- Plan 9 - The Scheme API for system communication and namespaces
- BSD - The capability-based security system from Capsicum (FreeBSD)
- Linux - System calls and performance optimizations
- seL4 - The microkernel design and component separation

### Device Drivers

- Drivers are user-space daemons that access devices through mmap-like system calls and IRQ event messages
- A panic or hang on a user-space driver won't halt the kernel - the driver can be restarted

### Filesystems

- A filesystem on Redox is a user-space daemon that provides open, read, write functions and communicates with a device driver daemon

Why?

- The filesystem can be restarted without a system shutdown

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

### System Calls

- The Redox system call interface is Unix-like
- However, many POSIX and Linux system calls are emulated via libredox and relibc
- We support the most common Linux system calls and are adding more to relibc
- We do not intend 100% compatibility with Linux or POSIX


Why?

- More program compatibility without modifications.

### libredox

- A system library for Rust crates - libredox wraps system calls to provide a stable API/ABI while allowing Redox to change implementation details
- libredox works in conjunction with relibc - for emulated system calls, the implementation is mostly within relibc


### relibc

- A C/C++ standard library written in Rust.
- Redox uses it to offer C/C++ support for programs and libraries

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

### Kernel Schemes

These are the most important kernel schemes from Redox (not exaustive list):

- : - Namespace manager
- user: - Dispatch for user-space schemes
- event: - epoll-like file descriptor read/write "ready" events
- memory: - Physical memory mapping manager
- pipe: - Kernel manager for pipes
- irq: - Interrupt manager (converts interrupts to messages)
- time: - Real-time clock timer "ITimer" (Interval timer)
- sys: - System hardware resources information
- kernel.acpi: - Read the CPU configuration (number of cores, etc)
- proc: - Process context manager
- thisproc: - Process context manager

### User-Space Schemes

These are the most important user-space schemes from Redox (not exaustive list):

- file: - Main filesystem (redoxfs service)
- orbital: - Orbital GUI (window manager and virtual driver)
- ip: - IP protocol (smolnetd service)
- audiohw: - Sound boards (sb16d, ac97d and ihdad drivers)
- usb.*: - USB interfaces
- disk.*: - Storage drivers
- network: - Ethernet boards (alxd, rtl8168d, ixgbed, e1000d and virtio-netd drivers)
- escalate: - Privilege manager (escalated service)
- initfs: - Startup filesystem (bootstrap service)
- audio: - Audio manager and virtual device (audiod service)
- chan: - Inter-process communication (ipcd service)
- shm: - Shared memory manager (ipcd service)

### Sandbox

- Each process has a namespace (like Plan 9)

Why?

- Sandbox by default in all programs (resource isolation)
- Hardened against malwares.

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

### What It Runs?

- NetSurf browser
- A graphical terminal emulator (orbterm)
- Music (with rodioplay or ffplay from ffmpeg)
- Video (ffplay from ffmpeg)
- Some games like Doom (SDL1/SDL2 games)
- Some emulators
- Some tools like Git and Vim
- More soon

### Development Plans

Our current development plans.

### Server Plans

This is a higher priority than desktop because it is presumed to be a smaller scope.
We have some work to do on optimizing drivers, especially network drivers.
The foundation is very much there and porting common server programs is an important step to take - things like Apache, Nginx, etc.

### Self-Hosting the Redox Website

We would like to host the Redox website on Redox running in a virtual machine.
The Redox Summer of Code projects have been filling in some of the gaps we have had for virtual machines.
Having drivers like the ones Andy worked on for VirtIO will enable much lower latency and higher performance virtual machines running Redox.
It gets us to the point where we might be able to run Redox on a cloud machine, like EC2 or DigitalOcean.

### Virtual Machine Support

There is a lot of work to be done on virtual machine support, device virtualization and Redox as a hypervisor.
Some of the Summer of Code work was in this area, and we are hoping to see this work continue.

## Self-Hosting the Build System

We want to self host the build system.
This has been a very long term goal that we have been working on over the entire course of Redox, and a very important one.
We have been able to port GCC, Binutils and other aspects of the build system to Redox, but the one remaining issue is that `rustc` itself doesn’t work very well on Redox.
The work we have to fix that is pretty involved, it’s at all places in the stack.

- We have to continue porting functions to `relibc`, and we have to continue porting Rust crates that that need Redox-specific changes.
- We have to continue implementing more driver and kernel support.
- Importantly, we have to implement more POSIX process and job management system calls and C library functions. This is one area that has been particularly difficult and 4lDO2 has been working really hard on it. So we hope to have some progress made within the next year. 

### Desktop Plans

### Cosmic Desktop

The Cosmic Desktop is something being worked on at System 76, where Jeremy works.
It’s an open source Linux desktop environment that is written mostly in Rust, and that aligns it with the goals of Redox.
There are some things we need to work on to have Cosmic Desktop on Redox.

- First, we need to port some of the applications that are going into Cosmic.
For example, the text editor that Jeremy is working on for Cosmic can be ported to run on top of Orbital, which is our current window manager for Redox.
- The second stage will be to port Wayland and the Cosmic compositor.
That will allow us gain all the applications on Cosmic and the other elements of the shell by having an actual Wayland compositor.
That will also enable us to easily port GTK, Qt, and other app frameworks like Electron.
A large list of applications that run on Linux are able to be ported to any Wayland environment, and should require minimal patching to run on Redox.

### Developer Tools

We need to work on porting developer tools to Redox.
That involves identifying the tools we want to port and ensuring that they are running properly. We have a lot of the basic things there, like the ability to compile C programs, Python and Lua and some other scripting languages that have been partially ported.

But the big one is still `rustc`, and then after that, porting other popular languages. Elixir and Go, for example, have their own runtime and are therefore more difficult than C and C++.

We also still have a lot of work to fully support C and C++, because of missing aspects of `relibc` that need to be fleshed out, e.g. wide string support.

### Performance

We need to start giving more attention to performance and responsiveness, and making Desktop and Demo go faster is an important part of that.

A majority of how people will interact with the system is to download our ISO and try it out on real hardware.
That’s a common thing for Linux distributions and we should assume that people will be trying to do with Redox.
Right now we have some issues that are stopping us in that direction.
The support for virtual machines is far better than for real hardware.
Our device drivers try to target common hardware, but they don't always work well.
For example, the high definition audio driver may work on one machine but hang on another machine.
Preventing hangs like that across the entire set of drivers that we have is a very important part to getting people to be able to see our demo image running on real hardware at native performance.

Then there is optimization.

4lDO2 has been working on various optimizations, like on demand paging.

Jeremy likes to port a lot of games, emulators and things like that, because it’s another way to see the performance.

When you can make a game run properly at very high speeds and very low latency, then you know the rest of the stack is basically good.

It becomes an easy benchmark to compare as you make changes.

### Future Goals

- Wayland support (port the COSMIC compositor)
- USB and Wi-Fi support
- ACPI support
- More ported programs and libraries

### Project Goals

- Be an alternative for Linux, BSD, MacOSX, Windows and Android
- Bring the full Rust potential for the Unix world
- Bring more safety and stability for computing