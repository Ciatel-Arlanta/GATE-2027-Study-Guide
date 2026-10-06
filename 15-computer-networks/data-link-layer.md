# Data Link Layer: Errors, Flow Control, MAC and Ethernet

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Data-link layer; Error detection; Medium Access Control; Ethernet
> **Prerequisites:** [Layering, switching and performance](layering-switching-performance.md) · **Leads to:** [Routing](routing.md), [Transport layer and TCP](transport-layer-tcp.md)

## Quick glance

- The link layer moves **frames** between **adjacent** nodes: framing, error detection, flow control (sliding window), and medium access for shared links.
- **Parity** detects odd numbers of bit errors; **2D parity** corrects 1 bit; **Internet checksum** = one's-complement sum of 16-bit words, complemented; **CRC** = remainder of polynomial division mod 2 by generator $G(x)$ of degree $r$, appended as $r$ bits.
- **Hamming distance $d$**: detect $d-1$ errors, correct $\lfloor (d-1)/2 \rfloor$. Hamming code: $2^r \ge m + r + 1$, check bits at power-of-2 positions.
- $a = T_p/T_t$. **Stop-and-wait** $\eta = 1/(1+2a)$. **GBN / SR** $\eta = \min\{1,\ N/(1+2a)\}$. Window to fill the pipe $= 1 + 2a$.
- **Sequence-number bits $k$:** GBN window $\le 2^k - 1$, SR window $\le 2^{k-1}$, stop-and-wait needs 1 bit.
- **ALOHA:** pure $S = Ge^{-2G}$ (max $1/2e = 18.4\%$ at $G = 0.5$); slotted $S = Ge^{-G}$ (max $1/e = 36.8\%$ at $G = 1$).
- **CSMA/CD:** minimum frame length $= 2\,T_p \times$ bandwidth. Ethernet: 10 Mbps, 51.2 $\mu$s slot, **64 B min frame, 1518 B max**, 46-1500 B payload, 48-bit MAC.
- **Binary exponential backoff:** after the $n$-th collision pick $K \in \{0, \dots, 2^n - 1\}$ and wait $K \times 512$ bit times.
- **#1 trap:** GBN receiver window is **1**, SR receiver window is **N**; and CRC sends the remainder, **not** the quotient.

---

## 1. Services of the data link layer and framing

**Intuition.** The physical layer delivers a stream of bits with no boundaries. The link layer carves it into **frames**, adds addresses and error-check bits, and decides who may speak on a shared wire.

Services: framing, link addressing (MAC), error detection (and sometimes correction), flow control (do not overwhelm the receiver), medium access control (for shared media), reliable delivery (on lossy links such as Wi-Fi).

### 1.1 Framing methods

| Method | Idea | Cost |
| --- | --- | --- |
| Character count | first field = length | one error corrupts all later frames |
| **Byte stuffing** | frame starts/ends with flag byte (0x7E); an escape byte (0x7D) is inserted before any flag or escape in the data | extra bytes |
| **Bit stuffing** (HDLC) | flag $= 01111110$; sender inserts a 0 after **every five consecutive 1s** in the data; receiver removes a 0 after five 1s | extra bits |
| Physical coding violations | invalid signal patterns mark the boundary | needs redundancy in coding |

**Bit-stuffing worked example.** Data: `0111110111111` (13 bits).
Scan left to right: `0`, then `11111` (five 1s) so stuff a `0`; then data `0`; then `111111`: after the first five 1s stuff a `0`, then the last `1`.
Output: `0 11111 0 0 11111 0 1` $=$ `011111001111101` (15 bits: 2 stuffed bits). The receiver sees five 1s followed by 0 and deletes that 0.

---

## 2. Error detection and correction

Errors flip bits. We add **redundant bits**; the more redundancy, the more error patterns are caught.

### 2.1 Parity

- **Single parity bit** (even parity): choose the bit so the total number of 1s is even. **Detects every odd-weight error** (1, 3, 5 flipped bits); misses all even-weight errors (2, 4 ...). Cannot locate the error.
- **Two-dimensional parity:** arrange data in rows; add a parity bit per row and per column (plus a corner bit).

**Worked example** (even parity, 4 rows of 4 bits):

```text
data       row parity
1 0 1 1       1       (three 1s -> parity bit 1)
0 1 1 0       0
1 1 1 1       0
0 0 0 1       1
-------
col par: 0 0 1 1   corner 0   (row-parity column 1,0,0,1 has two 1s -> 0)
```

If one bit flips, exactly one row parity and one column parity fail; their intersection **locates and corrects** the bit. All 1, 2 and 3-bit errors are detected; some 4-bit patterns (a rectangle of flips) pass undetected.

### 2.2 Internet checksum

Used in IP, TCP and UDP. Treat data as 16-bit words, add with **one's-complement arithmetic** (carry out of bit 16 is added back), then take the **bitwise complement** as the checksum. The receiver adds all words including the checksum; the result must be $\texttt{FFFF}$ (complement = 0).

**Worked example.** Words $\texttt{1234},\ \texttt{F0F0},\ \texttt{8001}$.
1. $\texttt{1234} + \texttt{F0F0} = \texttt{10324}$. Carry 1 wraps: $\texttt{0324} + 1 = \texttt{0325}$.
2. $\texttt{0325} + \texttt{8001} = \texttt{8326}$.
3. Checksum $= \sim\texttt{8326} = \texttt{7CD9}$.
4. Verify: $\texttt{8326} + \texttt{7CD9} = \texttt{FFFF}$.

Weakness: it misses reordered words, and compensating changes (one word $+1$, another $-1$). It is cheap in software, weaker than CRC.

### 2.3 CRC (cyclic redundancy check)

**Idea.** Treat the bit string as a polynomial over GF(2) (coefficients 0/1, addition = XOR, no carries). Choose a **generator** $G(x)$ of degree $r$ (so $r+1$ bits, leading and trailing coefficient 1). Sender and receiver agree on $G$.

**Sender procedure.**
1. Append $r$ zeros to the $m$-bit data: $D\cdot x^r$.
2. Divide by $G$ using modulo-2 long division; let the remainder be $R$ ($r$ bits).
3. Transmit $T = D\cdot x^r + R$ (data followed by $R$). Now $T$ is exactly divisible by $G$.

**Receiver:** divide the received bits by $G$; **remainder 0 means accepted** (no detected error), non-zero means error.

**Worked example 1.** Data $= 1101011011$, generator $x^4+x+1 = 10011$ ($r=4$).
Dividend (append 4 zeros): `11010110110000`. At each step, if the leading bit is 1 XOR with `10011` else shift:

```text
11010110110000
10011              XOR at shift 0
-----
01001110110000
 10011             shift 1 (leading bit now 1)
 -----
00000010110000
      10011        skip zeros; shift 6
      -----
00000000101000
        10011      shift 8
        -----
00000000001110     remainder = 1110
```

Remainder $R = 1110$. Transmitted frame: `1101011011` + `1110` $=$ `11010110111110`. Check: dividing it by 10011 gives remainder 0000.

**Corruption check.** Flip the 3rd bit: `11110110111110`; divide by 10011: remainder $= 1110 \ne 0$, so the error is detected.

**Worked example 2.** Data $1101$, $G = x^3+x+1 = 1011$ ($r=3$). Dividend `1101000`:
- `1101000` XOR `1011` at shift 0 $\to$ `0110000`
- shift 1 XOR `1011` $\to$ `0011100`
- shift 2 XOR `1011` $\to$ `0001010`
- shift 3 XOR `1011` $\to$ `0000001`

Remainder $= 001$. Frame $= 1101001$.

**What CRC detects (generator degree $r$, $G$ has $x^0$ term):**

| Error | Detected? |
| --- | --- |
| All single-bit errors | Yes, if $G$ has at least two non-zero terms (including $x^0$) |
| All burst errors of length $\le r$ | **Yes** |
| Burst of length $r+1$ | Missed with probability $2^{-(r-1)}$ |
| Burst longer than $r+1$ | Missed with probability $2^{-r}$ |
| All odd numbers of bit errors | Yes, if $(x+1)$ divides $G$ |
| Any error pattern that is a multiple of $G$ | **No** (undetectable) |

CRC-32 (Ethernet) uses a degree-32 generator. **Number of CRC bits appended = degree of $G$**, i.e., (number of bits in $G$) $-1$.

### 2.4 Hamming distance and error-correcting codes

The **Hamming distance** between two equal-length codewords is the number of positions where they differ. The **minimum distance $d$** of a code (the smallest distance over all pairs of valid codewords) determines its power:

| Capability | Condition |
| --- | --- |
| Detect up to $e$ errors | $d \ge e+1$ (so $e = d-1$) |
| Correct up to $t$ errors | $d \ge 2t+1$ (so $t = \lfloor (d-1)/2 \rfloor$) |
| Detect $e$ and correct $t$ ($e \ge t$) at once | $d \ge e + t + 1$ |

Example: the repetition code $\{000, 111\}$ has $d = 3$: detects 2-bit errors, corrects 1-bit errors. Even-parity code has $d = 2$: detects 1, corrects 0.

**Hamming code (7,4).** For $m$ data bits and $r$ check bits we need the syndrome ($r$ bits) to name any of $m + r$ positions or "no error": $2^r \ge m + r + 1$. For $m = 4$: $r = 3$; $m = 8$: $r = 4$; $m = 11$: $r = 4$.

Check bits sit at positions $1, 2, 4, 8,\dots$; the bit at position $p$ (power of 2) is the parity of all positions whose binary representation contains $p$.

**Worked example.** Data $1011$ into positions $3, 5, 6, 7$ (positions: p1 p2 d1 p4 d2 d3 d4).
- $p_1$ covers positions $1,3,5,7$: $d_1 + d_2 + d_4 = 1+0+1 = 0$ (even) so $p_1 = 0$.
- $p_2$ covers $2,3,6,7$: $d_1 + d_3 + d_4 = 1+1+1 = 1$ so $p_2 = 1$.
- $p_4$ covers $4,5,6,7$: $d_2 + d_3 + d_4 = 0+1+1 = 0$ so $p_4 = 0$.

Codeword (positions 1 to 7): `0110011`. Suppose position 5 flips: `0110111`. Recompute checks: $c_1$ (1,3,5,7) $= 0+1+1+1 = 1$, $c_2$ (2,3,6,7) $= 1+1+1+1 = 0$, $c_4$ (4,5,6,7) $= 0+1+1+1 = 1$. Syndrome $c_4c_2c_1 = 101_2 = 5$, which is the position of the error; flip it back. **The syndrome value equals the position of the bad bit.** The (7,4) Hamming code has $d = 3$.

---

## 3. Flow control and reliable transmission (sliding window)

**Intuition.** The sender must not outrun the receiver, and over a lossy link it must retransmit. Think of a pipe of length $T_p$: stop-and-wait puts one frame in the pipe at a time; sliding window keeps several frames in flight.

### 3.1 Notation

- $T_t = L/R$: time to transmit one frame ($L$ bits, rate $R$). $T_p$: one-way propagation delay. ACK frames are assumed tiny, so their transmission time is ignored (state the assumption).
- $a = T_p / T_t$. Round trip $= T_t + 2T_p$ (send frame, then ACK arrives).
- **Number of frames that fill the pipe $= 1 + 2a$.**

### 3.2 Stop-and-wait

Send one frame, wait for its ACK (timeout $\to$ resend).

$$\eta = \frac{T_t}{T_t + 2T_p} = \frac{1}{1+2a}$$

**Worked example.** $R = 1$ Mbps, frame 1000 B $= 8000$ bits $\Rightarrow T_t = 8$ ms; $T_p = 20$ ms. $a = 2.5$. Cycle $= 8 + 40 = 48$ ms. $\eta = 8/48 = 1/6 = 16.67\%$. Throughput $= 8000/0.048 = 166.7$ kbps.

With frame error probability $p$, each frame needs $1/(1-p)$ transmissions on average so $\eta = (1-p)/(1+2a)$.

### 3.3 Go-Back-N (GBN)

- Sender window $N$: up to $N$ unacknowledged frames in flight. **Receiver window = 1**: accepts only the next in-order frame; out-of-order frames are **discarded**.
- ACKs are **cumulative** ("ACK $n$" = all up to $n$ received).
- On timeout of frame $i$, the sender **goes back and retransmits frame $i$ and every frame after it**.

$$\eta = \min\left\{1,\ \frac{N}{1+2a}\right\}$$

### 3.4 Selective Repeat (SR)

- Sender window $N$, **receiver window $N$**: out-of-order frames are **buffered**; each frame is acknowledged individually (or selectively).
- Only the **lost frame** is retransmitted on its timer expiry.
- Efficiency without errors is the same as GBN: $\min\{1,\ N/(1+2a)\}$; with errors SR is better.

With frame loss probability $p$ and a large enough window: stop-and-wait $\eta = (1-p)/(1+2a)$; GBN $\eta = (1-p)/(1+2ap)$ (hmm: each loss wastes $\approx 2a$ frames); SR $\eta = 1-p$. (State these only when the question asks for errors.)

### 3.5 Window size vs sequence-number bits (high-yield)

With $k$ bits there are $2^k$ sequence numbers $0, \dots, 2^k - 1$.

| Protocol | Max sender window | Reason |
| --- | --- | --- |
| Stop-and-wait | 1 (needs 1 bit: seq 0/1) | $N = 1$ |
| **Go-Back-N** | $2^k - 1$ | If $N = 2^k$, a lost ACK for a whole window makes the retransmitted frame 0 look like the next new frame 0 |
| **Selective Repeat** | $2^{k-1}$ | Sender and receiver windows must not overlap in sequence space: $N_s + N_r \le 2^k$ with $N_s = N_r$ |

Inverse: bits needed for window $N$: GBN: $\lceil \log_2 (N+1) \rceil$; SR: $\lceil \log_2 (2N) \rceil$.

**Worked example.** $R = 1$ Mbps, frame 1000 bits so $T_t = 1$ ms; one-way delay 12 ms so $T_p = 12$ ms, $a = 12$. Pipe holds $1 + 2a = 25$ frames.
- Utilisation with $N = 5$: $5/25 = 20\%$; with $N = 25$ or more: $100\%$.
- Sequence bits for $N = 25$: GBN needs $N \le 2^k - 1$: $2^4 - 1 = 15 < 25$, $2^5 - 1 = 31 \ge 25$ so **5 bits**. SR needs $2N = 50 \le 2^k$: $2^5 = 32 < 50$, $2^6 = 64$ so **6 bits**.
- If the sender only has $k = 4$ bits: GBN window $\le 15$ so $\eta = 15/25 = 60\%$; SR window $\le 8$ so $\eta = 8/25 = 32\%$.

### 3.6 Counting transmissions with losses (worked)

Send frames $0 \dots 9$; window $N = 4$; the propagation delay is large enough that the sender sends a whole window before the first ACK/timeout is seen. Frame 4 is lost on its first transmission only.

**GBN:** 
- Round 1: send $0,1,2,3$ (4 transmissions), all delivered.
- Round 2: send $4,5,6,7$ (4). Frame 4 is lost; the receiver discards $5,6,7$ (out of order).
- Timeout $\Rightarrow$ go back: resend $4,5,6,7$ (4).
- Then $8,9$ (2).
- **Total $= 4 + 4 + 4 + 2 = 14$ transmissions** (4 extra).

**Selective Repeat:**
- $0$-$3$ (4); $4$-$7$ (4, frame 4 lost; $5,6,7$ buffered); resend only $4$ (1); then $8,9$ (2).
- **Total $= 11$ transmissions** (1 extra).

**Stop-and-wait:** 10 frames $+ 1$ retransmission $= 11$ transmissions, but far slower.

If the loss is of an ACK in GBN, cumulative ACKs often cover it, so no retransmission is needed.

### 3.7 Piggybacking

In full-duplex communication, instead of sending a separate ACK frame, the receiver **attaches the ACK to the next data frame** going the opposite way (in the ACK field of its header). This saves bandwidth; a timer sends a separate ACK if no data frame is ready.

---

## 4. Medium access control (MAC)

On a **shared channel** (broadcast link) the **MAC sublayer** decides who transmits when; two simultaneous transmissions collide. Three families:

| Family | Idea | Examples | Good when |
| --- | --- | --- | --- |
| Channel partitioning | divide channel by time/frequency/code | TDMA, FDMA, CDMA | heavy, steady load |
| **Random access** | transmit when you want; handle collisions | ALOHA, CSMA, CSMA/CD, CSMA/CA | light/bursty load |
| Taking turns | pass permission around | polling, token ring | heavy load, bounded delay |

### 4.1 Pure ALOHA

Transmit immediately; if no ACK, wait a random time and resend. A frame (duration $T$) collides if any other frame starts in the **vulnerable period $2T$**.

With $G$ = average number of frames **offered** per frame time (new + retransmissions, Poisson): probability of no other start in $2T$ is $e^{-2G}$, so

$$S = G\,e^{-2G}$$

Maximum at $G = 0.5$: $S_{max} = \dfrac{1}{2e} \approx 0.184$ (**18.4%**).

### 4.2 Slotted ALOHA

Time is divided into slots of length $T$; transmissions begin only at slot boundaries. Vulnerable period shrinks to $T$.

$$S = G\,e^{-G}, \quad S_{max} = \frac{1}{e} \approx 0.368 \text{ at } G = 1$$

**Worked example.** A channel of 1 Mbps, frames of 1000 bits ($T = 1$ ms). Stations together offer 1000 frames/s so $G = 1$. Pure ALOHA: $S = e^{-2} = 0.135$ so 135 frames/s succeed. Slotted ALOHA: $S = e^{-1} = 0.368$ so 368 frames/s. In slotted ALOHA at $G=1$ the slots are 36.8% successful, 36.8% empty and 26.4% collided.

### 4.3 CSMA (carrier sense)

**Listen before talking.** Collisions can still occur because of propagation delay (two stations sense idle before the other's signal arrives).

| Variant | When channel busy | When idle |
| --- | --- | --- |
| **1-persistent** | keep sensing, send as soon as idle | send with probability 1 (Ethernet) |
| **Non-persistent** | wait a random time, sense again | send |
| **p-persistent** (slotted) | wait for next slot | send with prob. $p$, else defer one slot |

### 4.4 CSMA/CD (Ethernet)

Add **collision detection**: while transmitting, listen to the wire; if the signal differs from what was sent, a collision occurred. Stop, send a jam signal, back off.

**Why a minimum frame size.** The sender must still be transmitting when the collision signal returns, otherwise it assumes success. Worst case: the farthest station starts just before the first bit arrives; the collision signal needs another $T_p$ to return, so the round trip is $2T_p$.

$$T_t \ge 2T_p \quad\Longleftrightarrow\quad L_{min} = 2\,T_p \times R$$

**Worked example 1.** 1 Gbps, cable length 1 km, signal speed $2\times10^8$ m/s. $T_p = 1000/(2\times10^8) = 5\ \mu$s. $L_{min} = 2 \times 5\times10^{-6} \times 10^9 = 10{,}000$ bits $= 1250$ bytes.

**Worked example 2.** 10 Mbps, 2.5 km total, $v = 2\times10^8$ m/s: $T_p = 12.5\ \mu$s, $L_{min} = 2\times12.5\times10^{-6}\times10^7 = 250$ bits.

**Standard Ethernet (10 Mbps):** slot time $= 51.2\ \mu$s $=$ 512 bit times $= 64$ bytes. So $L_{min} = 512$ bits $= 64$ B; $T_p \le 25.6\ \mu$s. The same 64 B minimum is kept for 100 Mbps, which cuts the maximum network diameter by 10; Gigabit Ethernet uses carrier extension (512 B slot).

**Binary exponential backoff.** After the $n$-th successive collision for a frame, choose $K$ uniformly from $\{0, 1, \dots, 2^n - 1\}$ (the exponent capped at 10) and wait $K \times 512$ bit times. After 16 attempts, give up and report failure.

**Worked example (two stations).** A and B collide for the first time. Each now picks $K \in \{0, 1\}$. They collide again only if $K_A = K_B$: probability $1/2$. After the second collision, $K \in \{0,1,2,3\}$: they collide again with probability $1/4$. So the probability that A and B's first three attempts all collide $= 1 \times \tfrac12 \times \tfrac14 = 1/8$. The expected backoff after collision 1 is $0.5$ slots for each station.

**CSMA/CD efficiency.** With contention slots of length $2T_p$ and on average $e \approx 2.718$ contention slots before success:

$$\eta = \frac{T_t}{T_t + T_p + 2eT_p} = \frac{1}{1 + (1+2e)\,a} \approx \frac{1}{1+6.44a}$$

Example: $a = 0.1$ gives $\eta = 1/1.644 = 60.8\%$. Efficiency falls as $a$ grows (longer cables, higher rates, shorter frames).

### 4.5 CSMA/CA (wireless, brief)

Wi-Fi cannot detect collisions while sending (own signal dominates, hidden terminal). It **avoids** them: sense idle for DIFS, then wait a random backoff counter (frozen while busy), send, and **wait for an ACK**; absence of ACK means collision (stop-and-wait style reliability at the link layer). Optional RTS/CTS handshake handles hidden terminals.

### 4.6 Taking turns: polling and token passing (brief)

- **Polling:** a master asks each slave in turn. Overhead of poll messages; master is a single point of failure.
- **Token passing** (token ring/FDDI): a small token circulates; only the holder may send. No collisions, bounded delay; cost is token latency and token loss recovery. Efficiency is high under heavy load and poor under light load compared with random access.

---

## 5. Ethernet (IEEE 802.3)

### 5.1 Frame format

```text
| Preamble 7B | SFD 1B | Dest MAC 6B | Src MAC 6B | Type/Len 2B | Data 46-1500 B | FCS(CRC-32) 4B |
|<---- 8 bytes, not counted in frame size ---->|<----------- frame: 64 to 1518 bytes ----------->|
```

- **Preamble** (10101010 $\times 7$) + **SFD** (10101011) synchronise clocks.
- **MAC address:** 48 bits, written as 6 hex pairs, burned into the NIC. Broadcast $=$ `FF:FF:FF:FF:FF:FF`.
- **Type/Length:** upper-layer protocol (e.g., 0x0800 IPv4) in Ethernet II.
- **Minimum frame 64 B** (DA through FCS) so minimum payload 46 B (padding added if smaller); **maximum frame 1518 B**, **MTU = 1500 B** payload. Frame overhead (header + FCS) $= 18$ B.

### 5.2 Efficiency

- Largest payload: $1500/1518 = 98.8\%$ (counting only header and FCS); including preamble 8 B and the 12 B inter-frame gap: $1500/1538 = 97.5\%$.
- Smallest payload: $46/(64 + 8 + 12) = 54.8\%$.
- Under contention use $\eta \approx 1/(1 + 6.44a)$.

### 5.3 Hubs, bridges and switches

| | Hub | Switch (bridge) |
| --- | --- | --- |
| Layer | 1 | 2 |
| Forwards to | all ports | the port of the destination MAC |
| Collision domains | **1 for the whole hub** | **1 per port** (with full duplex, no collisions at all) |
| Broadcast domains | 1 | 1 (all ports, unless VLANs) |

**Learning (transparent) switch.** For each arriving frame the switch (1) **records source MAC $\to$ incoming port**, (2) looks up the destination MAC: if known, forward only to that port (if it is the same port, discard); if unknown or broadcast, **flood** to all other ports. Entries age out.

**Trace.** Hosts A, B, C, D on ports 1, 2, 3, 4; empty table.
1. A $\to$ B: learn A@1; B unknown $\to$ flood to 2, 3, 4.
2. B $\to$ A: learn B@2; A known $\to$ forward to port 1 only.
3. C $\to$ B: learn C@3; B known $\to$ port 2 only.
4. D $\to$ broadcast: learn D@4; flood to 1, 2, 3.

**Counting domains.** Four hosts per hub, three hubs connected to a switch, switch to a router with another LAN: collision domains = 3 (one per hub-switch link side, the hub's network) + switch-to-router link + router's other segment; broadcast domains = 2 (router separates them). Rule: **switch port = one collision domain; router interface = one broadcast domain.**

---

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Bit stuffing | insert 0 after five 1s; flag 01111110 | framing questions |
| CRC bits | degree of $G$ = bits in $G$ $-1$ | frame length |
| CRC frame | data + remainder; receiver remainder 0 | CRC |
| Checksum | one's-complement sum, complement | IP/TCP/UDP |
| Hamming $d$ | detect $d-1$; correct $\lfloor(d-1)/2\rfloor$ | code capability |
| Hamming check bits | $2^r \ge m+r+1$ | number of parity bits |
| $a$ | $T_p/T_t$ | all efficiency formulas |
| Stop-and-wait | $\eta = 1/(1+2a)$ | single frame |
| GBN / SR | $\min(1, N/(1+2a))$ | window protocols |
| Window to fill pipe | $1 + 2a$ | minimum $N$ |
| Seq bits | GBN: $N \le 2^k-1$; SR: $N \le 2^{k-1}$ | sequence space |
| Pure ALOHA | $Ge^{-2G}$, max 18.4% at $G=0.5$ | throughput |
| Slotted ALOHA | $Ge^{-G}$, max 36.8% at $G=1$ | throughput |
| CSMA/CD min frame | $2T_p \times R$ | cable length |
| Backoff | $K \in [0, 2^n-1]$ slots of 512 bit times | collision resolution |
| Ethernet frame | 64-1518 B; payload 46-1500; header 14 + FCS 4 | frame size |
| CSMA/CD efficiency | $1/(1+6.44a)$ | Ethernet utilisation |

## GATE traps

- **ACK time ignored?** Read the question: if ACK size is given, cycle $= T_t + T_{ack} + 2T_p$; otherwise assume zero.
- **GBN window $2^k - 1$, SR window $2^{k-1}$.** Using $2^k$ for either is wrong; with $N = 2^k$ in GBN the receiver cannot tell new frame 0 from a retransmitted one.
- **GBN receiver discards out-of-order frames;** SR buffers them. Hence GBN retransmits the whole tail.
- **CRC remainder, not quotient, is appended;** and the generator's bit count is $r+1$. Do not forget to append $r$ zeros first.
- **XOR, not subtraction with borrow,** in CRC division. Leading 1 of the generator aligns with the current leading 1.
- **Parity misses even numbers of errors.** CRC with $(x+1)$ as a factor catches all odd errors.
- **Hamming correct $\lfloor (d-1)/2 \rfloor$** — for $d = 4$ it corrects 1 and detects 2 simultaneously.
- **Minimum frame size = $2T_p \times$ rate,** not $T_p \times$ rate; $T_p$ is one-way. Check units (bits vs bytes).
- **Slotted ALOHA max is 1/e at $G = 1$; pure is $1/2e$ at $G = 0.5$.** Two different $G$ values.
- **Ethernet 64 B excludes preamble and SFD;** 1518 B excludes them too. Payload 46-1500, not 64-1518.
- **A switch does not reduce broadcast domains;** a router does.
- **Backoff counts collisions,** the range doubles each time but is capped at $2^{10}$.
- **Efficiency of 100% is the cap:** if $N \ge 1 + 2a$ answer 1.

## Connections

- [Layering, switching and performance](layering-switching-performance.md) — $T_t$, $T_p$, RTT and BDP underlie $a$.
- [Transport layer and TCP](transport-layer-tcp.md) — TCP flow control is a byte-oriented sliding window (like GBN with cumulative ACKs and SR-style buffering).
- [IPv4 addressing](ipv4-addressing.md) — Ethernet MTU 1500 B causes IP fragmentation.
- [Routing](routing.md) — routers connect broadcast domains; link-state is flooded over link-layer adjacencies.
- [Boolean algebra and minimization](../09-digital-logic/boolean-algebra-and-minimization.md) — parity and CRC are XOR (GF(2)) logic; CRC is an LFSR in hardware.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) and [probability basics](../02-probability-statistics/probability-basics.md) — Hamming spheres, ALOHA's Poisson analysis, backoff probabilities.
- [Discrete distributions](../02-probability-statistics/discrete-distributions.md) — Poisson arrivals behind $Ge^{-G}$.
- [Algebraic structures](../01-discrete-mathematics/algebraic-structures.md) — polynomial arithmetic over GF(2).
- [Operating systems: synchronization](../13-operating-systems/synchronization.md) — token passing and backoff mirror mutual-exclusion ideas.

## Practice

**Q1 (MCQ).** A code has minimum Hamming distance 5. What is the maximum number of errors it can always correct?
(A) 1 (B) 2 (C) 3 (D) 4

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** $t = \lfloor (5-1)/2 \rfloor = 2$. (It can detect up to 4.)

</details>

**Q2 (NUM).** Using generator $x^3 + x + 1$, find the CRC remainder for data $1101$.

<details><summary>Answer</summary>

**Answer:** 001  
**Solution:** divide $1101000$ by $1011$ as in section 2.3 (example 2): remainder $001$. Transmitted word $1101001$.

</details>

**Q3 (NUM).** In stop-and-wait, $R = 2$ Mbps, frame 2000 bits, one-way propagation delay 4 ms. Find the link efficiency in %.

<details><summary>Answer</summary>

**Answer:** 11.1%  
**Solution:** $T_t = 2000/(2\times10^6) = 1$ ms; $T_p = 4$ ms so $a = 4$. $\eta = 1/(1 + 2a) = 1/9 = 0.111$.

</details>

**Q4 (NUM).** A GBN protocol uses 3-bit sequence numbers. $R = 1$ Mbps, frame size 1000 bits, $T_p = 5$ ms. Find the maximum achievable efficiency (%).

<details><summary>Answer</summary>

**Answer:** 63.6%  
**Solution:** $T_t = 1$ ms, $a = 5$, $1 + 2a = 11$. GBN window $\le 2^3 - 1 = 7$. $\eta = 7/11 = 0.636$. (With SR the window would be 4: $4/11 = 36.4\%$.)

</details>

**Q5 (NUM).** Pure ALOHA channel: frames offered at a rate giving $G = 0.5$ per frame time. If the frame time is 2 ms, how many frames per second are successfully sent?

<details><summary>Answer</summary>

**Answer:** about 92 frames/s  
**Solution:** $S = Ge^{-2G} = 0.5e^{-1} = 0.1839$ successes per frame time. Frame times per second $= 500$. $0.1839\times500 = 91.97 \approx 92$.

</details>

**Q6 (MSQ).** Which are true for CSMA/CD on a 10 Mbps Ethernet?
(A) Minimum frame size is 64 B for a 2.5 km network. (B) After 4 collisions the backoff range is $\{0,\dots,15\}$. (C) A frame of 40 B payload must be padded. (D) Collision detection works because the sender listens while transmitting.

<details><summary>Answer</summary>

**Answer:** (B), (C), (D)  
**Solution:** (B) $2^4 - 1 = 15$ true. (C) payload must be $\ge 46$ B, so 40 B is padded to 46 true. (D) true. (A) is a statement about a particular cable length: the 64 B minimum comes from the 51.2 $\mu$s slot, which covers 2.5 km at the standard design including repeaters; but strictly $2T_p R$ for 2.5 km at $2\times 10^8$ m/s is only 250 bits $= 31.25$ B, so "minimum frame size is 64 B for 2.5 km" is not derived from that cable length; treat (A) as false.

</details>

**Q7 (NUM).** Minimum frame length for a CSMA/CD LAN of 1 Gbps and a 200 m cable, signal speed $2\times10^8$ m/s, in bytes? Then, if the standard requires 512 bytes (Gigabit Ethernet), what extra is needed?

<details><summary>Answer</summary>

**Answer:** 250 bits $\to$ 31.25 B... compute carefully below  
**Solution:** $T_p = 200/(2\times10^8) = 1\ \mu$s. $L_{min} = 2\times10^{-6}\times10^9 = 2000$ bits $= 250$ bytes. (Not 31.25 B — 250 bits was a different, 10 Mbps example.) Gigabit Ethernet keeps a 512-byte slot by padding with carrier extension.

</details>

**Q8 (NUM, hard).** Frames $0$-$11$ are sent using GBN with window 5. The window is always fully sent before the sender learns of loss; frame 7 is lost on its first transmission only. How many transmissions in total? Compare with Selective Repeat.

<details><summary>Answer</summary>

**Answer:** GBN: 17; SR: 13  
**Solution:** Windows: send $0$-$4$ (5), then $5$-$9$ (5): frame 7 is lost, $8, 9$ are discarded by GBN (and 5, 6 were accepted). On timeout GBN goes back to 7: resend $7, 8, 9, 10, 11$ (5 frames, window of 5 starting at 7). Total $= 5 + 5 + 5 = 15$ transmissions.  
**Recount:** after 5, 6 are acknowledged the window starts at 7 and the sender had already sent 7, 8, 9 in round 2 (frames 5-9). So retransmitting $7$ to $11$ = 5 frames. Total $= 5 + 5 + 5 = 15$ for GBN.  
SR: $0$-$4$ (5), $5$-$9$ (5, with 7 lost, 8 and 9 buffered), resend 7 (1), then $10, 11$ (2): $5+5+1+2 = 13$.  
**Final: GBN = 15, SR = 13.**

</details>
