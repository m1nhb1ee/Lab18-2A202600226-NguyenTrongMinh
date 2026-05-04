# Reflection - Lab 18 Lakehouse

**Sinh viên:** Nguyễn Trọng Minh - 2A202600226

## Anti-pattern nào nhóm mình dễ mắc phải nhất?

Anti-pattern mình lo nhất là **"Too Many Small Files"** (slide 5).

Hình dung trong một pipeline theo dõi LLM thật: mỗi lần có request gọi API là một lần append vào Delta table. Nếu không có bước compaction, table sẽ tích lũy hàng nghìn file nhỏ lẻ mà không ai để ý - vì nó không báo lỗi, pipeline vẫn chạy bình thường. Đúng cái NB2 đã mô phỏng: 200 lần append tạo ra 200 file, query chậm hơn tới 9× so với sau khi OPTIMIZE. Trong thực tế, team thường ưu tiên tốc độ ingest và hay nói "để sau sẽ OPTIMIZE" - nhưng cái "sau" đó hầu như không bao giờ đến khi deadline cứ dồn liên tục.

## Lab 18 giải quyết vấn đề này như thế nào?

NB2 cho thấy rõ tác động: từ 200 file xuống còn 55 file sau `OPTIMIZE + ZORDER`, tốc độ query tăng **9.4×**. Cách khắc phục không phức tạp - chỉ cần lên lịch OPTIMIZE chạy định kỳ vào giờ thấp điểm, ví dụ 3 giờ sáng mỗi ngày. NB4 cũng hỗ trợ góc nhìn này: thay vì append liên tục lên Gold, ta batch aggregate theo ngày, giảm hẳn số lần ghi từ đầu.

## Điều mình rút ra

Small-file problem nguy hiểm ở chỗ nó âm thầm - không crash, không alert, chỉ từ từ làm dashboard chậm lại cho đến khi timeout lúc 3 giờ sáng mới té ngửa. Xây `OPTIMIZE` vào pipeline ngay từ đầu, không phải chờ lúc "có vấn đề mới sửa", là thói quen vận hành quan trọng nhất mình học được từ lab này.
