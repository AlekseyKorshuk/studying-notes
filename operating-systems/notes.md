# Short

# What is OS?

An OS is a layer of software, that provides user programs with simpler and cleaner model of the computer and handles managing all the resources.

# Kernel and user modes

In kernel mode the code has complete access to the hardware. In User mode the code cannot directly access hardware, and has only a subset of machine instructions. As crashes in kernel mode are catastrophic, separation and giving user mode for most programs protects computer.

# Process

Process — abstraction for program execution, which encapsulates its memory, program counter, stack pointer and other service information.

### Process have lots of **states**

![Untitled](Short%2018a59/Untitled.png)

1. Created
2. Ready for execution, swapped
3. Ready for execution, in memory
4. Execution in Kernel Mode
5. Executing in User Mode
6. Preempted when tried to enter Kernel
7. Blocked, in memory
8. Blocked, swapped
9. Zombie (process exited or dead)

### How to create process

Use `fork()` in all POSIX

Use `clone()` in Linux

### Implementation of processes | Model

**Process table** — table that OS maintains with to implement the process model.

P**rocess control blocks —** one entry that a process has.

P**rocess control blocks structure:**

- program counter
- stack pointer
- memory allocation
- the status of its open files
- its accounting
- scheduling information

# Threads

**Thread —** part of a program, which is associated with process. It's like lightweight process. Threads have their own program counter, registers and stack. Other parts (text memory, data memory, etc.) are shared.

Every process have threads. At least one, main thread.

### Thread usage

Threads are used to achieve simultaneous execution of program parts (for example, process user requests to server). If there are multiple cores, threads may execute really simultaneously.

- Multiple activities are going on at once
- Threads are lighter weight than processes, they are easier (i.e., faster) to create and destroy than processes
- Performance: threads yield no performance gain when all of them are CPU bound
- Are useful on systems with multiple CPUs, where real parallelism is possible

### The classical thread model

The process model is based on two independent concepts: resource grouping and execution.

`TODO`

### States

- Running
- Blocked
- Ready
- Terminated

# Interprocess communication (IPC)

## Race conditions

**Race condition** — where two or more processes are reading or writing some shared data and the final result depends on the running queue.

## Avoiding race condition

- Critical Regions
- Mutual Exclusion with Busy Waiting
    - Disabling Interrupts
    - Lock Variables
    - Strict Alternation
    - Peterson's Solution
    - The TSL instruction
- Sleep and Wakeup
    - The Producer-Consumer Problem
- Semaphores
    - Solving the Producer-Consumer Problem Using Semaphores
- Mutexes
    - Futexes
- Monitors
- Message Passing
    - The Producer-Consumer Problem with Message Passing
- Barriers
- Avoiding Locks: Read-Copy-Update

## Critical region

**Critical region** — part of the program where the shared memory is accessed.

- No two processes may be simultaneously inside their critical regions.
- No two processes may be simultaneously inside their critical regions.
- No process running outside its critical region may block any process.
- No process should have to wait forever to enter its critical region.

## Mutual Exclusion with Busy Waiting

**Mutual exclusion** — prohibit more than one process from reading and writing the shared data at the same time.

### **Disabling Interrupts**

On a single-processor system, the simplest solution is to have each process disable all interrupts just after entering its critical region and re-enable them just before leaving it. Once a process has disabled interrupts, it can examine and update the shared memory without fear that any other process will intervene.

### Lock Variables

Having a single **shared (lock) variable,** initially 0. When a process wants to enter its critical region, it first tests the lock. If the lock is already 1, the process just waits until it becomes 0.

### Strict Alternation

Have variable `turn`. And 2 processes `A` and `B`. 

If `turn == 0` process `A` can access critical region, else busy wait.

If `turn == 1` process `B` can access critical region, else busy wait.

**Spin lock** — a lock that uses busy waiting.

### Peterson's Solution

![Untitled](Short%2018a59/Untitled%201.png)

Before entering its critical region, each process calls `enter_region` with its own process number, 0 or 1, as a parameter. This call will cause it to wait if need be until it is safe to enter.

After it has finished with the shared variables, the process calls `leave_region` to indicate that it is done and to allow the other process to enter, if it so desires.

### The TSL instruction

Some computers, especially those designed with multiple processors in mind, have an instruction like `TSL RX,LOCK` (Test and Set Lock).

It reads the contents of the memory word lock into register `RX` and then stores a nonzero value at the memory address lock. The operations of reading the word and storing into it are guaranteed to be indivisible — no other processor can access the memory word until the instruction is finished. The CPU executing the `TSL` instruction locks the memory bus to prohibit other CPUs from accessing memory until it is done.

To use the TSL instruction, we will use a shared variable, `lock`, to coordinate access to shared memory. When `lock` is 0, any process may set it to 1 using the `TSL` instruction and then read or write the shared memory. When it is done, the process sets `lock` back to 0 using an ordinary `move` instruction.

![Untitled](Short%2018a59/Untitled%202.png)

## Sleep and Wakeup

Solves the problem of having **busy waiting**.

One of the simplest inter-process communication primitives, that **blocks instead of wasting CPU time when they are not allowed to enter their critical regions** is the pair `sleep` and `wakeup`.

### The Producer-Consumer Problem

Two processes share a common, fixed-size buffer. One of them, the **producer**, puts information into the buffer, and the other one, the **consumer**, takes it out.

Problem: when the producer wants to put a new item in the buffer, but it is already full.

Solution with `sleep` and `wakeup`:

![Untitled](Short%2018a59/Untitled%203.png)

## Semaphores

**Semaphore** — a new variable type, an integer variable to count the number of wakeups saved for future use.

Two operations on semaphores:

- `down` — checks to see if the value is greater than 0. If so, it decrements the value and just continues. If the value is 0, the process is put to sleep without completing the `down` for the moment.
- `up` — increments the value of the semaphore addressed. If one or more processes were sleeping on that semaphore, unable to complete an earlier `down` operation, one of them is chosen by the system at random and is allowed to complete its `down`.

### Solving the Producer-Consumer Problem Using Semaphores

![Untitled](Short%2018a59/Untitled%204.png)

This solution uses three semaphores: one called `full` for counting the number of slots that are full, one called `empty` for counting the number of slots that are empty, and one called `mutex` to make sure the producer and consumer do not access the buffer at the same time. `Full` is initially 0, `empty` is initially equal to the number of slots in the buffer, and `mutex` is initially 1. 

## Mutexes

**Mutex —** binary version of the semaphore: unlocked or locked.

### Futexes

Solves the busy waiting for **mutex**.

**Futex** — fast user space mutex, a feature of Linux that implements basic locking (much like a mutex) but avoids dropping into the kernel unless it really has to.

A futex consists of two parts: a kernel service and a user library.

## Monitors

**Monitor** — a collection of procedures, variables, and data structures that are all grouped together in a special kind of module or package. Processes may call the procedures in a monitor whenever they want to, but they cannot directly access the monitor’s internal data structures from procedures declared outside the monitor.

![Untitled](Short%2018a59/Untitled%205.png)

Monitors have an important property that makes them useful for achieving mutual exclusion: only one process can be active in a monitor at any instant. 

## Message passing

This method of inter-process communication uses two primitives, `send(destination, &message)` and `receive(source, &message)`.

### Issue

We can not be sure that the message is reached the receiver.

### The Producer-Consumer Problem with Message Passing

![Untitled](Short%2018a59/Untitled%206.png)

## Barriers

**Barrier** — the rule that no process may proceed into the next phase until all processes are ready to proceed to the next phase.

![Untitled](Short%2018a59/Untitled%207.png)

## Avoiding Locks: Read-Copy-Update

![Untitled](Short%2018a59/Untitled%208.png)

First, we make A’s left child pointer point to C. All readers that were in A will continue with node C and never see B or D. In other words, they will see only the new version. Likewise, all readers currently in B or D will continue following the original data structure pointers and see the old version. All is well, and we never need to lock anything. The main reason that the removal of B and D works without locking the data structure, is that **RCU** (Read-Copy-Update), decouples the `removal` and `reclamation` phases of the update.

# Scheduling

S**cheduler** — the part of the operating system that makes the choice

S**cheduling** **algorithm —** the algorithm **scheduler** uses.

## When to Schedule

1. When a new process is created, we need to decide to run the parent process or the child process. 
2. A scheduling decision must be made when a process exits.
3. When a process blocks on I/O, on a semaphore, or for some other reason, another process has to be selected to run.
4. When an I/O interrupt occurs, a scheduling decision may be made.

## N**on-preemptive**

A **non-preemptive** scheduling algorithm picks a process to run and then just lets it run until it blocks (either on I/O or waiting for another process) or voluntarily releases the CPU. In effect, no scheduling decisions are made during clock interrupts.

## P**reemptive**

A **preemptive** scheduling algorithm picks a process and lets it run
for a maximum of some fixed time. If it is still running at the end of the time interval, it is suspended and the scheduler picks another process to run (if one is available). Doing preemptive scheduling requires having a clock interrupt occur at the end of the time interval to give control of the CPU back to the scheduler.

## Categories of Scheduling Algorithms

- Batch
    
    In batch systems, there are no users impatiently waiting at their terminals for a quick response to a short request. Consequently, non-preemptive algorithms or preemptive algorithms with long time periods for each process are often acceptable. This approach reduces process switches and thus improves performance. 
    
- First-Come, First-Served
- Shortest Job First
- Shortest Remaining Time Next
- Interactive
    
    In an environment with interactive users, preemption is essential to keep one process from hogging the CPU and denying service to the others. Preemption is needed to prevent bugs, where one process might shut out all the others indefinitely. Servers also fall into this category, since they normally serve multiple (remote) users. Computer users are always in a big hurry.
    
- Round-Robin Scheduling
- Priority Scheduling
- Multiple Queues
- Shortest Process Next
- Guaranteed Scheduling
- Lottery Scheduling
- Fair-Share Scheduling
- Real time
    
    In systems with real-time constraints, preemption is sometimes not needed because the processes know that they may not run for long periods of time and usually do their work and block quickly. 
    
    The difference with interactive systems is that real-time systems run only programs that are intended to communicate further to solve the task. Interactive systems are general purpose and may run arbitrary programs that don't cooperate or even possibly damage each other.
    
- Scheduling policy Vs Scheduling mechanism
- Thread Scheduling

## Scheduling Algorithm Goals

![Untitled](Short%2018a59/Untitled%209.png)

## Scheduling in Batch Systems

### First-Come, First-Served

With this algorithm, processes are assigned the CPU in the order they request it. Basically, there is a single queue of ready processes.

### Shortest Job First

It is another non-preemptive batch algorithm that assumes the run times are known in advance. 

### Shortest Remaining Time Next

A preemptive version of shortest job first is **shortest remaining time next**. 

With this algorithm, the scheduler always chooses the process whose remaining run time is the shortest.

## Scheduling in Interactive Systems

### Round-Robin Scheduling

Each process is assigned a time interval, called its **quantum**, during which it is allowed to run. Processes just switches one by one and runs for a quantum time.

### Priority Scheduling

Each process has a priority (integer value). And processes with highest priority are in the front of the queue.

### Multiple Queues

Their solution was to set up priority classes. Processes in the highest class were run for one quantum. Processes in the next-highest class were run for two quanta. Processes in the next one were run for four quanta, etc. Whenever a process used up all the quanta allocated to it, it was moved down one class.

![Untitled](Short%2018a59/Untitled%2010.png)

### Shortest Process Next

Same as **shortest job first** in batch, but now for ****interactive processes as
well.

Suppose that the estimated time per command for some process is T0. Its next run is measured to be T1. We could update our estimate by taking a weighted sum of these two numbers:  $aT_0 + (1 − a)T_1$. Through the choice of $a$ we can decide to forget old runs quickly, or remember them for a long time. With $a = 1/2$, we get successive estimates of $T_0$:
$T_0\ \rightarrow\ \frac{T_0}{2} + \frac{T_1}{2}\ \rightarrow\ \frac{T_0}{4} + \frac{T_1}{4} + \frac{T_2}{2}\ \rightarrow\ \frac{T_0}{8} + \frac{T_1}{8} + \frac{T_2}{4} + \frac{T_3}{2}$ 
After three new runs, the weight of $T_0$ in the new estimate has dropped to 1/8.

### Guaranteed Scheduling

Make real promises to the users about performance and then live up to those promises.

On a single-user system with n processes running, all things being equal, each one should get $\frac{1}{n}$ of the CPU cycles. 

To make good on this promise, the system must keep track of how much CPU
each process has had since its creation.

A ratio of 0.5 means that a process has only had half of what it should have had, and a ratio of 2.0 means that a process has had twice as much as it was entitled to.  

### **Lottery Scheduling**

The basic idea is to give processes lottery tickets for various system resources,
such as CPU time.

For example, when a client process sends a message to a server process and then blocks, it may give all of its tickets to the server, to increase the chance of the server running next. When the server is finished, it returns the tickets so that the client can run again. In fact, in the absence of clients, servers need no tickets at all.

### Fair-Share Scheduling

To prevent unfair situations, some systems take into account which user owns a process before scheduling it. In this model, each user is allocated some fraction of the CPU and the scheduler picks processes in such a way as to enforce them.

## Scheduling in Real-Time Systems

Real-time systems are generally categorized as **hard real time**, meaning there
are absolute deadlines, and **soft real time**, meaning that missing an occasional deadline is undesirable, but tolerable. 

A real-time system is **schedulable**, if it can actually be implemented.

### Scheduling policy Vs Scheduling mechanism

Up until now, we assumed that all the processes in the system belong to different users and are thus competing for the CPU. However, sometimes it happens that one process has many children running under its control. For example, a database-management-system process may have many children and know which of its children are the most important (or time-critical) and which the least.

The solution to this problem is to separate the **scheduling mechanism** from
the **scheduling policy.** This means, that the scheduling algorithm is parameterized in some way.

### Thread Scheduling

When several processes each have multiple threads, we have two levels of parallelism present: processes and threads. Scheduling in such systems differs substantially depending on whether user-level threads or kernel-level threads (or both) are supported.

# No memory abstraction

The simplest memory abstraction is to have no abstraction at all.

![Untitled](Short%2018a59/Untitled%2011.png)

The operating system may be:

1. At the bottom of memory in RAM — used on mainframes and minicomputers
2. At the top of memory in ROM — used on some handheld computers and embedded systems
3. The device drivers may be at the top of memory in a ROM
and the rest of the system in RAM down below — used by early personal computers

## Running Multiple Programs Without a Memory Abstraction

Even with no memory abstraction, it is possible to run multiple programs at the same time. What the operating system has to do is save the entire contents of memory to a disk file, then bring in and run the next program. As long as there is only one program at a time in memory, there are no conflicts. This concept is called **swapping**.

With the addition of some special hardware, it is possible to run multiple programs concurrently, even without swapping.

# A memory abstraction: address spaces

To allow multiple applications to be in memory at the same time without interfering with each other we need two solve problems with protection and relocation.

The solution for this is to invent a new abstraction for memory: the **address space**.

An **address space** is the set of addresses that a process can use to address the memory. Each process has its own address space, independent of those belonging to other processes (except in some special circumstances where processes want to share their address spaces).

### Base and Limit Registers

This solution uses a particularly simple version of **dynamic relocation**.
It maps each process’ address space onto a different part of physical memory in a simple way. The classical solution is to equip each CPU with two special hardware registers, usually called the **base** and **limit** registers.

### Swapping

Two general approaches to dealing with memory overload have been developed over the years. The simplest strategy, called **swapping**, consists of bringing in each process in its entirety, running it for a while, then putting it back on the disk.

The other strategy, called **virtual memory**, allows programs to run even when they are only partially in main memory. 

# Managing Free Memory

When memory is assigned dynamically, the operating system must manage it.
Ways to keep track of memory usage:

- bitmaps
- free lists

## Memory Management with Bitmaps

With a bitmap, memory is divided into allocation units with size from a few words to several kilobytes. Corresponding to each allocation unit is a bit in the bitmap, which is 0 if the unit is free and 1 if it is occupied (or vice versa). 

![Untitled](Short%2018a59/Untitled%2012.png)

## Memory Management with Linked Lists

Another way of keeping track of memory is to maintain a linked list of allocated and free memory segments, where a segment either contains a process or is
an empty hole between two processes.

![Untitled](Short%2018a59/Untitled%2013.png)

### Algorithms to allocate memory for a created process

1. **First fit**
    
    The memory manager scans along the list of segments until it finds a hole that is big enough. The hole is then broken up into two pieces, one for the process and one for the unused memory, except in the statistically unlikely case of an exact fit. First fit is a fast algorithm because it searches as little as possible.
    
2. **Next fit**
    
    It works the same way as first fit, except that it keeps track of where it is whenever it finds a suitable hole. The next time it is called to find a hole, it starts searching the list from the place where it left off last time, instead of always at the beginning, as first fit does.
    
3. **Best fit**
    
    Best fit searches the entire list, from beginning to end, and takes the smallest hole that is adequate.
    
4. **Quick fit**
    
    Maintains separate lists for some of the more common sizes requested.
    
    With quick fit, finding a hole of the required size is extremely fast, but it has the same disadvantage as all schemes that sort by hole size: when a process terminates or is swapped out, finding its neighbors to see if a merge with them is possible is quite expensive. If merging is not done, memory will quickly fragment into a large number of small holes into which no processes fit.
    

# Virtual memory

**Virtual memory** — the method to turn the job of splitting pages to the computer.

The idea is, that each program has its own address space, which is broken up into chunks called pages. Each page is a contiguous range of addresses. These pages are mapped onto physical memory, but not all pages have to be in physical memory at the same time.

## Paging

The virtual address space consists of fixed-size units called **pages**. The corresponding units in the physical memory are called **page frames.** The pages and page frames are generally the same size.

![Untitled](Short%2018a59/Untitled%2014.png)

In the actual hardware, a **Present/absent bit** keeps track of which pages are physically present in memory. If the program references an unmapped address,  MMU notices that and causes the CPU to trap to the operating system. This trap is called a **page fault**.

The page number is used as an index into the **page table**, yielding the number
of the page frame corresponding to that virtual page.

## Page Tables

In a simple implementation, the mapping of virtual addresses onto physical addresses can be summarized as follows: the virtual address is split into a virtual page number (high-order bits) and an offset (low-order bits).

![Untitled](Short%2018a59/Untitled%2015.png)

### Structure of a Page Table Entry

![Untitled](Short%2018a59/Untitled%2016.png)

## Speeding Up Paging

### Translation Lookaside Buffers

Access to the page table needs at least one additional memory reference and it significantly reduces the performance of the CPU. 

The devised **solution** is to equip computers with a small hardware device for mapping virtual addresses to physical addresses without going through the page table.

The device is called a **TLB (Translation Lookaside Buffer)** or sometimes an **associative memory**.

It is usually inside the MMU and consists of a small number of entries. These fields have a one-to-one correspondence with the fields in the page table, except for the virtual page number, which is not needed in the page table.

Hardware first checks to see if its virtual page number is present in the TLB by comparing it to all the entries simultaneously.

### Software TLB Management

The TLB entries are explicitly loaded by the operating system.

When a TLB miss occurs it just generates a TLB fault and tosses the problem to the operating system. The system must find the page, remove an entry from the TLB, enter the new one, and restart the faulted instruction.

## Page Table for Large Memories

TLBs can be used to speed up virtual-to-physical address translation, but another problem is how to deal with very large virtual address spaces.

### Multilevel Page Tables

![Untitled](Short%2018a59/Untitled%2017.png)

Consider the use of a **multilevel page table.** The secret to the multilevel page table method is to avoid keeping all the page tables in memory all the time. In particular, those that are not needed should not be kept around.

The two-level page table system can be expanded to three, four, or more levels. Additional levels give more flexibility. For instance, Intel’s processor (launched in 1985) was able to address up to 4-GB of memory, using a two-level page table that consisted of a **page directory** whose entries pointed to page tables, which, in turn, pointed to the actual 4-KB page frames. Both the page directory and the page tables each contained 1024 entries, giving a
total of  $2^{10} × 2^{10} × 2^{12} = 2^{32}$ addressable bytes.

### Inverted Page Tables

An alternative to ever-increasing levels in a paging hierarchy is known as **inverted page tables**.

In this design, there is one entry per page frame in real memory, rather than one entry per page of virtual address space. 

Although inverted page tables save lots of space, they have a downside: virtual-to-physical translation becomes much harder. 

![Untitled](Short%2018a59/Untitled%2018.png)

# Page replacement algorithm

When a page fault occurs, the operating system has to choose a page to evict
(remove from memory) to make room for the incoming page.

![Untitled](Short%2018a59/Untitled%2019.png)

All in all, the two best algorithms are **Aging** and **WSClock**. They are based on LRU and the working set, respectively. Both give good paging performance and can be implemented efficiently. 

## **Not Recently Used (NRU)**

When a page fault occurs, the operating system inspects all the pages and
divides them into four categories based on the values of `R` and `M`:
Class 0: not referenced, not modified.
Class 1: not referenced, modified (this case is possible, when `R` bit is cleared).
Class 2: referenced, not modified.
Class 3: referenced, modified.

Algorithm removes a page at random from the lowest-numbered nonempty class.

## The First-In, First-Out (FIFO)

The operating system maintains a list of all pages currently in memory, with the most recent arrival at the tail and the least recent arrival at the head.

## The second-Chance

Inspect the R bit of the oldest page. 

If it is 0, the page is both old and unused, so it is replaced immediately.

If the R bit is 1, the bit is cleared, the page is put onto the end of the list of pages, and its load time is updated as though it had just arrived in memory.

## The Clock

![Untitled](Short%2018a59/Untitled%2020.png)

## The Least Recently Used (LRU)

When a page fault occurs, throw out the page that has been unused for the longest time.

## Aging

First, the counters are each shifted right 1 bit before the R bit is added in. Second, the R bit is added to the leftmost rather than the rightmost bit. This modified algorithm is called **aging**.

![Untitled](Short%2018a59/Untitled%2021.png)

## The Working Set

The basic idea is to find a page that is not in the working set and evict it.

![Untitled](Short%2018a59/Untitled%2022.png)

## The WSClock

![Untitled](Short%2018a59/Untitled%2023.png)

The data structure needed is a circular list of page frames, as in the clock algorithm. Initially, this list is empty. When the first page is loaded, it is added to the list. As more pages are added, they go into the list to form a ring. Each entry contains the `Time of last use` field from the basic working
set algorithm, as well as the `R` bit (shown) and the `M` bit (not shown).
At each page fault the page pointed to by the hand is examined first. If the `R == 1`, the page has been used during the current tick so it is not an ideal candidate to remove. The `R` bit is then set to 0, the hand advanced to the next page, and the algorithm repeated for that page.
If the page pointed to has `R == 0` and `age > τ` and the page is clean, it is not in the working set and a valid copy exists on the disk.

# Segmentation

**Segmentation** is a technique of memory management that instead of using **page table**, divides whole memory used by a process into **segments** (separate address spaces for different usages). Below picture will show how's done.

![Untitled](Short%2018a59/Untitled%2024.png)

Every part of a process - instructions, data, stacks, etc. has separate **memory segment** that grows separately from the others. As **segments** are usually pretty large (whatever that means) it is unusual for it to run out of free space.

## Compare Paging & Segmentation

![Untitled](Short%2018a59/Untitled%2025.png)

# Files

## File Structure

Files can be structured in any of several ways:

- Byte sequence
- Record sequence
- Tree

![Untitled](Short%2018a59/Untitled%2026.png)

## File Types

Many operating systems support several types of files.

UNIX and Windows, for example, have **regular files** and **directories**. UNIX also has **character** and **block special files**.

**Regular files** are the ones that contain user information. 

**Directories** are system files for maintaining the structure of the file system.

**Character special files** are related to input/output and are used to model serial I/O devices, such as terminals, printers, and networks.

**Block special files** are used to model disks. 

![Untitled](Short%2018a59/Untitled%2027.png)

## File Attributes

![Untitled](Short%2018a59/Untitled%2028.png)

## File Operations

1. **Create**. The file is created with no data. The purpose of the call is to announce that the file is coming and to set some of the attributes.
2. **Delete**. When the file is no longer needed, it has to be deleted to free up disk space. 
3. **Open**. Before using a file, a process must open it. The purpose of the open call is to allow the system to fetch the attributes and list of disk addresses into main memory for rapid access on later calls.
4. **Close**. When all the accesses are finished, the attributes and disk addresses are no longer needed, so the file should be closed to free up internal table space. 
5. **Read**. Data are read from file. Usually, the bytes come from the current position. The caller must specify how many data are needed and must also provide a buffer to put them in.
6. **Write**. Data are written to the file again, usually at the current position. If the current position is the end of the file, the file’s size increases. If the current position is in the middle of the file, existing data are overwritten and lost forever.
7. **Append**. This call is a restricted form of write. It can add data only to the end of the file. Systems that provide a minimal set of system calls rarely have append, but many systems provide multiple ways of doing the same thing, and these systems sometimes have append.
8. **Seek**. For random-access files, a method is needed to specify from where to take the data. One common approach is a system call, seek, that repositions the file pointer to a specific place in the file.
9. **Get attributes**. Processes often need to read file attributes to do their work. 
10. **Set attributes**. Some of the attributes are user-settable and can be changed after the file has been created. 
11. **Rename**

# Directories

## Single-Level Directory Systems

The simplest form of a directory system is having one directory containing all the files.

![Untitled](Short%2018a59/Untitled%2029.png)

## Hierarchical Directory Systems

With this approach, there can be as many directories as are needed to group the files in natural ways. Furthermore, if multiple users share a common file server, as is the case on many company networks, each user can have a private root directory for his or her own hierarchy.

![Untitled](Short%2018a59/Untitled%2030.png)

## Path Names

- **Absolute path name**
- **Relative path name (working directory)**

## Directory Operations

1. **Create**. A directory is created. It is empty except for `dot` and `dotdot`, which are put there automatically by the system 
2. **Delete**. A directory is deleted. Only an empty directory can be deleted. 
3. **Opendir**. Directories can be read. For example, to list all the files in a directory, a listing program opens the directory to read out the names of all the files it contains. Before a directory can be read, it must be opened, analogous to opening and reading a file.
4. **Closedir**. When a directory has been read, it should be closed to free up internal table space.
5. **Readdir**. This call returns the next entry in an open directory. Formerly, it was possible to read directories using the usual read system call, but that approach has the disadvantage of forcing the programmer to know and deal with the internal structure of directories. In contrast, `readdir` always returns one entry in a standard format, no matter which of the possible directory structures is being used.
6. **Rename**. In many respects, directories are just like files and can be renamed the same way files can be.
7. **Link**. Linking is a technique that allows a file to appear in more than one directory. This system call specifies an existing file and a path name, and creates a link from the existing file to the name specified by the path. In this way, the same file may appear in multiple directories. A link of this kind, which increments the counter in the file’s i-node (to keep track of the number of directory entries containing the file), is sometimes called a **hard link**.
8. **Unlink**. A directory entry is removed. If the file being unlinked is only present in one directory (the normal case), it is removed from the file system. If it is present in multiple directories, only the path name specified is removed. The others remain. In UNIX, the system call for deleting files (discussed earlier) is, in fact, unlink.

## **File systems implementation**

### **File-system layout**

![An example structure of the disc](Short%2018a59/Untitled%2031.png)

An example structure of the disc

**Master Boot Record (MBR)** — used to boot the computer.

The end of the MBR contains the partition table which table gives
the starting and ending addresses of each partition

### **Implementing files**

The most important thing when it comes to file is to know where are the boundaries (in terms of **disc blocks**) of the specific file. Here are usual solutions that can be used for that.

**Contiguous allocation** — we allocate files in the contiguous blocks. So finding every file is simple - just find the starting block and add the number of blocks the file occupies in total.

**Linked list allocation** — we use linked list to keep track of the next block containing the rest of the file.

**Linked-List Allocation Using a Table in Memory** — the problems with simple **linked list allocation** can be omitted when we keep the pointers in the memory.

**I-nodes** - instead of keeping everything in a memory, **I-nodes** (from *index-nodes*) are put there only when the file is actually open/used. **I-node** contains blocks addresses that makes the file a whole. Of course there's a possibility that there's a lot of them (due to the file size) - in such case the last block contains reference to the block that contains more information about the pointers.S

![An example i-node](Short%2018a59/Untitled%2032.png)

An example i-node

# **Input/Output**

## **I/O devices**

**I/O devices** can be roughly separated into two categories:

- **block** device
- **character** device

A **block device** is one that stores information in fixed-size blocks, each one with its own address. The essential property of a block device is that it is possible to read or write each block independently of all the other ones. Hard disks, Blu-ray discs, and USB sticks are common block devices.

Second type are **character** devices, which accept a stream of characters. That type include hardware like printers, mouse or network interfaces. Of course this division is just a rough one - there is hardware that does not fit in, but for our purposes it will suffice.

### Controller

I/O units typically consist of:

- A **mechanical** component - the device itself.
- An **electronic** component - the device controller or adapter.

Each controller has control registers, used for communicating with the CPU.

### Memory-Mapped I/O

There are two approaches for mapping the control registers of the I/O devices:

1. Assign I/O port number from the I/O port space which is protected from accessing by user programs.
2. Map all the registers in to the main memory space.
3. Hybrid approach with memory-mapped I/O data buffers and separate I/O ports for the control registers.

![Untitled](Short%2018a59/Untitled%2033.png)

### Direct Memory Access (DMA)

- has access to the system bus independent of the CPU.
- contains registers that can be written and read by the CPU.

How DMA controller works:

- The CPU programs the DMA controller by setting its registers so it knows what to transfer where.
- The DMA controller initiates the transfer by issuing a read request over the bus to the disk controller.
- The disk controller starts the data transfer and does not care whether the request came from the CPU or from the DMA controller.
- The disk controller sends an acknowledgement signal to the DMA controller over the bus, when the write is complete.

![Untitled](Short%2018a59/Untitled%2034.png)

### Interrupts Revisited

**Interrupt** — is a signal emitted by hardware (I/O device) or software when a process or an event needs immediate attention.

How an interrupt occurs:

- When an I/O device has finished the work given to it by the CPU, it causes an interrupt by asserting a signal on a bus line that it has been assigned.
- This signal is detected by the interrupt controller chip, which then decides what to do.
- The interrupt controller handles the interrupt immediately, if no other interrupts are pending.

![Untitled](Short%2018a59/Untitled%2035.png)

**Precise Interrupt** — an interrupt that leaves the machine in a well-defined state, otherwise it is an **imprecise interrupt**.

## Software

### Ways of Performing I/O

- Programmed I/O
- Interrupt-driven I/O
- I/O using DMA

### Programmed I/O

CPU does all the work.

For instance, printing a string on the printer via a serial interface.

![Untitled](Short%2018a59/Untitled%2036.png)

### Interrupt-Driven I/O

![Untitled](Short%2018a59/Untitled%2037.png)

### I/O using DMA

![Untitled](Short%2018a59/Untitled%2038.png)

## I/O Software Layers

![Untitled](Short%2018a59/Untitled%2039.png)

## Interrupt Handlers

Typical steps after hardware interrupt completes:

- Save registers (including the PSW) not already saved by interrupt hardware
- Set up context for interrupt service procedure
- Set up a stack for the interrupt service procedure
- Acknowledge interrupt controller. If no centralized interrupt controller, reenable interrupts
- Copy registers from where saved to process table
- Run interrupt service procedure. Extract information from interrupting device controller’s registers
- Choose which process to run next
- Set up the MMU context for process to run next
- Load new process’ registers, including its PSW
- Start running the new process

# **Deadlocks**

There are two types of resources:

- **preemptable** - is one that can be taken away from the process owning it with no ill effects, an example of printer and memory is given.
- **nonpreemptable** - is one that cannot be taken away from its current owner without potentially causing failure, an example of DVD writer is given. As long as it is in use (eg. writing new DVD with data), there's no way to use it without doing wrong to the current process.

## Introduction to Deadlocks

A set of processes is **deadlocked** if:

- Each process in the set is waiting for an event.
- That event can only be caused by another process in this set.

![Untitled](Short%2018a59/Untitled%2040.png)

## Conditions for Resource Deadlocks

- Mutual exclusion condition.
- Hold and wait condition.
- No preemption condition.
- Circular wait condition.

### Mutual exclusion condition

Each resource is either currently assigned to exactly one process or is available.

### Hold-and-wait condition

Processes currently holding resources that were granted earlier can request new resources.

### No-preemption condition

Resources previously granted cannot be forcibly taken away from a process. They must be explicitly released by the process holding them.

### Circular wait condition

There must be a circular list of two or more processes, each of which is waiting for a resource held by the next member of the chain.

## Dealing with Deadlocks

Strategies:

- Ignore the problem, maybe it will go away
- Detection and recovery
- Dynamic avoidance
- Prevention

### **The ostrich algorithm**

Just ignore the **deadlock** 😉

### **Deadlock detection and recovery**

Detection:

1. Find the deadlock. There is a circle in the graph
2. There are **more than one instance of resource type**.

Recovery:

- R**ecovery through preemption**, which means taking the resource from one of the processes and giving it to the other one.
- R**ecovery through rollback,** process authors must implement in them the functionality of a **checkpoint.**
- R**ecovery through killing processes**

### **Deadlock avoidance**

**Banker's algorithm — t**he algorithm checks for every request whether it fullfillment leads to the **unsafe state**.

**Single resource:**

![Untitled](Short%2018a59/Untitled%2041.png)

Let's go from left to right. In the situation *(a)* we can assume that it is **safe state**, because there is a possible series of steps that will allow all the processes to complete their jobs. From the *(a)* state we see, that with *3 free* resources, we can serve process *B*. Even if it suddenly wants to use *Max* amount of resources, we have *3 free* instances, so *Max - Has <= Free*. That situation is presented in the second picture *(b)*. When *B* finishes, we have the state presented in the *(c)*. Right now it's possible for us to serve *C*, as with *5 free* instances we add them to the already locked *2* and in total use *7* of them (which is defined as *Max* for *C*). The process *C* finishes then, releases its resources and therefore *A* can be served.

**Multiple resources:**

![Untitled](Short%2018a59/Untitled%2042.png)

- The one on the left shows how many of each resource are currently assigned to each of the five processes
- The matrix on the right shows how many resources each process still needs in order to complete

The algorithm for checking to see if a state is safe can now be stated.

1. Look for a row, **R**, whose unmet resource needs are all smaller than or equal to **A**. If no such row exists, > the system will eventually deadlock since no process can run to completion (assuming processes keep all resources until they exit).
2. Assume the process of the chosen row requests all the resources it needs (which is guaranteed to be possible) > and finishes. Mark that process as terminated and add all of its resources to the **A vector**.
3. Repeat **steps 1** and **2** until either all processes are marked terminated (in which case the initial state was safe) or no process is left whose resource needs can be met (in which case the system was not safe). If several processes are eligible to be chosen in step 1, it does not matter which one is selected: the pool of > available resources either gets larger, or at worst, stays the same.

### Deadlock Prevention

A solution to this could be **ordering of resources**, and then making scheduler not to enable processes run that are trying to lock the resources which are present lower on the list than the one they're currently processing. Example provided below shows it.

![Untitled](Short%2018a59/Untitled%2043.png)

Let's assume that resource *i* is number *2* (printer), and resource *j* is number *4* (tape drive). If a process *A* wants to lock first resource *number *1* is imagesetter), it is not allowed to do so. Only the processes that are trying to lock the higher items than they're currently locking are allowed to run. The problem with this approach is that it is impossible to find satisfying ordering of the resources to make all the processes happy.

## Livelock

Processes in a livelock are **not blocked**. Instead, they continue to execute **checking for a condition to become true** that will never become true. Thus, in addition to the resources they are holding, processes in livelock continue to **consume precious CPU time**. 

## Starvation

Starvation of a process occurs because of the presence of other processes as well as a stream of new **incoming processes** that end up **with higher priority** that the process being starved. Unlike deadlock or livelock, starvation can terminate on its own, e.g. when existing processes with higher priority terminate and no new processes with higher priority arrive.