# Event Contract sơ bộ — dùng cho dependency Queue async

> File này chỉ dùng cho các cặp Queue async ở Lab 02 để ghi nhận thỏa thuận ban đầu. Đặc tả chi tiết bằng AsyncAPI sẽ chuyển sang Lab 03.

## 1. Thông tin dependency

- Dependency số: 07 (pair-07-camera-analytics-async)
- Producer: Service A2 (Camera Stream)
- Consumer: Service A5 (Analytics)
- Cơ chế: Queue async
- Event/topic dự kiến: `camera.frame.processed`
- Người ghi: Nhóm 1
- Ngày: 15/05/2026

## 2. Mục đích nghiệp vụ

Sự kiện (Event) này sinh ra định kỳ (vd: mỗi 10 giây) hoặc mỗi khi Service A2 tiếp nhận thành công một lô frame ảnh từ Camera.
Consumer (Analytics - A5) sẽ hứng các event này để lấy dữ liệu vẽ biểu đồ thống kê (uptime của camera, tổng số frame đã quét, thời gian quét), phục vụ dashboard cho người quản trị. A2 bắn event xong là xong, không cần đợi A5 phản hồi.

## 3. Event name / topic

| Mục | Giá trị |
|---|---|
| Event name | `camera.frame.processed` |
| Topic/queue | `analytics.camera.logs` |
| Producer | `Service A2 (Camera Stream)` |
| Consumer | `Service A5 (Analytics)` |

## 4. Payload tối thiểu

```json
{
  "eventId": "123e4567-e89b-12d3-a456-426614174000",
  "eventType": "camera.frame.processed",
  "occurredAt": "2026-05-13T08:30:00Z",
  "correlationId": "987fcdeb-51a2-43d7-9012-426614174111",
  "source": "service-a2-camera-stream",
  "data": {
    "camera_id": "cam-gate-01",
    "frames_received": 150,
    "frames_dropped": 5,
    "timestamp_start": "2026-05-13T08:29:50Z",
    "timestamp_end": "2026-05-13T08:30:00Z"
  }
}
