# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` - host `Windows-AMD64` - llama.cpp `b10488`
Settings: `threads=8` `ngl=0` `ctx=2048`
`max_tokens=64` - warm-up discarded
Completed requests: `Q4_K_M` 10/10 - `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3666 | 1382 / 1429 | 17.6 / 18.5 | 2480 / 2579 / 2579 | 57.0 |
| UD-Q2_K_XL | 0.39 | 1355 | 1459 / 1490 | 17.1 / 18.0 | 2548 / 2595 / 2595 | 58.4 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it grows.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.02x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## Your observation

`UD-Q2_K_XL` nho hon `Q4_K_M` 0.11 GB, tuong duong khoang 22% dung luong tren dia, va toc do decode nhanh hon khoang 1.02x (58.4 tok/s so voi 57.0 tok/s). Tuy nhien TTFT va E2E trong lan do nay lai cham hon nhe, nen loi ve toc do thuc te khong ro rang; diem dang tien nhat cua 2-bit la load nhanh hon va file nho hon.

Voi may nay, 2-bit chi dang doi neu uu tien tiet kiem dung luong/bo nho hoac can load model nhanh. Neu muc tieu la chat luong cau tra loi on dinh hon, minh se giu `Q4_K_M`, vi kich thuoc tang khong qua lon va toc do decode gan nhu tuong duong.
