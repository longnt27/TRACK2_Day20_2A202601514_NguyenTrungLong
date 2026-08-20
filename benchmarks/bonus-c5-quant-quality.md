# Bonus C5 - Model nhỏ nhất vẫn hữu ích

Qwen3.5 0.8B, temperature 0, cùng 5 prompt ngắn, chấm strict theo đúng nội dung và format.

| Bài test | Q4_K_M | UD-Q2_K_XL |
|:--|:--:|:--:|
| 17 x 23 = 391 | đúng | đúng nhưng lặp sai tiếp |
| đảo `abcdef` thành `fedcba` | sai | sai |
| thủ đô Australia = Canberra | sai | đúng nhưng thêm fact sai |
| JSON `{name: An, age: 20}` đúng type | đúng | sai type tuổi |
| `sum([2,4,6])` = 12 | sai | sai |
| Tổng strict | 2/5 | 1/5 |

Q2 tiết kiệm 22% dung lượng và nhanh hơn 1.07x, nhưng hay bỏ constraint, dài dòng và
hallucinate. Q4 cũng yếu vì model chỉ 0.8B, nhưng vẫn là mức thấp nhất mình chấp nhận
cho demo có structured output. Production cần model lớn hơn và eval sát use case.
