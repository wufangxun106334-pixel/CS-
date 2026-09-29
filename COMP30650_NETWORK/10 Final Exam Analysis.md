# Final Exam Analysis

> Complete analysis and knowledge point summary based on 2022-23 and 2024-25 past papers.

---

## Exam Structure Comparison

| | 2022-23 | 2024-25 |
|---|---|---|
| Section A | Q1-Q4 choose 3 (20 marks each) | Q1-Q4 choose 3 (20 marks each) |
| Section B | Q5 compulsory (40 marks) | Q5 compulsory (40 marks) |
| Total | 100 marks | 100 marks |
| Duration | 2 hours | 2 hours |

---

## High-Frequency Topic Distribution

```
Physical Layer:   ████░░░░░░  Modulation, Encoding, Multiplexing (every year)
Link Layer:       ████████░░  Framing, ALOHA, CSMA, Flow Control (every year)
Network Layer:    ████████░░  IP Addressing, Subnetting, Routing, NAT (every year)
Transport Layer:  ██████░░░░  TCP/UDP, Congestion Control (every year)
Application Layer: ████░░░░░░  HTTP, DNS (occasional)
Security:         ████░░░░░░  Firewall, IPSec, TLS (new in 2425)
```

---

# Section A Analysis

## Q1: Network Fundamentals + Physical Layer (2223)

### (a) WAN, LAN, MAN Comparison

| Network Type | Scope | Typical Technology | Characteristics |
|---|---|---|---|
| **LAN** | Building/Campus | Ethernet, WiFi | High speed, low latency, self-managed |
| **MAN** | City-wide | Metro Ethernet | Medium distance, ISP-managed |
| **WAN** | Cross-city/Country | MPLS, Internet | Lower speed, high latency, leased |

**Direct Link:** A dedicated physical link between two devices, not passing through other devices.

**Protocol Layering:**
- **Advantages:** Modularity, independent development, easy debugging, standardisation
- **Disadvantages:** Redundant functions, overhead between layers, changes in one layer may affect adjacent layers

**OSI 7-Layer Model:**
```
7. Application     → HTTP, FTP, DNS
6. Presentation    → Encryption, Compression, Encoding
5. Session         → Session Management
4. Transport       → TCP, UDP
3. Network         → IP, ICMP, Routing
2. Data Link       → Ethernet, WiFi, Frames
1. Physical        → Bit Transmission, Signals
```

### (b) ASK, FSK, PSK Modulation

```
ASK (Amplitude Shift Keying): Represents 0/1 by amplitude
  0 → Low amplitude
  1 → High amplitude
  Characteristic: Simple, poor noise resistance

FSK (Frequency Shift Keying): Represents 0/1 by frequency
  0 → Low frequency
  1 → High frequency
  Characteristic: Better noise resistance

PSK (Phase Shift Keying): Represents 0/1 by phase
  0 → 0° phase
  1 → 180° phase
  Characteristic: Best noise resistance, high spectral efficiency
```

### (c) Circuit Switching Calculation

**Given:** N=100 users, each active 1% of time, link capacity C

**Bandwidth per user:**
```
Bandwidth per user = C / N = C / 100
```

**Circuit Switching vs Packet Switching:**

|                      | Circuit Switching | Packet Switching |
| -------------------- | ----------------- | ---------------- |
| Connection Setup     | Required          | Not required     |
| Resource Reservation | Yes               | No               |
| Bandwidth Guarantee  | Yes               | No               |
| Delay                | Fixed             | Variable         |
| Typical Example      | Telephone         | Internet         |

---

## Q1: Physical Layer Details (2425)

### (a) Why Protocol Layering is Necessary

**Three reasons:**
1. **Modularity:** Each layer designed independently, reduces complexity
2. **Flexibility:** Technology changes in one layer don't affect others
3. **Standardisation:** Different vendors can interoperate

### (b) Error Detection Methods

```
Parity:
  → Append 1 bit to make total number of 1s odd (odd parity) or even (even parity)
  → Can detect 1-bit errors, cannot detect 2-bit errors

Checksum:
  → Split data into 16-bit segments, sum and complement
  → Receiver recalculates and compares
  → Used in IP/TCP/UDP headers

CRC (Cyclic Redundancy Check):
  → Use generator polynomial for modulo-2 division
  → Remainder appended as FCS
  → Can detect all odd-number errors, burst errors
  → Used in Ethernet frames
```

### (c) Signal Encoding

```
NRZ (Non-Return to Zero):
  1 → High level
  0 → Low level
  Problem: No synchronisation with consecutive identical bits

Manchester Encoding:
  1 → First half high, second half low (falling edge)
  0 → First half low, second half high (rising edge)
  Advantage: Every bit has transition, self-synchronising
  Disadvantage: Bandwidth doubled
```

### (d) Modulation Methods

```
ASK: Amplitude variation
FSK: Frequency variation
PSK: Phase variation

Bit rate vs Baud rate:
  Bit rate = Baud rate × bits per symbol
  PSK can carry multiple bits per symbol (e.g. QPSK carries 2 bits)
```

### (e) Multiplexing Techniques

| Multiplexing | Division Dimension | Typical Application |
|---|---|---|
| **FDM** | Frequency | Radio, ADSL |
| **TDM** | Time | T1/E1 lines |
| **WDM** | Wavelength | Fibre optic |

---

## Q2: Data Link Layer (2223)

### (a) Framing Methods

```
4 Framing Methods:

1. Byte Count:
   Frame header contains a field indicating frame length
   Problem: Count error causes all subsequent frames to misalign

2. Byte Stuffing (Flag + ESC):
   Use special byte Flag to mark frame start/end
   Add ESC escape when Flag appears in data
   → Used by HDLC

3. Bit Stuffing (Flag + bit stuffing):
   Use bit pattern 01111110 to mark frame boundary
   Insert 0 after 5 consecutive 1s in data
   → Used by HDLC

4. Physical Layer Coding Violation:
   Use signal patterns that cannot occur in physical layer as boundaries
   → Ethernet uses preamble
```

### (b) CRC Calculation

**Given:** Data D=1010001101, Generator G=110101

**Steps:**
```
1. Append r zeros to D (r = number of bits in G - 1 = 5)
   D' = 101000110100000

2. Perform modulo-2 division (XOR) of D' by G
   Get remainder R (5 bits)

3. Transmitted data = D + R

4. Receiver divides received data by G, remainder 0 means no error
```

### (c) Slotted ALOHA

```
Principle:
  Time divided into equal slots
  Stations transmit only at slot beginning
  On collision, retransmit with probability p in subsequent slots

Maximum throughput: S = 1/e ≈ 0.368 (36.8%)
Occurs when G=1 (average 1 station attempts per slot)

G < 1: Many idle slots, low throughput
G > 1: Many collisions, throughput decreases
```

### (d) CSMA/CD vs CSMA/CA

|                         | CSMA/CD                   | CSMA/CA                                         |
| ----------------------- | ------------------------- | ----------------------------------------------- |
| **Usage**               | Wired Ethernet            | Wireless WiFi                                   |
| **Collision Detection** | Detect while transmitting | Cannot detect                                   |
| **Collision Handling**  | Stop on detection         | Avoid using RTS/CTS                             |
| **Why Different**       | Wired can detect voltage  | Wireless signal overwhelmed by own transmission |

**Hidden Terminal Problem in Wireless Networks:**
```
A ←→ B ←→ C

A and C can both communicate with B
But A and C cannot hear each other
A and C may transmit to B simultaneously → collision

Solution: RTS/CTS
  A sends RTS to B
  B replies CTS (heard by both A and C)
  C receives CTS and knows B is busy, waits
```

### (e) MAC Address and ARP

```
MAC Address:
  48 bits (6 bytes), e.g. 00:1A:2B:3C:4D:5E
  First 24 bits: Vendor OUI
  Last 24 bits: Device serial number
  All F (FF:FF:FF:FF:FF:FF) = Broadcast address

ARP (Address Resolution Protocol):
  IP address → MAC address
  1. Broadcast ARP request: "Who has IP 192.168.1.1?"
  2. Target unicast reply: "My MAC is xx:xx:xx:xx:xx:xx"
  3. Result cached in ARP table
```

---

## Q2: Data Link Layer (2425)

### (a) Framing + Error/Flow Control

**Error Control:**
```
ACK: Receiver acknowledges receipt
Timeout Retransmission: Sender retransmits after ACK timeout
NAK: Receiver requests retransmission
Sequence Number: Detect duplicate and out-of-order frames
```

**Three Flow Control Protocols:**

```
Stop-and-Wait:
  Send 1 frame, wait for ACK, then send next
  Low efficiency, channel utilisation = 1/(1+2a), where a = propagation delay/transmission time

Go-Back-N:
  Send window > 1, transmit continuously
  On error, retransmit all frames from the error frame
  Receive window = 1

Selective Repeat:
  Send window > 1, transmit continuously
  Only retransmit the erroneous frame
  Receive window > 1, buffer correct frames
```

### (b) Slotted ALOHA vs Pure ALOHA

| | Slotted ALOHA | Pure ALOHA |
|---|---|---|
| **Slots** | Slotted | Unslotted |
| **Transmission Time** | At slot beginning | Any time |
| **Max Throughput** | 1/e ≈ 0.368 | 1/(2e) ≈ 0.184 |
| **Collision Window** | 1 slot | 2 slots |

**Handling Collisions:**
```
After collision:
  1. Detect collision (or no ACK received)
  2. Retransmit with probability p in subsequent slots
  3. Wait with probability 1-p
  4. Increase retransmission count decreases p (backoff)
```

### (c) CSMA/CD Working Principle

```
CSMA/CD Steps:
  1. Listen first (Carrier Sense): Transmit only when channel is idle
  2. Listen while transmitting (Collision Detection): Detect collisions
  3. On collision, send Jam signal, stop transmitting
  4. Binary exponential backoff, then retransmit

Minimum Frame Length Requirement:
  Frame length ≥ 2 × propagation delay × data rate
  → Ensures collision can be detected before transmission completes
  → Ethernet minimum frame length: 64 bytes
```

### (d) Bridges/Switches

```
Bridges connect different LANs:
  Learning: Record MAC address of each port
  Forwarding: Unicast forward to known destination port
  Flooding: Broadcast when destination port unknown
  Filtering: Drop when source and destination are on the same port
```

---

## Q3: Network Layer (2223)

### (a) DHCP + NAT + Subnetting

```
DHCP (Dynamic Host Configuration Protocol):
  4-step process:
  1. DHCP Discover (broadcast)
  2. DHCP Offer (server provides IP)
  3. DHCP Request (client accepts)
  4. DHCP ACK (server confirms)

  Provides: IP address, subnet mask, default gateway, DNS server

NAT (Network Address Translation):
  Private IP → Public IP
  10.0.0.1:1234 → 203.0.113.1:5678
  NAT table records mapping
  Solves IPv4 address exhaustion

Subnetting:
  192.168.1.0/24 → Split into 4 subnets
  Each subnet /26 (64 addresses)
  Subnet mask: 255.255.255.192
```

### (b) IPv4 vs IPv6

| | IPv4 | IPv6 |
|---|---|---|
| **Address Length** | 32 bits | 128 bits |
| **Representation** | Dotted decimal | Colon hexadecimal |
| **Header** | Variable (20-60 bytes) | Fixed 40 bytes |
| **Fragmentation** | Router and host | Source host only |
| **Checksum** | Yes | No (delegated to upper layer) |
| **Broadcast** | Yes | No (use multicast instead) |

**IPv4 to IPv6 Transition:**
```
1. Dual Stack: Run both IPv4 and IPv6 simultaneously
2. Tunneling: IPv6 encapsulated in IPv4 for transmission
3. Translation: NAT64/DNS64
```

### (c) IP Fragmentation

```
MTU (Maximum Transmission Unit): Ethernet MTU = 1500 bytes

Fragmentation Calculation:
  Total data size = 4000 bytes (20-byte header + 3980 bytes data)
  MTU = 1500 bytes
  Data per fragment = 1480 bytes (1500 - 20 header)

  Fragment 1: Offset 0, Data 1480 bytes, MF=1
  Fragment 2: Offset 185 (1480/8), Data 1480 bytes, MF=1
  Fragment 3: Offset 370, Data 1020 bytes, MF=0

Offset unit: 8 bytes
MF=1: More fragments follow
MF=0: Last fragment
```

### (d) RIP vs OSPF vs BGP

| | RIP | OSPF | BGP |
|---|---|---|---|
| **Type** | Interior | Interior | Exterior |
| **Algorithm** | Bellman-Ford | Dijkstra | Path Vector |
| **Metric** | Hop count | Bandwidth (cost) | Policy |
| **Convergence** | Slow | Fast | Slow |
| **Scale** | Small networks | Large networks | Between ASes |
| **Protocol** | UDP 520 | IP 89 | TCP 179 |

---

## Q3: Network Layer (2425)

### (a) Binary/Decimal Conversion

```
172.16.2.16 to binary:
  172 = 10101100
  16  = 00010000
  2   = 00000010
  16  = 00010000

Answer: 10101100.00010000.00000010.00010000
```

### (b) Subnetting

```
Company requests 16 IPs starting from 172.16.2.16:
  Network address: 172.16.2.16/28
  Subnet mask: 255.255.255.240
  Address range: 172.16.2.16 - 172.16.2.31
  Broadcast address: 172.16.2.31
  Usable hosts: 172.16.2.17 - 172.16.2.30 (14 hosts)
```

**CIDR Notation:** 172.16.2.16/28 means the first 28 bits are the network portion.

### (c) NAT Address Multiplexing

```
How NAT allows multiple devices to share one public IP:

Internal Device       NAT Table                    External
10.0.0.1:1234    →   10.0.0.1:1234 ↔ 203.0.113.1:5678  → Server
10.0.0.2:1234    →   10.0.0.2:1234 ↔ 203.0.113.1:5679  → Server

Port numbers distinguish different internal devices
```

### (d) Routing Algorithms

```
Dijkstra (Link State):
  Start from source node, gradually expand to all nodes
  Each time select the nearest unvisited node
  Used by OSPF

Bellman-Ford (Distance Vector):
  Each node only exchanges information with neighbours
  d(x) = min{c(x,v) + d(v)} for all neighbours v
  Used by RIP
  Problem: Count to infinity
```

### (e) TTL and ICMP

```
TTL (Time to Live):
  Decremented by 1 at each router
  When TTL=0, packet dropped, ICMP timeout message sent
  Prevents packets from looping forever

ICMP Message Types:
  Echo Request/Reply → ping
  Time Exceeded → traceroute
  Destination Unreachable → port unreachable, etc.
  Redirect → routing redirection
```

---

## Q4: Transport + Application Layer (2223)

### (a) TCP vs UDP

| | TCP | UDP |
|---|---|---|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Reliable | Unreliable |
| **Ordering** | Guaranteed | Not guaranteed |
| **Flow Control** | Yes (sliding window) | No |
| **Congestion Control** | Yes | No |
| **Header** | 20 bytes | 8 bytes |
| **Usage** | Web, email, file transfer | DNS, video, gaming |

### (b) TCP 3-Way Handshake / 4-Way Teardown

```
3-Way Handshake (Connection Establishment):
  Client → SYN (seq=x) → Server
  Client ← SYN+ACK (seq=y, ack=x+1) ← Server
  Client → ACK (ack=y+1) → Server

4-Way Teardown (Connection Termination):
  Client → FIN → Server
  Client ← ACK ← Server
  Client ← FIN ← Server
  Client → ACK → Server
  Client enters TIME_WAIT (wait 2MSL)

Why 3-way handshake?
  → Prevents stale connection requests from reaching server
  → Both sides confirm each other's initial sequence number
```

### (c) HTTP Persistent/Non-Persistent Connections

```
Non-Persistent (HTTP 1.0):
  Each object requires a separate TCP connection
  Fetching 10 objects → 10 TCP connections
  RTT × 2 × 10

Persistent (HTTP 1.1):
  One TCP connection can transfer multiple objects
  Pipelining: Send next request without waiting for response
  RTT × 2 + RTT × 9 (best case)

Total time = 2RTT (TCP handshake + HTTP request) + transmission time
```

### (d) DNS Recursive vs Iterative

```
Recursive Query:
  Client asks local DNS, local DNS responsible for finding answer
  Client only waits for final result

Iterative Query:
  Local DNS queries level by level
  Root → TLD → Authoritative DNS
  Each level returns "who to ask next"
```

### (e) Socket Programming

```python
# TCP Server
server_socket = socket(AF_INET, SOCK_STREAM)
server_socket.bind(('', 12000))
server_socket.listen(1)
connection_socket, addr = server_socket.accept()
data = connection_socket.recv(1024)
connection_socket.send(modified_data)
connection_socket.close()

# TCP Client
client_socket = socket(AF_INET, SOCK_STREAM)
client_socket.connect(('server', 12000))
client_socket.send(sentence)
modified = client_socket.recv(1024)
client_socket.close()

# UDP Server
server_socket = socket(AF_INET, SOCK_DGRAM)
server_socket.bind(('', 12000))
data, addr = server_socket.recvfrom(2048)
server_socket.sendto(modified, addr)

# UDP Client
client_socket = socket(AF_INET, SOCK_DGRAM)
client_socket.sendto(sentence, ('server', 12000))
modified, addr = client_socket.recvfrom(2048)
client_socket.close()
```

---

## Q4: Transport + Security (2425)

### (a) TCP Flow Control

```
Sliding Window Mechanism:
  Receiver advertises window rwnd
  Sender transmits ≤ rwnd

  When rwnd=0: Sender stops transmitting
  Receiver sends window update (non-data packet) to notify
  Prevent sender from waiting forever: set persistence timer

Transmission Rounds:
  Round 1: Window grows exponentially from 1 MSS to threshold
  Round 2: After threshold, linear growth
  Timeout: Window reset to 1, threshold halved
```

### (b) TCP Congestion Control

```
Four phases:
  1. Slow Start: Window grows exponentially (1,2,4,8...)
  2. Congestion Avoidance: Window grows linearly (+1 per RTT)
  3. Fast Retransmit: 3 duplicate ACKs trigger immediate retransmission
  4. Fast Recovery: Don't return to slow start, start from threshold/2

Timeout vs 3 Duplicate ACKs:
  Timeout: Severe congestion → Slow Start (window=1)
  3 Duplicate ACKs: Mild loss → Fast Recovery (window=threshold/2)
```

### (c) DNS Round-Robin Load Balancing

```
Same domain maps to multiple IPs:
  www.example.com → 192.168.1.1
  www.example.com → 192.168.1.2
  www.example.com → 192.168.1.3

DNS server returns different order each time
Client selects first → distributed across different servers
```

### (d) Network Security

```
Firewall:
  Stateless firewall: Per-packet inspection (IP, port, protocol)
  Stateful firewall: Tracks connection state
  Application gateway: Deep inspection of application layer content

IPSec:
  Transport mode: Encrypts payload only, not IP header
  Tunnel mode: Encrypts entire original packet + new IP header
  Used for VPN

VPN:
  Dedicated tunnel through public network
  Remote users access as if on local network
  IP-in-IP encapsulation

TLS/SSL:
  Handshake: Negotiate encryption algorithms, exchange keys
  Transmission: Symmetric encryption of data
  Server proves identity with certificate
```

---

# Section B Analysis

## Q5 (2223): CSMA/CA + IP Routing + Duplicate ACKs

### (a) CSMA/CA Mechanism

```
CSMA/CA Steps:
  1. Sender listens to channel first
  2. If idle, wait DIFS time
  3. If busy, wait and add random backoff time
  4. If channel becomes busy during backoff, pause timer
  5. Send RTS → Wait CTS → Send data → Wait ACK
  6. If ACK timeout, backoff and retransmit

Timing Diagram:
  Sender:   |--DIFS--|---Backoff---|RTS|  |DATA|  |ACK timeout|---retransmit---|
  Channel:  IDLE     BUSY          IDLE    IDLE
  Receiver:                              |CTS|           |ACK|
```

### (b) IP Routing Table Lookup

```
Destination IP: 201.4.22.3

Routing Table:
  201.4.16.0/20 → Interface 0 (matches!)
  201.4.22.0/24 → Interface 1 (more specific, matches!)
  Default route  → Interface 2

Longest Prefix Match: Select /24 via Interface 1
```

### (c) TCP Duplicate ACK Handling

```
Scenario: 5 datagrams (A,B,C,D,E), B lost

  A → Success → ACK=B
  B → Lost
  C → Arrives → ACK=B (duplicate 1)
  D → Arrives → ACK=B (duplicate 2)
  E → Arrives → ACK=B (duplicate 3)

3 Duplicate ACKs received → Triggers Fast Retransmit
  Immediately retransmit B
  Don't wait for timeout
  Enter Fast Recovery (window halved, don't return to slow start)
```

---

## Q5 (2425): IPv4 Fragmentation + TCP Congestion Control + TLS

### (a) IPv4 Fragmentation Calculation

```
Packet: Total length 4000 bytes (20 header + 3980 data)
MTU = 1500 bytes

Max data per fragment = 1500 - 20 = 1480 bytes
1480 / 8 = 185 (fragment offset must be multiple of 8) ✓

Fragment 1: ID=x, MF=1, Offset=0, Data=1480 bytes
Fragment 2: ID=x, MF=1, Offset=185, Data=1480 bytes
Fragment 3: ID=x, MF=0, Offset=370, Data=1020 bytes
```

### (b) TCP Congestion Window Changes

```
Round   Window Size   Phase              Event
────────────────────────────────────────────────────
1       1             Slow Start         Normal
2       2             Slow Start         Normal
3       4             Slow Start         Normal
4       8             Slow Start         Reaches threshold
────────────────────────────────────────────────────
5       9             Congestion Avoid.  Linear growth
6       10            Congestion Avoid.  Linear growth
7       11            Congestion Avoid.  Linear growth
8       12            Congestion Avoid.  Linear growth
────────────────────────────────────────────────────
9       13            Congestion Avoid.  Timeout!
────────────────────────────────────────────────────
10      1             Slow Start         Threshold=6
11      2             Slow Start         Threshold=6
12      4             Slow Start         Threshold=6
────────────────────────────────────────────────────
13      5             Congestion Avoid.  Linear growth
14      6             Congestion Avoid.  Linear growth
```

```
Window Size ↑
    14 │                    ╱
    12 │                  ╱─╲ Timeout
    10 │              ╱──╱
     8 │          ╱──╱
     6 │─ ─ ─ ╱─╱─ ─ ─ ─ ─ ─ Threshold
     4 │    ╱╱
     2 │  ╱╱
     1 │╱╱
       └─────────────────────→ Rounds
        1 2 3 4 5 6 7 8 9 10 11 12 13 14
```

### (c) TLS Transport Security

```
Three mechanisms TLS uses to secure HTTP:

1. Confidentiality:
   → Symmetric encryption (e.g. AES) encrypts data
   → Eavesdropper sees ciphertext

2. Integrity:
   → Message Authentication Code (MAC/HMAC)
   → Detects tampering

3. Authentication:
   → Digital certificates + public key encryption
   → Server proves it is legitimate

TLS Handshake Process:
  1. Client Hello (supported encryption algorithms)
  2. Server Hello + Certificate + Public Key
  3. Client verifies certificate, generates pre-master secret
  4. Encrypt pre-master secret with server's public key, send
  5. Both sides generate session key
  6. Switch to symmetric encryption communication
```

---

# Formula Quick Reference

## Physical Layer

```
Bit rate = Baud rate × log₂(number of signal levels)
Nyquist: Max data rate = 2H × log₂(V) bps
Shannon: Max data rate = H × log₂(1 + S/N) bps
dB = 10 × log₁₀(signal power)
```

## Link Layer

```
Slotted ALOHA: S = G × e^(-G), Max S = 1/e ≈ 0.368
Pure ALOHA: S = G × e^(-2G), Max S = 1/(2e) ≈ 0.184
Efficiency = L/R / (L/R + 2×propagation delay) = 1 / (1 + 2a)
Stop-and-Wait efficiency = 1 / (1 + 2a)
GBN efficiency = N / (1 + 2a) (N = window size)
```

## Network Layer

```
Number of IP addresses = 2^(32 - prefix length)
Usable hosts = 2^(32 - prefix length) - 2
Subnet mask: prefix length number of 1s, rest 0s
```

## Transport Layer

```
Sequence number space = 2^32 (TCP)
TCP window = min(cwnd, rwnd)
Throughput ≈ W/RTT (W = window size)
```

---

# Exam Tips

1. **Physical Layer:** Practise drawing (ASK/FSK/PSK, NRZ/Manchester), calculate Nyquist/Shannon
2. **Link Layer:** Calculate CRC, ALOHA throughput, window efficiency
3. **Network Layer:** Binary conversion, subnetting, IP fragmentation, routing table lookup
4. **Transport Layer:** Understand TCP state machine, congestion control window diagram
5. **Security:** Understand TLS handshake, IPSec modes, firewall types
6. **Section B:** Draw diagrams first, then explain, clear steps earn partial marks
