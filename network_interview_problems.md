# Networking Interview Problems for SDE 2

## Table of Contents
1. [Network Fundamentals](#network-fundamentals)
2. [OSI and TCP/IP Models](#osi-and-tcpip-models)
3. [TCP and UDP](#tcp-and-udp)
4. [HTTP and HTTPS](#http-and-https)
5. [DNS and Name Resolution](#dns-and-name-resolution)
6. [Load Balancing](#load-balancing)
7. [Network Security](#network-security)
8. [Routing and Switching](#routing-and-switching)
9. [Network Protocols](#network-protocols)
10. [Network Topologies](#network-topologies)
11. [Subnetting and IP Addressing](#subnetting-and-ip-addressing)
12. [Network Performance](#network-performance)
13. [CDN and Content Delivery](#cdn-and-content-delivery)
14. [WebSocket and Real-Time Communication](#websocket-and-real-time-communication)
15. [Network Troubleshooting](#network-troubleshooting)

---

## Network Fundamentals

### Problem 1: OSI Model Layers
**Scenario**: Explain network communication using the OSI 7-layer model.

**Requirements**:
- Understand all 7 layers of OSI model
- Explain functions of each layer
- Map protocols to layers
- Understand data encapsulation

**Questions**:
1. What are the seven layers of the OSI model?
2. What is the function of each layer?
3. How does data encapsulation work across layers?
4. What protocols operate at each layer?

---

### Problem 2: TCP/IP Model
**Scenario**: Compare TCP/IP model with OSI model and explain protocol stack.

**Requirements**:
- Understand TCP/IP 4-layer model
- Map TCP/IP to OSI model
- Understand protocol relationships
- Explain internetworking

**Questions**:
1. What are the four layers of the TCP/IP model?
2. How does TCP/IP map to the OSI model?
3. What is the difference between TCP/IP and OSI models?
4. What protocols are used at each TCP/IP layer?

---

### Problem 3: Network Topologies
**Scenario**: Design network topology for different scenarios.

**Requirements**:
- Understand different network topologies
- Compare advantages and disadvantages
- Choose appropriate topology
- Handle scalability

**Questions**:
1. What are the different network topologies?
2. What are the advantages and disadvantages of star topology?
3. How does mesh topology provide redundancy?
4. When would you use bus topology vs ring topology?

---

### Problem 4: Network Devices
**Scenario**: Choose appropriate network devices for different network segments.

**Requirements**:
- Understand hubs, switches, routers
- Understand bridges and gateways
- Choose devices based on requirements
- Understand device functions

**Questions**:
1. What is the difference between a hub and a switch?
2. How does a router differ from a switch?
3. What is a bridge and when is it used?
4. What is the function of a gateway?

---

## OSI and TCP/IP Models

### Problem 5: Physical Layer
**Scenario**: Design physical layer implementation for network communication.

**Requirements**:
- Understand physical transmission media
- Understand signal encoding
- Handle transmission errors
- Optimize physical layer

**Questions**:
1. What are the different types of transmission media?
2. How does signal encoding work?
3. What is the difference between baseband and broadband?
4. How do you handle transmission errors at physical layer?

---

### Problem 6: Data Link Layer
**Scenario**: Implement data link layer protocols for reliable frame transmission.

**Requirements**:
- Understand framing
- Implement error detection (CRC, checksum)
- Implement flow control
- Handle media access control

**Questions**:
1. What is framing and why is it needed?
2. How does CRC (Cyclic Redundancy Check) work?
3. What is the difference between CSMA/CD and CSMA/CA?
4. How does the data link layer provide error detection?

---

### Problem 7: Network Layer
**Scenario**: Design network layer for packet routing and forwarding.

**Requirements**:
- Understand IP addressing
- Implement routing algorithms
- Handle packet fragmentation
- Understand IP protocol

**Questions**:
1. What is the function of the network layer?
2. How does IP routing work?
3. What is packet fragmentation and reassembly?
4. How does the network layer handle congestion?

---

### Problem 8: Transport Layer
**Scenario**: Compare TCP and UDP for different application requirements.

**Requirements**:
- Understand TCP characteristics
- Understand UDP characteristics
- Choose appropriate protocol
- Understand port numbers

**Questions**:
1. What are the functions of the transport layer?
2. What is the difference between TCP and UDP?
3. How do port numbers work?
4. When would you use TCP vs UDP?

---

### Problem 9: Session Layer
**Scenario**: Implement session management for network applications.

**Requirements**:
- Understand session establishment
- Handle session maintenance
- Implement session termination
- Understand session recovery

**Questions**:
1. What is the function of the session layer?
2. How does session establishment work?
3. What is session synchronization?
4. How do you handle session failures?

---

### Problem 10: Presentation Layer
**Scenario**: Implement data encoding and encryption at presentation layer.

**Requirements**:
- Understand data encoding (ASCII, Unicode)
- Implement data compression
- Handle encryption/decryption
- Understand data translation

**Questions**:
1. What is the function of the presentation layer?
2. How does data encoding work?
3. What is the difference between encryption and encoding?
4. How does data compression work?

---

### Problem 11: Application Layer
**Scenario**: Design application layer protocols for different services.

**Requirements**:
- Understand application protocols (HTTP, FTP, SMTP)
- Design custom protocols
- Handle application-level errors
- Understand client-server model

**Questions**:
1. What protocols operate at the application layer?
2. How does HTTP work at the application layer?
3. What is the difference between stateless and stateful protocols?
4. How do you design an application layer protocol?

---

## TCP and UDP

### Problem 12: TCP Connection Management
**Scenario**: Explain TCP three-way handshake and connection termination.

**Requirements**:
- Understand TCP handshake process
- Understand connection termination
- Handle connection states
- Understand TCP flags

**Questions**:
1. How does the TCP three-way handshake work?
2. How does TCP connection termination work (four-way handshake)?
3. What are the different TCP connection states?
4. What is a SYN flood attack and how is it prevented?

---

### Problem 13: TCP Reliability Mechanisms
**Scenario**: Implement TCP reliability features (acknowledgment, retransmission).

**Requirements**:
- Understand TCP acknowledgment
- Implement retransmission mechanism
- Handle sequence numbers
- Understand sliding window

**Questions**:
1. How does TCP ensure reliable delivery?
2. How does TCP acknowledgment work?
3. What is the sliding window protocol?
4. How does TCP handle packet loss?

---

### Problem 14: TCP Flow Control
**Scenario**: Implement TCP flow control to prevent receiver overflow.

**Requirements**:
- Understand receiver window
- Implement window advertisement
- Handle zero window
- Optimize flow control

**Questions**:
1. What is TCP flow control?
2. How does the receiver window work?
3. What happens when the receiver window is zero?
4. How does flow control differ from congestion control?

---

### Problem 15: TCP Congestion Control
**Scenario**: Implement TCP congestion control algorithms.

**Requirements**:
- Understand congestion causes
- Implement slow start
- Implement congestion avoidance
- Handle congestion detection

**Questions**:
1. What causes network congestion?
2. How does TCP slow start work?
3. What is the difference between slow start and congestion avoidance?
4. How does TCP detect congestion?

---

### Problem 16: UDP Characteristics
**Scenario**: Design UDP-based application with reliability mechanisms.

**Requirements**:
- Understand UDP characteristics
- Implement application-level reliability
- Handle UDP limitations
- Optimize UDP performance

**Questions**:
1. What are the characteristics of UDP?
2. When would you use UDP instead of TCP?
3. How do you implement reliability on top of UDP?
4. What are the advantages of UDP over TCP?

---

## HTTP and HTTPS

### Problem 17: HTTP Protocol
**Scenario**: Design HTTP client-server communication system.

**Requirements**:
- Understand HTTP methods
- Handle HTTP headers
- Implement request-response cycle
- Understand HTTP versions

**Questions**:
1. How does HTTP work?
2. What are the different HTTP methods?
3. What is the difference between HTTP/1.0 and HTTP/1.1?
4. How does HTTP/2 differ from HTTP/1.1?

---

### Problem 18: HTTP Methods and Status Codes
**Scenario**: Implement RESTful API using appropriate HTTP methods and status codes.

**Requirements**:
- Use correct HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes
- Handle idempotency
- Implement proper error handling

**Questions**:
1. What are the different HTTP methods and when to use them?
2. What do different HTTP status codes mean?
3. What is the difference between PUT and PATCH?
4. What is idempotency and which methods are idempotent?

---

### Problem 19: HTTPS and SSL/TLS
**Scenario**: Implement secure HTTP communication using SSL/TLS.

**Requirements**:
- Understand SSL/TLS handshake
- Implement certificate validation
- Handle encryption/decryption
- Understand certificate chain

**Questions**:
1. What is the difference between HTTP and HTTPS?
2. How does SSL/TLS provide security?
3. How does the SSL/TLS handshake work?
4. What is a certificate authority and how does it work?

---

### Problem 20: HTTP Caching
**Scenario**: Design HTTP caching strategy for web applications.

**Requirements**:
- Understand cache headers
- Implement cache validation
- Handle cache invalidation
- Optimize cache performance

**Questions**:
1. How does HTTP caching work?
2. What are the different cache headers?
3. What is the difference between cache-control and expires?
4. How does cache validation work (ETag, Last-Modified)?

---

### Problem 21: HTTP/2 and HTTP/3
**Scenario**: Explain improvements in HTTP/2 and HTTP/3.

**Requirements**:
- Understand HTTP/2 features
- Understand HTTP/3 features
- Compare with HTTP/1.1
- Implement HTTP/2 server push

**Questions**:
1. What are the key features of HTTP/2?
2. How does HTTP/2 multiplexing work?
3. What is HTTP/3 and how does it differ from HTTP/2?
4. What is server push in HTTP/2?

---

## DNS and Name Resolution

### Problem 22: DNS Resolution Process
**Scenario**: Explain how domain names are resolved to IP addresses.

**Requirements**:
- Understand DNS hierarchy
- Understand DNS resolution process
- Handle DNS caching
- Understand DNS record types

**Questions**:
1. How does DNS resolution work?
2. What are the different types of DNS records?
3. What is the difference between iterative and recursive DNS queries?
4. How does DNS caching work?

---

### Problem 23: DNS Record Types
**Scenario**: Configure DNS records for different services.

**Requirements**:
- Understand A, AAAA, CNAME records
- Understand MX, TXT, NS records
- Configure DNS for web and email
- Handle DNS TTL

**Questions**:
1. What are the different DNS record types?
2. What is the difference between A and AAAA records?
3. When would you use CNAME vs A record?
4. What is DNS TTL and why is it important?

---

### Problem 24: DNS Security
**Scenario**: Implement DNS security measures (DNSSEC, DNS over HTTPS).

**Requirements**:
- Understand DNS vulnerabilities
- Implement DNSSEC
- Understand DNS over HTTPS (DoH)
- Handle DNS spoofing prevention

**Questions**:
1. What are the security vulnerabilities of DNS?
2. How does DNSSEC work?
3. What is DNS over HTTPS (DoH)?
4. How do you prevent DNS spoofing?

---

## Load Balancing

### Problem 25: Load Balancing Algorithms
**Scenario**: Design load balancer with different distribution strategies.

**Requirements**:
- Implement round-robin algorithm
- Implement least connections algorithm
- Implement weighted algorithms
- Compare load balancing strategies

**Questions**:
1. What is load balancing and why is it needed?
2. How does round-robin load balancing work?
3. What is the difference between least connections and weighted least connections?
4. What is consistent hashing and how is it used in load balancing?

---

### Problem 26: Load Balancer Types
**Scenario**: Choose appropriate load balancer type (Layer 4 vs Layer 7).

**Requirements**:
- Understand Layer 4 load balancing
- Understand Layer 7 load balancing
- Compare performance and features
- Choose based on requirements

**Questions**:
1. What is the difference between Layer 4 and Layer 7 load balancing?
2. When would you use Layer 4 vs Layer 7 load balancing?
3. What are the performance implications of each?
4. How does SSL termination work in load balancers?

---

### Problem 27: Health Checks and Failover
**Scenario**: Implement health checks and automatic failover for load balancer.

**Requirements**:
- Implement health check mechanisms
- Handle server failures
- Implement failover strategies
- Monitor backend health

**Questions**:
1. How do load balancer health checks work?
2. What happens when a backend server fails?
3. How do you implement graceful failover?
4. What is the difference between active and passive health checks?

---

## Network Security

### Problem 28: Firewall Configuration
**Scenario**: Design firewall rules for network security.

**Requirements**:
- Understand firewall types
- Implement packet filtering rules
- Understand stateful inspection
- Design security policies

**Questions**:
1. What is a firewall and how does it work?
2. What is the difference between packet filtering and stateful inspection?
3. What is a DMZ (Demilitarized Zone) and why is it used?
4. How do you configure firewall rules for a web server?

---

### Problem 29: VPN and Tunneling
**Scenario**: Implement VPN solution for secure remote access.

**Requirements**:
- Understand VPN protocols (IPsec, SSL VPN)
- Implement tunneling mechanisms
- Handle encryption
- Understand VPN types

**Questions**:
1. What is a VPN and how does it work?
2. What are the different VPN protocols?
3. How does IPsec tunneling work?
4. What is the difference between site-to-site and remote access VPN?

---

### Problem 30: Network Attacks and Prevention
**Scenario**: Implement defenses against common network attacks.

**Requirements**:
- Understand DDoS attacks
- Understand man-in-the-middle attacks
- Implement attack prevention
- Handle network intrusion detection

**Questions**:
1. What are common network attacks?
2. How do you prevent DDoS attacks?
3. What is a man-in-the-middle attack and how is it prevented?
4. How does intrusion detection work?

---

## Routing and Switching

### Problem 31: IP Routing
**Scenario**: Design IP routing system for network packet forwarding.

**Requirements**:
- Understand routing tables
- Implement routing algorithms
- Handle route selection
- Understand routing protocols

**Questions**:
1. How does IP routing work?
2. What is a routing table?
3. How do routers select the best route?
4. What are the different routing protocols?

---

### Problem 32: Routing Algorithms
**Scenario**: Implement distance vector and link state routing algorithms.

**Requirements**:
- Implement distance vector routing
- Implement link state routing
- Compare routing algorithms
- Handle routing convergence

**Questions**:
1. What is the difference between distance vector and link state routing?
2. How does RIP (Routing Information Protocol) work?
3. How does OSPF (Open Shortest Path First) work?
4. What is the count-to-infinity problem and how is it solved?

---

### Problem 33: Network Switching
**Scenario**: Design switching system for local area networks.

**Requirements**:
- Understand switch operation
- Implement MAC address learning
- Handle broadcast domains
- Understand VLANs

**Questions**:
1. How does a network switch work?
2. How does a switch learn MAC addresses?
3. What is the difference between a switch and a hub?
4. What are VLANs and how do they work?

---

### Problem 34: Spanning Tree Protocol
**Scenario**: Implement Spanning Tree Protocol to prevent loops in switched networks.

**Requirements**:
- Understand STP algorithm
- Handle bridge election
- Prevent loops
- Optimize STP convergence

**Questions**:
1. What is Spanning Tree Protocol and why is it needed?
2. How does STP prevent loops?
3. How does STP elect a root bridge?
4. What is the difference between STP and RSTP?

---

## Network Protocols

### Problem 35: ARP Protocol
**Scenario**: Explain Address Resolution Protocol for IP to MAC mapping.

**Requirements**:
- Understand ARP operation
- Handle ARP cache
- Understand ARP spoofing
- Implement ARP security

**Questions**:
1. How does ARP (Address Resolution Protocol) work?
2. What is an ARP cache?
3. What is ARP spoofing and how is it prevented?
4. What is the difference between ARP and RARP?

---

### Problem 36: ICMP Protocol
**Scenario**: Use ICMP for network diagnostics and error reporting.

**Requirements**:
- Understand ICMP message types
- Implement ping using ICMP
- Handle ICMP errors
- Understand traceroute

**Questions**:
1. What is ICMP and what is it used for?
2. How does ping work using ICMP?
3. How does traceroute use ICMP?
4. What are the different ICMP message types?

---

### Problem 37: DHCP Protocol
**Scenario**: Design DHCP server for automatic IP address assignment.

**Requirements**:
- Understand DHCP process
- Implement DHCP server
- Handle IP address leasing
- Understand DHCP options

**Questions**:
1. How does DHCP assign IP addresses?
2. What is the DHCP four-step process?
3. How does DHCP lease renewal work?
4. What are DHCP options and how are they used?

---

### Problem 38: NAT and PAT
**Scenario**: Implement Network Address Translation for private networks.

**Requirements**:
- Understand NAT operation
- Implement PAT (Port Address Translation)
- Handle NAT traversal
- Understand NAT types

**Questions**:
1. What is NAT and why is it used?
2. How does NAT work?
3. What is the difference between NAT and PAT?
4. How does NAT traversal work for applications?

---

## Network Topologies

### Problem 39: LAN Design
**Scenario**: Design Local Area Network for office environment.

**Requirements**:
- Choose appropriate topology
- Design network hierarchy
- Plan IP addressing
- Handle network segmentation

**Questions**:
1. How do you design a LAN?
2. What factors influence LAN topology choice?
3. How do you segment a LAN?
4. What is the difference between LAN and WAN?

---

### Problem 40: WAN Technologies
**Scenario**: Choose WAN technology for connecting remote offices.

**Requirements**:
- Understand WAN technologies
- Compare leased lines, MPLS, VPN
- Choose based on requirements
- Handle WAN optimization

**Questions**:
1. What are the different WAN technologies?
2. What is MPLS and how does it work?
3. What is the difference between leased lines and VPN?
4. How do you optimize WAN performance?

---

## Subnetting and IP Addressing

### Problem 41: IP Addressing and Subnetting
**Scenario**: Design IP addressing scheme using subnetting.

**Requirements**:
- Understand IP address classes
- Implement subnetting
- Calculate subnet masks
- Design network addressing

**Questions**:
1. What is subnetting and why is it used?
2. How do you calculate the number of subnets and hosts?
3. What is CIDR notation?
4. How do you determine if two IP addresses are in the same subnet?

---

### Problem 42: IPv4 vs IPv6
**Scenario**: Compare IPv4 and IPv6 and plan migration strategy.

**Requirements**:
- Understand IPv4 limitations
- Understand IPv6 features
- Plan IPv6 migration
- Handle dual-stack deployment

**Questions**:
1. What are the limitations of IPv4?
2. What are the key features of IPv6?
3. How do you migrate from IPv4 to IPv6?
4. What is dual-stack and how does it work?

---

### Problem 43: Private IP Addresses
**Scenario**: Design private IP addressing scheme for internal networks.

**Requirements**:
- Understand private IP ranges
- Implement NAT for private networks
- Handle address conflicts
- Design addressing hierarchy

**Questions**:
1. What are private IP address ranges?
2. Why are private IP addresses used?
3. How do private IP addresses work with NAT?
4. How do you avoid IP address conflicts?

---

## Network Performance

### Problem 44: Network Latency
**Scenario**: Measure and optimize network latency.

**Requirements**:
- Understand latency components
- Measure network latency
- Optimize for low latency
- Handle latency in applications

**Questions**:
1. What causes network latency?
2. How do you measure network latency?
3. How do you reduce network latency?
4. What is the difference between latency and bandwidth?

---

### Problem 45: Bandwidth and Throughput
**Scenario**: Optimize network bandwidth utilization and throughput.

**Requirements**:
- Understand bandwidth vs throughput
- Measure network throughput
- Optimize bandwidth usage
- Handle bandwidth limitations

**Questions**:
1. What is the difference between bandwidth and throughput?
2. How do you measure network throughput?
3. How do you optimize bandwidth usage?
4. What factors affect network throughput?

---

### Problem 46: Network Monitoring
**Scenario**: Implement network monitoring and performance analysis.

**Requirements**:
- Monitor network metrics
- Implement SNMP monitoring
- Handle network alerts
- Analyze network performance

**Questions**:
1. What metrics should you monitor in a network?
2. How does SNMP work for network monitoring?
3. What tools are used for network monitoring?
4. How do you identify network performance bottlenecks?

---

## CDN and Content Delivery

### Problem 47: CDN Architecture
**Scenario**: Design Content Delivery Network for global content distribution.

**Requirements**:
- Understand CDN architecture
- Design edge server placement
- Implement caching strategies
- Handle content distribution

**Questions**:
1. What is a CDN and how does it work?
2. How do CDNs reduce latency?
3. What is edge caching and how does it work?
4. How do you choose CDN server locations?

---

### Problem 48: CDN Caching Strategies
**Scenario**: Implement caching strategies for CDN content delivery.

**Requirements**:
- Understand cache headers
- Implement cache invalidation
- Handle cache warming
- Optimize cache hit rates

**Questions**:
1. How does CDN caching work?
2. How do you invalidate CDN cache?
3. What is cache warming and why is it important?
4. How do you optimize CDN cache hit rates?

---

## WebSocket and Real-Time Communication

### Problem 49: WebSocket Protocol
**Scenario**: Implement real-time communication using WebSocket.

**Requirements**:
- Understand WebSocket handshake
- Implement WebSocket server
- Handle WebSocket frames
- Manage WebSocket connections

**Questions**:
1. What is WebSocket and how does it differ from HTTP?
2. How does WebSocket handshake work?
3. When would you use WebSocket vs HTTP?
4. What are the advantages of WebSocket for real-time applications?

---

### Problem 50: Real-Time Communication Patterns
**Scenario**: Design real-time communication system for chat application.

**Requirements**:
- Choose between WebSocket, Server-Sent Events, polling
- Handle connection management
- Implement message queuing
- Handle reconnection

**Questions**:
1. What are the different real-time communication patterns?
2. What is the difference between WebSocket and Server-Sent Events?
3. When would you use long polling vs WebSocket?
4. How do you handle WebSocket reconnection?

---

## Network Troubleshooting

### Problem 51: Network Diagnostic Tools
**Scenario**: Use network tools to diagnose connectivity issues.

**Requirements**:
- Use ping for connectivity testing
- Use traceroute for path discovery
- Use netstat for connection monitoring
- Use Wireshark for packet analysis

**Questions**:
1. What tools do you use to troubleshoot network issues?
2. How do you use ping and traceroute?
3. What is netstat used for?
4. How do you use Wireshark for packet analysis?

---

### Problem 52: Common Network Issues
**Scenario**: Diagnose and fix common network problems.

**Requirements**:
- Identify connectivity issues
- Diagnose DNS problems
- Handle routing issues
- Fix firewall problems

**Questions**:
1. How do you diagnose network connectivity issues?
2. How do you troubleshoot DNS problems?
3. How do you identify routing issues?
4. How do you debug firewall problems?

---

### Problem 53: Network Performance Troubleshooting
**Scenario**: Identify and fix network performance problems.

**Requirements**:
- Identify bottlenecks
- Measure network performance
- Optimize network configuration
- Handle congestion

**Questions**:
1. How do you identify network performance bottlenecks?
2. How do you measure network performance?
3. How do you optimize network configuration?
4. How do you handle network congestion?

---

### Problem 54: Packet Loss and Retransmission
**Scenario**: Diagnose and handle packet loss in network communication.

**Requirements**:
- Understand packet loss causes
- Measure packet loss
- Handle retransmission
- Optimize for packet loss

**Questions**:
1. What causes packet loss?
2. How do you measure packet loss?
3. How does TCP handle packet loss?
4. How do you optimize applications for packet loss?

---

### Problem 55: Network Security Troubleshooting
**Scenario**: Diagnose and fix network security issues.

**Requirements**:
- Identify security vulnerabilities
- Handle certificate issues
- Debug SSL/TLS problems
- Fix authentication issues

**Questions**:
1. How do you identify network security vulnerabilities?
2. How do you troubleshoot SSL/TLS certificate issues?
3. How do you debug authentication problems?
4. How do you identify and prevent network attacks?

---

## Summary

This comprehensive guide covers all major Networking topics for SDE 2 interviews:

1. **Network Fundamentals**: OSI model, TCP/IP, topologies, devices
2. **OSI Layers**: All 7 layers in detail
3. **TCP and UDP**: Connection management, reliability, flow control, congestion control
4. **HTTP and HTTPS**: Protocol, methods, caching, SSL/TLS, HTTP/2, HTTP/3
5. **DNS**: Resolution, record types, security
6. **Load Balancing**: Algorithms, types, health checks, failover
7. **Network Security**: Firewalls, VPN, attacks, prevention
8. **Routing and Switching**: IP routing, algorithms, switching, STP
9. **Network Protocols**: ARP, ICMP, DHCP, NAT
10. **Network Topologies**: LAN, WAN design
11. **IP Addressing**: Subnetting, IPv4 vs IPv6, private IPs
12. **Network Performance**: Latency, bandwidth, monitoring
13. **CDN**: Architecture, caching strategies
14. **WebSocket**: Real-time communication
15. **Troubleshooting**: Tools, common issues, performance, security

**Total Coverage: 55 Problems covering all Networking topics**

Each problem includes scenario, requirements, and interview-style questions to help you prepare comprehensively.
