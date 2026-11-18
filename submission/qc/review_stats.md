# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 335; đối tượng sau duyệt: 397; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 223 | 223/335 = 66.6% | 335 box máy |
| edited (sửa) | 54 | 54/335 = 16.1% | 335 box máy |
| deleted (xoá) | 58 | 58/335 = 17.3% | 335 box máy |
| added (vẽ thêm) | 120 | 120/397 = 30.2% | 397 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 112.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 128 | 23 | 17 | 48 |
| bicycle | 2 | 0 | 2 | 6 |
| motorbike | 55 | 20 | 6 | 45 |
| car | 28 | 1 | 6 | 11 |
| bus | 1 | 0 | 1 | 4 |
| truck | 0 | 0 | 2 | 2 |
| three_wheeler | 9 | 10 | 24 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 57 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 34 | 9 | 23 | 79.1% (KTC 95%: 65–89%) | 59.6% (KTC 95%: 47–71%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - wrong_class: 2/3
  - missing: 3/3
  - loose: 2/3
  - lọt: img_029.jpg (loose), img_035.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
