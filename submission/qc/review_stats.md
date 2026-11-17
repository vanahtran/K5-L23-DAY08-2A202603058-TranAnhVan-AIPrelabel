# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 516; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 494 | 494/519 = 95.2% | 519 box máy |
| edited (sửa) | 17 | 17/519 = 3.3% | 519 box máy |
| deleted (xoá) | 8 | 8/519 = 1.5% | 519 box máy |
| added (vẽ thêm) | 5 | 5/516 = 1.0% | 516 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 5.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 261 | 6 | 1 | 5 |
| bicycle | 8 | 0 | 0 | 0 |
| motorbike | 128 | 2 | 7 | 0 |
| car | 54 | 0 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 0 |
| three_wheeler | 34 | 9 | 0 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 46 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 23 | 9 | 61.7% (KTC 95%: 49–73%) | 80.4% (KTC 95%: 67–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - missing: 3/3
  - loose: 2/3
  - extra: 3/3
  - lọt: img_018.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
