# Bonus B1 - Prebuilt vs source build

Host `Darwin-arm64` · CPU `Apple M4`
Vector extensions detected: NEON
llama.cpp `b10488` both sides · `threads=10` ·
**both pinned to `ngl=0`** so this isolates the compiler ·
metric `tg128`, 3 repetitions

| Binary | Built for | tg128 (tok/s) | Relative |
|:--|--:|--:|--:|
| prebuilt release | runtime CPU dispatch | 32.6 | 1.00x |
| your source build | this CPU (`-DGGML_NATIVE=ON`) | 35.1 | 1.08x |

On this machine, the source build is **1.08x faster**.

before: 32.6 tok/s (prebuilt release)
after:  35.1 tok/s (source build, -DGGML_NATIVE=ON)
speedup: 1.08x

Same source revision, same model, same backend, same `-ngl` -- the only difference
is what the compiler was allowed to assume about the CPU.


### Separately: what GPU offload is worth on the same binary

`tg128` on the source build at `-ngl 99` instead of `-ngl 0`:

| Source build | tg128 (tok/s) | vs its own CPU run |
|:--|--:|--:|
| `-ngl 0` (CPU) | 35.1 | 1.00x |
| `-ngl 99` (offloaded to MTL0: Apple M4 (12124 MiB, 12123 MiB free)) | 106.1 | 3.02x |

This number is **not** part of the B1 comparison above -- it is a different knob.
Reporting it separately is the point: a compiler flag and an accelerator are not
interchangeable explanations for a speedup.


## Giải thích

Native build dùng đúng ARM M4 (`-mcpu=native`, dotprod, i8mm, SME), nên CPU-only tăng
32.6 lên 35.1 tok/s, chỉ 1.08x. Gap nhỏ hợp lý vì decode còn bị giới hạn bởi bandwidth;
vector instruction tốt hơn không làm RAM nhanh hơn. Metal offload là knob khác nên mới
tăng 3.02x, không được trộn vào speedup compiler.
