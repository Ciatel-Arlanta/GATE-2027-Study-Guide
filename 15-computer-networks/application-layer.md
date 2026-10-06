# Application layer: DNS and HTTP

> **Paper:** CS · **Priority:** P1 · **Plan topics:** DNS, HTTP
> **Prerequisites:** [TCP and transport](transport-layer-tcp.md)

## Quick glance
- DNS maps names to records; hierarchy delegates from root through TLD to authoritative servers.
- A record maps name to IPv4; AAAA to IPv6; CNAME aliases another name; MX names mail exchanger.
- HTTP is request-response; methods include GET, POST, PUT, DELETE; status code classes 2xx success, 3xx redirect, 4xx client, 5xx server.
- HTTP is stateless at protocol level; cookies/session tokens can maintain application state.
- Persistent HTTP reuses a transport connection; HTTPS is HTTP protected by TLS.

## 1. DNS resolution
A stub resolver asks a recursive resolver. If uncached, resolver follows root → TLD → authoritative hierarchy (iterative lookups) and caches answers according to TTL. DNS commonly uses UDP for ordinary queries; larger responses and zone transfers can use TCP.

## 2. HTTP exchange
Client sends request line, headers, optional body. Server returns status line, headers, optional body. GET retrieves a representation; POST commonly submits data; PUT replaces/creates at a known URI; DELETE requests removal. Idempotency concerns effect of repeating a request, not whether response bytes match.

## GATE traps
- DNS name lookup is hierarchical and cached; it is not one giant flat table.
- HTTP status class, not exact code alone, indicates broad outcome.
- Stateless means each request is interpreted independently at protocol level; sessions are layered mechanisms.
- DNS TTL is cache lifetime, not a guarantee that data stays unchanged.

## Connections
- [TCP](transport-layer-tcp.md) — carries HTTP exchanges in the common model.
- [IPv4](ipv4-addressing.md) — DNS A records yield IPv4 addresses.
- [Layering](layering-switching-performance.md) — app payload incurs transport/network/link overhead.

## Practice
**Q1.** Which DNS record maps a hostname to IPv4?
<details><summary>Answer</summary> A record.</details>

**Q2.** What class is HTTP status 404?
<details><summary>Answer</summary> 4xx client error.</details>

**Q3.** What does DNS TTL control?
<details><summary>Answer</summary> How long a cached record may be reused before refresh.</details>
