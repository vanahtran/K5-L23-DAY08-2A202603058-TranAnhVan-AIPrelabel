# Báo cáo Day 08 — 2A202603058 - Tran Anh Van

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.30 (mặc định) | `run_01`, `run_02`: Phát hiện tốt xe và người phổ biến trên đường phố, ít box rác |
| three_wheeler | YOLOE-26s, prompt: three_wheeler | 0.15 | `run_03`: Bắt được xe tuk-tuk và xích lô châu Á mà YOLO26 COCO không nhận ra |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 base (`sam2.1_b.pt`)** vì model đạt cân bằng tối ưu giữa độ chính xác phân đoạn đường biên (boundary) và thời gian xử lý, chạy ổn định không cần quyền truy cập gated như SAM 3 và chất lượng mask vượt trội so với SAM 1 nhẹ.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`): Lỗi phân loại sai bản chất (semantic misclassification) của closed-set detector khi gặp phương tiện đặc thù ngoài danh mục COCO (tuk-tuk bị gán là `car` hoặc `truck`). Cách sửa duy nhất là bổ sung detector open-vocabulary như YOLOE hoặc Grounding DINO chuyên phát hiện `three_wheeler`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Ở `img_011.jpg`, YOLO26 gán box tuk-tuk là `car` với độ tin cậy rất cao (conf > 0.8) nhưng sai nhãn; ngược lại YOLOE phát hiện tuk-tuk ở conf chỉ 0.22 nhưng đúng lớp.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Bộ ảnh gold chỉ có 4 ảnh (mẫu nhỏ, in-sample), việc quét ngưỡng và đánh giá trên cùng tập dữ liệu khiến chỉ số recall máy đạt 80.4% là con số ước lượng thô, khi áp dụng vào lô ảnh lớn thực tế độ chính xác có thể giảm.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 494 | 95.2% | 519 box máy |
| edited | 17 | 3.3% | 519 box máy |
| deleted | 8 | 1.5% | 519 box máy |
| added | 5 | 1.0% | 516 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` (như ở `img_011.jpg` và `img_013.jpg`) do hình dáng khung xe phức tạp khiến box thường bị lỏng hoặc YOLO26 nhầm thành ô tô.
- So với gold: precision / recall của **máy**: Precision **61.7%** (37/60), Recall **80.4%** (37/46) (mẫu số: 46 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **11/12 = 91.7%** (đạt ngưỡng >= 80%); loại lọt: 01 bẫy `loose` ở `img_018.jpg`; vì sao: Ở `img_018.jpg` có bãi đỗ xe máy mật độ rất dày, mắt người reviewer dễ bị thiên kiến tự động hóa (automation bias) khi nhìn thấy box màu bao quanh xe là mặc định chấp nhận mà không phóng to kiểm tra độ sát viền.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt: Accepted cao mà bắt bẫy thấp là dấu hiệu người duyệt lười kiểm tra, chỉ bấm chấp nhận nhanh; ngược lại bắt bẫy cao chứng tỏ reviewer chủ động soi lỗi độc lập với nhãn máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **6.25 phút/ảnh** vs duyệt **3.12 phút/ảnh** (tiết kiệm 50% thời gian). Thiên lệch: Khi duyệt ta đã quen thuộc với guideline và cấu trúc đối tượng trên đường phố nên tốc độ nhanh hơn, nhưng tốc độ tăng chủ yếu nhờ không phải tự vẽ các box ô tô và người đi bộ rõ nét.
- Kết luận: **Có, rất đáng dùng**, đặc biệt cho các lớp chuẩn có sẵn như `car`, `motorbike`, `person`. Đối với xe 3 bánh, cần kết hợp YOLOE và reviewer phải tập trung kiểm tra kỹ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét toàn ảnh bằng mắt thường trước khi bật hiển thị lớp box gợi ý của AI.
  2. Phóng to 200% kiểm tra các cụm xe đỗ dày đặc để phát hiện box lỏng hoặc sót.
  3. Đo tỷ lệ bắt bẫy định kỳ (yêu cầu >= 80%) trước khi chốt release nhãn.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 base | 2 dương, 1 âm | Không gồm người lái | 1 click dương ban đầu bị lan sang người lái, cần thêm click âm để bóc tách | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 base | 1 dương | Không | Mask bám sát thân người đi bộ | `person_img004.jpg` |
| `img_011.jpg` | SAM 2.1 base | 1 dương | Không | Mask ôm sát thân xe và bánh ô tô con | `car_img011.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc không vẽ lỗ thủng nhỏ (< 5% diện tích) và loại bỏ bóng đổ giúp polygon sạch sẽ; tuy nhiên guideline cần làm rõ quy tắc mask kính xe khi nhìn xuyên thấu.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Offline cho phép kiểm soát toạ độ click và phiên bản trọng số chính xác (`sam2.1_b.pt`), tránh phụ thuộc vào serverless Nuclio của CVAT.

## 6. Calibration (Part 6)

- Số quyết định: **3 quyết định** (`calibration_decisions.csv`); quan trọng nhất: `case_01` (xử lý xung đột nhận diện giữa YOLO26 và YOLOE cho xe 3 bánh).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Quy định rõ cách xử lý xe 3 bánh, hàng chở cồng kềnh và ngưỡng nhìn thấy người (>= 20%).
- Nguyên nhân gốc: Do giới hạn nhận diện của mô hình COCO và sự mơ hồ ban đầu của guideline đối với các phương tiện địa phương.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: **0/114 = 0.0%** · theo **ảnh**: **0/5 = 0.0%** (ngưỡng Release ≤ 2%).
- Top lỗi: Không phát hiện lỗi nghiêm trọng trong mẫu QC ngẫu nhiên.
- **Giới hạn mẫu nhỏ**: Cận trên khoảng tin cậy 95% Wilson của tỷ lệ ảnh lỗi là **43%**, vì vậy mẫu 5 ảnh chỉ mang tính sàng lọc thô, không khẳng định chắc chắn 100% lô ảnh dưới 2% lỗi.
- Script kiểm được / không kiểm được: Script kiểm tra được cú pháp, kích thước box, và danh mục lớp; không kiểm tra được độ ôm sát thẩm mỹ của box so với thực tế.

**Quyết định: Release**, ngày 2026-11-17. Lý do: Tỷ lệ ảnh lỗi đạt 0.0% (<= 2%), tỷ lệ bắt bẫy đạt 91.7% (>= 80%), không có lỗi hệ thống, guideline v1.1 đã hoàn thiện hiệu chuẩn.
Nếu Rework / Hold, việc cần làm tiếp: Không áp dụng.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng AI để hỗ trợ kiểm tra quy trình kỹ thuật, giải thích logic tính toán và rà soát lỗi cú pháp; toàn bộ số liệu đo đạc được trích xuất trực tiếp từ các file kết quả thực tế.
- Gold có bị nhiễm không: **Không**, 4 ảnh gold được gán tay hoàn toàn độc lập từ trước trên CVAT mà không qua bất kỳ mô hình gợi ý nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Máy macOS Apple Silicon; sử dụng CVAT trực tuyến và môi trường Python 3.12 để thực thi các script kiểm tra.
- Dùng cache `data/cache/` thay vì tự chạy? Có kết hợp kết quả chạy sẵn trong cache cho bộ quét toàn bộ ảnh để tối ưu hóa thời gian và tài nguyên máy tính.
- Lỗi / bug script đã gặp: Không có bug nghiêm trọng.
- Một điều bạn làm khác lần sau: Sẽ dành thời gian mở rộng kiểm tra trên nhiều ảnh bẫy hơn để kiểm soát triệt để thiên kiến tự động hóa.
