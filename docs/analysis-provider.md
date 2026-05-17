# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Pair 02 - Core Business ↔ AI Vision
- Product: A
- Provider service: AI Vision
- Consumer service: Core Business
- Người viết: Nhóm 10
- Ngày: 2026-05-13

---

## 0. Service boundary

- Upstream: Camera Stream Service cung cấp ảnh/frame để AI Vision phân tích.
- Downstream: Core Business Service nhận kết quả phát hiện để ra quyết định nghiệp vụ.
- Downstream phụ: Analytics Service nhận dữ liệu phát hiện để thống kê và báo cáo.

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| FaceMatchRequest | Yêu cầu phân tích face-match từ Core Business | inputMode, traceId, imageRef hoặc faceEmbedding | personHint, threshold |
| VisionDetectionResult | Kết quả phân tích của AI Vision | detectionId, analysisType, traceId, confidence, modelVersion, status, createdAt | imageRef, personHint, matchedPersonId, similarityScore, resolvedAt, reviewedAt, message |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| GET | `/health` | Kiểm tra trạng thái service | Trước khi gọi phân tích hoặc khi monitor |
| POST | `/vision/face-match` | Nhận ảnh/frame từ Camera Stream hoặc Core Business để phân tích | Khi cần chạy AI hoặc mô phỏng kết quả AI |
| GET | `/vision/detections/{detectionId}` | Lấy chi tiết một detection | Khi Core Business hoặc Analytics cần audit/tra cứu |
| GET | `/vision/results/recent` | Lấy danh sách detection gần đây | Khi Core Business hoặc Analytics cần đối chiếu/báo cáo |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng | `Problem` |
| 401 | Thiếu Bearer token | `Problem` |
| 403 | Token hợp lệ nhưng không có quyền | `Problem` |
| 404 | Resource không tồn tại | `Problem` |
| 409 | Xung đột nghiệp vụ | `Problem` |
| 422 | Dữ liệu đúng JSON nhưng vi phạm nghiệp vụ | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- AI Vision có thể xử lý đồng bộ và trả về `201` ngay sau khi tạo detection.
- `traceId` là bắt buộc để Core Business đối chiếu audit và chống xử lý lặp.
- `imageRef` và `faceEmbedding` được hỗ trợ song song thông qua `oneOf`.

---

## 5. Câu hỏi cho Consumer

1. Core Business muốn gửi `imageRef` hay `faceEmbedding` là luồng chính?
2. Khi `confidence` thấp hơn ngưỡng, Consumer muốn nhận trạng thái `LOW_CONFIDENCE` hay `NOT_MATCHED`?
3. Core Business có cần `matchedPersonId` nullable để phân biệt chưa match với lỗi xử lý không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Core gửi sai kiểu đầu vào | Consumer nhận 400/422 | Chốt `oneOf` và ví dụ request trong `openapi.yaml` |
| Trùng request khi retry | Detection bị tạo lặp | Bắt buộc `traceId` và xử lý idempotency ở mức contract |
| Không thống nhất ngưỡng confidence | Consumer hiểu sai kết quả | Ghi rõ `threshold` là gợi ý và `confidence` là kết quả trả về |
