# Unit 4: Processes

## Processes, introduction

A process is an abstraction that refers to each execution program. **Notice that a process has not got to be always executing, a process life cycle has several stages, execution is one of them**.

We can think of a process as:
- Each instance of an executing programm
- The entity the O.S. creates to execute a program

A process consists of:
- Adress space: the set of adresses the process may reference
- Control point: next instruction to be executed

### Regions

In its simplest form, a process adress space has three regions
- **Code:** code of all functions in the program that the process is running.
- **Data:** global variables of the program. Dynamically assigned memory (heap) is usually a part ofthis region.
- **Stack:** Used for parameter pssing and to store the return address once a function is called. It is also used by the function being called to store its local variables.

### Multitasking

When a system executes more than a process at the same time, they seem to be executing at the simoultaneously. Actually, the CPU keeps changing between them, so at any given instant **only one of them** is actually executing.

One CPU can only be executing one process at a time. Several CPUs are needed in order to have real parallel execution.

## Processes in UNIX

### What is the kernel?

The kernel is the term in which we usually refer to the O.S. itself. It resides in a file which gets loaded by the boot loader during the machine bootstrap procedure.

It initializes the system and creates the environment to execute processes. It creates some processes, which, in turn, will create the rest of the processes in the system.

The UNIX kernel is the only program to run directly on the system hardware. User processes do not interact directly with the hardware but use the system call interface instead.

### Kernel mode and user mode

Operating systems need more than a running mode to operate:
- **User mode:** user code runs in this mode
- **Kernel mode:** kernel runs in this mode
    - **System call:** A user process explicity requests some service from the kernel via the system call interface
    - **Exceptions:** Exectional situations (division by 0, addressing errors...) cause hardware traps that require kernel intervention
    - **Interrupts:** Devices use interrupts to notify the kernel of certain events (i/o completion, change of the state of a device...

Some processes or instructions can only be executed in kernel mode.

[Operating systems; Kernel Mode and User Mode](https://www.youtube.com/watch?v=o_JCGR_9HRs&ab_channel=IanTeachesThings)

