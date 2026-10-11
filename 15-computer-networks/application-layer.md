# Application Layer: DNS, HTTP, Email, and FTP

> **Paper:** CS · **Priority:** P1 · **Plan topics:** DNS, HTTP, SMTP, POP, IMAP, FTP, Email
> **Prerequisites:** [TCP and transport](transport-layer-tcp.md), [IPv4 addressing](ipv4-addressing.md)
> **PYQ weightage:** Medium · 6 Qs since 2015 (3 since 2021) · DA: 0 · Trend: steady · Most asked: HTTP (persistent vs non-persistent RTT delay calculations), DNS (hierarchy, resolution steps, record types), Email protocols (SMTP/POP3/IMAP), FTP dual connections

## Quick glance

- **Exam focus:** HTTP RTT delay calculations (non-persistent vs persistent without pipelining vs persistent with pipelining; single vs parallel TCP connections), DNS resolution mechanics (iterative vs recursive queries, record types `A`, `AAAA`, `CNAME`, `MX`, `NS`), Email protocol roles (SMTP for push/transfer vs POP3/IMAP for retrieval), and FTP dual-port structure (control port 21 vs data port 20). Numerical questions almost always test total time / RTT counts to fetch web pages with embedded objects.
- **DNS Hierarchy:** Root nameservers (`.`) $\to$ Top-Level Domain (TLD) servers (`.com`, `.edu`, `.in`) $\to$ Authoritative nameservers (`gate.iisc.ac.in`).
- **DNS Resolution:**
  - **Iterative:** Local resolver queries each server in turn; each server replies with referral to the next server down the hierarchy.
  - **Recursive:** Queried server takes responsibility to resolve the query completely and returns the final answer to the caller.
  - **Transport:** Uses **UDP port 53** for standard queries ($\le 512$ bytes); uses **TCP port 53** for zone transfers (AXFR) and responses $> 512$ bytes (EDNS0).
- **HTTP (HyperText Transfer Protocol):** Client-server, request-response, stateless.
  - **Non-persistent (HTTP/1.0):** 1 TCP connection per object. Latency per object $= 2\text{ RTT} + T_{\text{transmit}}$ ($1 \text{ RTT}$ TCP handshake $+ 1 \text{ RTT}$ HTTP request/response).
  - **Persistent without pipelining (HTTP/1.1 default):** TCP connection kept open. Base HTML takes $2 \text{ RTT}$; each referenced object takes $1 \text{ RTT}$. Total $= (k + 2) \text{ RTT}$.
  - **Persistent with pipelining:** After TCP handshake and base HTML ($2 \text{ RTT}$), all $k$ referenced objects requested back-to-back in $1 \text{ RTT}$. Total $= 3 \text{ RTT}$.
- **Email Architecture:** User Agent (MUA) $\xrightarrow{\text{SMTP}}$ Sender Mail Server $\xrightarrow{\text{SMTP}}$ Receiver Mail Server $\xrightarrow{\text{POP3/IMAP}}$ Receiver User Agent (MUA).
  - **SMTP:** Push protocol, TCP port 25, 7-bit ASCII text only. MIME encodes binary media using Base64 ($3 \text{ bytes} \to 4 \text{ ASCII chars}$, $33.3\%$ overhead).
  - **POP3:** Pull protocol, TCP port 110, downloads emails locally and deletes from server (stateless across sessions).
  - **IMAP:** Pull protocol, TCP port 143, organizes messages in server folders, stateful, supports partial header download.
- **FTP (File Transfer Protocol):** Out-of-band architecture using two separate connections:
  - **Control connection (TCP port 21):** Stays open for the entire interactive session. Sends commands (`USER`, `PASS`, `RETR`, `STOR`).
  - **Data connection (TCP port 20):** Created and closed dynamically for each individual file transfer.

## 1. Domain Name System (DNS): Architecture, Record Types, and Query Delay [PYQ: Medium]

DNS provides a distributed, hierarchical database mapping human-readable hostnames to IP addresses.

```text
                              Root DNS (".")
                             /      |       \
                            /       |        \
                       .com        .edu       .in (TLD Servers)
                       /            |          \
                  google.com     mit.edu    iisc.ac.in (Authoritative)
```

### Resource Record (RR) types
A DNS record is a 4-tuple: `(Name, Value, Type, TTL)`.

| Type | Name | Value | Purpose |
|---|---|---|---|
| **A** | Hostname | IPv4 Address | Standard forward lookup (`gate.org` $\to$ `192.0.2.1`) |
| **AAAA** | Hostname | IPv6 Address | 128-bit IPv6 address resolution |
| **CNAME** | Alias hostname | Canonical (real) name | Aliasing (`www.example.com` $\to$ `server1.example.com`) |
| **NS** | Domain name | Hostname of nameserver | Identifies authoritative nameserver for the domain |
| **MX** | Domain name | Hostname of mail server | Mail exchange server for the domain (includes priority) |
| **PTR** | Reverse IP (`.in-addr.arpa`) | Hostname | Reverse lookup (IP to hostname) |

### Iterative vs Recursive resolution
- **Recursive query:** Client places full resolution burden on the queried server. The local DNS server typically offers recursive service to stub resolvers on end hosts.
- **Iterative query:** Queried server returns the best referral it has ("I don't know the answer, but ask this TLD server"). The local DNS resolver issues iterative queries sequentially:
  $$\text{Local Resolver} \xrightarrow{\text{Query}} \text{Root} \xrightarrow{\text{Referral}} \text{TLD} \xrightarrow{\text{Referral}} \text{Authoritative} \xrightarrow{\text{IP Address}} \text{Local Resolver}$$

```text
Stub Resolver (Client)
      │ 1. Recursive Query
      ▼
Local DNS Server ── 2. Iterative ──> Root DNS Server
                 <── 3. Referral ───
                 ── 4. Iterative ──> TLD Server (.in)
                 <── 5. Referral ───
                 ── 6. Iterative ──> Authoritative DNS (iisc.ac.in)
                 <── 7. IP Address ─
      │ 8. Final IP Reply
      ▼
Client Cache
```

**Worked example 1 (DNS resolution delay calculation).** A client host issues a DNS request for `www.nptel.ac.in`. Assume:
- The client's local DNS server has a completely cold cache (no cached entries).
- $\text{RTT}$ between client and local DNS server is $\text{RTT}_0 = 2 \text{ ms}$.
- $\text{RTT}$ from local DNS server to Root server is $\text{RTT}_1 = 20 \text{ ms}$.
- $\text{RTT}$ from local DNS server to `.in` TLD server is $\text{RTT}_2 = 25 \text{ ms}$.
- $\text{RTT}$ from local DNS server to `.ac.in` server is $\text{RTT}_3 = 20 \text{ ms}$.
- $\text{RTT}$ from local DNS server to `nptel.ac.in` authoritative server is $\text{RTT}_4 = 15 \text{ ms}$.

Compute total time elapsed before the client receives the IP address if iterative queries are used by the local DNS server.

**Step-by-step solution:**
1. Client sends recursive query to Local DNS: $\text{RTT}_0 = 2 \text{ ms}$.
2. Local DNS queries Root server: $\text{RTT}_1 = 20 \text{ ms}$.
3. Local DNS queries `.in` TLD server: $\text{RTT}_2 = 25 \text{ ms}$.
4. Local DNS queries `.ac.in` server: $\text{RTT}_3 = 20 \text{ ms}$.
5. Local DNS queries authoritative `nptel.ac.in` server: $\text{RTT}_4 = 15 \text{ ms}$.
6. Local DNS returns result to client: included in round-trip $\text{RTT}_0$.
$$\text{Total Time} = \text{RTT}_0 + \text{RTT}_1 + \text{RTT}_2 + \text{RTT}_3 + \text{RTT}_4 = 2 + 20 + 25 + 20 + 15 = 82 \text{ ms}$$

If the local DNS server had cached the `.in` TLD server address, steps 2 and 3 would be skipped, saving $20 + 25 = 45 \text{ ms}$.

## 2. HyperText Transfer Protocol (HTTP): Request/Response, Status Codes, and Latency [PYQ: Medium]

HTTP runs over TCP (default port 80; HTTPS runs over TLS on port 443). HTTP is a stateless protocol: the server retains no information about past client requests. Application state is maintained using **Cookies** and session tokens.

### HTTP Methods and Idempotency
- **Idempotent method:** Executing the request multiple times produces the identical side-effect on the server as executing it once ($f(f(x)) = f(x)$).
  - `GET`, `HEAD`: Safe and Idempotent (read-only).
  - `PUT`: Idempotent (replaces entire resource).
  - `DELETE`: Idempotent (deleting resource multiple times leaves resource deleted).
- **Non-idempotent method:**
  - `POST`: Submits data to be processed (e.g. creating duplicate orders).

### HTTP Status codes

| Code Range | Category | Common Examples |
|---|---|---|
| **1xx** | Informational | `100 Continue` |
| **2xx** | Success | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | Redirection | `301 Moved Permanently`, `302 Found`, `304 Not Modified` (conditional GET cache hit) |
| **4xx** | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found` |
| **5xx** | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

### Connection models and latency calculations
Let a web page consist of **1 base HTML file** and **$k$ embedded objects**. Let $\text{RTT}$ be the round-trip time between client and server, and assume transmission delays are small ($T_{\text{transmit}} \approx 0$).

1. **Non-persistent HTTP without parallel connections (HTTP/1.0 serial):**
   - Each object requires a separate TCP connection: $1 \text{ RTT}$ (TCP handshake) $+ 1 \text{ RTT}$ (HTTP GET/response) $= 2 \text{ RTT}$.
   - Total time $= 2\text{ RTT} \text{ (base)} + k \times 2\text{ RTT} = 2(k + 1)\text{ RTT}$.
2. **Non-persistent HTTP with $p$ parallel connections:**
   - Base HTML takes $2\text{ RTT}$.
   - The $k$ objects are fetched in batches of $p$ parallel connections.
   - Total time $= 2\text{ RTT} + \lceil k / p \rceil \times 2\text{ RTT}$.
3. **Persistent HTTP without pipelining (HTTP/1.1 non-pipelined):**
   - Single TCP connection kept open. Handshake: $1\text{ RTT}$. Base HTML: $1\text{ RTT}$.
   - Each of the $k$ objects is requested sequentially over the open connection ($1\text{ RTT}$ each).
   - Total time $= 2\text{ RTT} + k \times 1\text{ RTT} = (k + 2)\text{ RTT}$.
4. **Persistent HTTP with pipelining (HTTP/1.1 pipelined):**
   - Single TCP connection. Handshake: $1\text{ RTT}$. Base HTML: $1\text{ RTT}$.
   - Client sends HTTP GET requests for all $k$ embedded objects back-to-back as soon as referenced links are parsed, taking $1\text{ RTT}$ for all objects combined.
   - Total time $= 2\text{ RTT} + 1\text{ RTT} = 3\text{ RTT}$.

```text
Non-persistent Serial: [TCP][GET HTML] [TCP][GET Obj1] [TCP][GET Obj2] ... = 2(k+1) RTT
Persistent Non-pipe:   [TCP][GET HTML] [GET Obj1]      [GET Obj2]      ... = (k+2) RTT
Persistent Pipelined:  [TCP][GET HTML] [GET Obj1, Obj2, ... Objk]          = 3 RTT
```

**Worked example 2 (Comprehensive HTTP download time).** A browser requests a web page containing a base HTML file and 5 embedded image objects. The round-trip time between client and server is $\text{RTT} = 50 \text{ ms}$. DNS lookup is already resolved. Neglect transmission delays.
Calculate the total time required to download the web page under:
1. Non-persistent HTTP without parallel connections.
2. Non-persistent HTTP with 5 parallel connections allowed simultaneously.
3. Persistent HTTP without pipelining.
4. Persistent HTTP with pipelining.

**Step-by-step solution:**
Here $k = 5$ objects, $\text{RTT} = 50 \text{ ms}$.
1. **Non-persistent serial:**
   $$\text{Time} = 2(k + 1)\text{ RTT} = 2(5 + 1) \times 50 \text{ ms} = 12 \times 50 = 600 \text{ ms}$$
2. **Non-persistent with 5 parallel connections:**
   - Base HTML: $2\text{ RTT} = 100 \text{ ms}$.
   - All 5 objects requested in parallel over 5 parallel TCP connections: $\lceil 5/5 \rceil \times 2\text{ RTT} = 2\text{ RTT} = 100 \text{ ms}$.
   $$\text{Total Time} = 100 + 100 = 200 \text{ ms} \quad (4 \text{ RTT})$$
3. **Persistent without pipelining:**
   $$\text{Time} = (k + 2)\text{ RTT} = (5 + 2) \times 50 \text{ ms} = 7 \times 50 = 350 \text{ ms}$$
4. **Persistent with pipelining:**
   $$\text{Time} = 3\text{ RTT} = 3 \times 50 \text{ ms} = 150 \text{ ms}$$

## 3. Electronic Mail: SMTP, POP3, IMAP, and MIME [PYQ: Medium]

```text
Sender Host                      Sender Server                   Receiver Server                  Receiver Host
┌───────────┐                    ┌───────────┐                   ┌───────────┐                    ┌───────────┐
│ Alice UA  │ ─── SMTP (Pushed) ─> │ Mail      │ ── SMTP (Relayed) ─> │ Mail      │ ── POP3/IMAP (Pulled) ─> │ Bob UA    │
│ (Client)  │     TCP Port 25    │ Server    │    TCP Port 25    │ Server    │    Port 110 / 143      │ (Client)  │
└───────────┘                    └───────────┘                   └───────────┘                    └───────────┘
```

### Protocol roles and characteristics
1. **Simple Mail Transfer Protocol (SMTP):**
   - **Push protocol:** Used by clients to push email to mail servers, and by mail servers to relay email between servers.
   - Runs over **TCP port 25** (submission often uses port 587 with STARTTLS).
   - In-band protocol: Commands and message data share the same TCP connection.
   - Character set limitation: Transmits **7-bit ASCII text only**.
   - Standard commands: `HELO` / `EHLO` (handshake), `MAIL FROM:`, `RCPT TO:`, `DATA`, `QUIT`.
2. **Multipurpose Internet Mail Extensions (MIME):**
   - Extends SMTP to carry non-ASCII binary attachments (audio, images, PDFs).
   - **Base64 encoding:** Converts binary data into printable ASCII:
     - Takes chunks of **3 bytes (24 bits)** and splits into **four 6-bit values**.
     - Each 6-bit value maps to one of 64 ASCII characters (`A-Z`, `a-z`, `0-9`, `+`, `/`).
     - **Size expansion:** Size increases by factor of $\frac{4}{3}$ ($33.3\%$ overhead).
3. **Post Office Protocol v3 (POP3):**
   - **Pull protocol:** Client downloads messages from mailbox server over **TCP port 110**.
   - Stateless across sessions; "download-and-delete" mode removes mail from server.
4. **Internet Message Access Protocol (IMAP):**
   - **Pull protocol:** Client accesses messages over **TCP port 143**.
   - Keeps messages centrally on server; organizes mail into folders; stateful across sessions.
   - Supports partial retrieval (e.g., fetch headers without large attachments).

**Worked example 3 (MIME Base64 encoding size).** Alice sends Bob an email with a 30 KB ($30 \times 1024$ bytes) image attachment encoded using Base64.
1. What is the size of the attachment after Base64 encoding?
2. What protocol and transport ports are used when Alice's client sends the email to her outgoing server, and when Bob's client fetches it via IMAP?

**Step-by-step solution:**
1. Original binary size $= 30 \times 1024 = 30,720$ bytes.
   Base64 encodes every 3 bytes into 4 ASCII bytes:
   $$\text{Encoded Size} = \frac{30,720}{3} \times 4 = 10,240 \times 4 = 40,960 \text{ bytes} = 40 \text{ KB}$$
   (An increase of exactly $\frac{10}{30} = 33.33\%$).
2. Alice $\to$ Outgoing Server: **SMTP** over **TCP Port 25** (or 587).
   Bob $\to$ Mail Server: **IMAP** over **TCP Port 143**.

## 4. FTP and Application Transport Mapping [PYQ: Low]

### File Transfer Protocol (FTP) architecture
FTP uses **two parallel connections** between client and server, known as an **out-of-band** control model:
1. **Control Connection (TCP port 21):**
   - Established first and maintained for the entire user session.
   - Transmits user commands (`USER`, `PASS`, `LIST`, `RETR`, `STOR`, `QUIT`) and 3-digit server status responses.
2. **Data Connection (TCP port 20):**
   - Created dynamically whenever a file or directory listing is transferred, and closed immediately upon completion.
   - Separate data connection opened for each file.

```text
FTP Client                                              FTP Server
    │              Control Connection (TCP Port 21)         │
    ├───────────────────────────────────────────────────────┤ (Persists entire session)
    │                                                       │
    │              Data Connection (TCP Port 20)            │
    ├=======================================================> (Created & closed per file)
```

### Active vs Passive FTP
- **Active Mode (`PORT` command):** Client opens a random port $P$ and sends `PORT IP, P` on control channel. Server initiates TCP data connection from server port 20 to client port $P$. **Fails behind client NAT/firewalls** because incoming connections are blocked.
- **Passive Mode (`PASV` command):** Client sends `PASV` on control channel. Server opens random unprivileged port $S$ and replies. Client initiates TCP connection to server port $S$. **Firewall/NAT friendly**.

### Comprehensive Application Protocol Reference

| Protocol | Full Name | Transport Protocol | Default Port | Mode / Notes |
|---|---|---|---|---|
| **DNS** | Domain Name System | UDP / TCP | 53 | UDP for queries; TCP for zone transfer |
| **HTTP** | HyperText Transfer Protocol | TCP | 80 | Request-response, stateless, in-band |
| **HTTPS** | HTTP over TLS | TCP | 443 | Encrypted HTTP |
| **FTP** | File Transfer Protocol | TCP | 21 (control), 20 (data) | Out-of-band dual connection |
| **SMTP** | Simple Mail Transfer Protocol | TCP | 25 (or 587) | Push protocol, 7-bit ASCII, in-band |
| **POP3** | Post Office Protocol v3 | TCP | 110 | Pull protocol, download-and-delete |
| **IMAP** | Internet Message Access Protocol | TCP | 143 | Pull protocol, server-side folders |
| **Telnet** | Telecommunication Network | TCP | 23 | Unencrypted remote login |
| **SSH** | Secure Shell | TCP | 22 | Encrypted secure remote login |
| **DHCP** | Dynamic Host Configuration | UDP | 67 (server), 68 (client) | DORA auto-configuration |

**Worked example 4 (Protocol identification).** Match each scenario with its primary protocol:
1. Translating `gate.iitk.ac.in` to IP address `172.27.16.5`: **DNS** (UDP 53).
2. Transferring a web page with persistent TCP connection: **HTTP/1.1** (TCP 80).
3. Pushing an outgoing email from a university mail server to Gmail: **SMTP** (TCP 25).
4. Retrieving email while synchronizing mailbox folders across phone and laptop: **IMAP** (TCP 143).

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Non-persistent serial HTTP | $\text{Delay} = 2(k + 1)\text{ RTT}$ | Base HTML $+ k$ objects via serial connections |
| Non-persistent $p$ parallel | $\text{Delay} = 2\text{ RTT} + \lceil k/p \rceil \times 2\text{ RTT}$ | Browser opening $p$ parallel TCP sockets |
| Persistent non-pipelined | $\text{Delay} = (k + 2)\text{ RTT}$ | Reusing connection with sequential requests |
| Persistent pipelined | $\text{Delay} = 3\text{ RTT}$ | Reusing connection with pipelined GETs |
| Base64 expansion | $\text{Size}_{\text{new}} = \text{Size}_{\text{orig}} \times \frac{4}{3}$ | MIME binary email encoding overhead |
| DNS query transport | UDP port 53 (standard) | DNS name resolution |
| DNS zone transfer | TCP port 53 | Primary to secondary DNS sync |
| FTP control port | TCP port 21 | FTP commands and session setup |
| FTP data port | TCP port 20 | Actual file transfer in active mode |

## GATE traps

- **Persistent pipelined HTTP delay is 3 RTT, not 2 RTT:** Handshake takes $1\text{ RTT}$, base HTML takes $1\text{ RTT}$, and pipelined object batch takes $1\text{ RTT}$ $\implies$ Total $= 3\text{ RTT}$.
- **FTP is out-of-band, HTTP/SMTP are in-band:** FTP uses separate control (port 21) and data (port 20) channels. HTTP and SMTP send commands and data over the same channel.
- **SMTP is a push protocol; POP3/IMAP are pull protocols:** A client cannot retrieve email using SMTP; an email server cannot push email to an offline client using POP3.
- **DNS uses both UDP and TCP:** Standard queries use UDP port 53 for speed. Zone transfers and responses exceeding 512 bytes use TCP port 53.
- **HTTP status code classes:** `3xx` is redirection, `4xx` is client error, `5xx` is server error. `404` is client error (requesting non-existent resource), not server fault.
- **Base64 overhead is 33.3%, not 25%:** 3 input bytes produce 4 output characters ($4/3 = 1.333$).

## Connections

- [TCP and transport](transport-layer-tcp.md) — HTTP, SMTP, POP3, and FTP control/data streams rely on TCP connections and port demultiplexing.
- [IPv4 addressing](ipv4-addressing.md) — DNS queries resolve domain names to IPv4 A records; Active FTP requires special ALG support to traverse NAT firewalls.
- [Layering and performance](layering-switching-performance.md) — RTT propagation delay dominates application response time in interactive web downloads.

## Practice

**Q1 (MCQ).** Which of the following application layer protocols uses **two separate TCP connections** (one for control commands and another for data transfer)?
(A) HTTP  
(B) SMTP  
(C) FTP  
(D) DNS  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**
FTP uses out-of-band architecture: TCP port 21 is maintained for control commands throughout the session, and TCP port 20 is established dynamically for each data transfer.

</details>

**Q2 (NAT).** A client browser downloads a web page consisting of 1 base HTML file and 8 referenced image objects. The round-trip time between client and server is $25 \text{ ms}$. Assume transmission time and DNS lookup delays are negligible. If the browser uses **persistent HTTP with pipelining**, what is the total time (in ms) taken to download the entire web page?

<details><summary>Answer</summary>

**Answer:** 75  
**Solution:**
Under persistent HTTP with pipelining:
1. TCP 3-way handshake: $1 \text{ RTT}$.
2. Request and receive base HTML file: $1 \text{ RTT}$.
3. After parsing base HTML, all 8 objects are requested together in a single pipelined batch: $1 \text{ RTT}$.
$$\text{Total Time} = 3 \text{ RTT} = 3 \times 25 \text{ ms} = 75 \text{ ms}$$

</details>

**Q3 (MCQ).** Which of the following DNS record types is used specifically to identify the mail server responsible for receiving email for a domain?
(A) A  
(B) CNAME  
(C) MX  
(D) NS  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**
`MX` (Mail Exchanger) records specify the mail server responsible for accepting messages on behalf of a domain. `A` maps to IPv4, `CNAME` is a canonical alias, and `NS` delegates a zone to an authoritative nameserver.

</details>

**Q4 (MSQ).** Which of the following statements about HTTP and SMTP are **CORRECT**?
(A) HTTP `GET` and `PUT` methods are both idempotent.  
(B) SMTP is an in-band protocol, while FTP is an out-of-band protocol.  
(C) HTTP status code `304 Not Modified` indicates that a cached resource is still fresh.  
(D) SMTP can natively transmit 8-bit binary multimedia files without MIME encoding.  

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:**
- (A) is correct: `GET` is read-only (idempotent); `PUT` replaces existing resource state (idempotent).
- (B) is correct: SMTP sends commands and data on TCP port 25; FTP separates control (21) and data (20).
- (C) is correct: Conditional `GET` returns 304 if ETag/If-Modified-Since matches cache.
- (D) is false: Standard SMTP supports only 7-bit ASCII text; MIME Base64 encoding is required for binary attachments.

</details>

**Q5 (NAT).** An email client attaches a binary file of size 12,000 bytes using MIME Base64 encoding. What is the size of the encoded attachment (in bytes) before transmitting it via SMTP?

<details><summary>Answer</summary>

**Answer:** 16000  
**Solution:**
Base64 groups binary data into 3-byte blocks and encodes each into 4 ASCII characters:
$$\text{Encoded Size} = \frac{12,000 \text{ bytes}}{3} \times 4 = 4,000 \times 4 = 16,000 \text{ bytes}$$

</details>

**Q6 (MCQ).** In DNS resolution, what is the key distinction between an **iterative** query and a **recursive** query?
(A) Iterative queries use TCP, while recursive queries use UDP.  
(B) In an iterative query, the queried server returns the best referral to the next nameserver, whereas in a recursive query, the queried server takes full responsibility to obtain the final answer.  
(C) Iterative queries are answered by root servers only; recursive queries are answered by authoritative servers only.  
(D) Iterative queries cannot be cached, while recursive queries are always cached.  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**
An iterative query asks for a referral or direct answer if known ("tell me who to ask next"), whereas a recursive query demands the server perform subsequent lookups and return the resolved IP.

</details>

**Q7 (NAT).** A web browser fetches a web page containing 1 base HTML file and 6 embedded images using **non-persistent HTTP without parallel connections**. The round-trip time between browser and server is $30 \text{ ms}$. Neglecting transmission time and DNS lookup, calculate the total download time in ms.

<details><summary>Answer</summary>

**Answer:** 420  
**Solution:**
Under non-persistent serial HTTP:
- Base HTML requires $1 \text{ RTT}$ (TCP handshake) $+ 1 \text{ RTT}$ (HTTP GET/response) $= 2 \text{ RTT}$.
- Each of the 6 embedded objects requires a separate TCP connection: $6 \times 2 \text{ RTT} = 12 \text{ RTT}$.
$$\text{Total Time} = 2 \text{ RTT} + 12 \text{ RTT} = 14 \text{ RTT} = 14 \times 30 \text{ ms} = 420 \text{ ms}$$

</details>
