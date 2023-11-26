# Redox OS Script

This script covers the first online presentation of Redox OS, I plan to follow the [FOSDEM](https://fosdem.org/) model for presentations because it brings more interest from the public.

**Each section will be a page**

### Prologue

On this presentation we will explain:

- What Is Redox OS?
- Origin Story
- Why Rust?
- Memory Safety
- Compiler Enforcement
- Why a New OS?
- Design
- Microkernel
- Device Drivers
- Filesystems
- System Calls
- libredox
- relibc
- An emulated system call example - epoll
- Bug Isolation
- Rare Kernel Panics
- Single System API
- Kernel Schemes
- User-Space Schemes
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
- What It Runs?
- Future Goals
- Project Goals

### What Is Redox OS?

- Redox OS is a general purpose operating system written in Rust. Our aim is to provide a fully functioning Unix-like microkernel-based operating system, that is secure, reliable and free

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
  - RedoxFS, Orbital Window Manager, relibc (libc written in Rust) and many Rust versions of system utilities

### Design

- Redox uses a microkernel-based architecture, with system services provided as "Schemes" (a communication API)
- In a microkernel design, the kernel is kept to a minimum, just providing coordination between applications, services and resources
- Most system services are user-space daemons using system calls for inter-process communication

Why?

- This kernel model improve the system security and reliability a lot

### Device Drivers

- Drivers are user-space daemons that access devices through mmap-style system calls and IRQ event messages
- A panic or hang on a user-space driver won't halt the kernel - the driver can be restarted

### Filesystems

- A filesystem on Redox is a user-space daemon that provides open, read, write functions and communicates with a device driver daemon

Why?

- The filesystem can be restarted without a system shutdown

### System Calls

- The Redox system call interface is Unix-like
- However, many POSIX and Linux system calls are emulated via libredox and relibc
- We support the most common Linux system calls and are adding more to relibc, but we do not intend 100% POSIX or Linux compatibility


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

### Kernel Schemes

- root: - Namespace manager
- user: - Dispatch for user-space schemes
- debug: - Debug messages that can't use the log: scheme
- event: - epoll-like file descriptor read/write "ready" events
- memory: - Physical memory mapping manager
- pipe: - Kernel manager for pipes
- serio: - Serial I/O (PS/2) driver (must stay in the kernel due to PS/2 protocol issues)
- irq: - Interrupt manager (converts interrupts to messages)
- time: - Real-time clock timerITimer (Interval timer)
- sys: - System hardware resources information
- procfull: - Process context manager (proc)
- procrestricted: - Process context manager (thisproc)

### User-Space Schemes

- file: - Redox filesystem (redoxfs service)
- orbital: - Orbital GUI (window manager and virtual driver)
- ip: - Network protocol (smolnetd service)
- udp: - Network protocol (smolnetd service)
- tcp: - Network protocol (smolnetd service)
- icmp: - Network protocol (smolnetd service)
- netcfg: - Network protocol (smolnetd service)
- dns: - Network protocol (dnsd service)
- pcspkr: - PC speaker  (pcspkrd driver)
- audiohw: - Audio boards (sb16d, ac97d and ihdad drivers)
- disk.usb-{id}+{port}-scsi: - USB SCSI  (usbscsid driver)
- network: - Ethernet adapters (alxd, rtl8168d, ixgbed, e1000d and virtio-netd drivers)
- bga: - Bochs emulator/debugger  (bgad driver)
- disk.live: -  (lived driver)
- acpi: - ACPI (acpid driver)
- input: - Virtual device (inputd driver)
- display.virtio-gpu: - VirtIO GPU (virtio-gpud driver)
- display.vesa: - VESA (vesad driver)
- zero: - Discard all writes, and always fill read buffers with zeroes (zerod service)
- null: - Discard all writes, and read no bytes (nulld service)
- escalate: - Privilege manager (escalated service)
- pty: - Terminal manager and virtual device (ptyd driver)
- rand: - Pseudo-random number generator (randd service)
- log: - Logging (logd service)
- initfs: - Startup filesystem (bootstrap service)
- audio: - Audio manager and virtual device (audiod service)
- chan: - Inter-process communication (ipcd service)
- shm: - Shared memory manager (ipcd service)

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