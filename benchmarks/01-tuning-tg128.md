# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` - host `Windows-AMD64` - llama.cpp `b10488`
CPU: **8 physical - 12 logical** cores - `ngl=0` - metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 19.5 | 33% |
| 4 | 54.3 | 93% |
| 8 | 58.3 | 100% |
| 12 | 52.8 | 91% |
| 24 | 23.1 | 40% |

**Best**: `-t 8` at 58.3 tok/s
**Slowest tested**: `-t 1` at 19.5 tok/s (2.99x spread)
**Against the physical-core default** (`-t 8`, 58.3 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=8 make bench
```

## Your explanation

Knee nam o `-t 8`, trung voi so physical cores cua CPU. Tu 1 len 4 threads throughput tang manh vi decode co them song song hoa huu ich; tu 4 len 8 chi tang nhe hon, cho thay may da gan cham tran memory bandwidth/cache. Khi tang len 12 logical threads va 24 threads, throughput giam ro: cac thread them canh tranh cung memory channels va scheduling overhead lon hon loi ich tinh toan.
