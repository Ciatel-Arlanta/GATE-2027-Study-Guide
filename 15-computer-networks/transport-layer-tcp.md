# TCP, Transport Layer, Flow Control, and Congestion Control

> **Paper:** CS · **Priority:** P0 · **Plan topics:** TCP flow/congestion control, transport service, sockets, TCP header, sequence numbers, 3-way handshake, connection termination, slow start, AIMD, wrap-around time, UDP, transport checksums
> **Prerequisites:** [Layering and performance](layering-switching-performance.md) · **Leads to:** [Application layer](application-layer.md)
> **PYQ weightage:** High · 21 Qs since 2015 (11 since 2021) · DA: 0 · Trend: steady · Most asked: TCP Congestion Control (slow start, AIMD, threshold updates), Sequence numbers & wrap-around calculations, Flow control & receiver window, 3-way handshake

## Quick glance

- **Exam focus:** TCP Congestion Control phases (Slow Start exponential growth, AIMD linear growth, threshold reductions on timeout vs 3 duplicate ACKs), Sequence number arithmetic (byte stream indexing, wrap-around time calculation), Window sizing ($\text{Effective Window} = \min(\text{rwnd}, \text{cwnd})$), and 3-way handshake sequence number negotiation. Heavily numerical; prime source of 2-mark NATs.
- **Service abstraction:** Connection-oriented, full-duplex, reliable byte stream between application processes identified by `(IP, Port)` tuples.
- **TCP Header:** 20 to 60 bytes. **Data Offset** (4 bits) is in **4-byte words** ($\text{Header Length} = \text{Data Offset} \times 4$).
- **Sequence & ACK numbers:** Sequence Number is the byte offset of the **first data byte** in the segment. Acknowledgement Number is **cumulative**: the sequence number of the **next byte expected** by the receiver.
- **Wrap-around time:** Time to consume all $2^{32}$ sequence numbers $= \dfrac{2^{32} \times 8}{\text{Bandwidth (bps)}}$ seconds. Must exceed Maximum Segment Lifetime (MSL, standard 120s) to avoid stale duplicate collisions.
- **Connection setup & teardown:** 3-way handshake: `SYN` (seq=x) $\to$ `SYN+ACK` (seq=y, ack=x+1) $\to$ `ACK` (seq=x+1, ack=y+1). SYN and FIN each consume **1 sequence number** even without data payload. Teardown uses 4-way handshake (`FIN` $\to$ `ACK`, `FIN` $\to$ `ACK`); initiator remains in `TIME-WAIT` for $2 \times \text{MSL}$.
- **Flow control (protects receiver):** Receiver advertises `rwnd` based on spare buffer space. Window size field is 16 bits (max 65,535 B; expanded up to $1\text{ GB}$ via Window Scale option). Silly Window Syndrome avoided by Clark's solution (receiver) and Nagle's algorithm (sender).
- **Congestion control (protects network):** Sender adjusts `cwnd`.
  - **Slow Start:** $\text{cwnd} \leftarrow \text{cwnd} + 1 \text{ MSS}$ per ACK received $\implies$ doubles every RTT ($1, 2, 4, 8, \dots$) until $\text{cwnd} \ge \text{ssthresh}$.
  - **Congestion Avoidance (AIMD):** $\text{cwnd} \leftarrow \text{cwnd} + 1 \text{ MSS}$ per RTT (additive increase) while $\text{cwnd} \ge \text{ssthresh}$.
  - **Loss via Timeout:** $\text{ssthresh} \leftarrow \lfloor \text{cwnd} / 2 \rfloor$; $\text{cwnd} \leftarrow 1 \text{ MSS}$; enters Slow Start.
  - **Loss via 3 Duplicate ACKs:** Fast Retransmit sends missing packet immediately. Under **TCP Tahoe**: $\text{ssthresh} \leftarrow \lfloor \text{cwnd} / 2 \rfloor$, $\text{cwnd} \leftarrow 1 \text{ MSS}$. Under **TCP Reno (Fast Recovery)**: $\text{ssthresh} \leftarrow \lfloor \text{cwnd} / 2 \rfloor$, $\text{cwnd} \leftarrow \text{ssthresh}$, continues in Congestion Avoidance.
- **UDP:** Connectionless, unreliable datagram service. Fixed 8-byte header (Source Port, Destination Port, Total Length, Checksum). Checksum is optional in IPv4, mandatory in IPv6, calculated over a 12-byte pseudo-header, UDP header, and data.

## 1. TCP Architecture, Byte Streams, and Sequence Number Calculations [PYQ: High]

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window Size        |
| (4b)  |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |        Urgent Pointer         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any) ...                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Key TCP header fields
1. **Ports (16 bits each):** Identifies source and destination applications ($0$ to $65,535$).
2. **Sequence Number (32 bits):** Byte sequence number of the **first byte of data** carried in this segment.
3. **Acknowledgment Number (32 bits):** Valid when the `ACK` flag is set. Contains the **next expected byte** (cumulative ACK). ACK $N$ confirms that all bytes up to $N - 1$ have been received correctly.
4. **Data Offset / HLEN (4 bits):** Length of TCP header in **4-byte words** ($20$ to $60$ bytes).
5. **Control Flags (6 bits):**
   - `URG`: Urgent pointer valid.
   - `ACK`: Acknowledgment field valid (set on all packets after initial SYN).
   - `PSH`: Push data immediately to application layer without waiting for full buffer.
   - `RST`: Abort and reset the connection.
   - `SYN`: Synchronize sequence numbers during connection establishment.
   - `FIN`: Sender has finished transmitting data; initiate teardown.
6. **Window Size (16 bits):** Number of bytes the receiver is willing to accept (`rwnd`).

### Sequence number arithmetic and wrap-around time
TCP sequences individual **bytes**, not segments. With a 32-bit sequence number space, there are $2^{32} = 4,294,967,296$ unique sequence numbers.
- **Wrap-around time ($T_{\text{wrap}}$):** Time required to cycle through all $2^{32}$ sequence numbers at a constant transmission rate $B$ (in bps):
  $$T_{\text{wrap}} = \frac{2^{32} \text{ bytes} \times 8 \text{ bits/byte}}{B \text{ bps}} = \frac{34,359,738,368}{B}$$
- **Maximum Segment Lifetime (MSL):** Time a packet can survive in the network before being dropped (typically $120$ seconds).
- **Rule against ambiguity:** To prevent an old delayed segment from being accepted as new data in the next sequence cycle, the sequence wrap-around time **must exceed MSL**:
  $$T_{\text{wrap}} > \text{MSL} \implies B < \frac{2^{32} \times 8}{\text{MSL}}$$

**Worked example 1 (Wrap-around time numerical).** A sender transfers data over a 1 Gbps ($10^9$ bps) full-duplex link using TCP.
1. Compute the sequence number wrap-around time in seconds.
2. If the link speed is upgraded to 10 Gbps ($10^{10}$ bps), what is the new wrap-around time?
3. If Maximum Segment Lifetime (MSL) is 60 seconds, what is the maximum transmission rate (in Mbps) that TCP can support without sequence number wrap-around ambiguity?

**Step-by-step solution:**
1. At 1 Gbps ($10^9$ bps):
   $$T_{\text{wrap}} = \frac{2^{32} \times 8}{10^9} = \frac{34,359,738,368}{10^9} \approx 34.36 \text{ seconds}$$
2. At 10 Gbps ($10^{10}$ bps):
   $$T_{\text{wrap}} = \frac{34,359,738,368}{10^{10}} \approx 3.44 \text{ seconds}$$
3. For no ambiguity with $\text{MSL} = 60$ s:
   $$B_{\max} = \frac{2^{32} \times 8}{60} = \frac{34,359,738,368}{60} \approx 572,662,306 \text{ bps} \approx 572.66 \text{ Mbps}$$

## 2. TCP Connection Management and State Transitions [PYQ: Medium]

```text
Client                                                   Server
  │                    1. SYN (seq = x)                    │  (LISTEN)
  ├───────────────────────────────────────────────────────>│  (SYN_RCVD)
  │                                                        │
  │              2. SYN + ACK (seq = y, ack = x + 1)       │
  │<───────────────────────────────────────────────────────┤
(ESTABLISHED)                                              │
  │                 3. ACK (seq = x + 1, ack = y + 1)      │
  ├───────────────────────────────────────────────────────>│  (ESTABLISHED)
```

### Three-way handshake mechanics
1. **Step 1 (Client $\to$ Server):** Client chooses random Initial Sequence Number (ISN) $x$, sends segment with flag `SYN=1`, `seq=x`. (Consumes 1 sequence number).
2. **Step 2 (Server $\to$ Client):** Server responds with `SYN=1, ACK=1`, chooses server ISN $y$, sets `seq=y`, `ack=x+1`. (Consumes 1 sequence number).
3. **Step 3 (Client $\to$ Server):** Client completes setup with `ACK=1`, `seq=x+1`, `ack=y+1`. May optionally carry data.

### Connection teardown (Four-way handshake)
1. Active closer sends `FIN` (`seq=u`).
2. Passive receiver sends `ACK` (`ack=u+1`), enters `CLOSE_WAIT`. Active closer enters `FIN_WAIT_2`.
3. Passive receiver finishes sending remaining data, then sends `FIN` (`seq=v, ack=u+1`), enters `LAST_ACK`.
4. Active closer sends `ACK` (`ack=v+1`), enters **`TIME-WAIT`** state for $2 \times \text{MSL}$ (ensuring final ACK is received and old duplicate segments die out).

**Worked example 2 (Byte stream and ACK trace).** A client establishes a TCP connection with a server. Client ISN $= 1000$, Server ISN $= 5000$.
- Client sends a data segment of $800$ bytes.
- Server acknowledges the data and sends $500$ bytes of response.
- Client initiates connection termination.

Trace the `seq`, `ack`, and payload length for every segment exchanged.

**Step-by-step solution:**
1. **Connection setup:**
   - Client $\to$ Server: `SYN`, $\text{seq} = 1000$, Data $= 0$.
   - Server $\to$ Client: `SYN+ACK`, $\text{seq} = 5000$, $\text{ack} = 1001$, Data $= 0$.
   - Client $\to$ Server: `ACK`, $\text{seq} = 1001$, $\text{ack} = 5001$, Data $= 0$.
2. **Data transfer:**
   - Client $\to$ Server: $\text{seq} = 1001$, Data $= 800$ bytes (byte range $1001$ to $1800$).
   - Server $\to$ Client: Data $= 500$ bytes (byte range $5001$ to $5500$). `seq` $= 5001$, cumulative `ack` $= 1801$.
   - Client $\to$ Server: `ACK`, $\text{seq} = 1801$, `ack` $= 5501$, Data $= 0$.
3. **Connection teardown:**
   - Client $\to$ Server: `FIN`, $\text{seq} = 1801$, `ack` $= 5501$. (FIN consumes 1 byte).
   - Server $\to$ Client: `ACK`, $\text{seq} = 5501$, `ack` $= 1802$.
   - Server $\to$ Client: `FIN`, $\text{seq} = 5501$, `ack` $= 1802$.
   - Client $\to$ Server: `ACK`, $\text{seq} = 1802$, `ack` $= 5502$.

## 3. Flow Control, Receiver Window, and Silly Window Syndrome [PYQ: High]

### Sliding window flow control
Flow control prevents a fast sender from overflowing a slow receiver's buffer:
$$\text{Effective Window} = \min(\text{rwnd}, \text{cwnd})$$
The receiver advertises its available buffer space via the 16-bit **Window Size** header field:
$$\text{rwnd} = \text{RcvBuffer} - (\text{LastByteReceived} - \text{LastByteReadByApp})$$

- **Zero-Window Advertisement:** When $\text{rwnd} = 0$, sender pauses. To prevent deadlock when receiver buffer clears, sender runs a **Persist Timer** and periodically sends 1-byte **Window Probe** segments.
- **Window Scale Option (RFC 1323):** Left-shifts the 16-bit window field by up to $14$ bits during handshake, expanding max window from $64\text{ KB}$ ($2^{16}-1$) to $1\text{ GB}$ ($2^{30}-1$) for High Bandwidth-Delay Product (BDP) networks.

### Silly Window Syndrome (SWS)
Occurs when data is transmitted in tiny 1-byte chunks, incurring enormous 40-byte header overhead (20B IP + 20B TCP).
1. **Receiver-side (Clark's solution):** Receiver advertises $\text{rwnd} = 0$ until it can offer a window of at least $\min(\text{MSS}, \text{RcvBuffer}/2)$.
2. **Sender-side (Nagle's algorithm):** If application produces data byte-by-byte:
   - Send the first byte immediately.
   - Buffer subsequent bytes until: (a) a full $\text{MSS}$ chunk accumulates, OR (b) an ACK for previously transmitted data arrives.

**Worked example 3 (BDP and Window Sizing).** A connection operates over a path with Bandwidth $B = 100 \text{ Mbps}$ ($10^8$ bps) and $\text{RTT} = 40 \text{ ms}$.
1. Find the Bandwidth-Delay Product (BDP) in bytes.
2. What is the maximum throughput achievable if the sender is constrained by standard 16-bit TCP window size without window scaling?

**Step-by-step solution:**
1. $\text{BDP} = B \times \text{RTT} = 10^8 \text{ bps} \times 0.04 \text{ s} = 4 \times 10^6 \text{ bits} = 500,000 \text{ bytes} \approx 500 \text{ KB}$.
2. Maximum 16-bit window $= 2^{16} - 1 = 65,535 \text{ bytes} = 524,280 \text{ bits}$.
   $$\text{Max Throughput} = \frac{\text{Window Size}}{\text{RTT}} = \frac{524,280 \text{ bits}}{0.04 \text{ s}} = 13,107,000 \text{ bps} \approx 13.11 \text{ Mbps}$$
   Channel utilization $= 13.11 / 100 \approx 13.11\%$.

## 4. Congestion Control: Slow Start, AIMD, and Loss Recovery [PYQ: High]

Congestion control limits transmission to match network path capacity using the congestion window (`cwnd`).

```text
cwnd (MSS)
 │                                          Timeout! cwnd=1
 │                                           │
 │                              Congestion   ▼
 │                              Avoidance   / \
 │                      ssthresh ───────┐  /   \
 │                             /        │ /     \
 │                   Slow     /         │/
 │                   Start   /          │
 │                          /           │
 │                         /            │
 └─────────────────────────┴────────────┴──────────────── RTT rounds
```

### The 4 phases of TCP congestion control
1. **Slow Start:**
   - Starts at initial window (typically $1 \text{ MSS}$).
   - Increases by $1 \text{ MSS}$ for every ACK received $\implies$ **doubles every RTT** ($1, 2, 4, 8, \dots$).
   - Continues while $\text{cwnd} < \text{ssthresh}$.
2. **Congestion Avoidance (AIMD):**
   - Triggered when $\text{cwnd} \ge \text{ssthresh}$.
   - **Additive Increase:** $\text{cwnd}$ increases by $1 \text{ MSS}$ per RTT.
3. **Loss Event 1: Retransmission Timeout (RTO):**
   - Severe congestion indication.
   - $\text{ssthresh} \leftarrow \max(2, \lfloor \text{cwnd} / 2 \rfloor)$.
   - $\text{cwnd} \leftarrow 1 \text{ MSS}$.
   - Enter **Slow Start**.
4. **Loss Event 2: Three Duplicate ACKs (Fast Retransmit & Fast Recovery):**
   - Mild congestion (out-of-order packets still triggering ACKs).
   - Fast Retransmit: Sender retransmits missing segment without waiting for timeout.
   - **TCP Tahoe:** $\text{ssthresh} \leftarrow \max(2, \lfloor \text{cwnd} / 2 \rfloor)$, $\text{cwnd} \leftarrow 1 \text{ MSS}$, Slow Start.
   - **TCP Reno:** $\text{ssthresh} \leftarrow \max(2, \lfloor \text{cwnd} / 2 \rfloor)$, $\text{cwnd} \leftarrow \text{ssthresh}$, enters **Congestion Avoidance directly**.

**Worked example 4 (Complete congestion window cycle trace).** A TCP connection begins with $\text{cwnd} = 1 \text{ MSS}$ and $\text{ssthresh} = 16 \text{ MSS}$.
- At RTT transmission round 8, a timeout occurs.
- At RTT transmission round 14, three duplicate ACKs are received.

Trace the values of $\text{cwnd}$ and $\text{ssthresh}$ for rounds 1 to 16 under TCP Tahoe and TCP Reno.

**Step-by-step trace table:**

| RTT Round | cwnd (Tahoe) | ssthresh (Tahoe) | cwnd (Reno) | ssthresh (Reno) | Phase |
|---|---|---|---|---|---|
| **1** | 1 | 16 | 1 | 16 | Slow Start |
| **2** | 2 | 16 | 2 | 16 | Slow Start |
| **3** | 4 | 16 | 4 | 16 | Slow Start |
| **4** | 8 | 16 | 8 | 16 | Slow Start |
| **5** | 16 | 16 | 16 | 16 | Slow Start $\to$ hits ssthresh |
| **6** | 17 | 16 | 17 | 16 | Congestion Avoidance (+1) |
| **7** | 18 | 16 | 18 | 16 | Congestion Avoidance (+1) |
| **8** | 19 $\to$ **Timeout!** | 16 | 19 $\to$ **Timeout!** | 16 | $\text{ssthresh} = \lfloor 19/2 \rfloor = 9$ |
| **9** | 1 | 9 | 1 | 9 | Slow Start |
| **10** | 2 | 9 | 2 | 9 | Slow Start |
| **11** | 4 | 9 | 4 | 9 | Slow Start |
| **12** | 8 | 9 | 8 | 9 | Slow Start |
| **13** | 9 | 9 | 9 | 9 | Slow Start $\to$ hits ssthresh (9) |
| **14** | 10 $\to$ **3 Dup ACKs!** | 9 | 10 $\to$ **3 Dup ACKs!** | 9 | $\text{ssthresh} = \lfloor 10/2 \rfloor = 5$ |
| **15** | 1 | 5 | 5 | 5 | Tahoe: SS; Reno: CA (+1) |
| **16** | 2 | 5 | 6 | 5 | Tahoe: SS; Reno: CA (+1) |

## 5. UDP and Transport Layer Checksums [PYQ: Low]

### User Datagram Protocol (UDP)
UDP provides minimal process-to-process communication without connection setup, flow control, or retransmissions.

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Data ...                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
- Fixed header length: **8 bytes**.
- Length field: Total length in bytes of UDP header $+$ payload ($\text{Minimum} = 8$).

### Internet Checksum algorithm
1. Data treated as a sequence of 16-bit words.
2. Sum all 16-bit words using **1's complement addition** (any carry out from the 16th bit is wrapped around and added to the least significant bit).
3. Take the 1's complement (bitwise NOT) of the final sum.
4. **Pseudo-Header:** Both TCP and UDP include a 12-byte IPv4 pseudo-header (Source IP, Destination IP, Zero, Protocol=17 for UDP / 6 for TCP, Transport Length) during checksum calculation to catch misdelivered packets.

**Worked example 5 (Checksum calculation).** Compute the 16-bit 1's complement checksum for three 16-bit words: `0x466F`, `0x7273`, `0x656E`.

**Step-by-step solution:**
1. Add first two words:
   $$\begin{aligned}
   0x466F + 0x7273 &= 0xB8E2
   \end{aligned}$$
2. Add third word:
   $$\begin{aligned}
   0xB8E2 + 0x656E &= 0x11E50
   \end{aligned}$$
3. Wrap around the carry:
   $$\text{Sum} = 0x1E50 + 1 = 0x1E51$$
4. Take bitwise NOT:
   $$\text{Checksum} = \sim(0x1E51) = 0xE1AE$$

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Sequence wrap-around time | $T_{\text{wrap}} = \dfrac{2^{32} \times 8}{B \text{ bps}}$ | High-speed link sequence reuse |
| Effective window | $\min(\text{cwnd}, \text{rwnd})$ | Sender transmission rate limit |
| Bandwidth-Delay Product | $\text{BDP} = B \times \text{RTT}$ | Required window size for 100% link utilization |
| TCP Header Length | $\text{Data Offset} \times 4$ bytes ($20 \le \text{HLEN} \le 60$) | Extracting payload from segment |
| Slow Start update | $\text{cwnd} \leftarrow \text{cwnd} \times 2$ per RTT | Exponential growth phase |
| Congestion Avoidance update | $\text{cwnd} \leftarrow \text{cwnd} + 1 \text{ MSS}$ per RTT | Additive increase phase |
| Multiplicative Decrease | $\text{ssthresh} \leftarrow \lfloor \text{cwnd} / 2 \rfloor$ | Any congestion loss event |
| Timeout recovery | $\text{cwnd} \leftarrow 1 \text{ MSS}$, enter Slow Start | Severe congestion |
| Fast Recovery (Reno) | $\text{cwnd} \leftarrow \text{ssthresh}$, enter Congestion Avoidance | 3 duplicate ACKs |
| UDP Header size | Exactly 8 bytes | Transport overhead calculations |

## GATE traps

- **Sequence numbers count bytes, not packets:** A sequence number increase of 500 means 500 bytes of data were transferred, not 500 segments.
- **SYN and FIN consume 1 sequence number:** Even if a SYN or FIN segment carries 0 bytes of data payload, it advances the sequence counter by 1. Pure ACKs without SYN/FIN and without data consume 0 sequence numbers.
- **Acknowledgment number is next expected byte:** An ACK of 2000 acknowledges receipt of bytes up to 1999; byte 2000 has NOT been received yet.
- **Slow start vs Congestion avoidance threshold:** If $\text{cwnd} = \text{ssthresh}$, TCP enters Congestion Avoidance (additive increase $+1$).
- **TCP Tahoe vs Reno distinction:** On 3 duplicate ACKs, Tahoe drops `cwnd` to $1 \text{ MSS}$, while Reno drops `cwnd` to `ssthresh` and continues linear increase. Always check whether the question specifies Tahoe or Reno.

## Connections

- [Layering and performance](layering-switching-performance.md) — Bandwidth-delay product directly governs the sliding window size needed to saturate links.
- [IPv4 addressing](ipv4-addressing.md) — TCP segments are encapsulated as IPv4 payloads; NAT translates TCP port numbers.
- [Application layer](application-layer.md) — HTTP, FTP, and SMTP run over TCP; DNS and DHCP use UDP.
- [OS processes and sockets](../13-operating-systems/processes-threads-syscalls.md) — Sockets provide the OS API endpoint mapped to `(IP, Port)` tuples.

## Practice

**Q1 (NAT).** A TCP connection uses a sequence number field of 32 bits. The maximum lifetime of a TCP segment in the network is 2 minutes. What is the maximum bandwidth (in Mbps, rounded to two decimal places) of the link such that sequence numbers do not wrap around within the maximum segment lifetime?

<details><summary>Answer</summary>

**Answer:** 286.33  
**Solution:**
1. Total sequence space $= 2^{32} \text{ bytes} = 2^{32} \times 8 = 34,359,738,368 \text{ bits}$.
2. Segment lifetime $= 2 \text{ minutes} = 120 \text{ seconds}$.
3. Maximum bandwidth:
   $$B = \frac{34,359,738,368 \text{ bits}}{120 \text{ s}} \approx 286,331,153 \text{ bps} \approx 286.33 \text{ Mbps}$$

</details>

**Q2 (MCQ).** In a TCP 3-way handshake, client sends `SYN` with sequence number $x$. Server replies with `SYN+ACK` with sequence number $y$. The acknowledgment number sent by the client in the 3rd handshake packet is:
(A) $x + 1$  
(B) $y$  
(C) $y + 1$  
(D) $x + y$  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**
The server's SYN consumes 1 sequence number at sequence position $y$. The client acknowledges this by setting the acknowledgment number to the next expected byte: $y + 1$.

</details>

**Q3 (NAT).** Consider a TCP connection with Maximum Segment Size (MSS) of 1 KB. The current congestion window is 32 KB and the slow start threshold is 16 KB. If a timeout occurs, what will be the value of the slow start threshold (in KB) for the next transmission round?

<details><summary>Answer</summary>

**Answer:** 16  
**Solution:**
Upon a timeout loss event, the slow start threshold is updated to half of the current congestion window:
$$\text{ssthresh}_{\text{new}} = \left\lfloor \frac{\text{cwnd}}{2} \right\rfloor = \left\lfloor \frac{32 \text{ KB}}{2} \right\rfloor = 16 \text{ KB}$$

</details>

**Q4 (MSQ).** Which of the following statements about TCP and UDP are **CORRECT**?
(A) TCP provides full-duplex, stream-oriented reliable data transfer.  
(B) UDP header length is variable, depending on the presence of optional fields.  
(C) In TCP Reno, receiving three duplicate ACKs causes the congestion window to drop to 1 MSS.  
(D) The pseudo-header used in TCP and UDP checksum calculation includes the source and destination IP addresses.  

<details><summary>Answer</summary>

**Answer:** (A), (D)  
**Solution:**
- (A) is correct: TCP is full-duplex and stream-oriented.
- (B) is false: UDP header is fixed at exactly 8 bytes (no options).
- (C) is false: In TCP Reno, 3 duplicate ACKs trigger Fast Recovery ($\text{cwnd} \leftarrow \text{ssthresh}$); dropping to 1 MSS occurs in TCP Tahoe.
- (D) is correct: Both transport checksums verify IP layer source and destination addresses via the pseudo-header.

</details>

**Q5 (NAT).** A TCP sender is operating with $\text{cwnd} = 20 \text{ MSS}$ and $\text{ssthresh} = 16 \text{ MSS}$. Assuming each RTT delivers ACKs for all in-flight segments and no packet loss occurs, what will be the value of $\text{cwnd}$ (in MSS) after 4 RTTs?

<details><summary>Answer</summary>

**Answer:** 24  
**Solution:**
Since $\text{cwnd} \ge \text{ssthresh}$ ($20 \ge 16$), TCP is in the Congestion Avoidance phase.
In Congestion Avoidance, $\text{cwnd}$ increases additively by $1 \text{ MSS}$ per RTT:
$$\text{cwnd} = 20 + 4 \times 1 = 24 \text{ MSS}$$

</details>

**Q6 (MCQ).** What is the primary purpose of the `TIME-WAIT` state at the active closer of a TCP connection?
(A) To allow the operating system to flush kernel socket buffers.  
(B) To ensure the final ACK is received by the passive closer and old duplicate segments expire in the network.  
(C) To measure round-trip time for future connections.  
(D) To renegotiate port numbers for subsequent incoming connections.  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**
`TIME-WAIT` maintains state for $2 \times \text{MSL}$ so that if the final ACK is lost, the passive closer's retransmitted FIN can still be acknowledged, and all lingering packets from the incarnation expire harmlessly.

</details>

**Q7 (NAT).** Consider a 10 Mbps link with round-trip time RTT $= 20 \text{ ms}$. If the maximum receiver advertised window is 16 KB ($16 \times 1024$ bytes), what is the maximum link utilization achievable (in percentage, rounded to one decimal place)?

<details><summary>Answer</summary>

**Answer:** 65.5  
**Solution:**
1. Window size $= 16 \times 1024 \text{ bytes} = 16,384 \text{ bytes} = 131,072 \text{ bits}$.
2. Maximum throughput $= \frac{131,072 \text{ bits}}{0.020 \text{ s}} = 6,553,600 \text{ bps} = 6.5536 \text{ Mbps}$.
3. Link utilization $= \frac{6.5536 \text{ Mbps}}{10 \text{ Mbps}} \times 100\% = 65.536\% \approx 65.5\%$.

</details>
