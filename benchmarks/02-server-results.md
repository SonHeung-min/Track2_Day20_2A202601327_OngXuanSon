# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` - llama.cpp `b10488` -
`--parallel 4` - `ctx=2048` - `threads=8` -
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 73 | 1.27 | 6200 | 12000 | 14000 | 8.4 | 0.0% |
| 50 | 62 | 1.05 | 27000 | 49000 | 51000 | 28.9 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were really in flight, regardless of how many users locust simulated. It counts queued requests too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.83x** (17% of linear) |
| P95 latency | **4.08x** |
| Effective concurrency at 50 users | 28.9 vs `--parallel 4` slots (occupancy/slot ratio 7.22) |

**Saturated.** Throughput delivered only 0.83x for 5x the offered load, and effective concurrency (28.9) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load added beyond that point became queue time rather than throughput.

Throughput moved 0.83x while P95 moved 4.08x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if the SLO is a P95 target then the requests added are no longer served within it.

## Your reading

Server bao hoa truoc hoac tai muc 50 users. Bang chung thuyet phuc nhat la khi offered load tang 5x, throughput thuc te khong tang ma giam tu 1.27 RPS xuong 1.05 RPS, trong khi P95 tang tu 12000 ms len 49000 ms. Effective concurrency 28.9 vuot xa `--parallel 4`, va metrics ghi peak busy slots 3.91/4, nen phan latency tang them chu yeu la queue time.

Neu phai tang goodput tai SLO P95, minh se thu tang `--parallel` hoac giam do dai output/request truoc. Thread count da dat knee o 8, nen them thread khong giai quyet duoc hang doi; can tang slot phuc vu dong thoi hoac giam service time moi request.
