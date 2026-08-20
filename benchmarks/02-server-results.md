# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=10` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 137 | 2.34 | 3200 | 5000 | 5300 | 7.7 | 0.0% |
| 50 | 137 | 2.32 | 19000 | 23000 | 24000 | 40.3 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.99x** (20% of linear) |
| P95 latency | **4.60x** |
| Effective concurrency at 50 users | 40.3 vs `--parallel 4` slots (occupancy/slot ratio 10.08) |

**Saturated.** Throughput delivered only 0.99x for 5x the offered load, and effective concurrency (40.3) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.99x while P95 moved 4.60x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Nhận xét

Server đã bão hòa trước 50 users. Load tăng 5 lần nhưng RPS giảm nhẹ từ 2.34 xuống
2.32, trong khi P95 tăng 4.60 lần lên 23 giây. Effective concurrency là 40.3 với 4
slot, phần latency tăng thêm nằm trong queue. Với SLO P95 6 giây, mình sẽ thêm replica
trước, vì tăng slot trên cùng Metal có thể làm mỗi request chậm hơn.
