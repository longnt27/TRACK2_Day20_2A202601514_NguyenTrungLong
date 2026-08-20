# Bonus B5/C9 - Embedding serving

Qwen3.5 0.8B Q4 chạy `llama-server --embedding --pooling mean` trên Metal. Đây là chat
model dùng tạm, không phải embedding model chuyên dụng.

| Batch | Latency (ms) | Throughput (texts/s) |
|--:|--:|--:|
| 1 | 39.9 | 25.1 |
| 2 | 57.0 | 35.1 |
| 4 | 109.9 | 36.4 |
| 8 | 221.9 | 36.1 |
| 16 | 431.3 | 37.1 |

Batch 1 lên 16 làm throughput tăng 1.48x nhưng latency mỗi batch tăng 10.8x. Throughput
plateau từ batch 4, nên batch lớn hơn chỉ đổi latency lấy rất ít throughput. Embedding
chỉ có forward/prefill, không KV cache hay decode loop: static batch hợp lý hơn continuous
batching của chat. Production phải dùng encoder như Qwen3-Embedding/BGE-M3 và autoscale
riêng hai regime.
