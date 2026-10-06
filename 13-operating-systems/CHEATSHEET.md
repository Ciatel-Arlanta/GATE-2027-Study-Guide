# Operating Systems — revision sheet

| Topic | Recall |
|---|---|
| Turnaround | completion − arrival |
| Waiting | turnaround − CPU burst (single CPU burst model) |
| Response | first run − arrival |
| Paging | logical page → frame; offset unchanged |
| Page size | $2^k$ bytes ⇒ k offset bits |
| Effective access (one-level PT) | $h(t+m)+(1-h)(t+2m)$ under stated timing assumption |
| Banker’s Need | Max − Allocation; safe if a completion order exists |
| Critical section | mutual exclusion, progress, bounded waiting |
| Deadlock | mutual exclusion + hold/wait + no preemption + circular wait |
| FIFO anomaly | more frames can increase page faults |

Scheduling algorithms: FCFS, SJF/SRTF, priority, Round Robin. Track arrivals and remaining bursts on a timeline; recompute ready queue at each event. Distinguish deadlock, starvation, and race.
