# CS Fundamentals Interview Solutions for SDE 2
## Operating Systems, Networking, and DBMS

## Table of Contents
1. [Operating Systems Solutions](#operating-systems-solutions)
2. [Networking Solutions](#networking-solutions)
3. [Database Management Systems Solutions](#database-management-systems-solutions)

---

## Operating Systems Solutions

### Solution 1: Process vs Thread

**Fundamental Difference:**
- **Process**: Independent execution unit with its own memory space
- **Thread**: Lightweight unit of execution within a process, shares memory space

**Memory Sharing:**
- **Processes**: Each process has separate memory space (isolated)
- **Threads**: Threads within a process share memory space (heap, data segment)

**Context Switching Overhead:**
- **Process**: Higher overhead (save/restore entire memory space, registers, file descriptors)
- **Thread**: Lower overhead (save/restore only registers and stack)

**When to Use:**

**Use Processes when:**
- Need isolation and fault tolerance
- Different security contexts required
- Crash of one shouldn't affect others
- CPU-bound tasks that benefit from multiple cores

**Use Threads when:**
- Need to share data efficiently
- I/O-bound tasks (can wait while other threads run)
- Need fine-grained synchronization
- Lower memory overhead needed

**Example:**
```python
# Process example
import multiprocessing

def worker_process(data):
    # Isolated memory space
    result = process_data(data)
    return result

# Thread example
import threading

def worker_thread(data, shared_list):
    # Shares memory with other threads
    result = process_data(data)
    shared_list.append(result)
```

**Key Points:**
- Processes provide isolation, threads provide efficiency
- Processes have higher overhead, threads have lower overhead
- Choose based on isolation needs vs performance needs
- Modern systems often use hybrid approach

---

### Solution 2: Process Scheduling Algorithms

**Scheduling Algorithms:**

1. **First Come First Served (FCFS)**
   - Non-preemptive
   - Simple but can cause convoy effect
   - Good for batch systems

2. **Shortest Job First (SJF)**
   - Non-preemptive (SJF) or preemptive (SRTF - Shortest Remaining Time First)
   - Minimizes average waiting time
   - Requires knowledge of execution time

3. **Priority Scheduling**
   - Preemptive or non-preemptive
   - Higher priority processes run first
   - Can cause starvation of low-priority processes

4. **Round Robin (RR)**
   - Preemptive
   - Each process gets time quantum
   - Good for time-sharing systems
   - Prevents starvation

5. **Multilevel Queue**
   - Multiple queues with different priorities
   - Processes assigned to queues based on type
   - Each queue can have different scheduling algorithm

**Preemptive vs Non-Preemptive:**
- **Preemptive**: OS can interrupt running process
- **Non-preemptive**: Process runs until completion or voluntary yield

**Round Robin Example:**
```
Time Quantum = 4ms
Processes: P1(24ms), P2(3ms), P3(3ms)

Execution:
P1: 0-4ms (20ms remaining)
P2: 4-7ms (completed)
P3: 7-10ms (completed)
P1: 10-14ms (16ms remaining)
P1: 14-18ms (12ms remaining)
...
```

**Choosing Algorithm:**
- **FCFS**: Simple, fair, but poor for interactive systems
- **SJF**: Optimal for minimizing waiting time, but requires execution time knowledge
- **Round Robin**: Good for interactive systems, prevents starvation
- **Priority**: Good when priorities matter, but need aging to prevent starvation

**Key Points:**
- Different algorithms optimize for different metrics
- Preemptive scheduling provides better responsiveness
- Round Robin balances fairness and responsiveness
- Choose based on system requirements

---

### Solution 3: Deadlock Detection and Prevention

**Four Necessary Conditions (Coffman Conditions):**
1. **Mutual Exclusion**: Resources cannot be shared
2. **Hold and Wait**: Process holds resource while waiting for another
3. **No Preemption**: Resources cannot be forcibly taken away
4. **Circular Wait**: Circular chain of processes waiting for resources

**Deadlock Prevention:**
- **Eliminate Mutual Exclusion**: Not always possible (some resources inherently exclusive)
- **Eliminate Hold and Wait**: Require processes to request all resources at once
- **Allow Preemption**: Force release of resources if needed
- **Eliminate Circular Wait**: Order resources, request in order

**Deadlock Detection:**
```python
# Resource Allocation Graph (RAG)
# Nodes: Processes and Resources
# Edges: Request edges and Assignment edges

# Detection Algorithm:
# 1. Build wait-for graph
# 2. Check for cycles
# 3. If cycle exists, deadlock detected

def detect_deadlock(processes, resources):
    # Build wait-for graph
    wait_graph = build_wait_graph(processes, resources)
    
    # Check for cycles using DFS
    visited = set()
    rec_stack = set()
    
    for process in processes:
        if has_cycle(process, wait_graph, visited, rec_stack):
            return True
    
    return False
```

**Deadlock Recovery:**
1. **Process Termination**: Kill one or more processes
2. **Resource Preemption**: Take resources from processes
3. **Rollback**: Rollback processes to safe state

**Banker's Algorithm (Deadlock Avoidance):**
```python
# Check if system is in safe state before allocating resources
def is_safe_state(available, allocation, need):
    work = available.copy()
    finish = [False] * len(processes)
    safe_sequence = []
    
    while True:
        found = False
        for i in range(len(processes)):
            if not finish[i] and need[i] <= work:
                work += allocation[i]
                finish[i] = True
                safe_sequence.append(i)
                found = True
        
        if not found:
            break
    
    return all(finish), safe_sequence
```

**Key Points:**
- All four conditions must hold for deadlock
- Prevention is better than detection
- Banker's algorithm avoids deadlocks
- Recovery strategies have trade-offs

---

### Solution 4: Memory Management - Paging vs Segmentation

**Paging:**
- Physical memory divided into fixed-size frames
- Logical memory divided into fixed-size pages
- Page table maps logical pages to physical frames
- No external fragmentation
- Internal fragmentation possible

**Segmentation:**
- Logical memory divided into variable-size segments
- Each segment represents logical unit (code, data, stack)
- Segment table maps segments to physical memory
- External fragmentation possible
- No internal fragmentation

**Paging Example:**
```
Logical Address: Page Number (4 bits) + Offset (12 bits)
Page Table: Page 0 -> Frame 5, Page 1 -> Frame 8, etc.

Address Translation:
Logical: 0x1234 (Page 1, Offset 0x234)
Physical: Frame 8 + Offset 0x234 = 0x8234
```

**Segmentation Example:**
```
Logical Address: Segment Number (2 bits) + Offset (14 bits)
Segment Table: Seg 0 (Base: 0x1000, Limit: 0x500)
              Seg 1 (Base: 0x2000, Limit: 0x300)

Address Translation:
Logical: Seg 1, Offset 0x100
Physical: Base(1) + Offset = 0x2000 + 0x100 = 0x2100
```

**Advantages/Disadvantages:**

**Paging:**
- ✅ No external fragmentation
- ✅ Simple memory allocation
- ❌ Internal fragmentation
- ❌ No logical structure

**Segmentation:**
- ✅ Logical structure
- ✅ Easy sharing
- ✅ Protection per segment
- ❌ External fragmentation
- ❌ Complex allocation

**Combined Approach:**
- Modern systems use paged segmentation
- Segments divided into pages
- Combines benefits of both

**Key Points:**
- Paging: Fixed-size, no external fragmentation
- Segmentation: Variable-size, logical structure
- Modern systems combine both approaches
- Choose based on requirements

---

### Solution 5: Virtual Memory and Page Replacement

**Virtual Memory:**
- Allows programs to use more memory than physically available
- Pages stored on disk when not in use
- Provides illusion of large memory space
- Enables multiprogramming

**Why Virtual Memory:**
- Allows running programs larger than physical memory
- Enables more processes to run simultaneously
- Simplifies memory management
- Provides memory protection

**FIFO Page Replacement:**
```python
def fifo_page_replacement(pages, frames):
    queue = []  # Queue of pages in memory
    page_faults = 0
    
    for page in pages:
        if page not in queue:
            page_faults += 1
            if len(queue) == frames:
                queue.pop(0)  # Remove oldest
            queue.append(page)
    
    return page_faults
```

**LRU Page Replacement:**
```python
def lru_page_replacement(pages, frames):
    memory = []  # Pages in memory (most recent at end)
    page_faults = 0
    
    for page in pages:
        if page in memory:
            # Move to end (most recently used)
            memory.remove(page)
            memory.append(page)
        else:
            page_faults += 1
            if len(memory) == frames:
                memory.pop(0)  # Remove least recently used
            memory.append(page)
    
    return page_faults
```

**Belady's Anomaly:**
- More frames can lead to more page faults
- Occurs with FIFO, not with LRU
- Example: FIFO with 3 frames has fewer faults than with 4 frames

**Other Algorithms:**
- **Optimal**: Replace page that won't be used for longest time (theoretical)
- **Clock**: Circular list with reference bit
- **LFU**: Least Frequently Used

**Key Points:**
- Virtual memory enables larger programs
- Page replacement when memory full
- LRU is practical and avoids Belady's anomaly
- Choose algorithm based on access pattern

---

### Solution 6: Inter-Process Communication (IPC)

**IPC Mechanisms:**

1. **Shared Memory**
   - Fastest IPC method
   - Processes share memory region
   - Requires synchronization
   - Example: POSIX shared memory

2. **Message Passing**
   - Processes communicate via messages
   - Slower but safer
   - No shared memory needed
   - Example: Pipes, sockets, message queues

3. **Pipes**
   - Unidirectional communication
   - Parent-child processes
   - Example: `pipe()` system call

4. **Named Pipes (FIFO)**
   - Named pipes for unrelated processes
   - Persistent in filesystem
   - Example: `mkfifo()`

5. **Message Queues**
   - Structured messages
   - Can have priorities
   - Example: POSIX message queues

6. **Sockets**
   - Network-based IPC
   - Can be local (Unix sockets) or network
   - Example: TCP/UDP sockets

**Shared Memory Example:**
```c
// Process 1 (Writer)
int shm_fd = shm_open("/my_shm", O_CREAT | O_RDWR, 0666);
ftruncate(shm_fd, sizeof(int));
int *ptr = mmap(NULL, sizeof(int), PROT_WRITE, MAP_SHARED, shm_fd, 0);
*ptr = 42;

// Process 2 (Reader)
int shm_fd = shm_open("/my_shm", O_RDONLY, 0666);
int *ptr = mmap(NULL, sizeof(int), PROT_READ, MAP_SHARED, shm_fd, 0);
printf("Value: %d\n", *ptr);
```

**Pipe Example:**
```c
int fd[2];
pipe(fd);

if (fork() == 0) {
    // Child: write
    close(fd[0]);
    write(fd[1], "Hello", 5);
} else {
    // Parent: read
    close(fd[1]);
    char buf[10];
    read(fd[0], buf, 10);
}
```

**When to Use:**
- **Shared Memory**: High performance, need synchronization
- **Message Passing**: Simpler, safer, slower
- **Pipes**: Simple parent-child communication
- **Sockets**: Network communication or local IPC

**Key Points:**
- Shared memory fastest but needs synchronization
- Message passing safer but slower
- Choose based on performance vs safety trade-off
- Modern systems use multiple IPC mechanisms

---

### Solution 7: Synchronization Primitives

**Mutex (Mutual Exclusion):**
- Binary semaphore (0 or 1)
- Only one thread can hold lock
- Must be released by same thread that acquired it
- Used for critical sections

**Semaphore:**
- Counting semaphore (can be > 1)
- Multiple threads can access resource
- Binary semaphore (0 or 1) = mutex
- Used for resource counting

**Condition Variables:**
- Allow threads to wait for conditions
- Used with mutexes
- Thread waits until condition is true
- Used for producer-consumer, reader-writer

**Mutex Example:**
```python
import threading

mutex = threading.Lock()
counter = 0

def increment():
    global counter
    mutex.acquire()
    try:
        counter += 1
    finally:
        mutex.release()
```

**Semaphore Example:**
```python
import threading

# Allow 3 threads to access resource
semaphore = threading.Semaphore(3)

def access_resource():
    semaphore.acquire()
    try:
        # Use resource
        pass
    finally:
        semaphore.release()
```

**Producer-Consumer with Semaphores:**
```python
import threading
import queue

buffer = queue.Queue(maxsize=10)
empty = threading.Semaphore(10)  # Empty slots
full = threading.Semaphore(0)      # Filled slots
mutex = threading.Lock()

def producer():
    while True:
        item = produce_item()
        empty.acquire()  # Wait for empty slot
        mutex.acquire()
        buffer.put(item)
        mutex.release()
        full.release()   # Signal item available

def consumer():
    while True:
        full.acquire()   # Wait for item
        mutex.acquire()
        item = buffer.get()
        mutex.release()
        empty.release()  # Signal slot empty
        consume_item(item)
```

**Key Points:**
- Mutex: Binary lock for critical sections
- Semaphore: Counting resource access
- Condition variables: Wait for conditions
- Use appropriate primitive for scenario

---

### Solution 8: Race Conditions and Critical Sections

**Race Condition:**
- Occurs when multiple threads access shared data concurrently
- Outcome depends on timing of execution
- Can lead to incorrect results

**Critical Section:**
- Code section that accesses shared resources
- Must be executed atomically
- Only one thread should execute at a time

**Race Condition Example:**
```python
# Unsafe - race condition
counter = 0

def increment():
    global counter
    temp = counter
    temp += 1
    counter = temp  # Not atomic!

# Thread 1: temp=0, temp=1, counter=1
# Thread 2: temp=0, temp=1, counter=1 (should be 2!)
```

**Fixing Race Condition:**
```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    lock.acquire()
    try:
        counter += 1  # Atomic operation
    finally:
        lock.release()
```

**Mutual Exclusion Solutions:**
1. **Locks/Mutexes**: Simple, but can deadlock
2. **Semaphores**: More flexible
3. **Atomic Operations**: Hardware-level atomicity
4. **Transactional Memory**: Database-like transactions

**Atomic Operations:**
```python
import threading

counter = 0
lock = threading.Lock()

# Using atomic increment (if supported)
# counter += 1  # If atomic in hardware

# Or using lock
def safe_increment():
    global counter
    with lock:
        counter += 1
```

**Key Points:**
- Race conditions cause unpredictable behavior
- Critical sections need mutual exclusion
- Use locks, semaphores, or atomic operations
- Always protect shared data access

---

### Solution 9: File System Design

**File System Components:**
1. **Boot Block**: Contains boot code
2. **Super Block**: File system metadata (size, free blocks, etc.)
3. **Inode Table**: File metadata (permissions, size, block pointers)
4. **Data Blocks**: Actual file data

**Directory Structure:**
- Directory is a special file containing (filename, inode number) pairs
- Hierarchical tree structure
- Root directory at top

**File Allocation Methods:**

1. **Contiguous Allocation**
   - Files stored in contiguous blocks
   - Simple, fast sequential access
   - External fragmentation
   - Example: Early file systems

2. **Linked Allocation**
   - Files stored as linked list of blocks
   - No external fragmentation
   - Slow random access
   - Example: FAT file system

3. **Indexed Allocation**
   - Index block contains pointers to data blocks
   - Fast random access
   - Overhead of index block
   - Example: Unix file systems

**Inode Structure:**
```
Inode contains:
- File type and permissions
- Owner and group
- File size
- Timestamps
- Direct block pointers (12)
- Single indirect pointer
- Double indirect pointer
- Triple indirect pointer
```

**File System Consistency:**
- **Consistency Check**: fsck checks for inconsistencies
- **Journaling**: Log changes before applying (ext3, ext4)
- **Copy-on-Write**: Write to new location, then update pointers (ZFS, Btrfs)

**Journaling Example:**
```
1. Write intent to journal
2. Write data to journal
3. Commit journal entry
4. Write data to actual location
5. Remove journal entry
```

**Key Points:**
- File systems organize data on disk
- Different allocation methods have trade-offs
- Inodes store file metadata
- Journaling improves consistency

---

### Solution 10: Disk Scheduling Algorithms

**Why Disk Scheduling:**
- Disk I/O is slow compared to CPU
- Seek time is major component
- Optimize order of requests to minimize seek time

**Scheduling Algorithms:**

1. **FCFS (First Come First Served)**
   - Simple, fair
   - No optimization
   - Can have high seek time

2. **SSTF (Shortest Seek Time First)**
   - Serve request closest to current head position
   - Minimizes seek time
   - Can cause starvation

3. **SCAN (Elevator Algorithm)**
   - Head moves in one direction, serves requests
   - Reverses direction at end
   - Fair, no starvation
   - Example: Elevator movement

4. **C-SCAN (Circular SCAN)**
   - Like SCAN but returns to start immediately
   - More uniform wait time
   - Better for high load

5. **LOOK**
   - Like SCAN but reverses at last request
   - More efficient than SCAN

**Example:**
```
Current head position: 50
Requests: 98, 183, 37, 122, 14, 124, 65, 67

FCFS: 50->98->183->37->122->14->124->65->67
      Total seek: 640

SSTF: 50->65->67->37->14->98->122->124->183
      Total seek: 236

SCAN (right first): 50->65->67->98->122->124->183->199->37->14
      Total seek: 236
```

**Minimizing Starvation:**
- Use SCAN or C-SCAN instead of SSTF
- Use aging for SSTF
- Use deadline scheduling

**Key Points:**
- Disk scheduling optimizes I/O performance
- SSTF minimizes seek time but can starve
- SCAN provides fairness
- Choose based on workload

---

### Solution 11: CPU Scheduling and Context Switching

**CPU Scheduling Goals:**
- Maximize CPU utilization
- Maximize throughput
- Minimize turnaround time
- Minimize waiting time
- Minimize response time

**Turnaround Time:**
- Time from submission to completion
- Turnaround Time = Completion Time - Arrival Time

**Waiting Time:**
- Time process spends waiting in ready queue
- Waiting Time = Turnaround Time - Burst Time

**Example Calculation:**
```
Process  Arrival  Burst  Completion  Turnaround  Waiting
P1       0        24     24          24          0
P2       1        3      27          26          23
P3       2        3      30          28          25

Average Turnaround: (24+26+28)/3 = 26
Average Waiting: (0+23+25)/3 = 16
```

**Multilevel Queue Scheduling:**
- Multiple queues with different priorities
- Processes assigned to queues based on type
- Each queue can have different scheduling algorithm
- Higher priority queues served first

**Example:**
```
Queue 1 (System): Round Robin, quantum=10ms
Queue 2 (Interactive): Round Robin, quantum=20ms
Queue 3 (Batch): FCFS
```

**Context Switching:**
- Saving state of current process
- Loading state of next process
- Overhead: ~1-1000 microseconds
- Minimize for better performance

**Key Points:**
- Different metrics for different goals
- Turnaround vs response time trade-off
- Multilevel queues for different process types
- Context switching has overhead

---

### Solution 12: Memory Allocation Strategies

**Memory Allocation Concepts:**
- **Hole**: Block of free memory
- **Allocation**: Assigning memory to process
- **Fragmentation**: Wasted memory

**Allocation Algorithms:**

1. **First Fit**
   - Allocate first hole large enough
   - Fast but can fragment
   ```python
   def first_fit(process_size, holes):
       for hole in holes:
           if hole.size >= process_size:
               allocate(hole, process_size)
               return hole
       return None
   ```

2. **Best Fit**
   - Allocate smallest hole large enough
   - Minimizes waste but slow
   ```python
   def best_fit(process_size, holes):
       best_hole = None
       for hole in holes:
           if hole.size >= process_size:
               if best_hole is None or hole.size < best_hole.size:
                   best_hole = hole
       if best_hole:
           allocate(best_hole, process_size)
       return best_hole
   ```

3. **Worst Fit**
   - Allocate largest hole
   - Leaves large holes for future
   - Can fragment more

**External Fragmentation:**
- Free memory scattered in small pieces
- Total free memory sufficient but not contiguous
- Solution: Compaction (move processes to consolidate holes)

**Compaction:**
- Move processes to consolidate free memory
- Expensive (requires updating addresses)
- Can be done when system idle

**Key Points:**
- First Fit: Fast, simple
- Best Fit: Minimizes waste, slower
- External fragmentation is common problem
- Compaction solves fragmentation but expensive

---

### Solution 13: System Calls and Kernel Mode

**User Mode vs Kernel Mode:**
- **User Mode**: Restricted mode, cannot access hardware directly
- **Kernel Mode**: Privileged mode, full hardware access
- Mode bit in CPU indicates current mode

**System Calls:**
- Interface between user programs and OS
- Switch from user mode to kernel mode
- OS performs privileged operation
- Return to user mode

**System Call Process:**
```
1. User program calls system call
2. Trap instruction executed (mode switch to kernel)
3. OS handles system call
4. Return to user mode
5. User program continues
```

**When Mode Switching Occurs:**
- System calls
- Interrupts (I/O completion, timer)
- Exceptions (divide by zero, page fault)
- Privileged instructions

**Kernel Protection:**
- Memory protection (user cannot access kernel memory)
- Privileged instructions only in kernel mode
- System call interface is controlled gateway
- Prevents user programs from crashing system

**Example System Calls:**
- `open()`: Open file
- `read()`: Read from file
- `write()`: Write to file
- `fork()`: Create process
- `exec()`: Execute program
- `exit()`: Terminate process

**Key Points:**
- Two modes for protection
- System calls provide controlled access
- Mode switching has overhead
- Kernel protected from user programs

---

### Solution 14: Process States and State Transitions

**Process States:**
1. **New**: Process being created
2. **Ready**: Process ready to run, waiting for CPU
3. **Running**: Process executing on CPU
4. **Waiting/Blocked**: Process waiting for event (I/O, signal)
5. **Terminated**: Process finished execution

**State Transitions:**
```
New -> Ready: Process admitted
Ready -> Running: CPU scheduler selects process
Running -> Ready: Time slice expired (preemption)
Running -> Waiting: Process waits for I/O or event
Waiting -> Ready: I/O complete or event occurred
Running -> Terminated: Process completes or killed
```

**Blocking and Unblocking:**
- **Blocking**: Process waits for I/O or event
- **Unblocking**: Event occurs, process moves to ready

**Example:**
```
Process reading file:
Running -> Waiting (initiate disk I/O)
Waiting -> Ready (disk I/O complete)
Ready -> Running (scheduled)
```

**State Queue Management:**
- Ready queue: Processes ready to run
- Wait queues: Processes waiting for specific events
- Scheduler selects from ready queue

**Key Points:**
- Five main process states
- Transitions based on events
- Blocking for I/O improves CPU utilization
- State queues managed by OS

---

### Solution 15: Threading Models

**User-Level Threads:**
- Managed by user-level library
- OS unaware of threads
- Fast creation and switching
- Blocking one thread blocks all threads

**Kernel-Level Threads:**
- Managed by OS kernel
- OS aware of each thread
- Slower creation and switching
- Blocking one thread doesn't block others

**Threading Models:**

1. **Many-to-One**
   - Many user threads to one kernel thread
   - Fast but blocking issue
   - Example: Early Java threads

2. **One-to-One**
   - One user thread to one kernel thread
   - No blocking issue but overhead
   - Example: Linux threads

3. **Many-to-Many**
   - Many user threads to many kernel threads
   - Best of both worlds
   - Complex to implement
   - Example: Solaris threads

**Comparison:**
```
                User-Level    Kernel-Level
Creation         Fast          Slow
Switching        Fast          Slow
Blocking         All blocked   One blocked
Scalability      Limited       Better
```

**Key Points:**
- User-level: Fast but blocking issue
- Kernel-level: Slower but better concurrency
- Many-to-many: Best balance
- Modern systems use hybrid approaches

---

## Networking Solutions

### Solution 16: OSI Model and TCP/IP Stack

**OSI Model (7 Layers):**
1. **Physical**: Bits over physical medium
2. **Data Link**: Frames, error detection/correction
3. **Network**: Routing, IP addresses
4. **Transport**: End-to-end communication, TCP/UDP
5. **Session**: Session management
6. **Presentation**: Data encoding/encryption
7. **Application**: User applications

**TCP/IP Model (4 Layers):**
1. **Link Layer**: Physical + Data Link
2. **Internet Layer**: Network layer (IP)
3. **Transport Layer**: TCP/UDP
4. **Application Layer**: Application + Presentation + Session

**Mapping:**
```
OSI                    TCP/IP
Application  ────────┐
Presentation ────────┤ Application
Session      ────────┘
Transport    ────────  Transport
Network      ────────  Internet
Data Link    ────────┐
Physical     ────────┘ Link
```

**Protocols by Layer:**
- **Application**: HTTP, HTTPS, FTP, SMTP, DNS
- **Transport**: TCP, UDP
- **Network**: IP, ICMP, ARP
- **Data Link**: Ethernet, WiFi
- **Physical**: Cables, radio waves

**Encapsulation:**
```
Application Data
    ↓
+ TCP Header (Transport)
    ↓
+ IP Header (Network)
    ↓
+ Ethernet Header (Data Link)
    ↓
Physical Transmission
```

**Key Points:**
- OSI: Theoretical 7-layer model
- TCP/IP: Practical 4-layer model
- Each layer adds header
- Protocols operate at specific layers

---

### Solution 17: TCP vs UDP

**TCP (Transmission Control Protocol):**
- Connection-oriented
- Reliable delivery
- Ordered delivery
- Flow control
- Congestion control
- Slower, higher overhead

**UDP (User Datagram Protocol):**
- Connectionless
- Unreliable delivery
- No ordering guarantee
- No flow control
- No congestion control
- Faster, lower overhead

**TCP Header (20 bytes minimum):**
```
Source Port (16 bits)
Destination Port (16 bits)
Sequence Number (32 bits)
Acknowledgment Number (32 bits)
Flags (SYN, ACK, FIN, etc.)
Window Size (16 bits)
Checksum (16 bits)
```

**UDP Header (8 bytes):**
```
Source Port (16 bits)
Destination Port (16 bits)
Length (16 bits)
Checksum (16 bits)
```

**When to Use TCP:**
- Web browsing (HTTP)
- Email (SMTP)
- File transfer (FTP)
- When reliability matters

**When to Use UDP:**
- DNS queries
- Video streaming
- Online gaming
- Real-time applications
- When speed matters more than reliability

**Key Points:**
- TCP: Reliable but slower
- UDP: Fast but unreliable
- Choose based on requirements
- Many applications use both

---

### Solution 18: TCP Connection Management

**Three-Way Handshake (Connection Establishment):**
```
Client                          Server
  |                               |
  |-------- SYN (seq=x) -------->|
  |<------ SYN-ACK (seq=y, ack=x+1) --|
  |-------- ACK (ack=y+1) ------->|
  |                               |
Connection Established
```

**Four-Way Handshake (Connection Termination):**
```
Client                          Server
  |                               |
  |-------- FIN (seq=x) -------->|
  |<------ ACK (ack=x+1) --------|
  |                               |
  |<------ FIN (seq=y) ----------|
  |-------- ACK (ack=y+1) ------->|
  |                               |
Connection Closed
```

**TCP Connection States:**
- **CLOSED**: No connection
- **LISTEN**: Server waiting for connection
- **SYN_SENT**: Client sent SYN
- **SYN_RECEIVED**: Server received SYN
- **ESTABLISHED**: Connection established
- **FIN_WAIT_1**: Client sent FIN
- **FIN_WAIT_2**: Client received ACK for FIN
- **CLOSE_WAIT**: Server received FIN
- **LAST_ACK**: Server sent FIN
- **TIME_WAIT**: Waiting for final ACK
- **CLOSED**: Connection closed

**SYN Flood Attack:**
- Attacker sends many SYN packets
- Server allocates resources for each
- Server runs out of resources
- Prevention: SYN cookies, rate limiting

**Key Points:**
- Three-way handshake establishes connection
- Four-way handshake closes connection
- States track connection lifecycle
- SYN flood is common attack

---

### Solution 19: HTTP and HTTPS

**HTTP (HyperText Transfer Protocol):**
- Application layer protocol
- Stateless (each request independent)
- Request-response model
- Port 80

**HTTP Methods:**
- **GET**: Retrieve resource
- **POST**: Submit data, create resource
- **PUT**: Update resource
- **DELETE**: Delete resource
- **PATCH**: Partial update
- **HEAD**: Get headers only
- **OPTIONS**: Get allowed methods

**HTTP Request:**
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

**HTTP Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<html>...</html>
```

**HTTPS (HTTP Secure):**
- HTTP over SSL/TLS
- Encrypted communication
- Port 443
- Prevents eavesdropping and tampering

**SSL/TLS Handshake:**
```
Client                          Server
  |                               |
  |-------- ClientHello -------->|
  |<------ ServerHello ----------|
  |<------ Certificate ----------|
  |-------- ClientKeyExchange -->|
  |<------ ChangeCipherSpec -----|
  |-------- ChangeCipherSpec ---->|
  |                               |
Encrypted Communication
```

**Key Points:**
- HTTP: Unencrypted, stateless
- HTTPS: Encrypted, secure
- SSL/TLS provides encryption
- Use HTTPS for sensitive data

---

### Solution 20: DNS Resolution Process

**DNS Hierarchy:**
```
Root (.)
  |
  +-- com, org, net, etc. (TLD)
        |
        +-- example (Domain)
              |
              +-- www, mail, etc. (Subdomain)
```

**DNS Resolution Process:**
```
1. Check local cache
2. Check hosts file
3. Query local DNS server
4. Local DNS queries root server
5. Root server refers to TLD server
6. TLD server refers to authoritative server
7. Authoritative server returns IP address
8. Response cached
```

**DNS Record Types:**
- **A**: IPv4 address
- **AAAA**: IPv6 address
- **CNAME**: Canonical name (alias)
- **MX**: Mail exchange
- **NS**: Name server
- **TXT**: Text record
- **PTR**: Pointer (reverse DNS)

**Recursive vs Iterative:**
- **Recursive**: DNS server does all queries
- **Iterative**: DNS server returns referral, client queries

**DNS Caching:**
- Responses cached at multiple levels
- TTL (Time To Live) controls cache duration
- Reduces DNS queries
- Can cause stale data

**Example:**
```
Query: www.example.com
1. Local DNS cache: Not found
2. Query root server: Refer to .com server
3. Query .com server: Refer to example.com server
4. Query example.com server: Return 192.0.2.1
5. Cache result with TTL
```

**Key Points:**
- Hierarchical name resolution
- Multiple record types
- Caching improves performance
- TTL controls cache duration

---

### Solution 21: Load Balancing Algorithms

**Load Balancing:**
- Distribute requests across multiple servers
- Improve performance and availability
- Prevent overload

**Algorithms:**

1. **Round Robin**
   - Distribute requests sequentially
   - Simple, fair
   - Doesn't consider server load
   ```python
   servers = ['server1', 'server2', 'server3']
   current = 0
   
   def get_server():
       global current
       server = servers[current]
       current = (current + 1) % len(servers)
       return server
   ```

2. **Least Connections**
   - Route to server with fewest active connections
   - Good for long-lived connections
   ```python
   def get_server():
       return min(servers, key=lambda s: s.active_connections)
   ```

3. **Weighted Round Robin**
   - Round robin with weights
   - More powerful servers get more requests
   ```python
   servers = [('server1', 3), ('server2', 2), ('server3', 1)]
   # server1 gets 3x requests
   ```

4. **Consistent Hashing**
   - Hash request to server
   - Minimal redistribution on server changes
   - Good for caching
   ```python
   import hashlib
   
   def consistent_hash(key, servers):
       hash_val = int(hashlib.md5(key.encode()).hexdigest(), 16)
       return servers[hash_val % len(servers)]
   ```

**Key Points:**
- Multiple algorithms available
- Choose based on requirements
- Round robin: Simple
- Consistent hashing: Minimal redistribution

---

### Solution 22: Network Topologies

**Network Topologies:**

1. **Bus Topology**
   - All devices connected to single cable
   - Simple, cheap
   - Single point of failure
   - Limited distance

2. **Star Topology**
   - All devices connected to central hub/switch
   - Easy to add/remove devices
   - Hub failure affects all
   - Most common LAN topology

3. **Ring Topology**
   - Devices connected in ring
   - Data travels in one direction
   - Single break affects all
   - Token ring networks

4. **Mesh Topology**
   - Every device connected to every other
   - High redundancy
   - Expensive, complex
   - Used in critical systems

5. **Tree Topology**
   - Hierarchical structure
   - Scalable
   - Root failure affects all
   - Used in large networks

**Comparison:**
```
Topology    Cost    Scalability  Fault Tolerance
Bus         Low     Limited      Poor
Star        Medium  Good         Medium
Ring        Medium  Limited      Poor
Mesh        High    Limited      Excellent
Tree        Medium  Good         Medium
```

**Key Points:**
- Different topologies for different needs
- Star most common for LANs
- Mesh for high availability
- Choose based on requirements

---

### Solution 23: Subnetting and CIDR

**Subnetting:**
- Dividing network into smaller networks
- Improves organization and security
- Reduces broadcast traffic

**Subnet Mask:**
- 32-bit number separating network and host
- 1s for network bits, 0s for host bits
- Example: 255.255.255.0 = /24

**CIDR Notation:**
- Classless Inter-Domain Routing
- Format: IP/prefix_length
- Example: 192.168.1.0/24

**Calculating Subnets:**
```
Network: 192.168.1.0/24
Subnet mask: 255.255.255.0
Network bits: 24
Host bits: 8
Hosts per subnet: 2^8 - 2 = 254 (subtract network and broadcast)
```

**Subnetting Example:**
```
Network: 192.168.1.0/24
Divide into 4 subnets:

Subnet 1: 192.168.1.0/26 (64 hosts)
Subnet 2: 192.168.1.64/26 (64 hosts)
Subnet 3: 192.168.1.128/26 (64 hosts)
Subnet 4: 192.168.1.192/26 (64 hosts)
```

**Same Subnet Check:**
```python
def same_subnet(ip1, ip2, subnet_mask):
    ip1_int = ip_to_int(ip1)
    ip2_int = ip_to_int(ip2)
    mask_int = ip_to_int(subnet_mask)
    
    return (ip1_int & mask_int) == (ip2_int & mask_int)
```

**Key Points:**
- Subnetting divides networks
- CIDR notation: IP/prefix_length
- Calculate hosts: 2^(host_bits) - 2
- Subnet mask separates network/host

---

### Solution 24: Routing Algorithms

**Routing:**
- Finding path from source to destination
- Routers maintain routing tables
- Updates based on routing algorithms

**Distance Vector Routing:**
- Each router knows distance to neighbors
- Exchanges distance vectors with neighbors
- Example: RIP (Routing Information Protocol)
- Problem: Slow convergence, count-to-infinity

**Link State Routing:**
- Each router knows entire network topology
- Uses Dijkstra's algorithm
- Example: OSPF (Open Shortest Path First)
- Faster convergence

**RIP (Distance Vector):**
```
Router A knows:
- To B: 1 hop
- To C: 2 hops (via B)

Router B knows:
- To A: 1 hop
- To C: 1 hop

Exchange updates periodically
```

**OSPF (Link State):**
```
Each router:
1. Discovers neighbors
2. Builds link state database
3. Runs Dijkstra's algorithm
4. Builds shortest path tree
5. Updates routing table
```

**Count-to-Infinity Problem:**
- In distance vector, bad news travels slowly
- Can cause routing loops
- Solved by: Split horizon, poison reverse, maximum hop count

**Key Points:**
- Distance vector: Simple, slow convergence
- Link state: Complex, fast convergence
- RIP: Distance vector
- OSPF: Link state

---

### Solution 25: Network Protocols - ARP, ICMP, DHCP

**ARP (Address Resolution Protocol):**
- Maps IP address to MAC address
- Used on local network
- Broadcast request, unicast response

**ARP Process:**
```
Host A wants to send to Host B (IP: 192.168.1.2)
1. Check ARP cache
2. If not found, broadcast ARP request
3. Host B responds with MAC address
4. Host A caches mapping
```

**ICMP (Internet Control Message Protocol):**
- Error reporting and diagnostics
- Used by ping and traceroute
- Messages: Echo Request/Reply, Destination Unreachable, Time Exceeded

**DHCP (Dynamic Host Configuration Protocol):**
- Automatically assigns IP addresses
- Four-step process: Discover, Offer, Request, Acknowledge

**DHCP Process:**
```
Client                          DHCP Server
  |                               |
  |-------- DHCP Discover ------->| (Broadcast)
  |<------ DHCP Offer ------------| (IP address offer)
  |-------- DHCP Request -------->| (Request specific IP)
  |<------ DHCP ACK --------------| (Confirmation)
  |                               |
IP Address Assigned
```

**Protocol Interactions:**
- ARP: Resolve MAC for IP
- ICMP: Error reporting
- DHCP: IP configuration
- All work together for network communication

**Key Points:**
- ARP: IP to MAC mapping
- ICMP: Error reporting
- DHCP: Automatic IP assignment
- Protocols work together

---

### Solution 26: Firewall and Network Security

**Firewall:**
- Network security device
- Filters traffic based on rules
- Protects network from unauthorized access

**Firewall Types:**

1. **Packet Filtering**
   - Examines packet headers
   - Rules based on IP, port, protocol
   - Fast but limited
   - Stateless

2. **Stateful Inspection**
   - Tracks connection state
   - More intelligent filtering
   - Slower but more secure
   - Understands context

3. **Application Layer (Proxy)**
   - Examines application data
   - Most secure but slowest
   - Can filter content

**DMZ (Demilitarized Zone):**
- Network segment between internal and external
- Public-facing servers placed here
- Additional security layer

**Firewall Rules Example:**
```
Allow: HTTP (port 80) -> Web Server
Allow: HTTPS (port 443) -> Web Server
Allow: SSH (port 22) -> Admin Server (from specific IP)
Deny: All other traffic
```

**Key Points:**
- Firewalls filter network traffic
- Different types for different needs
- DMZ provides additional security
- Rules control access

---

### Solution 27: Network Congestion Control

**Congestion Causes:**
- Too many packets in network
- Slow links
- Router buffer overflow
- Results in packet loss

**TCP Congestion Control:**

1. **Slow Start**
   - Start with small window
   - Double window each RTT
   - Exponential growth

2. **Congestion Avoidance**
   - Linear growth after threshold
   - Additive increase

3. **Fast Retransmit**
   - Retransmit on 3 duplicate ACKs
   - Don't wait for timeout

4. **Fast Recovery**
   - Reduce window by half
   - Continue congestion avoidance

**TCP Congestion Control States:**
```
Slow Start: cwnd = 1, doubles each RTT
Congestion Avoidance: cwnd += 1 each RTT
Fast Retransmit: On 3 duplicate ACKs
Fast Recovery: cwnd = cwnd/2, then additive increase
```

**Handling Packet Loss:**
- Timeout: Slow start
- Duplicate ACKs: Fast retransmit
- Reduce sending rate
- Gradually increase

**Key Points:**
- Congestion causes packet loss
- TCP adjusts sending rate
- Slow start + congestion avoidance
- Fast retransmit improves recovery

---

### Solution 28: WebSocket vs HTTP

**WebSocket:**
- Full-duplex communication
- Persistent connection
- Low overhead after handshake
- Real-time bidirectional

**HTTP:**
- Request-response model
- Stateless
- New connection per request
- Higher overhead

**WebSocket Handshake:**
```
Client                          Server
  |                               |
  |-------- HTTP Upgrade Request ->|
  |<------ HTTP 101 Switching -----|
  |         Protocols              |
  |                               |
WebSocket Connection Established
```

**When to Use WebSocket:**
- Real-time chat
- Live updates
- Gaming
- Collaborative editing
- When bidirectional needed

**When to Use HTTP:**
- REST APIs
- Traditional web pages
- File downloads
- When request-response sufficient

**Key Points:**
- WebSocket: Persistent, bidirectional
- HTTP: Request-response, stateless
- Choose based on communication pattern
- WebSocket for real-time, HTTP for traditional

---

### Solution 29: CDN and Content Distribution

**CDN (Content Delivery Network):**
- Distributed servers worldwide
- Cache content close to users
- Reduces latency
- Improves performance

**How CDN Works:**
```
User Request -> DNS -> Nearest CDN Server
                |
                v
         Cache Hit? -> Return Content
                |
                v
         Cache Miss -> Origin Server -> Cache -> Return
```

**Edge Server Placement:**
- Geographic distribution
- Close to users
- Reduces latency
- Improves performance

**Caching Strategies:**
- Cache static content
- TTL-based expiration
- Cache invalidation
- Origin server fallback

**Benefits:**
- Reduced latency
- Reduced bandwidth costs
- Improved availability
- Better user experience

**Key Points:**
- CDN distributes content globally
- Edge servers cache content
- Reduces latency and bandwidth
- Improves performance

---

### Solution 30: Network Monitoring and Troubleshooting

**Diagnostic Tools:**

1. **ping**
   - Test connectivity
   - Measure latency
   - ICMP echo request/reply

2. **traceroute**
   - Show path to destination
   - Identify bottlenecks
   - TTL-based discovery

3. **netstat**
   - Show network connections
   - Show listening ports
   - Network statistics

4. **tcpdump/Wireshark**
   - Packet capture
   - Analyze network traffic
   - Debug protocols

**HTTP Status Codes:**
- **2xx**: Success (200 OK)
- **3xx**: Redirection (301 Moved)
- **4xx**: Client Error (404 Not Found)
- **5xx**: Server Error (500 Internal Error)

**Troubleshooting Steps:**
1. Check connectivity (ping)
2. Check DNS resolution
3. Check routing (traceroute)
4. Check firewall rules
5. Analyze packets (tcpdump)

**Key Points:**
- Multiple tools available
- Systematic troubleshooting
- Check connectivity, DNS, routing
- Use packet analysis for deep debugging

---

## Database Management Systems Solutions

### Solution 31: ACID Properties

**ACID Properties:**

1. **Atomicity**
   - All or nothing
   - Transaction either completes or fails
   - No partial execution
   - Implemented using undo log

2. **Consistency**
   - Database remains in valid state
   - Constraints maintained
   - Before and after transaction valid

3. **Isolation**
   - Concurrent transactions don't interfere
   - Each transaction sees consistent snapshot
   - Implemented using locking/MVCC

4. **Durability**
   - Committed changes persist
   - Survives system failures
   - Implemented using write-ahead log

**Atomicity Example:**
```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both succeed or both fail
```

**Consistency:**
- Database constraints enforced
- Referential integrity maintained
- Business rules followed

**Isolation:**
- Transactions isolated from each other
- Different isolation levels
- Prevents dirty reads, phantom reads

**Durability:**
- Changes written to disk
- Transaction log ensures durability
- Survives crashes

**Key Points:**
- ACID ensures reliable transactions
- Atomicity: All or nothing
- Consistency: Valid state
- Isolation: Concurrent transactions
- Durability: Persistence

---

### Solution 32: Database Normalization

**Normalization:**
- Eliminate redundancy
- Reduce anomalies
- Improve data integrity
- Trade-off: More joins, better integrity

**Normal Forms:**

1. **1NF (First Normal Form)**
   - Atomic values
   - No repeating groups
   - Each cell single value

2. **2NF (Second Normal Form)**
   - 1NF + no partial dependencies
   - All non-key attributes depend on full key

3. **3NF (Third Normal Form)**
   - 2NF + no transitive dependencies
   - Non-key attributes don't depend on other non-key attributes

**Example:**
```
Unnormalized:
Orders (OrderID, CustomerName, ProductID, ProductName, Quantity, Price)

1NF:
Orders (OrderID, CustomerName)
OrderItems (OrderID, ProductID, ProductName, Quantity, Price)

2NF:
Orders (OrderID, CustomerName)
Products (ProductID, ProductName, Price)
OrderItems (OrderID, ProductID, Quantity)

3NF:
Orders (OrderID, CustomerID)
Customers (CustomerID, CustomerName)
Products (ProductID, ProductName, Price)
OrderItems (OrderID, ProductID, Quantity)
```

**When to Denormalize:**
- Performance critical
- Read-heavy workloads
- Acceptable redundancy
- Reduce join overhead

**Key Points:**
- Normalization reduces redundancy
- 1NF, 2NF, 3NF eliminate different anomalies
- Trade-off: Integrity vs Performance
- Denormalize for performance

---

### Solution 33: Indexing Strategies

**Index:**
- Data structure improving query speed
- Like book index
- Speeds up SELECT queries
- Slows down INSERT/UPDATE/DELETE

**B-Tree Index:**
- Balanced tree structure
- O(log n) search time
- Good for range queries
- Most common index type

**Hash Index:**
- Hash table structure
- O(1) average search time
- Only equality queries
- Not good for ranges

**When to Create Index:**
- Frequently queried columns
- Foreign keys
- Columns in WHERE clause
- Columns in JOIN conditions

**Composite Index:**
- Index on multiple columns
- Order matters
- Useful for multi-column queries
- Example: (last_name, first_name)

**Index Trade-offs:**
- ✅ Faster SELECT queries
- ✅ Faster JOINs
- ❌ Slower INSERT/UPDATE/DELETE
- ❌ Additional storage space

**Key Points:**
- Indexes speed up queries
- B-tree: General purpose
- Hash: Equality only
- Balance query speed vs write performance

---

### Solution 34: SQL Joins

**Join Types:**

1. **INNER JOIN**
   - Returns matching rows
   - Most common join
   ```sql
   SELECT * FROM A INNER JOIN B ON A.id = B.id;
   ```

2. **LEFT JOIN (LEFT OUTER JOIN)**
   - Returns all rows from left table
   - NULL for non-matching right rows
   ```sql
   SELECT * FROM A LEFT JOIN B ON A.id = B.id;
   ```

3. **RIGHT JOIN (RIGHT OUTER JOIN)**
   - Returns all rows from right table
   - NULL for non-matching left rows

4. **FULL OUTER JOIN**
   - Returns all rows from both tables
   - NULL for non-matching rows

**Join Performance:**
- Use indexes on join columns
- Smaller table first in join
- Avoid unnecessary joins
- Use WHERE to filter early

**Example:**
```sql
-- Employees and Departments
SELECT e.name, d.name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- All employees, even without department
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
```

**Key Points:**
- INNER: Matching rows only
- LEFT: All left rows
- RIGHT: All right rows
- FULL: All rows from both
- Optimize with indexes

---

### Solution 35: Transaction Isolation Levels

**Isolation Levels:**

1. **READ UNCOMMITTED**
   - Lowest isolation
   - Dirty reads possible
   - No locks

2. **READ COMMITTED**
   - No dirty reads
   - Non-repeatable reads possible
   - Locks held during read

3. **REPEATABLE READ**
   - No dirty reads
   - No non-repeatable reads
   - Phantom reads possible
   - Locks held for transaction

4. **SERIALIZABLE**
   - Highest isolation
   - No anomalies
   - Serial execution
   - Highest overhead

**Concurrency Problems:**

1. **Dirty Read**
   - Read uncommitted data
   - Prevented by READ COMMITTED+

2. **Non-Repeatable Read**
   - Different values on re-read
   - Prevented by REPEATABLE READ+

3. **Phantom Read**
   - New rows appear
   - Prevented by SERIALIZABLE

**Choosing Isolation Level:**
- Balance consistency vs performance
- Higher isolation = more locking = slower
- Lower isolation = faster but anomalies possible

**Key Points:**
- Four isolation levels
- Trade-off: Consistency vs Performance
- Higher isolation prevents more anomalies
- Choose based on requirements

---

### Solution 36: Database Locking

**Lock Types:**

1. **Shared Lock (Read Lock)**
   - Multiple readers allowed
   - No writers allowed
   - Used for SELECT

2. **Exclusive Lock (Write Lock)**
   - No other locks allowed
   - Used for INSERT/UPDATE/DELETE

**Lock Granularity:**
- Row-level: Most granular, best concurrency
- Page-level: Medium granularity
- Table-level: Coarsest, worst concurrency

**Deadlock Prevention:**
- Lock ordering
- Timeout
- Deadlock detection
- Transaction rollback

**Lock Escalation:**
- Many row locks -> table lock
- Reduces lock overhead
- Reduces concurrency
- Automatic by database

**Example:**
```sql
-- Shared lock
SELECT * FROM accounts WHERE id = 1;  -- Shared lock

-- Exclusive lock
UPDATE accounts SET balance = 100 WHERE id = 1;  -- Exclusive lock
```

**Key Points:**
- Shared: Multiple readers
- Exclusive: Single writer
- Row-level: Best concurrency
- Deadlocks prevented/detected

---

### Solution 37: Query Optimization

**Query Optimization Techniques:**

1. **Use Indexes**
   - Index frequently queried columns
   - Index join columns
   - Index WHERE clause columns

2. **Avoid SELECT ***
   - Select only needed columns
   - Reduces data transfer

3. **Use WHERE Early**
   - Filter before joins
   - Reduce dataset size

4. **Avoid Functions in WHERE**
   - Prevents index usage
   - Example: WHERE UPPER(name) = 'JOHN'

5. **Use LIMIT**
   - Limit result set
   - Stop early when possible

**Execution Plan Analysis:**
```sql
EXPLAIN SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'USA';
```

**Common Optimizations:**
- Index missing columns
- Rewrite subqueries as joins
- Use EXISTS instead of IN for large sets
- Avoid correlated subqueries

**Key Points:**
- Analyze execution plans
- Use indexes effectively
- Filter early
- Avoid unnecessary operations

---

### Solution 38: Database Replication

**Replication Types:**

1. **Master-Slave (Primary-Replica)**
   - One master, multiple slaves
   - Master handles writes
   - Slaves handle reads
   - Simple, common

2. **Master-Master**
   - Multiple masters
   - All handle writes
   - More complex
   - Conflict resolution needed

**Replication Lag:**
- Delay between master and slave
- Causes: Network, load, processing
- Problem: Stale reads
- Solution: Monitor lag, read from master for critical reads

**Consistency:**
- Synchronous: Wait for all replicas (slow, consistent)
- Asynchronous: Don't wait (fast, eventual consistency)

**Use Cases:**
- High availability
- Read scaling
- Geographic distribution
- Backup

**Key Points:**
- Replication provides redundancy
- Master-slave: Simple, common
- Replication lag is common
- Choose sync vs async based on requirements

---

### Solution 39: CAP Theorem and Database Consistency

**CAP Theorem:**
- Can only guarantee 2 of 3:
  - **Consistency**: All nodes see same data
  - **Availability**: System remains operational
  - **Partition Tolerance**: System continues despite network partitions

**Trade-offs:**

1. **CP (Consistency + Partition Tolerance)**
   - SQL databases
   - Strong consistency
   - May sacrifice availability

2. **AP (Availability + Partition Tolerance)**
   - NoSQL databases (DynamoDB, Cassandra)
   - High availability
   - Eventual consistency

3. **CA (Consistency + Availability)**
   - Not possible in distributed systems
   - Single-node systems only

**Consistency Models:**

1. **Strong Consistency**
   - All reads see latest write
   - ACID transactions
   - SQL databases

2. **Eventual Consistency**
   - Eventually all nodes consistent
   - NoSQL databases
   - Acceptable for many use cases

**Choosing Database:**
- **SQL**: Strong consistency needed
- **NoSQL**: Scale, eventual consistency OK

**Key Points:**
- CAP: Can only guarantee 2 of 3
- SQL: CP (Consistency + Partition)
- NoSQL: AP (Availability + Partition)
- Choose based on requirements

---

### Solution 40: Database Sharding

**Sharding:**
- Horizontal partitioning
- Split data across multiple databases
- Improves scalability
- Each shard independent

**Sharding Strategies:**

1. **Range Sharding**
   - Shard by value ranges
   - Example: UserID 1-1000 -> Shard1, 1001-2000 -> Shard2
   - Simple but can hotspot

2. **Hash Sharding**
   - Shard by hash of key
   - Even distribution
   - Example: hash(user_id) % num_shards
   - Good distribution

3. **Directory Sharding**
   - Lookup table for shard
   - Flexible
   - Single point of failure

**Sharding Challenges:**
- Cross-shard queries
- Rebalancing
- Joins across shards
- Transaction management

**Example:**
```python
def get_shard(user_id, num_shards):
    return hash(user_id) % num_shards

# Query specific shard
shard = get_shard(user_id, 4)
query_shard(shard, user_id)
```

**Key Points:**
- Sharding splits data horizontally
- Hash sharding: Even distribution
- Cross-shard queries expensive
- Rebalancing complex

---

### Solution 41: NoSQL Database Types

**NoSQL Types:**

1. **Document Databases**
   - Store documents (JSON, BSON)
   - Example: MongoDB, CouchDB
   - Flexible schema
   - Good for content management

2. **Key-Value Stores**
   - Simple key-value pairs
   - Example: Redis, DynamoDB
   - Fast, simple
   - Good for caching

3. **Column-Family Stores**
   - Store columns together
   - Example: Cassandra, HBase
   - Good for analytics
   - Wide tables

4. **Graph Databases**
   - Store relationships
   - Example: Neo4j
   - Good for social networks
   - Complex queries

**When to Use:**
- **Document**: Flexible schema, JSON data
- **Key-Value**: Simple, fast, caching
- **Column-Family**: Analytics, time-series
- **Graph**: Relationships, social networks

**Key Points:**
- Different types for different needs
- Document: Flexible
- Key-Value: Simple
- Column-Family: Analytics
- Graph: Relationships

---

### Solution 42: Database Backup and Recovery

**Backup Types:**

1. **Full Backup**
   - Complete database copy
   - Slow, large
   - Foundation for recovery

2. **Incremental Backup**
   - Only changes since last backup
   - Fast, small
   - Need full + all incrementals

3. **Differential Backup**
   - Changes since last full backup
   - Medium speed, size
   - Need full + latest differential

**Recovery:**
- Restore full backup
- Apply incremental/differential backups
- Apply transaction log
- Point-in-time recovery

**Checkpoint:**
- Flush dirty pages to disk
- Update checkpoint record
- Speeds up recovery
- Reduces log replay

**Backup Strategy:**
- Full backup: Weekly
- Incremental: Daily
- Transaction log: Continuous
- Test recovery regularly

**Key Points:**
- Regular backups essential
- Full + incremental common
- Test recovery procedures
- Point-in-time recovery possible

---

### Solution 43: Database Connection Pooling

**Connection Pooling:**
- Reuse database connections
- Avoid connection overhead
- Limit concurrent connections
- Improve performance

**Pool Configuration:**
- Min connections: Always available
- Max connections: Maximum allowed
- Idle timeout: Close idle connections
- Connection timeout: Wait time for connection

**Benefits:**
- Faster (reuse connections)
- Resource efficient
- Connection limit enforcement
- Better performance

**Challenges:**
- Pool exhaustion
- Stale connections
- Connection leaks
- Proper sizing

**Example:**
```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20,
    pool_timeout=30
)
```

**Key Points:**
- Reuse connections
- Configure pool size
- Monitor pool usage
- Handle pool exhaustion

---

### Solution 44: Database Views and Materialized Views

**Views:**
- Virtual table
- Based on query
- No storage
- Always current

**Materialized Views:**
- Physical storage
- Pre-computed results
- Faster queries
- Need refresh

**View Example:**
```sql
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active';

SELECT * FROM active_users;  -- Always current
```

**Materialized View Example:**
```sql
CREATE MATERIALIZED VIEW user_stats AS
SELECT COUNT(*) as total_users, AVG(age) as avg_age
FROM users;

REFRESH MATERIALIZED VIEW user_stats;  -- Update manually
```

**When to Use:**
- **Views**: Simplify queries, always current
- **Materialized Views**: Expensive queries, can tolerate stale data

**Key Points:**
- Views: Virtual, always current
- Materialized Views: Physical, faster, need refresh
- Choose based on freshness vs performance

---

### Solution 45: Database Triggers and Stored Procedures

**Triggers:**
- Automatic actions on events
- INSERT, UPDATE, DELETE
- BEFORE or AFTER
- Enforce business rules

**Trigger Example:**
```sql
CREATE TRIGGER update_timestamp
BEFORE UPDATE ON orders
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END;
```

**Stored Procedures:**
- Pre-compiled SQL code
- Reusable logic
- Better performance
- Business logic in database

**Stored Procedure Example:**
```sql
CREATE PROCEDURE transfer_funds(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL
)
BEGIN
    START TRANSACTION;
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;
    COMMIT;
END;
```

**When to Use:**
- **Triggers**: Automatic actions, audit trails
- **Stored Procedures**: Complex logic, performance

**Key Points:**
- Triggers: Automatic on events
- Stored Procedures: Reusable logic
- Trade-off: Database vs application logic
- Performance vs maintainability

---

### Solution 46: Database Security

**Security Measures:**

1. **Authentication**
   - Verify user identity
   - Passwords, certificates
   - Multi-factor authentication

2. **Authorization**
   - Control access
   - Roles and permissions
   - Principle of least privilege

3. **Encryption**
   - Data at rest: Encrypt database files
   - Data in transit: SSL/TLS
   - Encrypt sensitive columns

4. **Auditing**
   - Log access
   - Track changes
   - Compliance

**SQL Injection Prevention:**
```python
# Vulnerable
query = f"SELECT * FROM users WHERE name = '{user_input}'"

# Safe - Parameterized queries
query = "SELECT * FROM users WHERE name = ?"
cursor.execute(query, (user_input,))
```

**Key Points:**
- Authentication: Who you are
- Authorization: What you can do
- Encrypt sensitive data
- Prevent SQL injection

---

### Solution 47: Database Transactions and Concurrency

**Concurrency Control:**

1. **Locking**
   - Pessimistic approach
   - Lock before access
   - Prevents conflicts

2. **MVCC (Multi-Version Concurrency Control)**
   - Optimistic approach
   - Multiple versions
   - No blocking reads

**Lost Update Problem:**
```
Transaction 1: Read balance = 100
Transaction 2: Read balance = 100
Transaction 1: Write balance = 150
Transaction 2: Write balance = 120  -- Lost update!
```

**Prevention:**
- Locking: Exclusive lock
- MVCC: Version checking
- Optimistic locking: Check version before update

**MVCC Example:**
- Each row has version number
- Read sees consistent snapshot
- Write checks version
- Conflict if version changed

**Key Points:**
- Locking: Pessimistic
- MVCC: Optimistic
- Prevent lost updates
- Choose based on workload

---

### Solution 48: Database Index Types

**Index Types:**

1. **B-Tree Index**
   - Most common
   - Balanced tree
   - Good for ranges
   - General purpose

2. **Hash Index**
   - Hash table
   - Equality only
   - Very fast
   - Limited use

3. **Bitmap Index**
   - Bitmap per value
   - Good for low cardinality
   - Fast AND/OR operations
   - Data warehouse

4. **Full-Text Index**
   - Text search
   - Inverted index
   - Good for documents
   - LIKE queries

**When to Use:**
- **B-Tree**: General purpose, ranges
- **Hash**: Equality queries only
- **Bitmap**: Low cardinality, analytics
- **Full-Text**: Text search

**Key Points:**
- B-Tree: Most common
- Hash: Equality only
- Bitmap: Analytics
- Full-Text: Text search

---

### Solution 49: Database Partitioning

**Partitioning Types:**

1. **Range Partitioning**
   - Partition by value ranges
   - Example: By date, by ID ranges
   - Good for time-series

2. **Hash Partitioning**
   - Partition by hash
   - Even distribution
   - Good for load balancing

3. **List Partitioning**
   - Partition by list of values
   - Example: By region, by status
   - Good for categories

**Partition Pruning:**
- Query optimizer eliminates partitions
- Only scans relevant partitions
- Improves performance
- Requires partition key in WHERE

**Example:**
```sql
CREATE TABLE orders (
    id INT,
    order_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);

-- Only scans p2023 partition
SELECT * FROM orders WHERE order_date = '2023-06-15';
```

**Key Points:**
- Partitioning splits large tables
- Range: Time-series
- Hash: Even distribution
- Partition pruning improves performance

---

### Solution 50: Database Performance Tuning

**Performance Tuning:**

1. **Identify Bottlenecks**
   - Slow queries
   - High CPU usage
   - High I/O
   - Lock contention

2. **Optimize Queries**
   - Use indexes
   - Rewrite queries
   - Avoid N+1 queries
   - Use EXPLAIN

3. **Tune Configuration**
   - Buffer pool size
   - Connection limits
   - Query cache
   - Log settings

4. **Monitor Metrics**
   - Query execution time
   - Cache hit ratio
   - Connection pool usage
   - Lock waits

**Common Optimizations:**
- Add missing indexes
- Increase buffer pool
- Optimize queries
- Partition large tables
- Archive old data

**Key Points:**
- Identify bottlenecks first
- Optimize queries
- Tune configuration
- Monitor continuously

---

## Summary

This comprehensive guide covers all major CS fundamentals topics for SDE 2 interviews:

1. **Operating Systems**: Processes, threads, scheduling, memory management, IPC, synchronization, file systems, disk scheduling
2. **Networking**: OSI/TCP-IP, protocols (TCP/UDP/HTTP), DNS, load balancing, routing, security, CDN
3. **Database Management Systems**: ACID, normalization, indexing, transactions, replication, sharding, NoSQL, security, performance

**Total Coverage: 50 Problems with Detailed Solutions**

Each solution includes explanations, examples, code snippets, and key points. Use these as a reference for interview preparation and real-world implementations.
