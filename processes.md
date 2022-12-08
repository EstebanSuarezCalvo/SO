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

### Threads and processes

In a traditional Unix system a process is defined by:
- **Adress space:** Set of memory adresses the process can reference
- **Control point:** Indicates which is the next instruction to execute

In a modern Unix system a process can have several control points (threads).

### Adress space

Processes use virtual adresses. A part of their virtual address space corresponds to the kernel code and data. It is called system space or kernel space and it can only be reached when in kernel mode.

The unix kernel is a C program, and as such it has:
- **Kernel Code:** what is run when the system is executing in kernel mode: system calls code, interrupt and exception handlers
- **Kernel Data:** global variables of the kernel, accesible for all the processes in the system (process table, inode table...)
- **Kernel Stack:** part of memory used as stack when executing in kernel mode: parameter passing inside the kernel, local variables of kernel functions...

### Reentrant kernel

Unix kernel is reentrant:
- Several processes can be running simoultaneously several kernel functions
- Several processes can be running simoultaneously the same kernel function

For the kernel to be reentrant:
- **Kernel Code** must be read only
- **Kernel Data** must be protected from concurrent acces
- Each process has its own **kernel stack**

#### Kernel data protection

Traditional approach (non preemptible kernel)
- A process running in kernel mode can not be preempted, it only leaves the CPU if it ends, blocks or returns to user mode
- Only certain kernel data structures need to be protected. Protecting them is simple, just a flag in use/not in use

Modern approach (preemptible kernel)
- A process running in kernel mode can be preempted if a higher priority process apears ready
- All kernel data structures must be protected by more sophisticated means (like semaphores)

More complex mechanisms are needed in miltiprocessor systems

## Data structures

In order to manage processes, the O.S. needs:
- To asign memory for the program to be loaded.
- As several proceses can run the same program, the O.S. has to identify them (*Process Descriptor* and *Process Identifier*)
- When the process is not being executed, the O.S. needs to keep execution information: registers, memory, resources.
- The O.S. needs to know the list of processes in the system and the state in which each of them is. Usually, it uses lists of Process Descriptors, one for each state or even one for each i/o device.

### System Control Block

**A *System Control Block (SCB)* is a set of data structures used by the O.S. to control the execution of processes by the system.**

It usually incluces:
- List of all Process Descriptors
- Pointer to the process currently in CPU (its Process Descriptor)
- Pointer to lists of processes in different states: lists of runnable processes, list of i/o  blocked processes...
- Pointer to a list of resource descriptors
- References to the hardware and software interrupt routines and to the error handling routines.

### Process Control Block

**A *Process Control Block (PCB)* is a data structure used by the O.S. to store all the information about a process.**

- The O.S. has a table of processes where it keeps information of each process in the system.
- Each entry in this table has information on ONE PROCESS
- The Process Control Block keeps the data relevant to one process that the O.S. uses to manage it (PCB)

It usually includes:
- Identification:
    - Process identifier
    - Parent process identifier
    - User and group identifiers
- Scheduling:
    - State of the process
    - If the process is blocked, event the process is waiting for
    - Scheduling parameters: process priority and other information relevant to the scheduling algorithm
- References to the asigned memory regions:
    - Data region
    - Code region
    - Stack region
- Assigned resources:
    - Open files: file descriptor table
    - Assigned communication ports
- Inter-process comunication information

[Process Control Block](https://www.youtube.com/watch?v=4s2MKuVYKV8&ab_channel=NesoAcademy)

## Data structures in UNIX

To implement the concept of a process, unix uses concepts and structures that allow a program to execute.

- **User Adress Space:** Code, data, stack, shared memroy regions, mapped files...
- **Control Information:** 
    - *proc* structure
    - *u_area*
    - kernel stack
    - address translation maps
- **Credentials:** indicate which user is behind the execution of that process
- **Environment variables:** an alternate method to pass information to the process
- **Hardware context:** the contents of the hardware registers (PC, PSW, ...). When a context switch happens these are stored in a part of the *u_area* called PCB (Process Control Block)

Some ot these entities, although conceptually different share implementation: for example, the *kernel stack* of a process is usually implemented as a part of the *u_area*, and the credentials go in the *proc* structure.

In linux, instead of the *u_area* and the *proc* structure, there exists the *task_struct* structure.

### *proc* structure

The *proc* structure contains all the information that the kernel needs at all times. The *proc* structure of a process is always directly accesible, even when the process is not in the CPU.

The kernel keeps an array of *proc* structures called *process table*. It is in the kernel space.

### *u_area*

- It is in user space but it is only accesible when in kernel mode and when the process is in CPU
- Always at the same virtual address
- Contains information only needed when the process is running

### Credentials

Credentials of a process allow the system to determine what privileges a proess has relating to fiels and to other process in the system
- Each user in the system is identified by a number: *user id* or *uid*
- Each group in the system is identified by a number: *group id* or *gid*
- There is a special user in the system: *root (uid=0)*
    - Can access all files
    - Can send signals to every process
    - Can make privileged system calls

Access to a file is conditioned by:
- Owner: (file *uid*)
- Group: (file *gid*)
- Permissions: (file *mode*)

A process has its **credentials**, which specify what files it can acces (and how) and what processes it can send signals to (and from what processes it can get signals sent)
- User credential (process *uid*)
- Group credential (process *gid*)
- When a process tries to acess a file, the following procedure applies
    - If the process uid matches the file uid: owner permissions apply
    - If the process gid matches the file gid: group permissions apply
    - Otherwise *rest of the world permissions apply


