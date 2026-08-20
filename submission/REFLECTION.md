# Reflection - Day 20 Lab (Personal Report)

**Họ Tên:** Ong Xuan Son
**Cohort:** A20-K3
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 - 10 điểm)*

- **OS:** Windows 11 AMD64
- **CPU:** 12th Gen Intel(R) Core(TM) i5-12450H
- **Cores:** 8 physical / 12 logical
- **CPU extensions:** không được probe báo ra
- **RAM:** 15.6 GB
- **Accelerator:** phát hiện NVIDIA GeForce RTX 3050 Laptop GPU 4GB và Vulkan; lần chạy base dùng CPU runtime (`ngl=0`)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cpu-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** primary `Q4_K_M` + compare `UD-Q2_K_XL`

**Chạy ở đâu:** laptop của tôi

**Setup story**: Máy có đủ RAM và có RTX 3050, nhưng CUDA runtime download bị lỗi/treo trong lần setup đầu. Để hoàn thành base track ổn định, tôi dùng prebuilt CPU runtime `win-cpu-x64`; base track không yêu cầu GPU. Model dùng Qwen3.5 0.8B để tải nhanh và chạy gọn hơn.

---

## 2. Đo lường  *(rubric 3, 4, 5 - 20 điểm)*

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3666 | 1382 / 1429 | 17.6 / 18.5 | 2480 / 2579 / 2579 | 57.0 |
| UD-Q2_K_XL | 0.39 | 1355 | 1459 / 1490 | 17.1 / 18.0 | 2548 / 2595 / 2595 | 58.4 |

**Quan sát**: `UD-Q2_K_XL` nhỏ hơn 0.11 GB và decode nhanh hơn 1.02x, nhưng TTFT/E2E lại chậm hơn nhẹ trong run này. Với máy này, 2-bit đáng dùng khi ưu tiên dung lượng hoặc load nhanh; nếu cần chất lượng và latency ổn định, tôi sẽ giữ `Q4_K_M`.

---

## 3. Serving under load  *(rubric 8, 9, 10 - 20 điểm)*

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.27 | 6200 | 12000 | 14000 | 8.4 | 0.0% |
| 50 | 1.05 | 27000 | 49000 | 51000 | 28.9 | 0.0% |

- **Offered load tăng 5x, throughput thực tăng:** 0.83x
- **P95 tăng:** 4.08x
- **Effective concurrency ở 50 users:** 28.9 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`**: 3.91 / 4 slots

**Saturation reading**: Server bão hòa trước hoặc tại 50 users. RPS giảm từ 1.27 xuống 1.05 khi users tăng 5x, trong khi P95 tăng từ 12s lên 49s. Metrics cho thấy busy slots đạt 3.91/4 và requests deferred đạt 46, nên latency tăng thêm là queue time. Để tăng goodput@SLO, tôi sẽ thử tăng `--parallel` hoặc giảm max output tokens trước, vì thread count đã tới knee ở 8.

---

## 4. Integration  *(rubric 12, 13 - 15 điểm)*

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost laptop | stub |
| N17 Data pipeline | không có DAG/batch job trong lần chạy này | stub |
| N18 Lakehouse | toy in-memory docs | stub |
| N19 Vector + features | keyword overlap retrieval | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 6223.8 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection**: Bottleneck nằm ở LLM stage, đúng như kỳ vọng vì embed/retrieve đang là stub rất nhẹ. Nếu phải giảm latency pipeline 2x, tôi sẽ giảm output tokens hoặc dùng runtime/offload nhanh hơn cho LLM trước; tối ưu retrieval không có nhiều ý nghĩa khi nó gần 0 ms.

---

## 5. The single change that mattered most  *(rubric 11 - 10 điểm)*

**Change:** tune thread count cho decode, so sánh `-t 1` với `-t 8`

```text
before:  19.5 tok/s at -t 1
after:   58.3 tok/s at -t 8
speedup: 2.99x
```

**Tại sao nó work**:

Decode của llama.cpp trên CPU bị giới hạn nhiều bởi memory bandwidth và cache behavior, nhưng vẫn cần đủ threads để song song hóa các phép tính trên mỗi token. Từ `-t 1` lên `-t 4`, throughput tăng mạnh vì CPU có thêm core làm việc thật. Từ `-t 4` lên `-t 8`, vẫn tăng nhưng nhỏ hơn, cho thấy máy đã gần chạm knee của memory bandwidth.

Qua `-t 8`, thêm logical/oversubscribed threads không giúp nữa: `-t 12` giảm còn 52.8 tok/s và `-t 24` rơi mạnh còn 23.1 tok/s. Các thread thêm cạnh tranh chung cache và memory channels, đồng thời tạo scheduling overhead. Vì vậy best setting trùng với 8 physical cores, không phải số logical threads cao nhất.

---

## 6. Bonus  *(optional - tối đa 20 điểm)*

**Đã làm:** không làm bonus trong lần nộp base track này.

**Numbers:**

```text
before:  n/a
after:   n/a
speedup: n/a
```

**Điều này nói lên gì mà deck chưa nói:**

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

Offered load tăng từ 10 lên 50 users nhưng RPS không tăng; thay vào đó P95 tăng hơn 4 lần. Đây là minh chứng rất rõ rằng sau saturation, thêm user chủ yếu tạo queue time chứ không tạo throughput.

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section required trong các file `benchmarks/*.md` đã được thay bằng nhận xét của tôi
- [x] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` / `scripts\verify.py` exit 0
- [ ] Repo GitHub ở chế độ public
- [ ] Đã paste public URL vào VinUni LMS
- [x] Không commit `models/*.gguf` hay `runtime/`
