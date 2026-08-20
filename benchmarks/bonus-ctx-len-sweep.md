# Bonus - Context-length sweep (prefill cost)

Host `Darwin-arm64` · llama.cpp `b10488` ·
`threads=10` `ngl=99` · RAM 16.0 GB

| Prompt tokens | Prefill (tok/s) | TTFT contribution (ms) | vs linear scaling |
|:--|--:|--:|--:|
| 256 | 2059.1 | 124.3 | 1.00x |
| 1024 | 2087.2 | 490.6 | 0.99x |
| 2048 | 2041.0 | 1003.4 | 1.01x |
| 4096 | 1961.7 | 2088.0 | 1.05x |
| 8192 | 1780.0 | 4602.2 | 1.16x |

At 8192 tokens, prefill costs **4602 ms** --
1.16x what linear scaling from the smallest point would predict. That excess
is attention's O(N^2) term becoming visible, and every millisecond of it lands in TTFT
before the user sees a single token.

Either way, this is the number to remember when someone proposes stuffing more retrieved
context into a RAG prompt "because the context window allows it". Prefill is paid in full,
on every request, before the first token appears.

## Nhận xét

Từ 4096 token, prefill 2.09 giây đã lớn hơn mean LLM latency 1.33 giây của pipeline.
Ở 8192 token nó lên 4.60 giây và chậm hơn tuyến tính 16%, nên quadratic attention đã
lộ rõ. RAG không nên nhét context chỉ vì còn cửa sổ; mình sẽ giới hạn khoảng 2K token,
rerank rồi chỉ giữ chunk liên quan nhất.
