# Networking Cheat Sheet (SDE 2 Interviews)

## OSI / TCP-IP (rapid mapping)
- **OSI**: Physical, Data Link, Network, Transport, Session, Presentation, Application
- **TCP/IP**: Link, Internet, Transport, Application
- **Encapsulation**: App data → TCP/UDP segment → IP packet → L2 frame → bits

## TCP vs UDP (what to say)
- **TCP**: connection-oriented, reliable, ordered, flow + congestion control.
- **UDP**: connectionless, best-effort, unordered; reliability must be added at app layer.
- **Use TCP**: web/app APIs, payments, file transfer. **Use UDP**: DNS, streaming, gaming, VoIP (latency wins).

## TCP handshake / teardown (must know)
- **3-way handshake**: SYN → SYN-ACK → ACK (establishes initial seq numbers).
- **4-way close**: FIN/ACK in each direction; half-close possible.
- **TIME_WAIT**: side that actively closes waits to absorb delayed packets (2×MSL).
- **SYN flood**: too many half-open conns; mitigations: SYN cookies, rate limiting, backlog tuning.

## Reliability (core mechanisms)
- **Seq numbers + ACKs**: detect loss/reordering.
- **Retransmit**: timeout or fast retransmit (dup ACKs).
- **Sliding window**: multiple in-flight segments.

## Flow vs congestion control (interview trap)
- **Flow control (rwnd)**: protects receiver buffer (end-to-end).
- **Congestion control (cwnd)**: protects the network (sender adapts).
- **Slow start**: cwnd grows exponentially until ssthresh.
- **Congestion avoidance**: additive increase, multiplicative decrease (AIMD).

## HTTP essentials
- **HTTP is stateless** (state via cookies/tokens).
- **Methods**:
  - **GET** (safe, idempotent),
  - **POST** (not idempotent),
  - **PUT** (idempotent replace),
  - **PATCH** (partial update; not guaranteed idempotent),
  - **DELETE** (idempotent).
- **Common status codes**: 200, 201, 204, 301/302, 304, 400, 401, 403, 404, 409, 429, 500, 502, 503, 504.

## HTTP/1.1 vs HTTP/2 vs HTTP/3 (high-level)
- **HTTP/1.1**: keep-alive helps; head-of-line blocking at request pipelining level.
- **HTTP/2**: multiplexing streams over 1 TCP conn, HPACK header compression, server push (rarely used now).
- **HTTP/3**: runs over **QUIC (UDP)** → avoids TCP HOL, faster handshake (TLS 1.3 integrated).

## HTTPS / TLS (what interviewers expect)
- **TLS provides**: confidentiality (encryption), integrity (MAC/AEAD), authentication (certs).
- **Certificate chain**: leaf → intermediate(s) → root CA (trusted).
- **Common terms**:
  - **SNI**: client indicates hostname in TLS handshake.
  - **ALPN**: negotiate app protocol (e.g., h2).
- **mTLS**: client also presents cert (service-to-service auth).

## HTTP caching (practical)
- **Cache-Control**: `max-age`, `public/private`, `no-cache` (revalidate), `no-store`.
- **ETag / If-None-Match** and **Last-Modified / If-Modified-Since**: validation → 304 Not Modified.
- **CDN**: caches at edge; invalidation is expensive—prefer versioned assets.

## DNS (must know flow)
- **Recursive resolver** asks: root → TLD → authoritative → returns record; caches by TTL.
- **Records**: A/AAAA, CNAME, MX, NS, TXT, SRV.
- **Common debugging**: stale cache/TTL, wrong CNAME chain, NXDOMAIN, split-horizon DNS.
- **Security**: **DNSSEC** signs records; **DoH/DoT** encrypts transport to resolver.

## IP addressing / subnetting (quick math)
- **CIDR**: `a.b.c.d/len` where `len` = network bits.
- **Hosts per subnet**: $2^{(32-len)} - 2$ (IPv4).
- **Private ranges**: 10/8, 172.16/12, 192.168/16.

## NAT / PAT (common system design detail)
- **NAT**: translate private ↔ public IPs.
- **PAT**: many internal hosts share one public IP via port mapping.
- **NAT traversal**: STUN/TURN/ICE (relevant for WebRTC).

## Switching / VLAN / STP (LAN basics)
- **Switch**: L2; learns MAC→port table; floods unknown unicast/broadcast.
- **VLAN**: splits broadcast domains on same physical switch (802.1Q tags).
- **STP/RSTP**: prevents loops by blocking redundant links; elects root bridge.

## Routing (what to mention)
- **Routing table**: longest-prefix match picks next hop.
- **RIP**: distance-vector (hop count), slower convergence.
- **OSPF**: link-state + Dijkstra, faster convergence.
- (Optional to know) **BGP**: inter-domain, policy-based (internet backbone).

## Load balancers (L4 vs L7)
- **L4**: TCP/UDP forwarding; faster; no app awareness.
- **L7**: HTTP-aware routing (path/host/header); supports auth, rate limit, rewrites.
- **Sticky sessions**: cookie/IP hash; avoid if possible (prefer stateless services).
- **Health checks**: active probes + passive failure detection; drain connections on shutdown.
- **Consistent hashing**: stable mapping for caches/shards.

## Performance knobs (high signal)
- **Latency vs bandwidth**: latency is time; bandwidth is capacity.
- **MTU**: max L2 payload; avoid fragmentation.
- **MSS**: TCP payload size (derived from MTU).
- Symptoms:
  - High RTT: distance/congestion.
  - Loss: congestion/bad links.
  - Jitter: queueing/contended paths.

## Troubleshooting playbook (say this)
- **Name resolution**: `dig`/`nslookup`.
- **Reachability**: `ping` (ICMP), `traceroute`/`mtr`.
- **Ports/conns**: `ss`/`netstat`, `lsof -i`.
- **HTTP**: `curl -v`, check status/headers.
- **Packets**: `tcpdump`, Wireshark.
- Approach: “DNS → TCP connect → TLS → HTTP → app logs/metrics.”

