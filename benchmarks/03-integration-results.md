# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` - llama.cpp `b10488` -
retrieval backend: **keyword overlap** - 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 7077.2 | 7077.2 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 5654.9 | 5655.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 5939.2 | 5939.3 |

Mean per stage (ms): embed **0.0** - retrieve **0.0** - llm **6223.8** - total **6223.8**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, Goodput is more useful than raw throughput because it focuses on the actual performance metrics required for a system to meet its Service Level Objectives.

**What problem does PagedAttention actually solve?**

> PagedAttention solves internal fragmentation in GPU memory by storing the KV cache in non-contiguous pages.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when prefill is compute-bound and decode is memory-bandwidth-bound, so each stage benefits from a different scheduling strategy.

## Which N16-N19 pieces are real

N16 Cloud/IaC: stub, chay localhost tren laptop. N17 Data pipeline: stub, khong co DAG/batch job that trong lan chay nay. N18 Lakehouse: stub, dung toy docs trong code thay cho Delta/Iceberg/SQLite production. N19 Vector + features: stub, retrieval dung keyword overlap fallback, khong dung vector index/Feast. N20 Serving: real, goi `llama-server` OpenAI-compatible tren port 8080.

Dominant stage la `llm` voi 6223.8 ms trung binh, chiem 100% total, dung nhu ky vong vi embed va retrieve dang la stub cuc nhe. Neu can giam latency 2x, minh se tan cong LLM stage truoc: giam max tokens, dung quant/runtime nhanh hon, hoac dung GPU/offload; toi uu retrieval khong tao duoc nhieu loi ich khi no gan 0 ms.
