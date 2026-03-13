# OS Cheat Sheet (SDE 2 Interviews)

## High-frequency definitions (say these crisply)
- **Process**: program in execution with **own address space**.
- **Thread**: execution unit within a process; **shares address space** with peer threads.
- **Context switch**: save CPU state of one runnable entity + restore another; cost comes from **scheduler + cache/TLB effects**.
- **System call**: controlled entry into kernel (trap) to perform privileged operations.
- **Interrupt vs exception**: interrupt is external (I/O, timer); exception is internal (page fault, divide-by-zero).

## Process lifecycle (must know)
- **States**: New → Ready → Running → Waiting/Blocked → Ready → … → Terminated
- **Zombie**: exited child whose **exit status not collected** (parent didn’t `wait()`).
- **Orphan**: parent died; adopted by init/systemd (on Unix-like systems).

## `fork()` / `exec()` / `wait()` (classic)
- **`fork()`**: creates child; returns **0 in child**, **child PID in parent**.
- **`exec*()`**: replaces current process image (PID stays same, code/data replaced).
- **`wait()` / `waitpid()`**: parent collects child status → prevents zombies.
- **Copy-on-write (COW)**: `fork()` is cheap because pages are shared until a write.

## Scheduling (core algorithms + what to say)
- **FCFS**: simple; can cause **convoy effect**.
- **SJF/SRTF**: minimizes avg waiting time; needs burst prediction; SRTF is preemptive.
- **Round Robin (RR)**: time quantum $q$; good responsiveness; too small $q$ ⇒ overhead, too large ⇒ FCFS-like.
- **Priority**: needs **aging** to avoid starvation; watch **priority inversion** (fix: priority inheritance).
- **MLFQ**: interactive stays high priority; CPU-bound drifts down; good general-purpose.

## Scheduling metrics (often asked)
- **Turnaround** = completion − arrival
- **Waiting** = turnaround − CPU burst
- **Response** = first scheduled − arrival
- **Throughput** = completed / time
- **CPU utilization** ≈ busy time / total time

## Concurrency & synchronization (must be fluent)
- **Race condition**: result depends on timing/interleaving.
- **Critical section requirements**: mutual exclusion, progress, bounded waiting.
- **Mutex**: exclusive lock (ownership matters).
- **Semaphore**: counter-based (binary or counting); good for resource counting.
- **Condition variable**: wait/signal for a predicate; always `while (!cond) wait()`.
- **Spurious wakeup**: can happen; reason for `while` loop.

## Classic problems (know the patterns)
- **Producer–Consumer (bounded buffer)**: semaphores `empty/full` + mutex.
- **Readers–Writers**: reader-preference can starve writers; use fair/queued locks to avoid starvation.
- **Dining Philosophers**: avoid deadlock by ordering, asymmetric pickup, or limiting concurrency.

## Deadlocks (the 4 conditions + strategies)
- **Coffman conditions**: mutual exclusion, hold-and-wait, no preemption, circular wait.
- **Prevention**: break one condition (e.g., impose global resource ordering → breaks circular wait).
- **Avoidance**: **Banker’s algorithm** (requires max demand known).
- **Detection**: wait-for graph cycle detection (periodic).
- **Recovery**: kill/rollback/preempt resources (cost trade-offs).

## Memory management (what interviewers probe)
- **Paging**: fixed-size pages/frames; no external fragmentation; possible internal fragmentation.
- **Segmentation**: variable-size logical segments; external fragmentation possible.
- **Page table**: maps virtual page → physical frame + bits (valid, dirty, prot).
- **TLB**: cache of page table translations; TLB miss ≠ page fault.
- **MMU**: hardware translating VA→PA using page tables/TLB.
- **Multi-level page tables**: reduce page-table memory for sparse address spaces.
- **Inverted page table**: one entry per frame; smaller, lookup more complex.

## Virtual memory (page faults + replacement)
- **Demand paging**: bring page in on first access.
- **Page fault handling** (high level): trap → validate → locate page on disk → pick frame (evict if needed) → read → update PTE/TLB → restart instruction.
- **Replacement**:
  - **FIFO**: simple; suffers **Belady’s anomaly**.
  - **LRU**: good; implement via hash map + DLL; approximations: clock/second-chance.
  - **Optimal**: theoretical lower bound (needs future knowledge).
- **Thrashing**: excessive faults; mitigate via **working set** / reduce degree of multiprogramming.

## File systems (talk track)
- **Inode**: metadata + block pointers (direct/indirect).
- **Directories**: map name → inode (entries).
- **Allocation**:
  - contiguous (fast, fragments),
  - linked (no fragments, poor random access),
  - indexed (good random access, index overhead).
- **Journaling**: write intent/log first → crash recovery by replay; modes: metadata-only vs data journaling.

## I/O basics
- **Buffering**: single vs double vs circular (overlap I/O and CPU).
- **Spooling**: queue device requests (e.g., printers).
- **Disk scheduling**: SSTF (can starve), SCAN/C-SCAN (fairer).

## Kernel & virtualization (enough for SDE2)
- **Monolithic**: fast, large trusted base (Linux).
- **Microkernel**: smaller kernel, services in user space (more isolation, more IPC).
- **Modules**: loadable kernel extensions.
- **Containers vs VMs**: containers share kernel (lightweight); VMs virtualize hardware (stronger isolation).

## Quick “debugging toolbox” phrases
- “First check **CPU vs I/O bound**, then **contention**, then **memory pressure/page faults**, then **I/O saturation**.”
- OS perf tools: `top/htop`, `vmstat`, `iostat`, `free`, `sar`, `strace`, `perf`.

