# Bonus B5 - Semantic cache offline

Demo offline dùng bag-of-words stub, threshold 0.80. Trong 8 request có 3 hit (38%),
tiết kiệm mô phỏng khoảng 750 ms decode. Sweep 0.70-0.95 vẫn luôn 3/8 vì similarity
chỉ ra 0 hoặc 1; đây là artifact của stub, không phải kết quả chất lượng.

Finding chính: semantic-cache hit bỏ qua cả prefill lẫn decode, khác prefix/KV cache.
Nhưng số hit offline không đủ để chọn threshold production. Cần embedding model thật,
đo false hit/miss, TTL và salt theo tenant; nếu không cache có thể trả sai hoặc rò timing.
