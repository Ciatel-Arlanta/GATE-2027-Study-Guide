# Routing: distance-vector and link-state

> **Paper:** CS · **Priority:** P0 · **Plan topics:** routing, distance-vector, link-state, forwarding
> **Prerequisites:** [Graph theory](../01-discrete-mathematics/graph-theory.md) · [IPv4 addressing](ipv4-addressing.md)

## Quick glance
- Routing computes paths; forwarding sends each packet using a local table.
- Distance-vector is distributed Bellman–Ford: exchange distances with neighbours.
- Link-state floods topology; each router runs Dijkstra locally.
- Count-to-infinity is a distance-vector convergence problem; split horizon/poisoning can mitigate it.
- Link-state uses more topology/memory but converges with a global map.

## 1. Distance-vector update
Router x updates distance to destination y as $D_x(y)=\min_v[c(x,v)+D_v(y)]$. Example x has neighbours a cost 1 with advertised y-cost 5, b cost 3 with y-cost 1. Choose via b: total 4 rather than 6.

## 2. Link-state
Routers discover neighbours/costs, flood link-state advertisements, build identical graph, then run Dijkstra from themselves. Each computes next hop from predecessor tree. Changes propagate as new advertisements.

## 3. Forwarding and prefixes
Destination address is matched against routing entries; longest prefix wins. Routing protocols populate routes, while forwarding plane performs per-packet lookup.

## GATE traps
- A route is not necessarily a direct neighbour; table stores next hop.
- Distance-vector advertisements contain destination costs, not full topology.
- Link-state routers compute independently from flooded common topology.
- Dijkstra assumes nonnegative link costs.

## Connections
- [Shortest paths](../08-algorithms/shortest-paths.md) — Bellman–Ford and Dijkstra are routing foundations.
- [IPv4 addressing](ipv4-addressing.md) — CIDR route aggregation and longest match.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — network as weighted graph.

## Practice
**Q1.** x to a cost 2; a advertises distance to z=5. Candidate x→z via a?
<details><summary>Answer</summary> 7.</details>

**Q2.** Which approach floods topology and runs shortest path locally?
<details><summary>Answer</summary> Link-state.</details>

**Q3.** Route table has matching /16 and /24. Which is used?
<details><summary>Answer</summary> /24, the longest prefix.</details>
