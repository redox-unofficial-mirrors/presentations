# Redox OS Script

This script covers the first online presentation of Redox OS, I plan to follow the [FOSDEM](https://fosdem.org/) model for presentations because it brings more interest from the public.

**Each section will be a page**

### Prologue

On this presentation we will explain:

- What Is Redox OS?
- Design
- Influences
- How It Works?
- How System Interfaces Are Implemented?
- Hardware Support 
- Program Compatibility
- Future Goals
- Project Goals

### What Is Redox OS?

Redox OS is a general purpose operating system written in Rust. Our aim is to provide a fully functioning Unix-like microkernel-based operating system, that is secure, reliable and free

### Design

- Redox follow a microkernel-based architecture and an unified system API called Scheme
- Most system components are isolated from other components on memory (different address spaces like normal programs)
- Hardened against kernel panics (most system components runs on user-space)
- Bugs can't spread over components
- All resources are files (single system interface for everything)
- Each process has a namespace (sandbox)

### Influences

- Minix - The microkernel architecture and IPC model
- Plan 9 - The Scheme API for system communication and namespaces
- Linux - System calls and performance optimizations
- BSD - The capability-based security system from Capsicum (FreeBSD)
- seL4 - The microkernel design and separation

### How It Works?

- Most system components are user-space daemons with inter-process communication
- System components use system calls instead of internal kernel APIs
- There are user-space schemes and kernel schemes for many use cases

### How System Interfaces Are Implemented?

- Schemes are the way to use system resources
- Each system component offer a scheme
- libredox is a system library for Rust crates
- The relibc library offer C/C++ support for programs and libraries

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