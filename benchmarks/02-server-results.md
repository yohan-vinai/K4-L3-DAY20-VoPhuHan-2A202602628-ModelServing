# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 45 | 0.83 | 11000 | 16000 | 17000 | 8.6 | 0.0% |
| 50 | 37 | 0.64 | 27000 | 57000 | 57000 | 18.3 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.78x** (16% of linear) |
| P95 latency | **3.56x** |
| Effective concurrency at 50 users | 18.3 vs `--parallel 4` slots (occupancy/slot ratio 4.58) |

**Saturated.** Throughput delivered only 0.78x for 5x the offered load, and effective concurrency (18.3) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.78x while P95 moved 3.56x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

The server is saturated by the tested 10-user load and remains saturated at 50 users: effective concurrency is 8.6 versus four slots at 10 users, while at 50 users P95 reaches 57 s and throughput falls from 0.83 to 0.64 RPS despite five times as many users. At a 20 s P95 SLO, the 10-user run is within target and the 50-user run is not. I would first cap long prompt/output sizes or admission rate to reduce slot holding time and queue growth; adding more slots alone would not add M1 memory bandwidth.
