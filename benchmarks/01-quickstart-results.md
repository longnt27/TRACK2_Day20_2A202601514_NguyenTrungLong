# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=10` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 1067 | 47 / 57 | 8.8 / 9.5 | 598 / 658 / 658 | 113.8 |
| UD-Q2_K_XL | 0.39 | 1029 | 45 / 46 | 8.2 / 8.3 | 560 / 568 / 568 | 122.3 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.07x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## Nhận xét

Q2 nhỏ hơn 0.11 GB (22%) và decode nhanh hơn 1.07x. Nhưng test 5 câu cho thấy Q2 hay
dài dòng, sai format và chỉ đúng strict 1/5, trong khi Q4 đúng 2/5. Máy có 16 GB RAM
nên mình chọn Q4; mức tăng 7% không đáng đổi lấy chất lượng thấp hơn.
