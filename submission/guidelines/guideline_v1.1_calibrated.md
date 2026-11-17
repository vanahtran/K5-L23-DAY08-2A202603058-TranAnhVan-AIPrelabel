# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603058 - Tran Anh Van

> Bản bổ sung v1.1 cập nhật quy tắc sau khi hiệu chuẩn từ các lỗi của mô hình AI, bẫy lọt và các ca khó trong quá trình gán nhãn thực tế.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người ngồi trong xe ba bánh có mui che | Gán box `person` riêng nếu diện tích nhìn thấy >= 20% cơ thể; nếu bị che > 80% thì bỏ qua | `img_013.jpg` | case_02 |
| 2 | Xe ba gác chở hàng cồng kềnh nhô ra ngoài | Box xe chỉ ôm thân xe và khung gắn cố định; hàng rời nhô quá 20% chiều dài xe không tính vào box xe | `img_002.jpg` | case_01 |
| 3 | Xe đạp điện / xe máy điện cỡ nhỏ | Phân loại dựa vào bàn đạp: có bàn đạp trợ lực tính là `bicycle`, không có bàn đạp tính là `motorbike` | `img_006.jpg` | case_04 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_011.jpg` | Xe tuk-tuk 3 bánh chạy trên đường phố | YOLO26 nhận nhầm thành `car`/`truck` do tập COCO không có nhãn 3 bánh | Sửa thành `three_wheeler`, bổ sung detector YOLOE | Mục 2: Mọi xe 3 bánh gắn liền thùng/mui quy về `three_wheeler` |
| `img_018.jpg` | Bãi đỗ xe máy dày đặc có box bẫy lỏng lẻo | Bẫy loose trên xe máy bị lọt do thiên kiến tự động hóa | Siết chặt quy tắc review: quét toàn cảnh bằng mắt trước khi bật nhãn gợi ý | Mục 4.1: Kiểm tra kỹ độ khít của box tại các bãi đỗ xe dày đặc |
| `img_004.jpg` | Người điều khiển xe xích lô và khách ngồi | SAM tạo mask lan từ thân xe sang người lái | Tách 2 polygon/box độc lập cho người và phương tiện | Mục 4.2: Mask xe không được gộp người lái |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Khung tiêu chuẩn ban đầu |
| v1.1 | Bổ sung quy tắc phân định xe 3 bánh, người bị che khuất và kiểm soát bãi đỗ xe dày đặc | Hiệu chuẩn từ thực nghiệm review (case_01, case_02, case_03) |
