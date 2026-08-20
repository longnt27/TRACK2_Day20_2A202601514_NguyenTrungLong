Reflection Day 20.

Họ tên Nguyễn Trung Long.

Cohort 4.

Ngày submit 2026 08 20.

1. Hardware và runtime.

Mình chạy bài trên Mac mini với macOS 26.5.2. Máy dùng Apple M4, 10 core vật lý, 10 core logic, 16 GB RAM và Apple Metal. Runtime là llama.cpp b10488. Model là Qwen3.5 0.8B. Hai bản quant là Q4 K M và UD Q2 K XL.

Setup nhìn chung khá thẳng. Lần probe đầu trong sandbox đọc RAM thành 0 GB vì sysctl bị chặn. Chạy setup ngoài sandbox thì nhận đúng Apple M4 và 16 GB RAM. Mình chọn Qwen nhỏ để load test hoàn thành được nhiều request hơn.

2. Đo lường.

Q4 K M có dung lượng 0.50 GB và load trong 1067 ms. TTFT P50 và P95 là 47 ms và 57 ms. TPOT P50 và P95 là 8.8 ms và 9.5 ms. E2E P50, P95 và P99 là 598 ms, 658 ms và 658 ms. Decode đạt 113.8 token mỗi giây.

UD Q2 K XL có dung lượng 0.39 GB và load trong 1029 ms. TTFT P50 và P95 là 45 ms và 46 ms. TPOT P50 và P95 là 8.2 ms và 8.3 ms. E2E P50, P95 và P99 là 560 ms, 568 ms và 568 ms. Decode đạt 122.3 token mỗi giây.

Q2 nhỏ hơn 22 phần trăm và decode nhanh hơn 1.07 lần. Nhưng khi thử 5 câu, Q2 chỉ đúng hoàn toàn 1 câu, Q4 đúng 2 câu. Q2 cũng hay trả lời dài và sai format. Máy mình đủ RAM nên mình chọn Q4. Tăng 7 phần trăm tốc độ không đáng để đổi lấy chất lượng thấp hơn.

3. Serving khi có tải.

Ở 10 users có 137 request, RPS là 2.34. P50, P95 và P99 là 3200 ms, 5000 ms và 5300 ms. Effective concurrency là 7.7. Không có request lỗi.

Ở 50 users có 137 request, RPS là 2.32. P50, P95 và P99 là 19000 ms, 23000 ms và 24000 ms. Effective concurrency là 40.3. Không có request lỗi.

Load tăng 5 lần nhưng throughput còn giảm nhẹ xuống 0.99 lần. P95 tăng 4.60 lần. Peak busy slot là 3.96 trên 4 và có lúc 46 request bị deferred. Server đã bão hòa trước 50 users. Latency tăng thêm chủ yếu là thời gian nằm trong queue. Nếu cần giữ P95 dưới 6 giây, mình sẽ thêm replica trước. Tăng slot trên cùng Metal có thể làm mỗi request chậm hơn.

4. Integration.

N16 Cloud và IaC là stub. N17 data pipeline là stub. N18 lakehouse là stub. N19 retrieval và features là stub, phần retrieval đang dùng keyword overlap. N20 llama server là phần real.

Mean latency của embed là 0.0 ms. Retrieve là 0.0 ms. LLM là 1327.8 ms và chiếm gần như toàn bộ thời gian. Kết quả này đúng với dự đoán vì corpus nhỏ và retrieval rất nhẹ. Nếu cần giảm một nửa latency, mình sẽ giảm số output token hoặc cache câu trả lời. Tối ưu retrieval gần 0 ms sẽ không tạo khác biệt đáng kể.

5. Thay đổi quan trọng nhất.

Mình tăng số thread từ 1 lên 10. Tốc độ tăng từ 111.0 lên 114.2 token mỗi giây, tương đương 1.03 lần.

Mức tăng nhỏ vì phần nặng đã chạy trên Metal với toàn bộ layer được offload. CPU chủ yếu chuẩn bị graph và scheduling. Từ 1 lên 10 thread vẫn giảm được phần việc tuần tự. Khi tăng lên 20 thread, tốc độ giảm còn 112.1 token mỗi giây. Máy chỉ có 10 core vật lý nên thread thêm bắt đầu tranh lịch và cache. Thêm CPU thread cũng không tạo thêm GPU hay memory bandwidth.

6. Bonus.

Mình đã build llama.cpp native, chạy context sweep, làm challenge chất lượng quant và đo embedding serving.

Native CPU đạt 35.1 token mỗi giây, prebuilt CPU đạt 32.6 token mỗi giây. Speedup là 1.08 lần. Native build dùng instruction phù hợp hơn với Apple M4, nhưng khoảng cách nhỏ vì decode vẫn bị giới hạn nhiều bởi memory bandwidth.

Context 8192 token tốn 4.60 giây prefill, cao hơn tuyến tính 1.16 lần. Điều này cho thấy không nên nhét đầy context chỉ vì model còn chỗ.

Embedding throughput tăng từ 25.1 lên 37.1 text mỗi giây khi batch tăng từ 1 lên 16. Throughput gần như phẳng từ batch 4. Batch lớn hơn làm latency tăng nhưng không đem lại nhiều throughput.

7. Điều làm mình bất ngờ nhất.

Load tăng 5 lần nhưng RPS không tăng, còn P95 tăng hơn 4 lần. Queue phình nhanh hơn mình nghĩ.
