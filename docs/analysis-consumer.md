# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: pair01-camera-ai
- Product: A - Smart Campus
- Consumer service: Service A2 (Camera Stream)
- Provider service: Service A4 (AI Vision)
- Người viết: Nhóm 1
- Ngày: 13/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| `Frame / Image` (Gửi) | Gửi đường dẫn ảnh tĩnh để AI nhận diện dị thường (có người lạ, đồ vật bị bỏ quên...). | `frame_url`, `camera_id`, `timestamp` | `resolution`, `camera_type` |
| `DetectionResult` (Nhận) | Đọc kết quả phân tích để quyết định xem có cần sinh Alert gửi cho Core Business (A6) hay không. | `anomaly_detected` (boolean), `confidence_score` (float) | `bounding_boxes`, `object_labels` (Vì A2 không làm UI nên không cần vẽ khung) |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/api/v1/vision/analyze` | Gọi ngay khi thiết bị biên báo có chuyển động (`motion_detected` = true). | HTTP 200 OK kèm cục JSON chứa `anomaly_detected` và `confidence_score`. |
| GET | `/api/v1/vision/health` | Gọi định kỳ (ping) để xem AI có đang bị sập không trước khi đẩy data. | HTTP 200 OK với status "up". |

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Payload sai định dạng (VD: gửi URL lỗi). | Drop frame này, ghi log cảnh báo lỗi cấu trúc, không retry. |
| 401 | Thiếu hoặc sai API Key / Token khi gọi AI. | Bắn alert về team Dev/Ops kiểm tra lại cấu hình biến môi trường. |
| 422 | AI từ chối xử lý (Ví dụ: ảnh mờ quá, độ phân giải quá thấp). | Bỏ qua frame đó, log lại lý do ảnh không đạt chất lượng. |
| 503 | Server AI đang quá tải hoặc sập. | Tạm thời ngưng gửi request mới (Circuit Breaker) trong vài giây, drop các frame hiện tại để tránh nghẽn RAM của A2. |
| 504 | Gateway Timeout (AI phân tích quá lâu, vượt ngưỡng chờ). | Drop request, coi như frame đó không có gì dị thường để duy trì tốc độ stream. |

---

## 4. Giả định bổ sung

- Giả định 1: Các `frame_url` được gửi đi đều là public link (hoặc internal link trong cùng mạng ảo), thằng AI tự dùng lệnh wget/curl để tải ảnh về phân tích chứ A2 không nhét cục file ảnh Base64 vào body payload.
- Giả định 2: Thời gian chờ phản hồi (Latency) tối đa của AI cho mỗi ảnh là 2 giây. Vượt mức này A2 sẽ tự ngắt kết nối (Timeout).
- Giả định 3: AI không lưu trữ ảnh, xử lý xong và trả kết quả là xong để tránh phình to ổ cứng.

---

## 5. Câu hỏi cho Provider

1. Tốc độ nhận diện trung bình của mô hình (Inference Time) là bao nhiêu mili-giây/ảnh? 
2. Hệ thống AI có hỗ trợ nhận diện theo dạng mảng (Batch Processing) không, hay bắt buộc phải gửi từng ảnh một?
3. Có giới hạn số lượng request tối đa trong 1 giây (Rate Limit) cho mỗi IP/Camera không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi cấu trúc JSON trả về (ví dụ đổi tên biến). | Consumer không parse được data, hệ thống báo động bị tê liệt hoàn toàn. | Tuân thủ chặt chẽ hợp đồng OpenAPI. Có thay đổi phải nâng version đường dẫn (v1 -> v2). |
| Thời gian AI xử lý quá lâu làm nghẽn luồng. | Dữ liệu tích tụ làm cạn kiệt RAM của service A2. | Triển khai Timeout cứng (2s) và cơ chế Circuit Breaker bên phía A2. |
| URL ảnh gửi sang bị chặn quyền truy cập (403 Forbidden). | AI không thể phân tích, mọi kết quả đều fail. | Đảm bảo Object Storage (chỗ lưu ảnh) đã config cho phép dải mạng của AI vào đọc. |