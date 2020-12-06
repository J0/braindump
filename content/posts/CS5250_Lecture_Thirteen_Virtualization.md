+++
title = "CS5250: Memory"
author = ["Joel Lee"]
draft = false
tags = ["Operating Systems", "CS5250"]
date = "2020-04-20"
+++

## Device Drivers {#device-drivers}
- Input-Output : Subsystem responsible for IO 
- 16-bit I/O devices. Seperate address space for I/O instructions 
- I/O devices are operated by means of registers. Registers are known by I/O port address 

## Linux uses a tree-like structure rooted at ioport\_resources to reflect hierachy of connections 

- I/O interface is a hardware circuitry inserted between a group of I/O ports and their corresponding device controller {
- Different types of I/O interfaces: Graphics, Disk, Mouse, network 
- Device controller interprets high-level commands from I/o interface and forces device to execute specific actions by sending proper sequences of electrical signals to it 

## Memory-mapped I/O. Suitable for fast-devices and DMA. PCI Express -> Concept of a lane which allows you to transport data up to 32 lanes 

- Device Drivers hide details of the hardware from software. They use standardized calls 
- Device Driver does not enforce policies. It just implements mechanism to access hardware.
- Easier user model. Driver behaves like an array of blocks. We split the kernel
- Loadable module -> Add/Remove kernel features at runtime. Modules are the non-core parts of an operating system.
- Classes of Device and modules
- Character Devices are a stream of bytes. Supports open, close, read, write. Fundamental virtual file system(VFS)
- Network devices->Send and receive data packets. They have unique names like eth0. File system modules are software drivers and not device drivers.
- Drivers do not define security policies -> They provide mechanisms to enforce policies


#### SysFS {#sysfs}

- Linux driver model: Everything is a file with exception of network cards
- SysFS exposes a hierachical relatoinship between devices.
- Device drivers are held using KObjects->Correpsonds to directory in sysfs. KObjects are grouped into Ksets. Device has a device object and each device
- Each device has a major-minor number. Each device will request.
- Character devices
- Kernel maintains a hash table chrdevs containing interfals of device numbers with handlers
- Prime example of a block device is a disk. We have a mapping layer to determine physical location of block
- Below that is a scheduler that will fire up a block device driver.
- I/O Schedule rwill figure out what's happening.
- Hardware transfer data in sector. VFS uses logical unit called blocks. Blcoks can work with data in segments. Disk caches work with pages.
- Zero copy -> Put data directly into user space
- Logical Volumte Management/RAID. \`bio\` contains all details about an ongoing operation
- Generic block layer fires off random block I/O opeartions. But random access is bad for performance. I/O Scheduler tries to group I/O operations for best performance
 - I/O Scheduler must ensure deadlocks and all works as well.


## Virtual Machines {#virtual-machines}

- Real resource/host platform and a guest system which runs on top of the virtual machine. Allow other virtual machines to run the entire stack. On x86 hardware you can have a different OS running. 
- User has full control of hardware. From user point of view it looks like a disk. 
- Time sharing VMM maintain overall control of all the hardware ressources. First VM context switching, switch to new VMM. VMM maintains control of overall hardware. 
- VMM ->Hypervisor is part of the hardware. We can use Virtual box-> Run hypervisor in non-privilleged mode. VMM can run in both modes 
- Emulation/Binary Translation -> Processor virtualizes 


## Privilleged instructions ->Trap if the machine is in user mode but does not trap if the machine is in system mode. 
- Two cateogries of instructions 
- Control-sensitive isntructions ->Attempt to change the configuration of resources in the system
- Behavior-sensitve instruction -> Result depends on the configruation of the resources 
- Innocuous instructions -> Neither control/behavior sensitive. 
- VMM must satisfy efficiency, resource control, equivalence 

## Theorem: VM privillegd instructions VMM will do the right thing and then return control to Guest OS 

- Example: set CPU timer. Load the difference in internal time so that time can be restored. Should the wall clock be different from reality?
- Can in theory go for recursive virtualization but it's not clear if you really want to.
