# Reflection: Lakehouse Anti-Patterns

Trong số các anti-pattern của Lakehouse, hệ thống dữ liệu của chúng tôi dễ gặp rủi ro nhất với **"Small-File Problem" (Vấn đề tệp tin nhỏ)**. 

### Lý do:
1. **Đặc thù dữ liệu đầu vào:** Hệ thống ghi nhận log cuộc gọi LLM (LLM Observability) liên tục theo thời gian thực (real-time streaming ingestion) hoặc theo các lô nhỏ (micro-batches) từ các ứng dụng client.
2. **Hậu quả tích lũy:** Nếu ghi dữ liệu trực tiếp vào Delta Lake mà không có cơ chế gom cụm, sau một thời gian ngắn hệ thống sẽ tích lũy hàng chục ngàn file Parquet kích thước siêu nhỏ (vài KB đến vài MB). Điều này làm phình to metadata trong transaction log (`_delta_log`) và khiến hiệu năng truy vấn đọc (query read performance) bị suy giảm nghiêm trọng do công cụ phải quét quá nhiều tệp tin.

### Giải pháp khắc phục:
Chúng tôi áp dụng kỹ thuật tối ưu hóa trong bài học bằng cách chạy định kỳ lệnh `OPTIMIZE` kết hợp `Z-ORDER BY (model, user_id)` để nén (compact) các tệp nhỏ thành các tệp lớn hơn (~256KB - 8MB tùy quy mô) và gom cụm dữ liệu để hỗ trợ tính năng bỏ qua tệp (file skipping), giúp tối ưu hóa hiệu năng truy vấn cho các dashboard quan sát.
