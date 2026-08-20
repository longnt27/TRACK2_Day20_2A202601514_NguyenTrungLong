# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 1600.7 | 1600.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 934.0 | 934.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 1448.6 | 1448.6 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **1327.8** · total **1327.8**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput is more useful than raw throughput** because it focuses on **SLOs (Service Level Objectives)** rather than ignoring them.

Here is the breakdown:
*   **Raw Throughput** ignores SLOs: The text states, "Throughput at saturation ignores SLOs."
*   **Goodput** counts requests per second that met the **TTFT** (Total Throughput to Failure) and **TPOT** (Total Thr

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing key-value pairs (KV cache) in non-contiguous pages.

By organizing the KV cache into non-contiguous pages, the model avoids the wasted space that would otherwise exist if all data were packed tightly into a single contiguous block of memory. This optimization allows the model to utilize more of the avai

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when the **prefill operation is compute-bound** and the **decode operation is memory-bound**, as stated in the context.

This is particularly useful in scenarios where the system needs to optimize for different resource constraints:
*   **Prefill (Compute-bound):** Requires significant CPU/GPU time. Splitting allows the engine to process the data in smaller chunk


## Thành phần và bottleneck

N16 Cloud/IaC, N17 data pipeline, N18 lakehouse và N19 vector/features đều là stub;
N19 dùng keyword overlap, không phải embedding thật. Chỉ N20 llama-server là real.
LLM chiếm 1327.8 ms, tức 100%, đúng kỳ vọng với corpus nhỏ. Muốn giảm một nửa latency,
mình giảm output token/model hoặc cache answer trước; tối ưu retrieve gần 0 ms không có ích.
