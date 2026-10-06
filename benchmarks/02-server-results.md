# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 75 | 1.27 | 6400 | 9200 | 11000 | 8.4 | 0.0% |
| 50 | 69 | 1.18 | 25000 | 29000 | 31000 | 24.9 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.93x** (about 19% of the linear 5x increase) |
| P95 latency | **3.15x** |
| Effective concurrency at 50 users | 24.9 vs `--parallel 4` slots (occupancy/slot ratio 6.22) |

**Saturation is visible by the 10-user run and persists at 50 users.** At 10 users, effective concurrency is already 8.4 against 4 decode slots. Increasing offered users fivefold raised delivered RPS only from 1.27 to 1.18, while P95 latency grew from 9.2 s to 29 s. The 50-user run also had 24.9 effective concurrent requests against 4 slots. These results show added load mainly increasing wait time rather than delivered throughput.

For a 20-second P95 target, the 10-user run is within target (P95 9.2 s; all 75 responses were below 20 s), while the 50-user run misses it (P50 is already 25 s, so fewer than half of its 69 responses met 20 s). The aggregate Locust CSV does not expose an exact per-request goodput count. I would first cap prompt/output length or admission rate to reduce slot holding time and queue growth; adding slots alone would not add M1 memory bandwidth.

## Your reading

Saturation is already evident at 10 users: effective concurrency is 8.4 for four decode slots. At 50 users, five times the offered users produced only 0.93x the RPS, while P95 rose 3.15x to 29 s. With a 20-second P95 target, the 10-user run met it (all 75 responses were below the target); the 50-user run did not (its 25 s median means fewer than half of 69 responses met it). The aggregate CSV cannot give an exact count of SLO-passing requests. I would first cap long prompts/outputs or admission rate to reduce slot holding time and queue growth; adding slots alone would not add M1 memory bandwidth.
