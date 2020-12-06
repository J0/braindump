+++
title = "CS5250 Lecture One(partial)"
author = ["Joel Lee"]
tags = ["Operating Systems", "CS5250"]
date = "2020-01-13"
draft = false
+++

## Traps and interrupts {#traps-and-interrupts}
-   interrupts  are async to program execution and cause by internal events
-   Traps are synchronous to program execution and are caused by internal events. The condition must be remedied by trap handler for instruction and it must stop the offending instruction midstream.
-   Offending instruction may be retried and the program may continue or it may be aborted.


### Interrupt handling {#interrupt-handling}

-   Arithmetic overflow: What happens if we divide by zero and carry on?
    -   Your result will be wrong
    -   You will have no idea where it went wrong
-   In divide by zero we can have two scenarios
    -   Signalling not a number: The moment computation goes to NaN it will escalate and bomb your program.
-   Undefined instruction: \`jmp\` to area that doesn't contain an instruction
-   TLB/Page fault
    -   Needed for virt memory maintainance
-   Segmentation fault
    -   You access an invalid address
-   I/O Service request
-   Hardware Malfunction
-   Timer interrupt


## Unix {#unix}

-   Some courses are taught using OpenBSD


## Windows {#windows}

-   Architecture of Windows
-   there is a system library ntdll which can't be deleted


## macOS {#macos}

-   Variant of Unix
-   Kernel with utilities put on top of it together with various core services


## AndroidOS {#androidos}

-   Based off linux kernel. It has restructured linux in a v different way.


## Generic OS {#generic-os}

-   Contains libraries that interact directly with the hardware


## Monolithic design {#monolithic-design}

-   Usually a single large process
-   Runs in a single address space- kernel space
-   Often a single binary file lodaded at boot time.


## Microkernel advantes {#microkernel-advantes}

-   Security advantage: Principle of least privillege
-   Performance disadvantage as we need to cross the kernel boundary


## Lecture Two: Introduction to Intel Architecture and Programming {#lecture-two-introduction-to-intel-architecture-and-programming}

We will focus on Intel x86

-   Requires basic knowledge of computer architecture


## Nomenclature {#nomenclature}

-   IA-32: 32bit x86. Intel partnered with HP to produce a 64 bit processor and they wanted to wipe the slate clean.
-   Intel64 is Intel's version of AMD64 which was AMD's 64bit extension of IA-32
-   Intel architecture is little endian. Memory is treated as an array of bytes

Every x86 processor supports 32 bit mode for backward compatibility.


## Intel 64-registers {#intel-64-registers}

-AX is 1 16 bit register. Consists of AH(8 bits) and AL(8 bits)


## EFLAGS {#eflags}

-   Interrupt flag


## Operating mode {#operating-mode}

-   Real-address mode. Impl prog env of Intel8086 within sec


## Protected mode {#protected-mode}

- Privillege

