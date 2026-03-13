# Operating Systems Interview Problems for SDE 2

## Table of Contents
1. [Process Management](#process-management)
2. [Thread Management](#thread-management)
3. [CPU Scheduling](#cpu-scheduling)
4. [Memory Management](#memory-management)
5. [Virtual Memory](#virtual-memory)
6. [File Systems](#file-systems)
7. [I/O Systems](#io-systems)
8. [Synchronization](#synchronization)
9. [Deadlocks](#deadlocks)
10. [Inter-Process Communication](#inter-process-communication)
11. [System Calls](#system-calls)
12. [Kernel Architecture](#kernel-architecture)
13. [Process Synchronization](#process-synchronization)
14. [Memory Allocation](#memory-allocation)
15. [Advanced Topics](#advanced-topics)

---

## Process Management

### Problem 1: Process Creation and Termination
**Scenario**: Design a process management system that handles process creation, execution, and termination.

**Requirements**:
- Understand process creation mechanisms (fork, exec, clone)
- Handle process termination and cleanup
- Manage process hierarchy (parent-child relationships)
- Implement process state tracking

**Questions**:
1. How does the `fork()` system call work?
2. What is the difference between `fork()` and `exec()`?
3. How does process termination work and what happens to child processes?
4. What is a zombie process and how do you prevent it?

---

### Problem 2: Process States and State Transitions
**Scenario**: Model and implement process lifecycle with all possible state transitions.

**Requirements**:
- Understand all process states (New, Ready, Running, Waiting, Terminated)
- Model state transitions accurately
- Handle state change events
- Implement process state machine

**Questions**:
1. What are the five states of a process?
2. What causes a process to transition from Running to Waiting?
3. What is the difference between Ready and Running states?
4. How does a process move from Waiting to Ready state?

---

### Problem 3: Process Control Block (PCB)
**Scenario**: Design a Process Control Block to store all process information.

**Requirements**:
- Understand PCB structure
- Store process state, registers, memory info
- Handle context switching
- Manage process metadata

**Questions**:
1. What information is stored in a Process Control Block?
2. Why is PCB important for context switching?
3. How does the OS use PCB for process management?
4. What happens to PCB when a process terminates?

---

### Problem 4: Process Scheduling Algorithms
**Scenario**: Implement different process scheduling algorithms and compare their performance.

**Requirements**:
- Implement FCFS, SJF, Priority, Round Robin
- Calculate turnaround time, waiting time, response time
- Compare algorithm performance
- Handle preemptive vs non-preemptive scheduling

**Questions**:
1. What are the different process scheduling algorithms?
2. How do you calculate average waiting time for FCFS?
3. What is the difference between preemptive and non-preemptive scheduling?
4. Which scheduling algorithm minimizes average waiting time?

---

### Problem 5: Multilevel Queue Scheduling
**Scenario**: Design a scheduler using multilevel queues for different process types.

**Requirements**:
- Implement multiple priority queues
- Assign processes to appropriate queues
- Schedule within and across queues
- Handle queue aging

**Questions**:
1. How does multilevel queue scheduling work?
2. What is the difference between multilevel queue and multilevel feedback queue?
3. How do you prevent starvation in multilevel queues?
4. When would you use multilevel queue scheduling?

---

### Problem 6: Process Synchronization Basics
**Scenario**: Implement synchronization mechanisms to coordinate multiple processes.

**Requirements**:
- Understand race conditions
- Implement critical sections
- Use mutual exclusion
- Prevent data corruption

**Questions**:
1. What is a race condition?
2. What is a critical section?
3. What are the requirements for a solution to the critical section problem?
4. How do you ensure mutual exclusion?

---

## Thread Management

### Problem 7: Thread Creation and Management
**Scenario**: Implement a multithreaded application with proper thread lifecycle management.

**Requirements**:
- Create and manage threads
- Handle thread synchronization
- Implement thread-safe operations
- Manage thread resources

**Questions**:
1. What is a thread and how does it differ from a process?
2. How do you create threads in different programming languages?
3. What is thread-local storage?
4. How do threads share memory compared to processes?

---

### Problem 8: User-Level vs Kernel-Level Threads
**Scenario**: Choose between user-level and kernel-level threading models.

**Requirements**:
- Understand threading models
- Compare user-level and kernel-level threads
- Understand many-to-one, one-to-one, many-to-many models
- Choose appropriate model for application

**Questions**:
1. What is the difference between user-level and kernel-level threads?
2. What are the advantages and disadvantages of each model?
3. How does the many-to-many threading model work?
4. When would you use user-level threads vs kernel-level threads?

---

### Problem 9: Thread Synchronization Primitives
**Scenario**: Implement thread synchronization using mutexes, semaphores, and condition variables.

**Requirements**:
- Use mutexes for mutual exclusion
- Use semaphores for resource counting
- Use condition variables for waiting
- Implement producer-consumer problem

**Questions**:
1. What is a mutex and how does it work?
2. What is the difference between a mutex and a semaphore?
3. How do condition variables work with mutexes?
4. How would you implement a producer-consumer problem using semaphores?

---

### Problem 10: Thread Pools
**Scenario**: Design and implement a thread pool for efficient thread management.

**Requirements**:
- Create thread pool with fixed size
- Queue tasks for execution
- Reuse threads efficiently
- Handle thread pool shutdown

**Questions**:
1. What is a thread pool and why is it used?
2. How do you determine the optimal thread pool size?
3. What are the advantages of thread pools?
4. How do you handle thread pool exhaustion?

---

## CPU Scheduling

### Problem 11: CPU Scheduling Metrics
**Scenario**: Calculate and optimize CPU scheduling metrics for different algorithms.

**Requirements**:
- Calculate turnaround time, waiting time, response time
- Calculate throughput and CPU utilization
- Compare metrics across algorithms
- Optimize for specific metrics

**Questions**:
1. What are the different CPU scheduling metrics?
2. How do you calculate turnaround time?
3. What is the difference between turnaround time and response time?
4. How do you maximize CPU utilization?

---

### Problem 12: Shortest Job First (SJF) Scheduling
**Scenario**: Implement SJF scheduling algorithm with and without preemption.

**Requirements**:
- Implement non-preemptive SJF
- Implement preemptive SJF (SRTF)
- Handle unknown burst times
- Compare with other algorithms

**Questions**:
1. How does Shortest Job First scheduling work?
2. What is the difference between SJF and SRTF?
3. How do you estimate burst time for SJF?
4. Why is SJF optimal for minimizing average waiting time?

---

### Problem 13: Round Robin Scheduling
**Scenario**: Implement Round Robin scheduling with optimal time quantum selection.

**Requirements**:
- Implement Round Robin algorithm
- Choose appropriate time quantum
- Handle context switching overhead
- Optimize for different workloads

**Questions**:
1. How does Round Robin scheduling work?
2. How do you choose an optimal time quantum?
3. What happens if the time quantum is too large or too small?
4. How does Round Robin prevent starvation?

---

### Problem 14: Priority Scheduling
**Scenario**: Implement priority scheduling with aging to prevent starvation.

**Requirements**:
- Implement priority scheduling
- Handle priority inversion problem
- Implement priority aging
- Compare preemptive vs non-preemptive priority

**Questions**:
1. How does priority scheduling work?
2. What is priority inversion and how is it solved?
3. How does priority aging prevent starvation?
4. What is the difference between static and dynamic priorities?

---

### Problem 15: Multilevel Feedback Queue Scheduling
**Scenario**: Design a multilevel feedback queue scheduler for adaptive process scheduling.

**Requirements**:
- Implement multiple queues with different priorities
- Move processes between queues based on behavior
- Handle I/O-bound and CPU-bound processes
- Prevent starvation

**Questions**:
1. How does multilevel feedback queue scheduling work?
2. How do processes move between queues?
3. How does it handle I/O-bound vs CPU-bound processes?
4. What are the advantages of multilevel feedback queues?

---

## Memory Management

### Problem 16: Contiguous Memory Allocation
**Scenario**: Implement memory allocation algorithms (First Fit, Best Fit, Worst Fit).

**Requirements**:
- Implement First Fit allocation
- Implement Best Fit allocation
- Implement Worst Fit allocation
- Handle external fragmentation

**Questions**:
1. How does First Fit memory allocation work?
2. What is the difference between Best Fit and Worst Fit?
3. What is external fragmentation and how do you reduce it?
4. What is compaction and when is it used?

---

### Problem 17: Paging System
**Scenario**: Design and implement a paging system for memory management.

**Requirements**:
- Understand paging mechanism
- Implement page table
- Handle page table lookups
- Optimize page table size

**Questions**:
1. How does paging work in memory management?
2. What is a page table and how is it used?
3. How do you translate logical addresses to physical addresses?
4. What are the advantages and disadvantages of paging?

---

### Problem 18: Segmentation
**Scenario**: Implement segmentation for logical memory organization.

**Requirements**:
- Understand segmentation mechanism
- Implement segment table
- Handle segment lookups
- Compare with paging

**Questions**:
1. How does segmentation differ from paging?
2. What are the advantages of segmentation?
3. How do you translate logical addresses in segmentation?
4. What are the disadvantages of segmentation?

---

### Problem 19: Paged Segmentation
**Scenario**: Design a memory management system combining paging and segmentation.

**Requirements**:
- Combine paging and segmentation
- Implement two-level address translation
- Handle segment-page lookups
- Optimize for performance

**Questions**:
1. How does paged segmentation work?
2. What are the benefits of combining paging and segmentation?
3. How do you translate addresses in paged segmentation?
4. How does it compare to pure paging or segmentation?

---

### Problem 20: Page Table Structure
**Scenario**: Design efficient page table structures for large address spaces.

**Requirements**:
- Understand hierarchical page tables
- Implement multi-level page tables
- Handle inverted page tables
- Optimize page table size

**Questions**:
1. What are the different page table structures?
2. How does a two-level page table work?
3. What is an inverted page table and when is it used?
4. How do you reduce page table size?

---

## Virtual Memory

### Problem 21: Virtual Memory Concept
**Scenario**: Explain and implement virtual memory system.

**Requirements**:
- Understand virtual memory concept
- Implement demand paging
- Handle page faults
- Optimize virtual memory performance

**Questions**:
1. What is virtual memory and why is it used?
2. How does virtual memory enable programs larger than physical memory?
3. What is demand paging?
4. What are the benefits of virtual memory?

---

### Problem 22: Page Replacement Algorithms
**Scenario**: Implement different page replacement algorithms and compare their performance.

**Requirements**:
- Implement FIFO page replacement
- Implement LRU page replacement
- Implement Optimal page replacement
- Compare algorithm performance

**Questions**:
1. What are the different page replacement algorithms?
2. How does FIFO page replacement work?
3. How does LRU page replacement work?
4. What is Belady's anomaly and which algorithms suffer from it?

---

### Problem 23: LRU Implementation
**Scenario**: Implement efficient LRU page replacement using different data structures.

**Requirements**:
- Implement LRU using stack
- Implement LRU using counters
- Implement LRU using hash map and doubly linked list
- Optimize for performance

**Questions**:
1. How do you implement LRU efficiently?
2. What data structures are used for LRU implementation?
3. What is the time complexity of LRU operations?
4. How do you approximate LRU when exact implementation is expensive?

---

### Problem 24: Thrashing
**Scenario**: Detect and prevent thrashing in virtual memory systems.

**Requirements**:
- Understand thrashing causes
- Detect thrashing conditions
- Implement thrashing prevention
- Optimize system to avoid thrashing

**Questions**:
1. What is thrashing and what causes it?
2. How do you detect thrashing?
3. How do you prevent thrashing?
4. What is the working set model?

---

### Problem 25: Page Fault Handling
**Scenario**: Implement complete page fault handling mechanism.

**Requirements**:
- Handle page fault interrupts
- Load pages from disk
- Update page tables
- Optimize page fault handling

**Questions**:
1. What happens when a page fault occurs?
2. What are the steps in page fault handling?
3. How do you minimize page faults?
4. What is the cost of a page fault?

---

## File Systems

### Problem 26: File System Structure
**Scenario**: Design a file system with boot block, superblock, inodes, and data blocks.

**Requirements**:
- Understand file system components
- Design file system layout
- Implement directory structure
- Handle file system metadata

**Questions**:
1. What are the main components of a file system?
2. What is stored in the superblock?
3. What is an inode and what information does it contain?
4. How are directories implemented in file systems?

---

### Problem 27: File Allocation Methods
**Scenario**: Implement different file allocation methods (contiguous, linked, indexed).

**Requirements**:
- Implement contiguous allocation
- Implement linked allocation
- Implement indexed allocation
- Compare allocation methods

**Questions**:
1. What are the different file allocation methods?
2. How does contiguous allocation work?
3. What are the advantages and disadvantages of linked allocation?
4. How does indexed allocation solve the problems of linked allocation?

---

### Problem 28: Directory Implementation
**Scenario**: Design directory structures for efficient file lookup.

**Requirements**:
- Implement linear list directory
- Implement hash table directory
- Implement B-tree directory
- Optimize directory operations

**Questions**:
1. How are directories implemented in file systems?
2. What is the difference between linear list and hash table directories?
3. How do you handle directory operations efficiently?
4. What is a directory entry?

---

### Problem 29: File System Consistency
**Scenario**: Implement file system consistency checking and recovery.

**Requirements**:
- Understand file system inconsistencies
- Implement consistency checker (fsck)
- Handle metadata corruption
- Recover from inconsistencies

**Questions**:
1. What causes file system inconsistencies?
2. How does a consistency checker work?
3. What is journaling in file systems?
4. How does journaling improve file system consistency?

---

### Problem 30: Journaling File Systems
**Scenario**: Design a journaling file system for improved reliability.

**Requirements**:
- Understand journaling concept
- Implement write-ahead logging
- Handle journal replay
- Optimize journal performance

**Questions**:
1. What is journaling in file systems?
2. How does journaling improve reliability?
3. What are the different journaling modes?
4. What is the performance impact of journaling?

---

## I/O Systems

### Problem 31: Disk Scheduling Algorithms
**Scenario**: Implement different disk scheduling algorithms to optimize I/O performance.

**Requirements**:
- Implement FCFS disk scheduling
- Implement SSTF disk scheduling
- Implement SCAN and C-SCAN algorithms
- Compare algorithm performance

**Questions**:
1. Why do we need disk scheduling algorithms?
2. How does SSTF (Shortest Seek Time First) work?
3. What is the SCAN algorithm and how does it differ from C-SCAN?
4. Which disk scheduling algorithm minimizes starvation?

---

### Problem 32: I/O Buffering
**Scenario**: Design I/O buffering mechanisms for efficient I/O operations.

**Requirements**:
- Understand single buffering
- Implement double buffering
- Implement circular buffering
- Optimize buffer management

**Questions**:
1. What is I/O buffering and why is it used?
2. What is the difference between single and double buffering?
3. How does circular buffering work?
4. How do you determine optimal buffer size?

---

### Problem 33: Spooling and Device Management
**Scenario**: Implement spooling system for device management.

**Requirements**:
- Understand spooling concept
- Implement print spooling
- Manage device queues
- Handle device allocation

**Questions**:
1. What is spooling and why is it used?
2. How does print spooling work?
3. How do you manage device queues?
4. What are the benefits of spooling?

---

## Synchronization

### Problem 34: Mutex Implementation
**Scenario**: Implement mutex from scratch using atomic operations.

**Requirements**:
- Understand mutex internals
- Implement mutex using test-and-set
- Implement mutex using compare-and-swap
- Handle mutex fairness

**Questions**:
1. How is a mutex implemented at the hardware level?
2. What is test-and-set and how is it used for mutex?
3. What is compare-and-swap and how does it differ from test-and-set?
4. How do you implement a fair mutex?

---

### Problem 35: Semaphore Implementation
**Scenario**: Implement semaphores using mutexes and condition variables.

**Requirements**:
- Implement binary semaphore
- Implement counting semaphore
- Handle semaphore operations atomically
- Prevent race conditions

**Questions**:
1. How are semaphores implemented?
2. What is the difference between binary and counting semaphores?
3. How do semaphore wait and signal operations work?
4. How do you implement semaphores using mutexes?

---

### Problem 36: Reader-Writer Problem
**Scenario**: Implement reader-writer locks with different priority policies.

**Requirements**:
- Implement reader-preference solution
- Implement writer-preference solution
- Implement fair solution
- Handle starvation prevention

**Questions**:
1. What is the reader-writer problem?
2. How do you implement reader-preference solution?
3. How do you implement writer-preference solution?
4. How do you prevent starvation in reader-writer locks?

---

### Problem 37: Dining Philosophers Problem
**Scenario**: Solve the dining philosophers problem with different strategies.

**Requirements**:
- Understand the problem
- Implement solution with semaphores
- Implement solution with mutexes
- Prevent deadlock and starvation

**Questions**:
1. What is the dining philosophers problem?
2. How do you solve it using semaphores?
3. How do you prevent deadlock in the solution?
4. How do you ensure fairness and prevent starvation?

---

### Problem 38: Producer-Consumer Problem
**Scenario**: Implement producer-consumer problem with bounded buffer.

**Requirements**:
- Implement using semaphores
- Implement using mutexes and condition variables
- Handle buffer management
- Ensure thread safety

**Questions**:
1. What is the producer-consumer problem?
2. How do you solve it using semaphores?
3. How do you solve it using mutexes and condition variables?
4. How do you handle buffer overflow and underflow?

---

## Deadlocks

### Problem 39: Deadlock Detection
**Scenario**: Implement deadlock detection algorithm using resource allocation graph.

**Requirements**:
- Build resource allocation graph
- Detect cycles in graph
- Identify deadlocked processes
- Handle deadlock recovery

**Questions**:
1. How do you detect deadlocks in a system?
2. What is a resource allocation graph?
3. How do you detect cycles in a resource allocation graph?
4. What happens after deadlock is detected?

---

### Problem 40: Deadlock Prevention
**Scenario**: Implement deadlock prevention by eliminating necessary conditions.

**Requirements**:
- Eliminate mutual exclusion (where possible)
- Eliminate hold and wait
- Eliminate no preemption
- Eliminate circular wait

**Questions**:
1. What are the four necessary conditions for deadlock?
2. How do you eliminate hold and wait condition?
3. How do you eliminate circular wait condition?
4. What are the trade-offs of deadlock prevention?

---

### Problem 41: Deadlock Avoidance - Banker's Algorithm
**Scenario**: Implement Banker's algorithm for deadlock avoidance.

**Requirements**:
- Understand Banker's algorithm
- Check safe state before allocation
- Implement safety algorithm
- Handle resource requests

**Questions**:
1. How does Banker's algorithm work?
2. What is a safe state?
3. How do you check if a state is safe?
4. What are the limitations of Banker's algorithm?

---

### Problem 42: Deadlock Recovery
**Scenario**: Implement deadlock recovery mechanisms.

**Requirements**:
- Process termination strategies
- Resource preemption strategies
- Rollback mechanisms
- Minimize recovery cost

**Questions**:
1. What are the strategies for deadlock recovery?
2. How do you choose which process to terminate?
3. How does resource preemption work?
4. What is the cost of deadlock recovery?

---

## Inter-Process Communication

### Problem 43: Shared Memory IPC
**Scenario**: Implement inter-process communication using shared memory.

**Requirements**:
- Create shared memory segments
- Attach/detach shared memory
- Synchronize access to shared memory
- Handle shared memory cleanup

**Questions**:
1. How does shared memory IPC work?
2. How do you create and use shared memory?
3. How do you synchronize access to shared memory?
4. What are the advantages and disadvantages of shared memory?

---

### Problem 44: Message Passing IPC
**Scenario**: Implement IPC using message queues and pipes.

**Requirements**:
- Implement named pipes (FIFO)
- Implement message queues
- Handle message synchronization
- Compare with shared memory

**Questions**:
1. How does message passing IPC work?
2. What is the difference between pipes and message queues?
3. How do you implement bidirectional communication?
4. What are the advantages of message passing over shared memory?

---

### Problem 45: Socket Programming
**Scenario**: Implement IPC using Unix domain sockets and network sockets.

**Requirements**:
- Create socket connections
- Handle socket communication
- Implement client-server model
- Handle socket errors

**Questions**:
1. How do Unix domain sockets work for IPC?
2. What is the difference between stream and datagram sockets?
3. How do you implement a client-server communication using sockets?
4. How do sockets compare to other IPC mechanisms?

---

## System Calls

### Problem 46: System Call Implementation
**Scenario**: Understand and trace system call execution flow.

**Requirements**:
- Understand system call interface
- Trace system call execution
- Understand mode switching
- Handle system call errors

**Questions**:
1. How do system calls work?
2. What happens when a system call is invoked?
3. How does the system switch from user mode to kernel mode?
4. What is the cost of a system call?

---

### Problem 47: File System System Calls
**Scenario**: Implement file operations using system calls (open, read, write, close).

**Requirements**:
- Use open() system call
- Implement read/write operations
- Handle file descriptors
- Manage file permissions

**Questions**:
1. How do file system system calls work?
2. What is a file descriptor?
3. How do you handle file I/O errors?
4. What is the difference between blocking and non-blocking I/O?

---

### Problem 48: Process Management System Calls
**Scenario**: Use system calls for process creation and management.

**Requirements**:
- Use fork() and exec() system calls
- Handle process synchronization
- Manage process groups
- Handle signals

**Questions**:
1. How does fork() system call work?
2. What is the difference between fork() and vfork()?
3. How do exec() system calls work?
4. How do you handle process synchronization using system calls?

---

## Kernel Architecture

### Problem 49: Monolithic vs Microkernel
**Scenario**: Compare monolithic and microkernel architectures.

**Requirements**:
- Understand kernel architectures
- Compare performance
- Compare maintainability
- Choose appropriate architecture

**Questions**:
1. What is the difference between monolithic and microkernel?
2. What are the advantages and disadvantages of each?
3. How does performance differ between architectures?
4. What are hybrid kernel architectures?

---

### Problem 50: Kernel Modules
**Scenario**: Design and implement kernel modules for extensibility.

**Requirements**:
- Understand kernel module concept
- Load and unload modules
- Handle module dependencies
- Implement module interfaces

**Questions**:
1. What are kernel modules and why are they used?
2. How do you load and unload kernel modules?
3. How do kernel modules interact with the kernel?
4. What are the security implications of kernel modules?

---

## Process Synchronization

### Problem 51: Atomic Operations
**Scenario**: Implement atomic operations for lock-free programming.

**Requirements**:
- Understand atomic operations
- Use compare-and-swap
- Implement lock-free data structures
- Handle memory ordering

**Questions**:
1. What are atomic operations?
2. How does compare-and-swap work?
3. What is lock-free programming?
4. What are memory ordering semantics?

---

### Problem 52: Spinlocks vs Mutexes
**Scenario**: Choose between spinlocks and mutexes for different scenarios.

**Requirements**:
- Understand spinlock implementation
- Compare with mutexes
- Choose based on wait time
- Optimize for performance

**Questions**:
1. What is a spinlock and how does it differ from a mutex?
2. When should you use a spinlock vs a mutex?
3. What is the performance difference?
4. How do you implement adaptive mutexes?

---

### Problem 53: Condition Variables
**Scenario**: Implement condition variables for efficient waiting.

**Requirements**:
- Understand condition variable concept
- Implement wait and signal operations
- Use with mutexes correctly
- Handle spurious wakeups

**Questions**:
1. What are condition variables and why are they used?
2. How do condition variables work with mutexes?
3. What is a spurious wakeup?
4. How do you use condition variables correctly?

---

## Memory Allocation

### Problem 54: Buddy System Allocation
**Scenario**: Implement buddy system for memory allocation.

**Requirements**:
- Understand buddy system algorithm
- Implement allocation and deallocation
- Handle memory coalescing
- Optimize for fragmentation

**Questions**:
1. How does the buddy system work?
2. What are the advantages of the buddy system?
3. How does memory coalescing work in buddy system?
4. What are the disadvantages of buddy system?

---

### Problem 55: Slab Allocation
**Scenario**: Implement slab allocator for kernel object allocation.

**Requirements**:
- Understand slab allocation concept
- Implement slab creation and destruction
- Handle object allocation from slabs
- Optimize for common object sizes

**Questions**:
1. What is slab allocation and why is it used?
2. How does slab allocation work?
3. What are the advantages of slab allocation?
4. How does it differ from buddy system?

---

## Advanced Topics

### Problem 56: Real-Time Scheduling
**Scenario**: Implement real-time scheduling algorithms (Rate Monotonic, EDF).

**Requirements**:
- Understand real-time systems
- Implement Rate Monotonic Scheduling
- Implement Earliest Deadline First (EDF)
- Handle real-time constraints

**Questions**:
1. What are real-time scheduling algorithms?
2. How does Rate Monotonic Scheduling work?
3. How does Earliest Deadline First work?
4. What is the difference between hard and soft real-time systems?

---

### Problem 57: Multi-Core Scheduling
**Scenario**: Design scheduling algorithms for multi-core systems.

**Requirements**:
- Understand multi-core challenges
- Implement load balancing
- Handle cache affinity
- Optimize for NUMA systems

**Questions**:
1. What are the challenges of multi-core scheduling?
2. How do you balance load across cores?
3. What is cache affinity and why is it important?
4. How does scheduling differ in NUMA systems?

---

### Problem 58: Virtualization and Containers
**Scenario**: Understand OS-level virtualization and containerization.

**Requirements**:
- Understand virtualization concepts
- Understand container technology
- Compare VMs and containers
- Handle resource isolation

**Questions**:
1. What is OS-level virtualization?
2. How do containers differ from virtual machines?
3. How does container isolation work?
4. What are the advantages of containers?

---

### Problem 59: Security in Operating Systems
**Scenario**: Implement security mechanisms in operating systems.

**Requirements**:
- Understand access control
- Implement capabilities
- Handle security policies
- Prevent privilege escalation

**Questions**:
1. How does access control work in operating systems?
2. What are capabilities and how do they differ from ACLs?
3. How do you prevent privilege escalation?
4. What is sandboxing and how does it work?

---

### Problem 60: Performance Monitoring and Profiling
**Scenario**: Implement performance monitoring and profiling tools.

**Requirements**:
- Monitor CPU usage
- Monitor memory usage
- Profile system calls
- Identify performance bottlenecks

**Questions**:
1. How do you monitor OS performance?
2. What tools are used for profiling?
3. How do you identify performance bottlenecks?
4. What metrics are important for OS performance?

---

## Summary

This comprehensive guide covers all major Operating Systems topics for SDE 2 interviews:

1. **Process Management**: Creation, termination, states, scheduling
2. **Thread Management**: Threads, synchronization, thread pools
3. **CPU Scheduling**: Algorithms, metrics, optimization
4. **Memory Management**: Allocation, paging, segmentation
5. **Virtual Memory**: Page replacement, thrashing, page faults
6. **File Systems**: Structure, allocation, consistency, journaling
7. **I/O Systems**: Disk scheduling, buffering, spooling
8. **Synchronization**: Mutexes, semaphores, condition variables
9. **Deadlocks**: Detection, prevention, avoidance, recovery
10. **IPC**: Shared memory, message passing, sockets
11. **System Calls**: Implementation, file operations, process management
12. **Kernel Architecture**: Monolithic, microkernel, modules
13. **Advanced Topics**: Real-time, multi-core, virtualization, security

**Total Coverage: 60 Problems covering all OS topics**

Each problem includes scenario, requirements, and interview-style questions to help you prepare comprehensively.
