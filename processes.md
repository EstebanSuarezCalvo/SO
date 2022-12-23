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

A process has actually three pairs of credentials:
- **Effective:** rule access to files
- **Real and effective:** rule sending and receiving signals (a signal is received if the real or effective *uid* of the sending process matches the real *uid* of the receiving process)
- **Real and saved:** rule what changes to the effective credential can be done via *setuid* and *setgid* system calls

#### Change of credentials

There are only **three** system calls able to change a process credentials:
- `setuid()` changes the *uid* of the calling process
    - The only changes allowed are *effective := real* and *effective := saved*
    - If the process has the efective credential of *root*, `setuid()` changes the three credentials
- `setgid()` changes the *gid* of the calling process
- `exec()` the *exec* system calls (*execl, execv, execlp, execve...*) can change the credentials of the calling process if the file to be executed has the adecuate permissions

### Environment variables

An environment variable is a variable whose value is set outside the program.

These variables are strings of characters placed at the bottom of the user stack.

There are several ways to acces them:
- Third argument to `main()`: NULL terminated array of the environment variables
- `extern char ** environ`: NULL terminated array of the environment variables
- Library functions: `putenv()`, `getenv()`, `setenv()`, `unsetenv()`

## Process life cycle

Every process in the system starts its life with a create process system call. The process that makes that system call is called parent process of the created process.

During its life cycle, a process goes through different states: running, ready to run, blocked...

Every process in the system ends with the terminate process system call.

### Swapping systems

On older systems, some parts of the secondary memory was used to swap out processes from the primary memory so that the degree of multiprogramming could be increased.

### Paging systems

On modern systems, some part of the secondary memory is used to swap out PIECES of processes from the primary memory so that the degree of multiprogramming can be increased. This also allows for a process that is not loaded completely into memory to be executed.

### Process states

- CPU / running / executing
- ready / ready to run / runnable
- blocked / asleep / waiting
- swapped out / suspended (a proccess in this state can either be runnable or blocked)

### State transitions
- Entering the running state: the first in the ready to run queue is scheduled to run
- Entering the ready to run state:
    - **A new process has been created and it enters the runnable queue**
    - **From CPU:** another process is scheduled to run via a context switch. We say the process has been preempted
    - **From blocked:** the event the process was waiting for (some i/o operation or whatever) has ocurred. We call this transition *unblock* or *wake up*.
    - **From ready to run:** the O.S. decides to bring it to primary memory. This transition is called swap in.

- Entering the blocked state:
    - **From CPU:** the process makes some system call (for example asks for some i/o to be done) that cannot be complete at the time so it blocks
    - **From blocked:** the O.S. swaps in a blocked process (not every O.S. accepts this transition)

- The *blocked* and *ready* states can be entered when the O.S. decides to swap out a process (ready or blocked) to free some primary memory

### Process creation

When a process makes a create process system call, the O.S. must:
1. Assign an Identifier to the new process
2. Create and initialize its PCB (Process Control Block)
3. Update the SCB (System Control Block) to include the new process
4. Assign memory to it and, if needed, load the program the new process is going to execute
5. Put it into the ready to run queue

The only way to create a new process is through a system call.

### Termination of a process

When a process is terminated, its PCB is deleted and the O.S. reclaims all the resources assigned to that process.

If the process has some children processes: it may wait for them to end, terminate or leave them be.

There are 2 ways of termination:
- **Normal termination:** the process calls voluntarily the terminate process system call
- **Abnormal termination:** not provided for in the process code. The process is *forced* to make the *terminate process system call*

## Process life cycle in UNIX

A process has a specific life span
- It is created by the `fork()` (or `vfork()`) system call
- Ends with the `exit()` system call
- Can execute a program with one of the `exec()` system calls

Every process has a parent process.

A process can have one (or more) child procecesses.

Tree like structure with the *init* the common ancestor to (almost) all processes in the system.

When a process ends its children processes are inherited by *init*.

### The states of a process

The process states in System V are:
- **idle:** the process is bein created but it is not yet ready to run
- **Runnable / ready to run**
- **Blocked / asleep:** in this state, as with the *runnable* state, the process can be in main memory or in the swap area (*swapped*)
- **User running**
- **Kernel running**
- **Zombie:** the process has terminated but the parent process has not yet performed one of the wait system calls on it: its *proc structure* has not been emptied so, for the system, the process still exists

Notice:
- The execution of a process starts in kernel mode.
- Transition to *blocked* is from kernel mode running.
- Transitions to and from runnable are from kernel mode running
- Execution ends in kernel mode
- When a process ends it goes into *zombie* state until its parent process performs one of the *wait* system calls on it

## `fork()`

Creates a process. The created process is a "klon" of the parent process, its address space is a replica of the parent process' address space. The only difference is the value returned by `fork()`: 0 to the child process and the child's process and the child's pid to the parent process.

### Tasks performed by `fork()`

1. Allocate swap space
2. Assign *pid* and allocate *proc* strucutre
3. Initialize *proc* structure
4. Assign addres translation maps for child processes
5. Allocate child process's *u_area* and copy data from parent process
6. Update fields in *u_area*
7. Add child process to set of processes sharing code
8. Duplicate data and stack segments from parent process and update tables
9. Initialize hardware context
10. Change state of child process to *ready to run*
11. return 0 to child process
12. return child's *pid* to parent process

#### Optimizing `fork()`
What we need to know in order to optimize `fork()`
- Among the tasks performed by `fork()`, duplicate data and stack segments from parent process and update tables implies:
    - Allocating memory for child's process data and stack
    - Copy parent process's data and stack
- It often happens that a process just created by `fork()` executes another program
    ~~~
    if ((pid = fork()) == 0) {
        if (execv("./programm", args) == -1) {
            perror("Error: in exec");
            exit(1);
        }
    }
    ~~~
- The exec() calls discard current address space and allocate a new one
- In this case, we have allocated memory, copied data on it and then ended up discarding all that memory

There are two optimizations:
- **Copy on write**
    - Data and stack are not copied: they are shared between parent and child processes
    - Data and stack are marked read only
    - When an attempt is made to modify any of them, as they are marked read only, an exception is produced
    - The exception handler copies ONLY THE PAGE that is being modified. Only modified pages of data and stack are copied
- **`vfork()` system call**
    - Used only if a call to `exec()` is to be made in a short time
    - Child process *borrows* parent process' space address until a call to `exec()` or `exit()` is made. At this moment the parent process is awaken and returned its address space
    - Nothing gets copied

## `exec()` 

(`execl()`, `execv()`, `execle()`, `execve()`, `execlp()`, `execvp()`) Makes an already created process execute a program: it **replaces the calling process address space** (code, data, stack...) with that of the program to be executed. **It does not create a new process**.

- An already created process executes a program; its address space is replaced by that one of the program to be executed
    - If the program was created by `vfork()`, `exec()` returns the address space to the parent process
    - If the program was created by `fork()`, `exec()` releases th address space
- A new address space is created and loaded with the new programs
- When `exec()` ends, execution starts and the new program's first instruction
- If the program to be executed has the adequate *mode*, `exec()`changes the efective and saved user and/or group credentials of the process calling `exec()`to the ones of the executable file

### Tasks performed by `exec()`

1. Get executable file from path
2. Check for execute access
3. Inspect file header and check if it is a valid executable
4. If the bits *setuid* and/or *setgid* are set, change the efective (and saved) *uid* or *gid* of the process
5. Save environment and arguments to `exec()` in kernel space (user space is being discarded)
6. Allocate new swap space (data and stack)
7. Release address space (if the process was created by `vfork()` return it to the parent process)
8. Allocate a new address space. If the code is already in use, share it, if not, load it from the executable file
9. Copy environment variables and arguments to `exec()` into new user stack
10. Restore signal handlers to de default action
11. Initialize hardware context, all registers to 0, except Program Counter, to entry point of program

## `exit()`

Ends a process.

### Tasks performed by `exit()`

1. Deactivate all signals
2. Close all process's open files
3. Free from Vnode Table vnodes of the code file, control terminal, root directory and current working directory
4. Save resource usage statistics and *exit state* into *proc* structure
5. Change process state to SZOMB and place *proc* structure into *zombie* list
6. Make *init* inherit all process's children processes
7. Deallocate address space, *u_area*, and swap space...
8. Send SIGCHLD to parent process
9. If parent is waiting for child process then awake parent process
10. Call *switch* to initiate context switch

## Waiting for a child to end

If a process needs to know how a child process has terminated, it can use one of the `wait()` system calls.

-`wait()` checks whether a child process has ended
    - If it has, `wait()` returns inmediately
    - If it has not, the process calling `wait()` waits until any of its children processes has ended

- The exit value that the child process passed to `exit()` is transfered to the variable `wait()`uses as a parameter.

- Deallocates child processes's *proc* structure.

- Returns *pid* of the child processes's *proc* structure

- `waitpid()`, `waitid()`, `wait3()` and `wait4()` do atmit options:
    - **WNOHANG** Does not wait for the child process to end
    - **WUNTRACED** In addition to reporting ending processes, stopping of a child process is also reported
    - **WCONTINUED** In addition to reporting ending processes, continuing of a stopped child process is also reported
    - **WNOWAIT** Does not deallocate child process's *proc* structure

- *proc* structure is not deallocated until one of the `wait()` system calls is used

- Creation of *zombie* processes can be avoided using flag SA_NOCLDWAIT in *sigaction* for the SIGCHLD signal

- Should that be the case `wait()`would return -1 setting errno to ECHILD

## CPU Scheduling

In a multiprogrammed O.S. several processes and/or threads compete for CPU.

The scheduler is the part of the O.S. which decides which process (among the runnable processes) obtains the CPU.

There are two kinds of scheduling algorithms:
- **non-preemptive algorithms**: The currently running process stays in the CPU until it ends its CPU burst
- **preemptive algorithms**: The scheduler can move out from the CPU the currently running process before it ends its CPU burst (preemption)

Types of scheduler:
- **short term scheduler**: decides which process enters the CPU among the runnable processes
- **medium term scheduler**: in *swapping* systems: decides which swapped out processes will be swapped in
- **long term scheduler**: in *batch systems*: decides which process(es) in the *spool* device will be loaded into main memory. It controls the degree of multiprogramming

The goals of a scheduler will vary depending on the environment it is used:
- **Batch environments**: It's main goal is to be efficient and have great throughput
- **Interactive environments**: Its main goal is to give at least some CPU to all processes in a timely manner
- **Real time environments**: Some processes in the system have very specific time constrains that need to be met. Typically priority based scheduling is used and those processes with special needs are assigned the greatest priorities in the system

In every system the sheduler has to provide
- **Fairness**: every process has to get a fair share of the CPU
- **Policy**: meet a certain criteria previously stablished
- **Balance**: different parts of the system share similar workloads

### Scheduling Evaluation

There are 3 methods to evaluate how an algorithm behaves on a given system

- Analytical methods (both deterministic and non deterministic)

    - Deterministic Models
        - We take a sample workload and evaluate how the system behaves. Important: the workload must be representative
        - We use some of the time measurements to assess the algorithm's performance
        - *Pros*: simplicity
        - *Cons*: misleading results if the workload is not correctly selected

    - Non deterministic Models
        - On many systems, the arrival time and length of the job cannot be predicted, so it is not possible to use a deterministic mode
        - We use probability distribution functions to model the CPU bursts and arrival times for the jobs in the system
        - With those two distributions we can estimate the mean values of throughput, watting time...

- Simulation
    - Another option is to simulate the system behaviour
    - Data for processes are either ramdomly generated or sampled from a real system
    - This method gives a real glimpse on how an scheduling algorithm actually performs
    - High computing cost

- Implantation

    - The algorithm is implemented on a running system to be evaluated
    - Data obtained correspondt o actual processes in a real system
    - The mere implantation of some specific algorithm in a running system can condition user behaviour so that the results thus obtained may be not as *authentic* as they should

### Scheduling algorithms

#### Non preemptive algorithms

##### Non preemptive: priorities

*Priority is a numeric value used to decide whether a process gets to use CPU before other processes.*

Depending on how they are asigned, priorities can be:
- **Internal**: Assigned by the O.S. from information on the process
- **External**: Assigned by the users or the System Administration
- **Mixed**: Combination of internal and external

Priorities can also be considered:
- **Static**: the priority of a process does not change (unless the system administrator or some user explicitly changes it)
- **Dynamic**: the system recalculates the processes' priorities

In non preentive priority scheduling, when the process in CPU voluntarily relenquishes CPU, the scheduler selects the highest priority process among all ready to run processes.

The main drawback is that a process with low priority waits forever. This problem is usually solved with dynamic priorities.

##### First-Come-First-Served (FCFS)

Pros:
- Easy to Implement. A FIFO queue is enough
- Fair

Cons:
- Drawback: risk of low throughput; "convoy" effect

##### Shortest Job First (SJF)

- Only theoretical usage, it needs to know beforehand the length of the CPU burts.

- Produces the shortest turnaround times with various processes arriving simoultaneously

- When two CPU bursts are the same lenght FCFS is used

- This algorithm produces the **best possible** results (unless the processes appear at different times)

#### Preemptive algorithms

##### Preemptive: priorities

- When a process with higher priority than the one in CPU becomes ready, it takes the CPU from the one using it, which goes into the *preempted state (ready to run)*.

##### Shortest Remaining Time First (SRTF)

- It is the preemptive implementation of SJF

- Every time new jobs appear ready, their CPU bursts are compared with the remaining time of the one in CPU

- If one of the new jobs has a CPU burts shorter than the remaining time of the one in CPU, the new job gets the CPU

##### Round-Robin (RR)

- Each process has a time limit in its CPU time called quantum (q)

- Ready to run processes are organiced in a FIFO queue

- If A is executing and reaches the quantum, then a context switch occurs

- The first process in the ready to run queue gets the CPU and A enters the queue

- A timer takes care of waking up the scheduler

Advantages:
- Easy to implement
- Fairness

Drawback:
- Finding the right *q* value

##### Multilevel Queues

A multilevel queue is an evolution from the priority scheduling.

We have one queue for each priority level. Each queue can have its own scheduling algoritm

### Real time scheduling

On real time systems, time is of critical importance, so the scheduler must organize the processes so the time limits are met

- Hard real time: all time limits MUST be met

- Soft real time: missing one hit, although not desirable, is tolerable

A program is usually divided into short and predictable bursts whose duration is known in advance.

On a real system we distiguish between two kinds of events, periodical and not periodical

___

**Schedudable real time system**: we define a real time system with m streams to me shedulable if it satisfies $$\sum_{i=1}^{m} C~i~ / P~i~$$

P~i~ = period at which the event occurs

C~i~ = CPU time needed to process the event

___


### Thread Sheculing

**A threat can be defined as the basic unit of CPU usage.**

Every process has at least one thread. If it has more than one thread it can perform several tasks concurrently.

The threads inside a process share: code segment, data segment, resources...

For each thread: identifier, program counter, registers, stack.

Advantages:

- Higher response capability: if one thread blocks, other threads can continue to execute

- There may be several threads sharing the same resources

- Less expensive than creating processes. Context switch is also lighter

- Can take advantage of multiprocessor architectures

We can think of scheduling at two levels: processes and threads. A process scheduler chooses a process, then a thread scheduler chooses the thread.

There is no preemption among threads. If a thread uses up all thre *quantum* another process is selected. When it returns to CPU the same thread will continue.

If the thread does not use all the *quantum*, the thread scheduler can select another thread inside the same process.

### Multiprocessor Scheduling

Scheduling gets more complicated when we have several processors. Assigning different sized jobs to several processors in an optimal way is a combinatory problem.

**Assymetric Multiprocessing**: One processor is in charge, the others just execute the processes they are assigned.

**Symmetric Multiprocessing**: Each processor has its own scheduling. Sometimes they share the ready to run queue, in this case extra care must be taken that one process doesn't end up in more than one processor.

If thread A has been executing longer in CPU1, its cache will be filled with data from A. We call this **affinity**.

## Unix process scheduling

### Traditional scheduling

Preemptive priorities recalculated dynamically:

- Always is run the process with the highest priority

- Smaller numbers indicate greater priorities

- Priority of a process decreases as the process uses CPU

- Priority of a process increases as it spends time in the ready to run queue

- When a process with higher priority than the one in CPU appears ready, it preempts the one in CPU (which goes into the ready to run state), unless the one in CPU is running in kernel mode, in which case it will be preempted when it returns to user mode

Processes of the same priority share the CPU in *round robin*.

User mode priority is recalculated attending to:

- *nice* factor: controlled by the `nice()` system call

- CPU usage: higher CPU usage (recent) means lower priority

The *proc* structure has the following members related to priority recalculation:

- *p_cpu*: cpu usage for the purpose of priority recalculation

- *p_nice*: *nice*

- *p_usrpri*: user mode priority, recalculated periodically from CPU usage and nice factor *nice*

- *p_pri*: process priority, this is the one used for scheduling

When the process runs in user mode  *p_pri* is identical to *p_usrpri*.

After a process was blocked, when it is awaken, *p_pri* is assigned a value depending on the reason the process was blocked. THis is called a **kernel priority** or **sleep priority**:

- This *kernel priorities* are smaller numbers thus higher priority than user mode priorities *p_usrpri*

- The goal is to make processes complete the system calls faster

#### Recalculation of user mode priorities

Every clock *tic*, the *handler* increments the *p_cpu* of the currently running process

Every second, the *p_cpu* is adjusted and the user *mode priorities* are recalculated so that the user mode priorities are lower than the kernel ones.

#### Traditional unix scheduling: Implementation

- Is implemented as an array of multilevel queues (normally 32 queues)

- After calculating its priority, a process is moved to the apropiate queue

- `swtch()` just loads the first process of the first non empty queue

- Each 100ms the `roundrobin()` routine changes to the next process in the same queue

### System V R4 scheduling

- It includes real time application

- It separates scheduling policy from implementation mechanisms

- New scheduling policies can be implemented

- It limits applications latency

- Priorities can be "inherited" to avoid *priority inversion*

Some *Scheduling classes* are defined and they determine the policies applied to the processes belonging to them.

#### System V R4 scheduling: main characteristics

- Priorities range from 0 to 159: the higher number the higher the priority:

    - 0-59: *time sharing class*

    - 60:69: *system priority*

    - 100-159: *real time*

- In the *proc* structure:

    - *p_cid* class identifier

    - *_clfuncs* pointer to class functions

    - *p_clproc* pointer to class independent data

- It is implemented as an array of multilevel queues.

#### System V R4 scheduling: predifined classes

##### *Time sharing* class

- User mode priorities are recalculated dynamically

- When a process blocks it is assigned a *sleep priority* depending on the reason it blocked. When it returns, its user mode priority is used

- Quantum depends on priority: higher priority -> shorter quantum

- ONLY the process leaving the CPU gets its priority recalculated

    - **used up all of its quantum**: its user mode priority gets lower

    - **blocked before using all of its quantum**: its priority raises

##### *Real time* class

- Priorities can only be changedd with the `priocntl()` system call

- Fixed priorities. Kernel maintains a table with the corresponding time quantums for each priority

- The higher the priority, the shorter the default quantum

- After using its quantum, the process returns to the same queue (at the end)

##### *System* class

- Used for special system processes

- Fixed priorities

### Linux scheduling

Linux distinguishes between 2 types of processes:

- **Real time processes**: Fixed static priority between 1 and 99. Its priority doesn't change and the higher priority runnable process gets the CPU

- **Normal processes**: correspond to a static priority of 0. They are executed if no real time process is ready to run. For them a preemptive dynamic priority algorithm is used. The system recalculates their priorities and quantums according to the values specified by *nice* and/or *setpriority*

CPU scheduling is done for time intervals called *epochs*. For each *epoch* every processes has its time *slice* depending on its priority.

The system has a *runqueue* for each processor and each process can only be in one *runqueue* at a time.

Each *runqueue* has 2 structures: the *active array* and the *expired array*. Each array has a process queue for each priority level

When a process uses up all of its *slice*, its *slice* gets recalculated and the process is moved to the *expired array*. When all processes have used up all their *slices* the *expired array* becomes *active array*

An array of bits indicates the non empty queues

### System calls for priority management

#### `nice()`

Changes the *niceness* of the calling process. For traditional unix systems that is the *p_nice* factor.

It takes the nice increment as its argument.

It returns the *niceness* minus 20. *niceness* is a number between 0 and 40.

#### `setpriority()` and `getpriority()`

They change the same scheduling parameters as the `nice()` system call.

Better interface to priority than `nice()`, as a process, with the right credentials, can check and/or modify other processes' priorities

## Unix Processes: Executing in kernel mode

