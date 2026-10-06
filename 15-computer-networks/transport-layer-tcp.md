# TCP, transport, flow control, and congestion

> **Paper:** CS · **Priority:** P0 · **Plan topics:** TCP flow/congestion control, transport service, sockets; selected legacy UDP contrast
> **Prerequisites:** [Layering and performance](layering-switching-performance.md) · **Leads to:** [Application layer](application-layer.md)

## Quick glance
- Transport multiplexes application processes using port numbers; TCP provides reliable ordered byte stream.
- TCP sequence number identifies byte position; ACK is next byte expected (cumulative ACK).
- Flow control protects receiver: advertised window limits unacknowledged bytes.
- Congestion control protects network: congestion window adapts to inferred capacity.
- Effective send window is approximately min(receiver window, congestion window).
- Classic TCP setup uses SYN, SYN-ACK, ACK; teardown is separate.

## 1. Sequence and acknowledgement
If a segment carries 500 bytes beginning at sequence 1000, next expected byte is 1500; ACK=1500 acknowledges bytes through 1499. A lost segment can be retransmitted; cumulative ACK may repeat the last contiguous byte boundary.

## 2. Flow versus congestion control
Receiver advertises rwnd based on buffer availability. Sender congestion window cwnd is sender-side estimate. Throughput is limited by smaller window over RTT. Flow control avoids receiver overflow; congestion control avoids overloading routers and links.

## 3. Congestion response and sockets
Slow start grows cwnd rapidly (roughly doubles each RTT absent loss), then congestion avoidance grows roughly linearly; loss triggers reduction depending on variant. A socket endpoint is identified by address/port; a TCP connection uses source/destination IP and ports.

## GATE traps
- TCP is a byte stream; segment boundaries are not application message boundaries.
- ACK number is next expected sequence number, not last received byte.
- Receiver window and congestion window solve different problems.
- TCP flow control does not guarantee low delay or prevent all network congestion.

## Connections
- [Application layer](application-layer.md) — HTTP commonly uses TCP.
- [IPv4/NAT](ipv4-addressing.md) — connection uses IP addresses and transport ports.
- [OS processes and sockets](../13-operating-systems/processes-threads-syscalls.md) — API boundary for network programs.

## Practice
**Q1.** Segment starts at seq 2000 and carries 100 bytes. Next cumulative ACK?
<details><summary>Answer</summary> 2100.</details>

**Q2.** What does rwnd protect?
<details><summary>Answer</summary> Receiver buffer capacity.</details>

**Q3.** What is TCP's service abstraction?
<details><summary>Answer</summary> Reliable, ordered byte stream between endpoints.</details>
