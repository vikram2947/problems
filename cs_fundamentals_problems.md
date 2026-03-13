# CS Fundamentals Interview Problems for SDE 2
## Operating Systems, Networking, and DBMS

## Table of Contents
1. [Operating Systems](#operating-systems)
2. [Networking](#networking)
3. [Database Management Systems](#database-management-systems)

---

## Operating Systems

### Problem 1: Process vs Thread
**Scenario**: Explain the difference between processes and threads, and when to use each.

**Requirements**:
- Understand process and thread concepts
- Compare memory sharing
- Understand context switching overhead
- Know when to use processes vs threads

**Questions**:
1. What is the fundamental difference between a process and a thread?
2. How do processes and threads share memory?
3. What is the overhead difference between process and thread context switching?
4. When should you use processes vs threads in your application?

---

### Problem 2: Process Scheduling Algorithms
**Scenario**: Design a process scheduler for different use cases.

**Requirements**:
- Understand different scheduling algorithms
- Compare their characteristics
- Choose appropriate algorithm for scenarios
- Understand preemption and non-preemption

**Questions**:
1. What are the different process scheduling algorithms?
2. What is the difference between preemptive and non-preemptive scheduling?
3. How does Round Robin scheduling work?
4. When would you use Priority Scheduling vs Shortest Job First?

---

### Problem 3: Deadlock Detection and Prevention
**Scenario**: Design a system that prevents and detects deadlocks.

**Requirements**:
- Understand deadlock conditions
- Implement deadlock prevention
- Implement deadlock detection
- Handle deadlock recovery

**Questions**:
1. What are the four necessary conditions for deadlock?
2. How can you prevent deadlocks?
3. How do you detect deadlocks in a system?
4. What are the strategies for deadlock recovery?

---

### Problem 4: Memory Management - Paging vs Segmentation
**Scenario**: Choose between paging and segmentation for memory management.

**Requirements**:
- Understand paging mechanism
- Understand segmentation mechanism
- Compare advantages and disadvantages
- Choose appropriate method

**Questions**:
1. How does paging work in memory management?
2. How does segmentation differ from paging?
3. What are the advantages and disadvantages of each?
4. When would you use paging vs segmentation?

---

### Problem 5: Virtual Memory and Page Replacement
**Scenario**: Implement page replacement algorithms for virtual memory.

**Requirements**:
- Understand virtual memory concept
- Implement FIFO page replacement
- Implement LRU page replacement
- Compare page replacement algorithms

**Questions**:
1. What is virtual memory and why is it used?
2. How does FIFO page replacement work?
3. How does LRU page replacement work?
4. What is the Belady's anomaly and which algorithms suffer from it?

---

### Problem 6: Inter-Process Communication (IPC)
**Scenario**: Design IPC mechanisms for different scenarios.

**Requirements**:
- Understand different IPC mechanisms
- Compare shared memory vs message passing
- Implement IPC using pipes
- Implement IPC using shared memory

**Questions**:
1. What are the different IPC mechanisms?
2. What is the difference between shared memory and message passing?
3. How do pipes work for IPC?
4. When would you use shared memory vs message passing?

---

### Problem 7: Synchronization Primitives
**Scenario**: Implement synchronization using mutexes, semaphores, and condition variables.

**Requirements**:
- Understand mutexes
- Understand semaphores
- Understand condition variables
- Implement producer-consumer problem

**Questions**:
1. What is a mutex and how does it differ from a semaphore?
2. How do binary semaphores differ from counting semaphores?
3. What are condition variables and when are they used?
4. How would you implement a producer-consumer problem using semaphores?

---

### Problem 8: Race Conditions and Critical Sections
**Scenario**: Identify and fix race conditions in concurrent code.

**Requirements**:
- Identify race conditions
- Understand critical sections
- Implement mutual exclusion
- Prevent data races

**Questions**:
1. What is a race condition?
2. What is a critical section?
3. How do you ensure mutual exclusion?
4. How would you fix a race condition in a counter increment operation?

---

### Problem 9: File System Design
**Scenario**: Design a file system with specific requirements.

**Requirements**:
- Understand file system structure
- Design directory structure
- Implement file allocation methods
- Handle file system consistency

**Questions**:
1. What are the main components of a file system?
2. How do different file allocation methods work (contiguous, linked, indexed)?
3. How does a file system maintain consistency?
4. What is journaling in file systems?

---

### Problem 10: Disk Scheduling Algorithms
**Scenario**: Optimize disk I/O using different scheduling algorithms.

**Requirements**:
- Understand disk scheduling concepts
- Implement FCFS scheduling
- Implement SSTF scheduling
- Implement SCAN and C-SCAN algorithms

**Questions**:
1. Why do we need disk scheduling algorithms?
2. How does SSTF (Shortest Seek Time First) work?
3. What is the SCAN algorithm and how does it differ from C-SCAN?
4. Which disk scheduling algorithm minimizes starvation?

---

### Problem 11: CPU Scheduling and Context Switching
**Scenario**: Optimize CPU utilization through efficient scheduling.

**Requirements**:
- Understand CPU scheduling goals
- Calculate turnaround time and waiting time
- Compare scheduling algorithms
- Optimize for different metrics

**Questions**:
1. What are the goals of CPU scheduling?
2. How do you calculate turnaround time and waiting time?
3. What is the difference between throughput and response time?
4. How does multilevel queue scheduling work?

---

### Problem 12: Memory Allocation Strategies
**Scenario**: Implement memory allocation algorithms (First Fit, Best Fit, Worst Fit).

**Requirements**:
- Understand memory allocation concepts
- Implement First Fit algorithm
- Implement Best Fit algorithm
- Compare allocation strategies

**Questions**:
1. How does First Fit memory allocation work?
2. How does Best Fit differ from Worst Fit?
3. What is external fragmentation and how do you reduce it?
4. What is compaction and when is it used?

---

### Problem 13: System Calls and Kernel Mode
**Scenario**: Understand the difference between user mode and kernel mode.

**Requirements**:
- Understand privilege levels
- Understand system calls
- Know when mode switching occurs
- Understand kernel protection

**Questions**:
1. What is the difference between user mode and kernel mode?
2. What are system calls and why are they needed?
3. When does a process switch from user mode to kernel mode?
4. How does the operating system protect the kernel?

---

### Problem 14: Process States and State Transitions
**Scenario**: Model process lifecycle with state transitions.

**Requirements**:
- Understand process states
- Model state transitions
- Handle state changes
- Understand blocking and unblocking

**Questions**:
1. What are the different states a process can be in?
2. What causes a process to transition from running to waiting?
3. What is the difference between ready and running states?
4. How does a process move from waiting to ready state?

---

### Problem 15: Threading Models
**Scenario**: Choose appropriate threading model for your application.

**Requirements**:
- Understand user-level threads
- Understand kernel-level threads
- Compare threading models
- Understand thread mapping

**Questions**:
1. What is the difference between user-level and kernel-level threads?
2. What are the advantages and disadvantages of each model?
3. How does many-to-one threading model work?
4. What is the many-to-many threading model?

---

## Networking

### Problem 16: OSI Model and TCP/IP Stack
**Scenario**: Explain network communication using OSI and TCP/IP models.

**Requirements**:
- Understand OSI model layers
- Understand TCP/IP stack
- Map protocols to layers
- Understand encapsulation

**Questions**:
1. What are the seven layers of the OSI model?
2. How does the TCP/IP model map to the OSI model?
3. What protocols operate at each layer?
4. How does data encapsulation work across layers?

---

### Problem 17: TCP vs UDP
**Scenario**: Choose between TCP and UDP for different applications.

**Requirements**:
- Understand TCP characteristics
- Understand UDP characteristics
- Compare reliability and performance
- Choose appropriate protocol

**Questions**:
1. What are the key differences between TCP and UDP?
2. When would you use TCP vs UDP?
3. How does TCP ensure reliable delivery?
4. What are the advantages of UDP over TCP?

---

### Problem 18: TCP Connection Management
**Scenario**: Explain TCP three-way handshake and connection termination.

**Requirements**:
- Understand TCP handshake
- Understand connection termination
- Handle connection states
- Understand SYN flood attacks

**Questions**:
1. How does the TCP three-way handshake work?
2. How does TCP connection termination work (four-way handshake)?
3. What are the different TCP connection states?
4. What is a SYN flood attack and how is it prevented?

---

### Problem 19: HTTP and HTTPS
**Scenario**: Design a secure web communication system.

**Requirements**:
- Understand HTTP protocol
- Understand HTTPS and SSL/TLS
- Compare HTTP methods
- Understand stateless nature

**Questions**:
1. How does HTTP work?
2. What is the difference between HTTP and HTTPS?
3. How does SSL/TLS provide security?
4. What are the different HTTP methods and when to use them?

---

### Problem 20: DNS Resolution Process
**Scenario**: Explain how domain names are resolved to IP addresses.

**Requirements**:
- Understand DNS hierarchy
- Understand DNS resolution process
- Understand DNS record types
- Handle DNS caching

**Questions**:
1. How does DNS resolution work?
2. What are the different types of DNS records?
3. What is DNS caching and why is it important?
4. How does iterative DNS query differ from recursive query?

---

### Problem 21: Load Balancing Algorithms
**Scenario**: Design a load balancer with different distribution strategies.

**Requirements**:
- Understand load balancing concepts
- Implement round-robin algorithm
- Implement least connections algorithm
- Compare load balancing strategies

**Questions**:
1. What is load balancing and why is it needed?
2. How does round-robin load balancing work?
3. What is the difference between least connections and weighted least connections?
4. What is consistent hashing and how is it used in load balancing?

---

### Problem 22: Network Topologies
**Scenario**: Choose appropriate network topology for different scenarios.

**Requirements**:
- Understand different topologies
- Compare advantages and disadvantages
- Choose topology based on requirements
- Understand scalability

**Questions**:
1. What are the different network topologies?
2. What are the advantages and disadvantages of star topology?
3. How does mesh topology provide redundancy?
4. When would you use bus topology vs ring topology?

---

### Problem 23: Subnetting and CIDR
**Scenario**: Design IP addressing scheme using subnetting.

**Requirements**:
- Understand IP addressing
- Understand subnetting
- Calculate subnet masks
- Design network addressing

**Questions**:
1. What is subnetting and why is it used?
2. How do you calculate the number of subnets and hosts?
3. What is CIDR notation?
4. How do you determine if two IP addresses are in the same subnet?

---

### Problem 24: Routing Algorithms
**Scenario**: Implement routing algorithms for network packet forwarding.

**Requirements**:
- Understand routing concepts
- Implement distance vector routing
- Implement link state routing
- Compare routing algorithms

**Questions**:
1. What is the difference between distance vector and link state routing?
2. How does RIP (Routing Information Protocol) work?
3. How does OSPF (Open Shortest Path First) work?
4. What is the count-to-infinity problem and how is it solved?

---

### Problem 25: Network Protocols - ARP, ICMP, DHCP
**Scenario**: Explain how different network protocols work together.

**Requirements**:
- Understand ARP protocol
- Understand ICMP protocol
- Understand DHCP protocol
- Understand protocol interactions

**Questions**:
1. How does ARP (Address Resolution Protocol) work?
2. What is ICMP used for?
3. How does DHCP assign IP addresses?
4. What happens when a device first connects to a network?

---

### Problem 26: Firewall and Network Security
**Scenario**: Design network security using firewalls and access control.

**Requirements**:
- Understand firewall types
- Implement packet filtering
- Understand stateful inspection
- Design security policies

**Questions**:
1. What is a firewall and how does it work?
2. What is the difference between packet filtering and stateful inspection?
3. What is a DMZ (Demilitarized Zone) and why is it used?
4. How do you configure firewall rules for a web server?

---

### Problem 27: Network Congestion Control
**Scenario**: Implement congestion control mechanisms.

**Requirements**:
- Understand congestion causes
- Implement TCP congestion control
- Understand congestion avoidance
- Handle network congestion

**Questions**:
1. What causes network congestion?
2. How does TCP congestion control work?
3. What is the difference between slow start and congestion avoidance?
4. How does TCP handle packet loss?

---

### Problem 28: WebSocket vs HTTP
**Scenario**: Choose between WebSocket and HTTP for real-time communication.

**Requirements**:
- Understand WebSocket protocol
- Compare with HTTP
- Understand use cases
- Implement WebSocket communication

**Questions**:
1. What is WebSocket and how does it differ from HTTP?
2. When would you use WebSocket vs HTTP?
3. How does WebSocket handshake work?
4. What are the advantages of WebSocket for real-time applications?

---

### Problem 29: CDN and Content Distribution
**Scenario**: Design a content delivery network.

**Requirements**:
- Understand CDN concepts
- Design edge server placement
- Implement caching strategies
- Handle content distribution

**Questions**:
1. What is a CDN and how does it work?
2. How do CDNs reduce latency?
3. What is edge caching and how does it work?
4. How do you choose CDN server locations?

---

### Problem 30: Network Monitoring and Troubleshooting
**Scenario**: Diagnose and fix network issues.

**Requirements**:
- Use network diagnostic tools
- Understand network metrics
- Troubleshoot connectivity issues
- Monitor network performance

**Questions**:
1. What tools do you use to troubleshoot network issues?
2. How do you use ping and traceroute?
3. What do different HTTP status codes mean?
4. How do you diagnose slow network performance?

---

## Database Management Systems

### Problem 31: ACID Properties
**Scenario**: Ensure database transactions maintain ACID properties.

**Requirements**:
- Understand ACID properties
- Implement atomicity
- Ensure consistency
- Handle isolation and durability

**Questions**:
1. What do ACID properties stand for?
2. How does a database ensure atomicity?
3. What is the difference between consistency and integrity?
4. How does durability differ from persistence?

---

### Problem 32: Database Normalization
**Scenario**: Normalize a database schema to reduce redundancy.

**Requirements**:
- Understand normalization forms
- Normalize to 1NF, 2NF, 3NF
- Understand BCNF
- Balance normalization vs performance

**Questions**:
1. What is database normalization and why is it important?
2. What are the different normal forms?
3. How do you normalize a table to 3NF?
4. When might you denormalize a database?

---

### Problem 33: Indexing Strategies
**Scenario**: Design indexes to optimize query performance.

**Requirements**:
- Understand index types
- Design B-tree indexes
- Design hash indexes
- Choose appropriate indexes

**Questions**:
1. What is a database index and how does it work?
2. What is the difference between B-tree and hash indexes?
3. When should you create an index?
4. What is a composite index and when is it useful?

---

### Problem 34: SQL Joins
**Scenario**: Write efficient SQL queries using different join types.

**Requirements**:
- Understand INNER JOIN
- Understand LEFT/RIGHT OUTER JOIN
- Understand FULL OUTER JOIN
- Optimize join performance

**Questions**:
1. What are the different types of SQL joins?
2. What is the difference between INNER JOIN and LEFT JOIN?
3. When would you use a FULL OUTER JOIN?
4. How do you optimize join queries?

---

### Problem 35: Transaction Isolation Levels
**Scenario**: Choose appropriate isolation level for different scenarios.

**Requirements**:
- Understand isolation levels
- Understand concurrency problems
- Choose isolation level
- Balance consistency vs performance

**Questions**:
1. What are the different transaction isolation levels?
2. What concurrency problems does each level prevent?
3. What is the difference between READ COMMITTED and REPEATABLE READ?
4. When would you use SERIALIZABLE isolation level?

---

### Problem 36: Database Locking
**Scenario**: Implement locking mechanisms to prevent data inconsistencies.

**Requirements**:
- Understand shared locks
- Understand exclusive locks
- Implement deadlock prevention
- Handle lock escalation

**Questions**:
1. What are the different types of database locks?
2. What is the difference between shared lock and exclusive lock?
3. How do databases prevent deadlocks?
4. What is lock escalation and when does it occur?

---

### Problem 37: Query Optimization
**Scenario**: Optimize slow database queries.

**Requirements**:
- Analyze query execution plans
- Optimize query structure
- Use indexes effectively
- Handle query performance

**Questions**:
1. How do you analyze a query execution plan?
2. What are common query optimization techniques?
3. How do you optimize a query with multiple joins?
4. What is query caching and how does it help?

---

### Problem 38: Database Replication
**Scenario**: Design database replication for high availability.

**Requirements**:
- Understand replication types
- Implement master-slave replication
- Handle replication lag
- Ensure consistency

**Questions**:
1. What is database replication and why is it used?
2. What is the difference between master-slave and master-master replication?
3. How do you handle replication lag?
4. How do you ensure consistency in replicated databases?

---

### Problem 39: CAP Theorem and Database Consistency
**Scenario**: Choose database based on CAP theorem requirements.

**Requirements**:
- Understand CAP theorem
- Understand consistency models
- Choose appropriate database
- Balance CAP properties

**Questions**:
1. What does CAP theorem state?
2. What are the trade-offs between consistency, availability, and partition tolerance?
3. How do SQL databases differ from NoSQL databases in terms of CAP?
4. When would you choose eventual consistency over strong consistency?

---

### Problem 40: Database Sharding
**Scenario**: Design database sharding strategy for scalability.

**Requirements**:
- Understand sharding concepts
- Design sharding key
- Implement horizontal partitioning
- Handle cross-shard queries

**Questions**:
1. What is database sharding and why is it used?
2. How do you choose a sharding key?
3. What are the challenges of database sharding?
4. How do you handle queries that span multiple shards?

---

### Problem 41: NoSQL Database Types
**Scenario**: Choose appropriate NoSQL database for different use cases.

**Requirements**:
- Understand document databases
- Understand key-value stores
- Understand column-family stores
- Understand graph databases

**Questions**:
1. What are the different types of NoSQL databases?
2. When would you use a document database vs a key-value store?
3. What are the advantages of column-family databases?
4. When would you use a graph database?

---

### Problem 42: Database Backup and Recovery
**Scenario**: Design backup and recovery strategy.

**Requirements**:
- Understand backup types
- Implement full backups
- Implement incremental backups
- Design recovery procedures

**Questions**:
1. What are the different types of database backups?
2. What is the difference between full backup and incremental backup?
3. How do you perform point-in-time recovery?
4. What is a database checkpoint?

---

### Problem 43: Database Connection Pooling
**Scenario**: Optimize database connections using connection pooling.

**Requirements**:
- Understand connection pooling
- Configure pool size
- Handle connection failures
- Optimize connection usage

**Questions**:
1. What is connection pooling and why is it important?
2. How do you determine the optimal pool size?
3. What happens when the connection pool is exhausted?
4. How do you handle connection failures in a pool?

---

### Problem 44: Database Views and Materialized Views
**Scenario**: Use views to simplify complex queries.

**Requirements**:
- Understand database views
- Understand materialized views
- Compare views vs tables
- Optimize view performance

**Questions**:
1. What is a database view and how does it differ from a table?
2. What is a materialized view and when would you use it?
3. Can you update a view?
4. How do views affect query performance?

---

### Problem 45: Database Triggers and Stored Procedures
**Scenario**: Implement business logic using triggers and stored procedures.

**Requirements**:
- Understand database triggers
- Understand stored procedures
- Compare triggers vs application logic
- Handle trigger execution

**Questions**:
1. What are database triggers and when are they used?
2. What is the difference between BEFORE and AFTER triggers?
3. What are stored procedures and what are their advantages?
4. When should you use triggers vs application-level logic?

---

### Problem 46: Database Security
**Scenario**: Implement database security measures.

**Requirements**:
- Understand authentication
- Implement authorization
- Encrypt sensitive data
- Audit database access

**Questions**:
1. How do you secure a database?
2. What is the difference between authentication and authorization?
3. How do you encrypt data at rest vs data in transit?
4. What is SQL injection and how do you prevent it?

---

### Problem 47: Database Transactions and Concurrency
**Scenario**: Handle concurrent transactions correctly.

**Requirements**:
- Understand transaction management
- Handle concurrent access
- Prevent lost updates
- Implement optimistic locking

**Questions**:
1. How do databases handle concurrent transactions?
2. What is a lost update problem and how is it prevented?
3. What is optimistic locking vs pessimistic locking?
4. How does MVCC (Multi-Version Concurrency Control) work?

---

### Problem 48: Database Index Types
**Scenario**: Choose appropriate index type for different scenarios.

**Requirements**:
- Understand B-tree indexes
- Understand hash indexes
- Understand bitmap indexes
- Understand full-text indexes

**Questions**:
1. What are the different types of database indexes?
2. When would you use a B-tree index vs a hash index?
3. What is a bitmap index and when is it useful?
4. How do full-text indexes work?

---

### Problem 49: Database Partitioning
**Scenario**: Partition large tables for better performance.

**Requirements**:
- Understand partitioning types
- Implement range partitioning
- Implement hash partitioning
- Handle partition pruning

**Questions**:
1. What is database partitioning and why is it used?
2. What are the different types of partitioning?
3. How does range partitioning work?
4. What is partition pruning and how does it improve performance?

---

### Problem 50: Database Performance Tuning
**Scenario**: Optimize overall database performance.

**Requirements**:
- Identify performance bottlenecks
- Optimize queries
- Tune database configuration
- Monitor database performance

**Questions**:
1. How do you identify database performance bottlenecks?
2. What database configuration parameters affect performance?
3. How do you optimize database I/O?
4. What metrics do you monitor for database performance?

---

## Summary

This comprehensive guide covers all major CS fundamentals topics for SDE 2 interviews:

1. **Operating Systems**: Processes, threads, scheduling, memory management, IPC, synchronization, file systems
2. **Networking**: OSI/TCP-IP, protocols, routing, load balancing, security, CDN
3. **Database Management Systems**: ACID, normalization, indexing, transactions, replication, sharding, NoSQL

**Total Coverage: 50 Problems covering OS, Networking, and DBMS**

Each problem includes scenario, requirements, and interview-style questions to help you prepare comprehensively.
