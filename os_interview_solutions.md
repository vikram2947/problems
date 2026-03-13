# Operating Systems Interview Solutions for SDE 2

## Table of Contents
1. [Process Management Solutions](#process-management-solutions)
2. [Thread Management Solutions](#thread-management-solutions)
3. [CPU Scheduling Solutions](#cpu-scheduling-solutions)
4. [Memory Management Solutions](#memory-management-solutions)
5. [Virtual Memory Solutions](#virtual-memory-solutions)
6. [File Systems Solutions](#file-systems-solutions)
7. [I/O Systems Solutions](#io-systems-solutions)
8. [Synchronization Solutions](#synchronization-solutions)
9. [Deadlocks Solutions](#deadlocks-solutions)
10. [Inter-Process Communication Solutions](#inter-process-communication-solutions)
11. [System Calls Solutions](#system-calls-solutions)
12. [Kernel Architecture Solutions](#kernel-architecture-solutions)
13. [Process Synchronization Solutions](#process-synchronization-solutions)
14. [Memory Allocation Solutions](#memory-allocation-solutions)
15. [Advanced Topics Solutions](#advanced-topics-solutions)

---

## Process Management Solutions

### Solution 1: Process Creation and Termination

**How fork() Works:**
- Creates exact copy of calling process
- Returns twice: once in parent (returns child PID), once in child (returns 0)
- Child gets copy of parent's memory space
- Both processes continue execution from after fork()

**Example:**
```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        printf("Child PID: %d\n", getpid());
    } else if (pid > 0) {
        // Parent process
        printf("Parent PID: %d, Child PID: %d\n", getpid(), pid);
    } else {
        // Error
        perror("fork failed");
    }
    return 0;
}
```

**fork() vs exec():**
- **fork()**: Creates new process (copy)
- **exec()**: Replaces current process image with new program
- Often used together: fork() to create, exec() to run new program

**Process Termination:**
- Process calls exit() or returns from main()
- OS cleans up resources (memory, file descriptors)
- Exit status sent to parent
- Parent can get status via wait() or waitpid()

**Zombie Process:**
- Process that has terminated but parent hasn't read exit status
- Still has entry in process table
- Prevented by: parent calling wait()/waitpid(), or using signal handler for SIGCHLD

**Key Points:**
- fork() creates process copy
- exec() replaces process image
- Parent must wait() to prevent zombies
- Process hierarchy maintained via parent-child relationships

---

### Solution 2: Process States and State Transitions

**Five Process States:**
1. **New**: Process being created
2. **Ready**: Ready to run, waiting for CPU
3. **Running**: Currently executing on CPU
4. **Waiting/Blocked**: Waiting for event (I/O, signal)
5. **Terminated**: Finished execution

**State Transitions:**
```
New → Ready: Process admitted to system
Ready → Running: CPU scheduler selects process
Running → Ready: Time slice expired (preemption)
Running → Waiting: Process waits for I/O or event
Waiting → Ready: I/O complete or event occurred
Running → Terminated: Process completes or killed
```

**Implementation:**
```c
typedef enum {
    NEW,
    READY,
    RUNNING,
    WAITING,
    TERMINATED
} ProcessState;

typedef struct {
    int pid;
    ProcessState state;
    // other fields
} Process;

void transition_state(Process *p, ProcessState new_state) {
    p->state = new_state;
    // Handle state-specific actions
}
```

**Key Points:**
- Five main states
- Transitions based on events
- Ready vs Running: Ready is in queue, Running is executing
- Blocking improves CPU utilization

---

### Solution 3: Process Control Block (PCB)

**PCB Contents:**
- Process state (New, Ready, Running, etc.)
- Program counter (next instruction)
- CPU registers (when not running)
- CPU scheduling information (priority, queues)
- Memory management information (page tables, limits)
- Accounting information (CPU time, I/O)
- I/O status information (open files, devices)

**PCB Structure:**
```c
typedef struct pcb {
    int pid;
    ProcessState state;
    int priority;
    int program_counter;
    int registers[16];
    MemoryInfo memory_info;
    FileDescriptor *open_files;
    struct pcb *next;  // For queues
} PCB;
```

**Context Switching:**
- Save current process PCB (registers, state)
- Load next process PCB
- Switch to new process context
- Overhead: ~1-1000 microseconds

**Key Points:**
- PCB stores all process information
- Essential for context switching
- OS maintains PCB for each process
- PCB destroyed when process terminates

---

### Solution 4: Process Scheduling Algorithms

**FCFS (First Come First Served):**
```python
def fcfs_scheduling(processes):
    processes.sort(key=lambda x: x.arrival_time)
    current_time = 0
    for p in processes:
        p.waiting_time = current_time - p.arrival_time
        current_time += p.burst_time
        p.turnaround_time = current_time - p.arrival_time
    return processes
```

**SJF (Shortest Job First):**
```python
def sjf_scheduling(processes):
    processes.sort(key=lambda x: (x.arrival_time, x.burst_time))
    current_time = 0
    ready_queue = []
    
    while processes or ready_queue:
        # Add arrived processes
        while processes and processes[0].arrival_time <= current_time:
            ready_queue.append(processes.pop(0))
        
        if ready_queue:
            # Select shortest job
            ready_queue.sort(key=lambda x: x.burst_time)
            p = ready_queue.pop(0)
            p.waiting_time = current_time - p.arrival_time
            current_time += p.burst_time
            p.turnaround_time = current_time - p.arrival_time
        else:
            current_time = processes[0].arrival_time
    
    return processes
```

**Round Robin:**
```python
def round_robin(processes, time_quantum):
    queue = processes.copy()
    current_time = 0
    waiting_times = {p.pid: 0 for p in processes}
    last_run_time = {p.pid: p.arrival_time for p in processes}
    
    while queue:
        p = queue.pop(0)
        if p.remaining_time > time_quantum:
            waiting_times[p.pid] += current_time - last_run_time[p.pid]
            current_time += time_quantum
            p.remaining_time -= time_quantum
            last_run_time[p.pid] = current_time
            queue.append(p)
        else:
            waiting_times[p.pid] += current_time - last_run_time[p.pid]
            current_time += p.remaining_time
            p.turnaround_time = current_time - p.arrival_time
    
    return processes
```

**Key Points:**
- FCFS: Simple but can cause convoy effect
- SJF: Optimal for minimizing waiting time
- Round Robin: Good for time-sharing, prevents starvation
- Preemptive allows better responsiveness

---

### Solution 5: Multilevel Queue Scheduling

**Implementation:**
```python
class MultilevelQueue:
    def __init__(self):
        self.system_queue = []  # High priority, RR
        self.interactive_queue = []  # Medium priority, RR
        self.batch_queue = []  # Low priority, FCFS
    
    def schedule(self):
        # Check system queue first
        if self.system_queue:
            return self.round_robin(self.system_queue, quantum=10)
        elif self.interactive_queue:
            return self.round_robin(self.interactive_queue, quantum=20)
        elif self.batch_queue:
            return self.fcfs(self.batch_queue)
        return None
```

**Aging:**
- Increase priority of waiting processes
- Prevents starvation
- Move processes to higher priority queues after waiting

**Key Points:**
- Multiple queues with different priorities
- Each queue can have different algorithm
- Higher priority queues served first
- Aging prevents starvation

---

## Thread Management Solutions

### Solution 7: Thread Creation and Management

**Thread Creation (Pthreads):**
```c
#include <pthread.h>

void* thread_function(void* arg) {
    int thread_id = *(int*)arg;
    printf("Thread %d running\n", thread_id);
    return NULL;
}

int main() {
    pthread_t threads[5];
    int ids[5];
    
    for (int i = 0; i < 5; i++) {
        ids[i] = i;
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    return 0;
}
```

**Thread vs Process:**
- Threads share memory space (heap, data segment)
- Processes have separate memory spaces
- Threads have lower creation overhead
- Threads need synchronization for shared data

**Thread-Local Storage:**
```c
__thread int thread_local_var;  // Each thread has own copy

void* thread_func(void* arg) {
    thread_local_var = 42;  // Independent per thread
    return NULL;
}
```

**Key Points:**
- Threads share memory, processes don't
- Lower overhead for threads
- Need synchronization for shared data
- Thread-local storage for per-thread data

---

### Solution 8: User-Level vs Kernel-Level Threads

**User-Level Threads:**
- Managed by user-level library
- OS unaware of threads
- Fast creation/switching
- Blocking one blocks all

**Kernel-Level Threads:**
- Managed by OS kernel
- OS aware of each thread
- Slower creation/switching
- Blocking one doesn't block others

**Many-to-Many Model:**
- Multiple user threads map to multiple kernel threads
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

### Solution 9: Thread Synchronization Primitives

**Mutex:**
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

void* increment(void* arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
```

**Semaphore:**
```c
#include <semaphore.h>

sem_t semaphore;

void* worker(void* arg) {
    sem_wait(&semaphore);  // Decrement
    // Critical section
    sem_post(&semaphore);  // Increment
    return NULL;
}
```

**Condition Variables:**
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition = PTHREAD_COND_INITIALIZER;
int data_ready = 0;

void* consumer(void* arg) {
    pthread_mutex_lock(&mutex);
    while (!data_ready) {
        pthread_cond_wait(&condition, &mutex);
    }
    // Process data
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void* producer(void* arg) {
    pthread_mutex_lock(&mutex);
    data_ready = 1;
    pthread_cond_signal(&condition);
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

**Key Points:**
- Mutex: Binary lock
- Semaphore: Counting resource
- Condition variables: Wait for conditions
- Use appropriate primitive for scenario

---

### Solution 10: Thread Pools

**Implementation:**
```python
import threading
import queue

class ThreadPool:
    def __init__(self, num_threads):
        self.task_queue = queue.Queue()
        self.threads = []
        self.shutdown = False
        
        for _ in range(num_threads):
            thread = threading.Thread(target=self._worker)
            thread.start()
            self.threads.append(thread)
    
    def _worker(self):
        while not self.shutdown:
            try:
                task = self.task_queue.get(timeout=1)
                task()
                self.task_queue.task_done()
            except queue.Empty:
                continue
    
    def submit(self, task):
        self.task_queue.put(task)
    
    def shutdown_pool(self):
        self.shutdown = True
        for thread in self.threads:
            thread.join()
```

**Optimal Pool Size:**
- CPU-bound: Number of CPU cores
- I/O-bound: Higher (2-4x cores)
- Consider: Task characteristics, system load

**Key Points:**
- Reuse threads efficiently
- Queue tasks for execution
- Size based on workload type
- Proper shutdown important

---

## CPU Scheduling Solutions

### Solution 11: CPU Scheduling Metrics

**Metrics:**
- **Turnaround Time**: Completion time - Arrival time
- **Waiting Time**: Turnaround time - Burst time
- **Response Time**: First response time - Arrival time
- **Throughput**: Number of processes completed per time unit
- **CPU Utilization**: Percentage of time CPU is busy

**Calculation Example:**
```python
def calculate_metrics(processes):
    for p in processes:
        p.turnaround_time = p.completion_time - p.arrival_time
        p.waiting_time = p.turnaround_time - p.burst_time
        p.response_time = p.first_response_time - p.arrival_time
    
    avg_turnaround = sum(p.turnaround_time for p in processes) / len(processes)
    avg_waiting = sum(p.waiting_time for p in processes) / len(processes)
    throughput = len(processes) / max(p.completion_time for p in processes)
    
    return avg_turnaround, avg_waiting, throughput
```

**Key Points:**
- Different metrics for different goals
- Turnaround: Total time
- Response: Time to first response
- Throughput: Processes per time unit

---

### Solution 12: Shortest Job First (SJF) Scheduling

**Non-Preemptive SJF:**
```python
def sjf_non_preemptive(processes):
    processes.sort(key=lambda x: (x.arrival_time, x.burst_time))
    current_time = 0
    completed = []
    
    while processes:
        # Find processes that have arrived
        available = [p for p in processes if p.arrival_time <= current_time]
        
        if available:
            # Select shortest job
            shortest = min(available, key=lambda x: x.burst_time)
            processes.remove(shortest)
            shortest.waiting_time = current_time - shortest.arrival_time
            current_time += shortest.burst_time
            shortest.turnaround_time = current_time - shortest.arrival_time
            completed.append(shortest)
        else:
            current_time = processes[0].arrival_time
    
    return completed
```

**Preemptive SJF (SRTF):**
- Can preempt running process if shorter job arrives
- More complex but better average waiting time

**Burst Time Estimation:**
- Use exponential average: τ(n+1) = α * t(n) + (1-α) * τ(n)
- α determines weight of recent vs historical

**Key Points:**
- SJF optimal for minimizing waiting time
- Requires burst time knowledge
- Preemptive version (SRTF) better
- Estimate burst time using exponential average

---

### Solution 13: Round Robin Scheduling

**Implementation:**
```python
def round_robin(processes, time_quantum):
    queue = []
    current_time = 0
    completed = []
    waiting_times = {p.pid: 0 for p in processes}
    last_run = {p.pid: p.arrival_time for p in processes}
    
    processes.sort(key=lambda x: x.arrival_time)
    i = 0
    
    while queue or i < len(processes):
        # Add arrived processes
        while i < len(processes) and processes[i].arrival_time <= current_time:
            queue.append(processes[i])
            i += 1
        
        if queue:
            p = queue.pop(0)
            waiting_times[p.pid] += current_time - last_run[p.pid]
            
            if p.remaining_time > time_quantum:
                current_time += time_quantum
                p.remaining_time -= time_quantum
                last_run[p.pid] = current_time
                queue.append(p)
            else:
                current_time += p.remaining_time
                p.turnaround_time = current_time - p.arrival_time
                p.waiting_time = waiting_times[p.pid]
                completed.append(p)
        else:
            current_time = processes[i].arrival_time
    
    return completed
```

**Time Quantum Selection:**
- Too small: High context switching overhead
- Too large: Approaches FCFS
- Optimal: 10-100ms typically
- Rule: 80% of CPU bursts should be less than quantum

**Key Points:**
- Fair scheduling algorithm
- Prevents starvation
- Time quantum critical
- Good for time-sharing systems

---

### Solution 14: Priority Scheduling

**Implementation:**
```python
def priority_scheduling(processes, preemptive=True):
    processes.sort(key=lambda x: (x.arrival_time, x.priority))
    current_time = 0
    completed = []
    ready_queue = []
    
    while processes or ready_queue:
        # Add arrived processes
        while processes and processes[0].arrival_time <= current_time:
            ready_queue.append(processes.pop(0))
        
        if ready_queue:
            ready_queue.sort(key=lambda x: x.priority)
            p = ready_queue[0]
            
            if preemptive and len(ready_queue) > 1:
                # Check if higher priority arrived
                if ready_queue[1].priority < p.priority:
                    p = ready_queue.pop(1)
                else:
                    p = ready_queue.pop(0)
            else:
                p = ready_queue.pop(0)
            
            p.waiting_time = current_time - p.arrival_time
            current_time += p.burst_time
            p.turnaround_time = current_time - p.arrival_time
            completed.append(p)
        else:
            current_time = processes[0].arrival_time
    
    return completed
```

**Priority Inversion:**
- Low priority holds lock needed by high priority
- Solution: Priority inheritance (temporarily raise priority)

**Priority Aging:**
- Increase priority of waiting processes
- Prevents starvation
- Implement: Increment priority after waiting time

**Key Points:**
- Higher priority runs first
- Can cause starvation without aging
- Priority inversion problem
- Aging prevents starvation

---

### Solution 15: Multilevel Feedback Queue Scheduling

**Implementation:**
```python
class MultilevelFeedbackQueue:
    def __init__(self):
        self.queues = [
            {'quantum': 8, 'algorithm': 'RR'},   # Q0: Highest priority
            {'quantum': 16, 'algorithm': 'RR'},  # Q1: Medium priority
            {'quantum': None, 'algorithm': 'FCFS'}  # Q2: Lowest priority
        ]
    
    def schedule(self, processes):
        current_time = 0
        queue_levels = [[] for _ in range(len(self.queues))]
        
        for p in processes:
            queue_levels[0].append(p)  # Start at highest priority
        
        while any(queue_levels):
            for level in range(len(queue_levels)):
                if queue_levels[level]:
                    p = queue_levels[level].pop(0)
                    quantum = self.queues[level]['quantum']
                    
                    if quantum and p.remaining_time > quantum:
                        # Time slice expired, move to lower priority
                        current_time += quantum
                        p.remaining_time -= quantum
                        if level < len(queue_levels) - 1:
                            queue_levels[level + 1].append(p)
                    else:
                        # Process completes
                        current_time += p.remaining_time
                        p.completion_time = current_time
                    break
        
        return processes
```

**Key Points:**
- Multiple queues with different priorities
- Processes move between queues
- I/O-bound stay in high priority
- CPU-bound move to lower priority
- Prevents starvation

---

## Memory Management Solutions

### Solution 16: Contiguous Memory Allocation

**First Fit:**
```python
def first_fit(memory_blocks, process_size):
    for block in memory_blocks:
        if block.size >= process_size and not block.allocated:
            # Allocate
            allocated_size = process_size
            remaining_size = block.size - process_size
            
            block.allocated = True
            block.allocated_size = allocated_size
            
            if remaining_size > 0:
                # Create new free block
                new_block = MemoryBlock(block.start + allocated_size, remaining_size)
                memory_blocks.insert(memory_blocks.index(block) + 1, new_block)
            
            return block
    return None  # No suitable block found
```

**Best Fit:**
```python
def best_fit(memory_blocks, process_size):
    best_block = None
    best_size = float('inf')
    
    for block in memory_blocks:
        if not block.allocated and block.size >= process_size:
            if block.size < best_size:
                best_size = block.size
                best_block = block
    
    if best_block:
        # Allocate similar to first fit
        allocate_block(best_block, process_size)
        return best_block
    return None
```

**External Fragmentation:**
- Free memory scattered in small pieces
- Total free sufficient but not contiguous
- Solution: Compaction (move processes to consolidate)

**Key Points:**
- First Fit: Fast, simple
- Best Fit: Minimizes waste, slower
- External fragmentation common
- Compaction expensive but solves fragmentation

---

### Solution 17: Paging System

**Address Translation:**
```
Logical Address = Page Number (p) + Offset (d)
Physical Address = Frame Number (f) + Offset (d)

Page Table Entry:
- Frame number
- Valid bit
- Protection bits
```

**Implementation:**
```python
class PagingSystem:
    def __init__(self, page_size, num_frames):
        self.page_size = page_size
        self.num_frames = num_frames
        self.page_table = {}  # page_num -> frame_num
        self.free_frames = list(range(num_frames))
    
    def translate_address(self, logical_address):
        page_num = logical_address // self.page_size
        offset = logical_address % self.page_size
        
        if page_num in self.page_table:
            frame_num = self.page_table[page_num]
            physical_address = frame_num * self.page_size + offset
            return physical_address
        else:
            # Page fault
            return self.handle_page_fault(page_num, offset)
    
    def handle_page_fault(self, page_num, offset):
        if self.free_frames:
            frame_num = self.free_frames.pop(0)
            self.page_table[page_num] = frame_num
            return frame_num * self.page_size + offset
        else:
            # Need page replacement
            return None
```

**Key Points:**
- Fixed-size pages and frames
- Page table maps pages to frames
- No external fragmentation
- Internal fragmentation possible

---

### Solution 18: Segmentation

**Address Translation:**
```
Logical Address = Segment Number (s) + Offset (d)
Physical Address = Base Address + Offset

Segment Table Entry:
- Base address
- Limit (segment length)
- Protection bits
```

**Implementation:**
```python
class SegmentationSystem:
    def __init__(self):
        self.segment_table = {}  # seg_num -> (base, limit)
    
    def translate_address(self, logical_address):
        seg_num = logical_address >> 16  # High bits
        offset = logical_address & 0xFFFF  # Low bits
        
        if seg_num in self.segment_table:
            base, limit = self.segment_table[seg_num]
            if offset < limit:
                return base + offset
            else:
                raise Exception("Segment fault: offset exceeds limit")
        else:
            raise Exception("Invalid segment number")
```

**Key Points:**
- Variable-size segments
- Logical structure (code, data, stack)
- External fragmentation possible
- Better for logical organization

---

### Solution 19: Paged Segmentation

**Two-Level Translation:**
```
Logical Address = Segment Number (s) + Page Number (p) + Offset (d)

1. Segment table: s -> base address of page table
2. Page table: p -> frame number
3. Physical address = frame_num * page_size + offset
```

**Key Points:**
- Combines benefits of both
- Segments divided into pages
- No external fragmentation
- Logical structure maintained

---

### Solution 20: Page Table Structure

**Hierarchical Page Tables:**
- Two-level: Outer page table + Inner page tables
- Reduces memory for page tables
- Example: 32-bit address, 4KB pages
  - 20 bits for page number (2^20 pages)
  - Two-level: 10 bits outer, 10 bits inner

**Inverted Page Table:**
- One entry per frame (not per page)
- Smaller for sparse address spaces
- Slower lookup (need to search)

**Key Points:**
- Hierarchical reduces page table size
- Inverted page table for sparse spaces
- Trade-off: Size vs lookup speed
- Modern systems use multi-level page tables

---

## Virtual Memory Solutions

### Solution 21: Virtual Memory Concept

**Virtual Memory:**
- Allows programs larger than physical memory
- Pages stored on disk when not in use
- Provides illusion of large memory space
- Enables multiprogramming

**Demand Paging:**
- Pages loaded only when needed
- Page fault triggers loading
- Lazy loading strategy

**Benefits:**
- More processes can run
- Programs can be larger than RAM
- Memory protection
- Simplified programming

**Key Points:**
- Virtual memory enables larger programs
- Demand paging loads on demand
- Page faults trigger loading
- Essential for modern systems

---

### Solution 22: Page Replacement Algorithms

**FIFO:**
```python
def fifo_page_replacement(pages, num_frames):
    memory = []
    page_faults = 0
    
    for page in pages:
        if page not in memory:
            page_faults += 1
            if len(memory) == num_frames:
                memory.pop(0)  # Remove oldest
            memory.append(page)
    
    return page_faults
```

**LRU:**
```python
def lru_page_replacement(pages, num_frames):
    memory = []  # Most recent at end
    page_faults = 0
    
    for page in pages:
        if page in memory:
            memory.remove(page)
            memory.append(page)  # Move to end
        else:
            page_faults += 1
            if len(memory) == num_frames:
                memory.pop(0)  # Remove least recently used
            memory.append(page)
    
    return page_faults
```

**Optimal:**
- Replace page that won't be used for longest time
- Theoretical optimal
- Requires future knowledge

**Belady's Anomaly:**
- More frames can lead to more page faults
- Occurs with FIFO
- LRU doesn't suffer from it

**Key Points:**
- FIFO: Simple but not optimal
- LRU: Good practical algorithm
- Optimal: Theoretical best
- Belady's anomaly affects FIFO

---

### Solution 23: LRU Implementation

**Using Hash Map + Doubly Linked List:**
```python
class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def _add_node(self, node):
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node
    
    def _remove_node(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def _move_to_head(self, node):
        self._remove_node(node)
        self._add_node(node)
    
    def get(self, key):
        if key in self.cache:
            node = self.cache[key]
            self._move_to_head(node)
            return node.value
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._move_to_head(node)
        else:
            if len(self.cache) >= self.capacity:
                lru = self.tail.prev
                self._remove_node(lru)
                del self.cache[lru.key]
            
            node = Node(key, value)
            self._add_node(node)
            self.cache[key] = node
```

**Key Points:**
- Hash map for O(1) lookup
- Doubly linked list for O(1) updates
- O(1) for all operations
- Efficient implementation

---

### Solution 24: Thrashing

**Thrashing:**
- Excessive page faults
- System spends more time swapping than executing
- CPU utilization drops
- Caused by too many processes for available frames

**Detection:**
- High page fault rate
- Low CPU utilization
- High disk I/O
- Monitor: page fault rate > threshold

**Prevention:**
- Working set model
- Limit multiprogramming level
- Increase physical memory
- Better page replacement algorithm

**Working Set:**
- Set of pages process needs in time window
- Allocate frames >= working set size
- Prevents thrashing

**Key Points:**
- Thrashing: Too many page faults
- Detect via monitoring
- Prevent with working set model
- Limit processes or increase memory

---

### Solution 25: Page Fault Handling

**Page Fault Steps:**
1. Invalid address → terminate process
2. Valid but not in memory:
   - Find free frame
   - If none, select victim (page replacement)
   - Write victim to disk if dirty
   - Read required page from disk
   - Update page table
   - Restart instruction

**Implementation:**
```python
def handle_page_fault(page_num, process):
    if page_num not in process.page_table:
        raise Exception("Invalid page")
    
    if not process.page_table[page_num].present:
        # Page not in memory
        frame = allocate_frame()
        if frame is None:
            # Need page replacement
            victim = select_victim_page()
            if victim.dirty:
                write_to_disk(victim)
            frame = victim.frame
        
        # Load page
        load_page_from_disk(page_num, frame)
        process.page_table[page_num].present = True
        process.page_table[page_num].frame = frame
    
    return process.page_table[page_num].frame
```

**Key Points:**
- Page fault triggers loading
- May need page replacement
- Write dirty pages to disk
- Restart instruction after loading

---

## File Systems Solutions

### Solution 26: File System Structure

**Components:**
- **Boot Block**: Boot code
- **Superblock**: File system metadata (size, free blocks, etc.)
- **Inode Table**: File metadata (permissions, size, block pointers)
- **Data Blocks**: Actual file data

**Inode Structure:**
```
- File type and permissions
- Owner and group IDs
- File size
- Timestamps (created, modified, accessed)
- Direct block pointers (12)
- Single indirect pointer
- Double indirect pointer
- Triple indirect pointer
```

**Directory:**
- Special file containing (filename, inode number) pairs
- Hierarchical structure
- Root directory at top

**Key Points:**
- Boot block: System startup
- Superblock: File system info
- Inodes: File metadata
- Data blocks: File content

---

### Solution 27: File Allocation Methods

**Contiguous Allocation:**
- Files stored in contiguous blocks
- Simple, fast sequential access
- External fragmentation
- Need to know file size in advance

**Linked Allocation:**
- Files stored as linked list of blocks
- No external fragmentation
- Slow random access
- Pointer overhead per block

**Indexed Allocation:**
- Index block contains pointers to data blocks
- Fast random access
- Index block overhead
- Can handle large files with multi-level indexing

**Comparison:**
```
Method          Sequential    Random    Fragmentation
Contiguous      Excellent    Good      External
Linked          Good         Poor      None
Indexed         Good         Excellent Minimal
```

**Key Points:**
- Contiguous: Fast but fragments
- Linked: No fragments but slow
- Indexed: Best balance
- Choose based on access pattern

---

### Solution 28: Directory Implementation

**Linear List:**
- Simple list of (filename, inode) pairs
- Linear search for lookup
- O(n) search time
- Simple to implement

**Hash Table:**
- Hash filename to directory entry
- O(1) average lookup
- Handle collisions
- Faster than linear list

**B-Tree:**
- Balanced tree structure
- O(log n) operations
- Good for large directories
- Used in modern file systems

**Key Points:**
- Linear list: Simple but slow
- Hash table: Fast lookup
- B-tree: Scalable
- Choose based on directory size

---

### Solution 29: File System Consistency

**Inconsistencies:**
- Lost blocks: Allocated but not referenced
- Missing blocks: Referenced but not allocated
- Incorrect link counts
- Orphaned inodes

**Consistency Checker (fsck):**
1. Check block consistency
2. Check inode consistency
3. Check directory consistency
4. Fix inconsistencies

**Journaling:**
- Log changes before applying
- Replay log on recovery
- Faster recovery than full check
- Example: ext3, ext4

**Key Points:**
- Inconsistencies from crashes
- fsck checks and fixes
- Journaling prevents inconsistencies
- Faster recovery with journaling

---

### Solution 30: Journaling File Systems

**Journaling Modes:**
- **Writeback**: Metadata only
- **Ordered**: Metadata after data
- **Data**: Full data journaling

**Write-Ahead Logging:**
1. Write intent to journal
2. Write data to journal
3. Commit journal entry
4. Write to actual location
5. Remove journal entry

**Recovery:**
- Replay journal entries
- Faster than full fsck
- Ensures consistency

**Key Points:**
- Journaling improves reliability
- Different modes for different needs
- Faster recovery
- Trade-off: Performance vs safety

---

## I/O Systems Solutions

### Solution 31: Disk Scheduling Algorithms

**FCFS:**
- Serve requests in order
- Simple, fair
- No optimization
- Can have high seek time

**SSTF:**
- Serve closest request
- Minimizes seek time
- Can cause starvation
- Not optimal overall

**SCAN (Elevator):**
- Move in one direction
- Serve requests along way
- Reverse at end
- Fair, no starvation

**C-SCAN:**
- Like SCAN but immediate return
- More uniform wait time
- Better for high load

**Example:**
```
Current: 50, Requests: [98, 183, 37, 122, 14, 124, 65, 67]

FCFS: 640 seeks
SSTF: 236 seeks
SCAN: 236 seeks (right first)
C-SCAN: 382 seeks
```

**Key Points:**
- SSTF: Minimizes seek but starves
- SCAN: Fair, no starvation
- C-SCAN: Uniform wait time
- Choose based on workload

---

### Solution 32: I/O Buffering

**Single Buffering:**
- One buffer
- Process waits while buffer fills
- Simple but inefficient

**Double Buffering:**
- Two buffers
- Fill one while processing other
- Overlaps I/O and processing
- Better performance

**Circular Buffering:**
- Multiple buffers in ring
- Producer fills, consumer processes
- Efficient for continuous I/O

**Key Points:**
- Buffering overlaps I/O and CPU
- Double buffering common
- Circular for continuous streams
- Reduces wait time

---

### Solution 33: Spooling and Device Management

**Spooling:**
- Simultaneous Peripheral Operations On-Line
- Queue jobs for devices
- Example: Print spooling
- Allows multiple processes to "use" device

**Device Queue:**
- Queue requests for device
- FIFO typically
- Manage device allocation

**Key Points:**
- Spooling allows device sharing
- Queue manages requests
- Improves device utilization
- Common for printers

---

## Synchronization Solutions

### Solution 34: Mutex Implementation

**Test-and-Set:**
```c
int test_and_set(int *lock) {
    int old = *lock;
    *lock = 1;
    return old;
}

void acquire_mutex(int *lock) {
    while (test_and_set(lock) == 1) {
        // Busy wait
    }
}

void release_mutex(int *lock) {
    *lock = 0;
}
```

**Compare-and-Swap:**
```c
int compare_and_swap(int *value, int expected, int new) {
    int old = *value;
    if (*value == expected) {
        *value = new;
    }
    return old;
}
```

**Fair Mutex:**
- Queue waiting threads
- Serve in order
- Prevents starvation

**Key Points:**
- Hardware support needed
- Test-and-set: Simple
- Compare-and-swap: More flexible
- Fair mutex prevents starvation

---

### Solution 35: Semaphore Implementation

**Using Mutex and Condition Variable:**
```c
typedef struct {
    int value;
    pthread_mutex_t mutex;
    pthread_cond_t condition;
} semaphore_t;

void sem_wait(semaphore_t *sem) {
    pthread_mutex_lock(&sem->mutex);
    while (sem->value <= 0) {
        pthread_cond_wait(&sem->condition, &sem->mutex);
    }
    sem->value--;
    pthread_mutex_unlock(&sem->mutex);
}

void sem_post(semaphore_t *sem) {
    pthread_mutex_lock(&sem->mutex);
    sem->value++;
    pthread_cond_signal(&sem->condition);
    pthread_mutex_unlock(&sem->mutex);
}
```

**Key Points:**
- Semaphore: Counting resource
- Wait decrements, post increments
- Blocks when value <= 0
- Can implement with mutex + condition variable

---

### Solution 36: Reader-Writer Problem

**Reader-Preference:**
```c
int readers = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t write_mutex = PTHREAD_MUTEX_INITIALIZER;

void reader() {
    pthread_mutex_lock(&mutex);
    readers++;
    if (readers == 1) {
        pthread_mutex_lock(&write_mutex);
    }
    pthread_mutex_unlock(&mutex);
    
    // Read
    
    pthread_mutex_lock(&mutex);
    readers--;
    if (readers == 0) {
        pthread_mutex_unlock(&write_mutex);
    }
    pthread_mutex_unlock(&mutex);
}

void writer() {
    pthread_mutex_lock(&write_mutex);
    // Write
    pthread_mutex_unlock(&write_mutex);
}
```

**Key Points:**
- Multiple readers allowed
- Single writer
- Reader-preference: Readers can starve writers
- Writer-preference: Writers prioritized

---

### Solution 37: Dining Philosophers Problem

**Solution with Semaphores:**
```c
#define N 5
sem_t chopsticks[N];
sem_t mutex;

void philosopher(int i) {
    while (1) {
        think();
        
        sem_wait(&mutex);
        sem_wait(&chopsticks[i]);
        sem_wait(&chopsticks[(i+1)%N]);
        sem_post(&mutex);
        
        eat();
        
        sem_post(&chopsticks[i]);
        sem_post(&chopsticks[(i+1)%N]);
    }
}
```

**Key Points:**
- Prevent deadlock: Acquire both atomically
- Prevent starvation: Fair scheduling
- Mutex ensures atomic acquisition
- Release both after eating

---

### Solution 38: Producer-Consumer Problem

**Using Semaphores:**
```c
#define BUFFER_SIZE 10
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

sem_t empty;  // Empty slots
sem_t full;   // Filled slots
pthread_mutex_t mutex;

void producer() {
    while (1) {
        item = produce();
        sem_wait(&empty);
        pthread_mutex_lock(&mutex);
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
}

void consumer() {
    while (1) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);
        item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&empty);
        consume(item);
    }
}
```

**Key Points:**
- Semaphores for synchronization
- Mutex for buffer access
- Empty semaphore: Available slots
- Full semaphore: Available items

---

## Deadlocks Solutions

### Solution 39: Deadlock Detection

**Resource Allocation Graph:**
- Nodes: Processes and Resources
- Edges: Request edges and Assignment edges
- Cycle detection: Deadlock exists if cycle in graph

**Algorithm:**
```python
def detect_deadlock(resource_allocation_graph):
    # Build wait-for graph
    wait_graph = build_wait_for_graph(resource_allocation_graph)
    
    # Detect cycles using DFS
    visited = set()
    rec_stack = set()
    
    for process in processes:
        if has_cycle(process, wait_graph, visited, rec_stack):
            return True
    
    return False

def has_cycle(node, graph, visited, rec_stack):
    visited.add(node)
    rec_stack.add(node)
    
    for neighbor in graph[node]:
        if neighbor not in visited:
            if has_cycle(neighbor, graph, visited, rec_stack):
                return True
        elif neighbor in rec_stack:
            return True
    
    rec_stack.remove(node)
    return False
```

**Key Points:**
- Build wait-for graph
- Detect cycles
- Cycle = deadlock
- Periodic detection

---

### Solution 40: Deadlock Prevention

**Eliminate Conditions:**
1. **Mutual Exclusion**: Not always possible (some resources inherently exclusive)
2. **Hold and Wait**: Request all resources at once
3. **No Preemption**: Allow preemption
4. **Circular Wait**: Order resources, request in order

**Resource Ordering:**
```python
# Assign order to resources
resource_order = {'R1': 1, 'R2': 2, 'R3': 3}

def request_resources(process, resources):
    # Sort resources by order
    sorted_resources = sorted(resources, key=lambda r: resource_order[r])
    
    # Request in order
    for resource in sorted_resources:
        acquire(resource)
```

**Key Points:**
- Eliminate one condition prevents deadlock
- Resource ordering prevents circular wait
- Request all at once prevents hold and wait
- Preemption prevents no preemption

---

### Solution 41: Deadlock Avoidance - Banker's Algorithm

**Safety Algorithm:**
```python
def is_safe_state(available, allocation, need, processes):
    work = available.copy()
    finish = [False] * len(processes)
    safe_sequence = []
    
    while True:
        found = False
        for i in range(len(processes)):
            if not finish[i]:
                # Check if need[i] <= work
                if all(need[i][j] <= work[j] for j in range(len(work))):
                    work = [work[j] + allocation[i][j] for j in range(len(work))]
                    finish[i] = True
                    safe_sequence.append(i)
                    found = True
        
        if not found:
            break
    
    return all(finish), safe_sequence

def request_resources(process_id, request, available, allocation, need):
    # Check if request <= need
    if not all(request[i] <= need[process_id][i] for i in range(len(request))):
        return False, "Error: Request exceeds need"
    
    # Check if request <= available
    if not all(request[i] <= available[i] for i in range(len(request))):
        return False, "Wait: Resources not available"
    
    # Try allocation
    new_available = [available[i] - request[i] for i in range(len(available))]
    new_allocation = [allocation[process_id][i] + request[i] for i in range(len(request))]
    new_need = [need[process_id][i] - request[i] for i in range(len(request))]
    
    # Check if safe
    if is_safe_state(new_available, new_allocation, new_need, processes):
        return True, "Safe: Allocation granted"
    else:
        return False, "Unsafe: Request denied"
```

**Key Points:**
- Check safe state before allocation
- Safe state: Can find sequence where all finish
- Deny request if leads to unsafe state
- Requires advance knowledge of max needs

---

### Solution 42: Deadlock Recovery

**Process Termination:**
- Abort all deadlocked processes
- Abort one at a time until deadlock resolved
- Choose process with minimum cost

**Resource Preemption:**
- Preempt resources from processes
- Rollback to safe state
- Restart process
- Need to handle preempted process

**Cost Considerations:**
- Priority of process
- How long process has computed
- Resources process has used
- What process needs to complete

**Key Points:**
- Terminate processes
- Preempt resources
- Minimize cost
- Rollback if needed

---

## Inter-Process Communication Solutions

### Solution 43: Shared Memory IPC

**Implementation:**
```c
#include <sys/shm.h>
#include <sys/ipc.h>

// Create shared memory
int shm_id = shmget(IPC_PRIVATE, 1024, IPC_CREAT | 0666);
char *shm_ptr = (char*)shmat(shm_id, NULL, 0);

// Write to shared memory
strcpy(shm_ptr, "Hello from process 1");

// Detach
shmdt(shm_ptr);

// Remove (in one process)
shmctl(shm_id, IPC_RMID, NULL);
```

**Synchronization:**
- Need mutexes/semaphores
- Shared memory itself not synchronized
- Multiple processes can access simultaneously

**Key Points:**
- Fastest IPC method
- Shared memory segment
- Need synchronization
- Good for high-performance

---

### Solution 44: Message Passing IPC

**Pipes:**
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

**Message Queues:**
```c
#include <sys/msg.h>

// Create message queue
int msg_id = msgget(IPC_PRIVATE, IPC_CREAT | 0666);

// Send message
struct msgbuf {
    long mtype;
    char mtext[100];
} message;

message.mtype = 1;
strcpy(message.mtext, "Hello");
msgsnd(msg_id, &message, sizeof(message.mtext), 0);

// Receive message
msgrcv(msg_id, &message, sizeof(message.mtext), 1, 0);
```

**Key Points:**
- Pipes: Simple, parent-child
- Message queues: Structured messages
- Safer than shared memory
- Slower than shared memory

---

### Solution 45: Socket Programming

**Unix Domain Sockets:**
```c
#include <sys/socket.h>
#include <sys/un.h>

// Server
int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strcpy(addr.sun_path, "/tmp/socket");
bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
listen(sockfd, 5);
int client = accept(sockfd, NULL, NULL);

// Client
int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
```

**Key Points:**
- Sockets for IPC or network
- Unix domain: Local IPC
- Stream: Reliable, ordered
- Datagram: Unreliable, unordered

---

## System Calls Solutions

### Solution 46: System Call Implementation

**System Call Flow:**
1. User program calls library function
2. Library function executes trap instruction
3. CPU switches to kernel mode
4. Kernel handles system call
5. Return to user mode
6. Library function returns to user

**Trap Instruction:**
- Switches to kernel mode
- Jumps to system call handler
- Passes system call number
- Kernel dispatches to appropriate handler

**Cost:**
- Mode switch overhead
- Context switching
- Typically 1-10 microseconds
- Optimized in modern systems

**Key Points:**
- Trap instruction switches mode
- Kernel handles in privileged mode
- Return to user mode after
- Overhead from mode switching

---

### Solution 47: File System System Calls

**File Operations:**
```c
// Open file
int fd = open("file.txt", O_RDWR | O_CREAT, 0644);

// Read
char buf[100];
ssize_t bytes_read = read(fd, buf, 100);

// Write
ssize_t bytes_written = write(fd, "Hello", 5);

// Close
close(fd);
```

**File Descriptors:**
- Small integer representing open file
- Per-process table
- 0: stdin, 1: stdout, 2: stderr
- Used for all file operations

**Key Points:**
- File descriptors represent files
- System calls for file operations
- Handle errors appropriately
- Close files when done

---

### Solution 48: Process Management System Calls

**fork() and exec():**
```c
pid_t pid = fork();
if (pid == 0) {
    // Child
    execl("/bin/ls", "ls", "-l", NULL);
} else {
    // Parent
    wait(NULL);  // Wait for child
}
```

**Signals:**
```c
#include <signal.h>

void signal_handler(int sig) {
    // Handle signal
}

signal(SIGINT, signal_handler);  // Handle Ctrl+C
```

**Key Points:**
- fork() creates process
- exec() replaces image
- wait() for child completion
- Signals for process communication

---

## Kernel Architecture Solutions

### Solution 49: Monolithic vs Microkernel

**Monolithic Kernel:**
- All OS services in kernel space
- Fast (no mode switches)
- Less modular
- Example: Linux

**Microkernel:**
- Minimal kernel, services in user space
- More modular, secure
- Slower (more mode switches)
- Example: Minix

**Hybrid:**
- Combination of both
- Some services in kernel, some in user space
- Balance performance and modularity
- Example: Windows, macOS

**Key Points:**
- Monolithic: Fast, less modular
- Microkernel: Modular, slower
- Hybrid: Balance
- Modern systems use hybrid

---

### Solution 50: Kernel Modules

**Module Structure:**
```c
#include <linux/module.h>
#include <linux/kernel.h>

static int __init module_init(void) {
    printk(KERN_INFO "Module loaded\n");
    return 0;
}

static void __exit module_exit(void) {
    printk(KERN_INFO "Module unloaded\n");
}

module_init(module_init);
module_exit(module_exit);
MODULE_LICENSE("GPL");
```

**Loading:**
```bash
insmod module.ko    # Load module
rmmod module        # Unload module
lsmod              # List loaded modules
```

**Key Points:**
- Extend kernel functionality
- Load/unload dynamically
- Interface with kernel
- Security considerations

---

## Process Synchronization Solutions

### Solution 51: Atomic Operations

**Compare-and-Swap:**
```c
int compare_and_swap(int *ptr, int expected, int new) {
    int old = *ptr;
    if (old == expected) {
        *ptr = new;
    }
    return old;
}

// Lock-free counter
void increment(int *counter) {
    int old, new;
    do {
        old = *counter;
        new = old + 1;
    } while (compare_and_swap(counter, old, new) != old);
}
```

**Memory Ordering:**
- Sequential consistency
- Acquire-release semantics
- Relaxed ordering
- Important for multi-core

**Key Points:**
- Atomic operations: Hardware support
- Lock-free programming possible
- Memory ordering matters
- Compare-and-swap powerful

---

### Solution 52: Spinlocks vs Mutexes

**Spinlock:**
- Busy wait for lock
- Good for short critical sections
- Wastes CPU cycles
- No context switch overhead

**Mutex:**
- Block and yield CPU
- Good for longer critical sections
- Saves CPU cycles
- Context switch overhead

**When to Use:**
- Spinlock: Short waits, multi-core
- Mutex: Longer waits, single core
- Adaptive: Spin then block

**Key Points:**
- Spinlock: Busy wait
- Mutex: Block
- Choose based on wait time
- Adaptive combines both

---

### Solution 53: Condition Variables

**Usage:**
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition = PTHREAD_COND_INITIALIZER;
int condition_met = 0;

// Waiter
pthread_mutex_lock(&mutex);
while (!condition_met) {
    pthread_cond_wait(&condition, &mutex);
}
// Condition met, proceed
pthread_mutex_unlock(&mutex);

// Signaler
pthread_mutex_lock(&mutex);
condition_met = 1;
pthread_cond_signal(&condition);
pthread_mutex_unlock(&mutex);
```

**Spurious Wakeups:**
- Thread can wake without signal
- Always check condition in loop
- while (!condition) not if (!condition)

**Key Points:**
- Wait for conditions efficiently
- Use with mutex
- Check condition in loop
- Signal when condition changes

---

## Memory Allocation Solutions

### Solution 54: Buddy System Allocation

**Algorithm:**
- Memory divided into blocks of size 2^k
- When allocating, find smallest block >= size
- Split blocks in half until right size
- When freeing, merge with buddy if both free

**Implementation:**
```python
class BuddySystem:
    def __init__(self, total_size):
        self.total_size = total_size
        self.free_lists = {}  # size -> list of blocks
        self.allocated = {}   # address -> size
    
    def allocate(self, size):
        # Find appropriate block size (power of 2)
        block_size = self._next_power_of_2(size)
        
        # Find free block
        if block_size in self.free_lists and self.free_lists[block_size]:
            block = self.free_lists[block_size].pop(0)
            self.allocated[block] = block_size
            return block
        
        # Split larger block
        larger_size = block_size * 2
        while larger_size <= self.total_size:
            if larger_size in self.free_lists and self.free_lists[larger_size]:
                block = self.free_lists[larger_size].pop(0)
                # Split
                while larger_size > block_size:
                    larger_size //= 2
                    buddy = block + larger_size
                    self.free_lists[larger_size].append(buddy)
                self.allocated[block] = block_size
                return block
            larger_size *= 2
        
        return None  # Out of memory
    
    def deallocate(self, address):
        if address not in self.allocated:
            return
        
        size = self.allocated[address]
        del self.allocated[address]
        
        # Merge with buddy
        self._merge_buddy(address, size)
    
    def _merge_buddy(self, address, size):
        buddy_address = address ^ size  # XOR to get buddy
        
        if buddy_address in self.free_lists.get(size, []):
            # Buddy is free, merge
            self.free_lists[size].remove(buddy_address)
            merged_address = min(address, buddy_address)
            self._merge_buddy(merged_address, size * 2)
        else:
            # Add to free list
            if size not in self.free_lists:
                self.free_lists[size] = []
            self.free_lists[size].append(address)
```

**Key Points:**
- Blocks are powers of 2
- Split to allocate
- Merge buddies when freeing
- Fast allocation/deallocation

---

### Solution 55: Slab Allocation

**Concept:**
- Pre-allocate objects of common sizes
- Cache objects in slabs
- Fast allocation for kernel objects
- Reduces fragmentation

**Structure:**
- Slab: Container for objects
- Cache: Collection of slabs for same object type
- Objects allocated from slabs
- Slabs can be full, partial, or empty

**Key Points:**
- Optimized for common object sizes
- Pre-allocation reduces overhead
- Good for kernel objects
- Reduces fragmentation

---

## Advanced Topics Solutions

### Solution 56: Real-Time Scheduling

**Rate Monotonic Scheduling:**
- Higher frequency = higher priority
- Preemptive
- Optimal for periodic tasks
- Utilization bound: Σ(Ci/Ti) ≤ n(2^(1/n) - 1)

**Earliest Deadline First (EDF):**
- Earlier deadline = higher priority
- Dynamic priority
- Optimal for periodic and aperiodic
- Utilization bound: 100%

**Key Points:**
- Real-time: Meet deadlines
- Rate Monotonic: Static priority
- EDF: Dynamic priority
- Hard vs soft real-time

---

### Solution 57: Multi-Core Scheduling

**Challenges:**
- Load balancing across cores
- Cache affinity
- False sharing
- NUMA considerations

**Load Balancing:**
- Migrate processes between cores
- Balance load
- Consider cache affinity
- Don't migrate unnecessarily

**Cache Affinity:**
- Keep process on same core
- Better cache utilization
- Reduce migration overhead
- Balance with load

**Key Points:**
- Balance load across cores
- Maintain cache affinity
- NUMA: Consider memory locality
- Optimize for multi-core

---

### Solution 58: Virtualization and Containers

**Virtual Machines:**
- Full OS virtualization
- Hypervisor manages VMs
- Isolation at hardware level
- Higher overhead

**Containers:**
- OS-level virtualization
- Share host kernel
- Lightweight
- Less isolation

**Comparison:**
```
                VMs          Containers
Isolation        High         Medium
Overhead         High         Low
Startup          Slow        Fast
Resource         More         Less
```

**Key Points:**
- VMs: Full isolation
- Containers: Lightweight
- Different use cases
- Containers popular for microservices

---

### Solution 59: Security in Operating Systems

**Access Control:**
- Discretionary Access Control (DAC)
- Mandatory Access Control (MAC)
- Role-Based Access Control (RBAC)
- Capabilities

**Capabilities:**
- Token-based access
- More flexible than ACLs
- Fine-grained control
- Example: Linux capabilities

**Sandboxing:**
- Isolate processes
- Limit resources
- Prevent privilege escalation
- Example: chroot, containers

**Key Points:**
- Multiple access control models
- Capabilities vs ACLs
- Sandboxing for isolation
- Defense in depth

---

### Solution 60: Performance Monitoring and Profiling

**Metrics:**
- CPU utilization
- Memory usage
- I/O statistics
- Context switches
- Page faults

**Tools:**
- top/htop: Process monitoring
- perf: Performance profiling
- strace: System call tracing
- vmstat: System statistics

**Profiling:**
- Identify bottlenecks
- CPU profiling
- Memory profiling
- I/O profiling

**Key Points:**
- Monitor key metrics
- Use profiling tools
- Identify bottlenecks
- Optimize based on data

---

## Summary

This comprehensive guide covers all major Operating Systems topics for SDE 2 interviews:

1. **Process Management**: Creation, termination, states, scheduling, PCB
2. **Thread Management**: Threads, synchronization, thread pools, models
3. **CPU Scheduling**: Algorithms, metrics, optimization, multilevel queues
4. **Memory Management**: Allocation, paging, segmentation, page tables
5. **Virtual Memory**: Page replacement, thrashing, page faults, LRU
6. **File Systems**: Structure, allocation, directories, consistency, journaling
7. **I/O Systems**: Disk scheduling, buffering, spooling
8. **Synchronization**: Mutexes, semaphores, condition variables, problems
9. **Deadlocks**: Detection, prevention, avoidance, recovery
10. **IPC**: Shared memory, message passing, sockets
11. **System Calls**: Implementation, file operations, process management
12. **Kernel Architecture**: Monolithic, microkernel, modules
13. **Process Synchronization**: Atomic operations, spinlocks, condition variables
14. **Memory Allocation**: Buddy system, slab allocation
15. **Advanced Topics**: Real-time scheduling, multi-core, virtualization, security, performance

**Total Coverage: 60 Problems with Detailed Solutions**

Each solution includes explanations, code examples, algorithms, and key points. Use these as a reference for interview preparation and real-world implementations.
