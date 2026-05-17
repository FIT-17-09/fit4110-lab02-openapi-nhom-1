# Chính sách Versioning

Mọi thay đổi trên Hợp đồng API đều phải tuân thủ quy tắc **Semantic Versioning (SemVer)**:

- **Major (v2.x.x):** Cập nhật khi có "breaking change" làm hỏng tính tương thích ngược. Ví dụ: Xóa một trường bắt buộc, thay đổi cấu trúc URL gốc, đổi kiểu dữ liệu bắt buộc (từ chuỗi sang số). Đường dẫn URL sẽ phải thay đổi (VD: từ `/api/v1/...` sang `/api/v2/...`).
- **Minor (v1.x.x):** Cập nhật khi bổ sung thêm tính năng mới mà không làm hỏng tính năng cũ. Ví dụ: Thêm một trường tùy chọn mới vào Request/Response, thêm một Endpoint mới hoàn toàn.
- **Patch (v1.0.x):** Cập nhật khi cần sửa lỗi nhẹ không làm thay đổi cấu trúc logic. Ví dụ: Sửa lỗi chính tả trong description của Swagger, bổ sung thêm Example minh họa.
