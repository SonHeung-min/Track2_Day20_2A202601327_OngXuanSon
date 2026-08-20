# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` - `--parallel 4` - 10 samples over 60s at 2.0s intervals - raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.91 of 4 slots (98%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a - not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 8683 |

Highest sampled value was **3.91 of 4** slots. This gauge is llama.cpp's average busy slots per decode step, so the number is the highest average sampled, not an instantaneous maximum batch width. A peak near 1 means requests were served one at a time; a peak approaching `--parallel` means the scheduler was packing concurrent requests into shared decode steps. `requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in P95.

## Your observation

Peak batch width was 3.91/4 slots, so continuous batching was active and almost all decode slots were busy under 50-user load. This matches the saturation story in `02-server-results.md`: effective concurrency was 28.9, much higher than 4 slots, so most of that concurrency was queued work, while the actual server compute side was capped around the 4 available decode slots. For slot utilisation I trust `n_busy_slots_per_decode`; for user-facing queuing pressure I trust effective concurrency from Little's Law.
