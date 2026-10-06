# IPv4 addressing, subnetting, and NAT

> **Paper:** CS · **Priority:** P0 · **Plan topics:** IPv4, subnetting, CIDR, fragmentation, NAT
> **Prerequisites:** [Layering and performance](layering-switching-performance.md) · **Leads to:** [Routing](routing.md)

## Quick glance
- IPv4 address is 32 bits; prefix `/p` leaves $32-p$ host bits.
- Block size is $2^{32-p}$ addresses; conventional subnet usable hosts often block size −2 (network and broadcast), with exceptions such as /31 point-to-point.
- Network address zeroes host bits; broadcast address ones them.
- Longest-prefix match chooses most specific route.
- NAT rewrites address/port mapping; IPv4 fragmentation uses offset in 8-byte units.

## 1. Subnet calculation
For `192.168.10.0/26`, host bits=6, block size=64. Subnets in last octet begin 0,64,128,192. First subnet network .0, broadcast .63, usual hosts .1–.62: 62 usable addresses.

To find subnet: apply prefix mask bitwise AND. For a /20, first two octets and top four bits of third octet are network; host bits are remaining 12.

## 2. Routing and fragmentation
Routers use longest-prefix match among entries that match destination. IPv4 fragments carry identification, offset, and MF flag. Fragment offset is measured in 8-byte blocks; all non-final fragment payloads must be multiples of 8 bytes. Reassembly occurs at destination.

## 3. NAT
Basic NAT maps private to public address; NAPT/PAT also maps transport ports, letting many internal flows share one public IPv4 address. NAT changes end-to-end addressing and may require translation state for incoming traffic.

## GATE traps
- Prefix length counts network bits, not host bits.
- Usable-host formula has edge-case conventions for /31 and /32.
- Fragment offset is not a byte count directly: multiply field by 8.
- Choose the longest matching prefix, not the first route listed.

## Connections
- [Routing](routing.md) — prefixes determine forwarding choice.
- [TCP](transport-layer-tcp.md) — NAT commonly tracks transport ports.
- [Layering](layering-switching-performance.md) — packet overhead adds to transmission and delay.

## Practice
**Q1.** How many addresses in a /27 block?
<details><summary>Answer</summary> $2^5=32$.</details>

**Q2.** Usable hosts in ordinary /27 Ethernet subnet?
<details><summary>Answer</summary> 30.</details>

**Q3.** Fragment offset field is 185. Byte offset?
<details><summary>Answer</summary> $185\cdot8=1480$ bytes.</details>
