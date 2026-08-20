# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **10 physical · 10 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 111.0 | 97% |
| 5 | 113.2 | 99% |
| 10 | 114.2 | 100% |
| 20 | 112.1 | 98% |

**Best**: `-t 10` at 114.2 tok/s
**Slowest tested**: `-t 1` at 111.0 tok/s (1.03x spread)
**Against the physical-core default** (`-t 10`, 114.2 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=10 make bench
```

## Giải thích

Knee ở 10 thread, đúng số core vật lý: 114.2 tok/s. Curve gần như phẳng vì toàn bộ
layer đã offload sang Metal, CPU chủ yếu lo scheduling. Tăng lên 20 thread không thêm
compute hữu ích, chỉ thêm tranh chấp và overhead nên tụt còn 112.1 tok/s.
