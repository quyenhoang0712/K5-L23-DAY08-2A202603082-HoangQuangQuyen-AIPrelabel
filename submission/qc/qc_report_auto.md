# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 151 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/151 = 4.0% | 151 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/8 = 50.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 223/335 = 66.6% / 54/335 = 16.1% / 58/335 = 17.3% | 335 box máy |
| Máy: added | 120/397 = 30.2% | 397 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 6 | img_003.jpg, img_004.jpg, img_008.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 4 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **78%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 50.0% > 2%
- lỗi hệ thống: 'thua' chiếm 6/6 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
