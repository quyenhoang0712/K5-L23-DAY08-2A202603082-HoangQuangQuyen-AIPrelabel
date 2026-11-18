# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603082 - Hoàng Quảng Quyền

**Người thực hiện:** Hoàng Quảng Quyền  
**Phiên bản:** v1.1  
**Phạm vi:** Bổ sung và làm rõ quy tắc gán nhãn dựa trên các tình huống thực tế trong Day 8. Giữ nguyên taxonomy và các ngưỡng QC của v1.0.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Mask xe ba bánh có bao gồm người ngồi trong xe và khoảng trống nhìn xuyên qua khung xe không? | Người lái và hành khách là các đối tượng `person` riêng. Mask `three_wheeler` giữ mái, khung, thân, bánh và kính gắn liền xe. Loại bỏ người và các khoảng trống lớn xuyên qua khung. | [img_013.jpg](../../data/images/img_013.jpg) | CAL-01 |
| 2 | Người gánh hàng có được tính chung đòn gánh và giỏ hàng vào bounding box `person` không? | Bounding box `person` ôm sát phần cơ thể nhìn thấy, không mở rộng để bao cả đòn gánh hoặc hai giỏ hàng. | [img_014.jpg](../../data/images/img_014.jpg) | CAL-03 |

**Những câu hỏi còn mở:** Các trường hợp như xe đạp điện, vật thể bị che khuất quá nhiều hoặc những phương tiện khó xác định số bánh chưa được chốt thành quy tắc mới nếu thiếu bằng chứng hình ảnh rõ ràng. Khi gặp, đánh dấu `can_review` để kiểm tra riêng.

## 2. Ca khó mới (từ lỗi máy và quá trình review)

| Ảnh | Tình huống | Máy làm gì | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| [img_009.jpg](../../data/images/img_009.jpg) | Xe máy hai bánh chở nhiều hàng hóa, dễ bị nhầm với xe ba gác. | Pre-label gán nhầm `three_wheeler` cho xe máy chở hàng. Chưa xác định riêng model nào tạo box sai. | Gán `motorbike` nếu phương tiện thực sự có cấu trúc hai bánh; chỉ gán `three_wheeler` nếu là xe ba bánh. | **Mục 2 — Định nghĩa lớp:** Không phân loại phương tiện dựa vào lượng hàng chở. Ưu tiên cấu trúc và số bánh thực tế. **CAL-02** |
| `img_013.jpg` | Xe ba bánh có người lái, hành khách và nhiều khoảng trống giữa các thanh khung. | SAM 2.1 Tiny tạo mask bao được nhiều phần của xe nhưng còn lẫn vùng người và có các vùng phân đoạn thiếu chính xác. | Tách người khỏi xe. Giữ các bộ phận xe, gồm kính gắn liền xe, và loại bỏ khoảng trống lớn nhìn xuyên qua khung. | **Mục 4.2 — Mask/polygon:** Kiểm tra riêng người, khoảng trống xuyên khung và đường biên sau khi SAM sinh mask. **CAL-01** |
| `img_014.jpg` | Người gánh hàng đứng sát hai giỏ và đòn gánh. | Đây là trường hợp cần làm rõ phạm vi đối tượng khi tạo mask và bounding box, không kết luận là lỗi máy nếu chưa có đối chiếu. | Nhãn `person` chỉ bao phần cơ thể người; dụng cụ gánh hàng và giỏ là vật riêng, không gộp vào box người. | **Mục 4.1 — Bounding box:** Với người mang hàng, xác định ranh giới cơ thể trước, không mở rộng box để bao toàn bộ hàng hóa. **CAL-03** |

### Quy tắc áp dụng khi review

1. **Phân biệt phương tiện:** Xác định loại xe từ cấu trúc thực tế, không dựa riêng vào hình dạng hàng hóa hoặc dự đoán ban đầu của AI.
2. **Tách đối tượng chồng lấn:** Người và phương tiện phải được đánh giá như hai đối tượng khác nhau, kể cả khi người đang ngồi trên xe.
3. **Kiểm tra mask SAM:** Sau khi click positive/negative, đối chiếu lại biên mask, vùng người bị dính và những khoảng trống lớn.
4. **Trường hợp chưa chắc chắn:** Không tự đổi lớp theo suy đoán; dùng `can_review` và ghi lại ảnh để kiểm tra.

Các quy tắc trên là phần làm rõ cách áp dụng v1.0 cho tình huống cụ thể, không thay đổi những quy định gốc về kích thước box tối thiểu, IoU hoặc ngưỡng QC.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Label contract gốc do giảng viên cung cấp. | — |
| v1.1 | Làm rõ việc tách người và xe trong segmentation, xử lý khoảng trống xuyên khung. | `img_013.jpg` — CAL-01 |
| v1.1 | Bổ sung ví dụ phân biệt xe máy chở hàng với xe ba bánh dựa vào cấu trúc phương tiện. | `img_009.jpg` — CAL-02 |
| v1.1 | Làm rõ phạm vi bounding box của người gánh hàng, loại bỏ giỏ và đòn gánh khỏi nhãn `person`. | `img_014.jpg` — CAL-03 |

**Người xác nhận quy tắc:** Hoàng Quảng Quyền — MSHV 2A202603082.

**Nguồn bằng chứng:** `submission/qc/calibration_decisions.csv`, ảnh gốc tương ứng và các kết quả quan sát trong Part 4–5.
