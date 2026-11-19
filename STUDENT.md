# Thông tin học viên

- **Họ và tên:** Hoàng Quảng Quyền
- **MSHV:** 2A202603082
- **Lớp:** L2
- **Tài khoản GitHub:** `quyenhoang0712`
- **Repo bài nộp:** [K5-L23-DAY08-2A202603082-HoangQuangQuyen-AIPrelabel](https://github.com/quyenhoang0712/K5-L23-DAY08-2A202603082-HoangQuangQuyen-AIPrelabel)
- **Commit chứa bộ dữ liệu và báo cáo Day 8:** [`499946f5`](https://github.com/quyenhoang0712/K5-L23-DAY08-2A202603082-HoangQuangQuyen-AIPrelabel/commit/499946f5c154a82c451009b1d59479e939694e33)
- **Môi trường:** macOS 26.6.2 (Apple Silicon M1, RAM 8 GB), Python 3.12.16, PyTorch MPS, CVAT 2.82.1 qua Docker.

## Kết quả tóm tắt

### 1. Gold Manual Annotation

- Hoàn thành gán nhãn thủ công **4 ảnh Gold với 57 đối tượng**.
- Thời gian thực hiện: **30 phút**, trung bình **7,5 phút/ảnh**.
- Dữ liệu: [gold_manual.json](submission/labels/gold_manual.json).

### 2. Model và Pre-label

- Sử dụng **YOLO26-n**, **YOLOE-s** và **SAM 2.1 Tiny** (`sam2.1_t.pt`).
- Thực hiện 6 lượt chạy model (r01–r06), thử nghiệm các ngưỡng confidence và kết hợp model cho từng nhóm lớp.
- Kết quả đánh giá pre-label trên 4 ảnh Gold:
  - **Precision:** 34/43 = 79,1%.
  - **Recall:** 34/57 = 59,6%.
- Các trường hợp xe ba bánh bị nhận nhầm thành `car`, `truck` hoặc `motorbike` cần kiểm tra trực quan.

[Nhật ký model](submission/model_log.csv) · [Quét ngưỡng](submission/qc/threshold_sweep.md)

### 3. Batch Review và AI Error Profile

- Hoàn thành review **24 ảnh Batch**.
- **Initial Blind Review:** Phát hiện 2/12 bẫy (16,7%), chưa đạt ngưỡng 80%. Bản đầu có 314 annotation.
- **Final Rework — Reviewed & Approved by Hoàng Quảng Quyền:** Phát hiện 10/12 bẫy (83,3%), đạt ngưỡng 80% sau khi có phản hồi và AI hỗ trợ. Bản cuối có 409 annotation.
- Hai bẫy còn lọt: `img_029.jpg` (`loose`) và `img_035.jpg` (`wrong_class`).

**AI Error Profile:**

| Trạng thái | Số đo | Tỷ lệ |
|---|---:|---:|
| Accepted | 223/335 | 66,6% |
| Edited | 54/335 | 16,1% |
| Deleted | 58/335 | 17,3% |
| Added | 120/397 | 30,2% |

Các phép đếm trên sử dụng mẫu số và bộ lọc của script đánh giá. `Added` không đồng nghĩa tất cả đều là vật thể máy bỏ sót, vì có trường hợp đổi lớp hoặc ghép box không thành công.

[Bản review đầu](submission/labels/batch_reviewed_initial.json) · [Bản nhãn cuối](submission/labels/batch_reviewed.json) · [Review Stats](submission/qc/review_stats.md)

### 4. Hiệu quả thời gian

- **Gold Manual:** 4 ảnh trong 30 phút, trung bình 7,5 phút/ảnh.
- **Batch Review:** 24 ảnh trong 120 phút, trung bình 5 phút/ảnh.
- Thời gian review thấp hơn 2,5 phút/ảnh so với Gold, nhưng hai tập ảnh khác nhau về nội dung và độ khó. Vì vậy chưa thể coi đây là phép đo chính xác mức thời gian AI tiết kiệm được.

[Nhật ký thời gian](submission/time_log.csv)

### 5. SAM Segmentation

Đã thực hiện và lưu **3 ảnh mask** sử dụng SAM 2.1 Tiny:

1. `img_013.jpg`: Xe ba bánh có người ngồi; mask còn dính một phần người và thiếu chính xác ở vùng khung xe.
2. `img_014.jpg`: Người phụ nữ gánh hàng; mask tách tương đối tốt cơ thể khỏi hai giỏ hàng.
3. `img_018.jpg`: Người đi bộ nhỏ; tạo được một polygon với 520 pixel mask, cần kiểm tra đường biên kỹ hơn khi phóng lớn.

[Bộ ảnh SAM](submission/masks)

### 6. Calibration và Guideline

Hoàn thành **3 quyết định calibration**:

- **CAL-01:** Tách người lái/hành khách khỏi mask `three_wheeler` và loại các khoảng trống lớn xuyên khung.
- **CAL-02:** Phân biệt `motorbike` chở hàng và `three_wheeler` dựa trên cấu trúc phương tiện.
- **CAL-03:** Bounding box `person` không bao gồm giỏ hàng và đòn gánh.

Đã cập nhật quy tắc tại **Guideline v1.1**, giữ nguyên taxonomy và các ngưỡng đánh giá gốc.

[Calibration Decisions](submission/qc/calibration_decisions.csv) · [Guideline v1.1](submission/guidelines/guideline_v1.1_calibrated.md)

### 7. Quality Control (QC)

Đã thực hiện QC trên **8/24 ảnh Batch**, gồm 3 ảnh ca khó và 5 ảnh chọn ngẫu nhiên, với tổng cộng 151 đối tượng.

| Chỉ số | Kết quả |
|---|---:|
| Lỗi ghi nhận theo đối tượng | 6/151 = 4,0% |
| Ảnh có lỗi theo phiếu QC | 4/8 = 50,0% |
| Ngưỡng lỗi theo ảnh | Tối đa 2% |
| Bắt bẫy ở Final Rework | 10/12 = 83,3% |

Các lỗi được ghi nhận tại `img_003.jpg`, `img_004.jpg`, `img_008.jpg` và `img_017.jpg`, chủ yếu liên quan đến bounding box thừa, trùng lặp hoặc nhãn không phù hợp. Phiếu QC có AI hỗ trợ; một số trường hợp khó vẫn cần đối chiếu lại khi sửa dữ liệu.

**Quyết định QC: REWORK — 18/11/2026.**

Lý do: Tỷ lệ ảnh có lỗi 50,0% vượt ngưỡng 2%. Kết quả bắt bẫy sau Rework đạt ngưỡng nhưng chưa đủ chứng minh dữ liệu đáp ứng yêu cầu Release.

Bộ dữ liệu đã được dựng thành `release/final_det.coco.json`, nhưng đây là bản xuất kỹ thuật phục vụ bài nộp, **không phải quyết định chấp thuận Release**.

[QC Report](submission/qc/qc_report_auto.md) · [Release Manifest](submission/release/release_manifest.json)

### 8. Công cụ AI và bài học cá nhân

Tôi sử dụng ChatGPT và Codex để hỗ trợ tìm hiểu yêu cầu lab, xử lý lỗi kỹ thuật, gợi ý prompt SAM, phân tích kết quả model, hỗ trợ Rework và rà soát QC.

Kết quả review mù ban đầu được lưu riêng để bảo đảm tính minh bạch. Các thay đổi có AI hỗ trợ được ghi nhận ở phiên bản Final Rework.

Qua bài thực hành, tôi nhận thấy AI Pre-label giúp tạo nhanh nhãn ban đầu, nhưng không thay thế được kiểm tra thủ công. Đặc biệt cần rà soát bounding box chồng lấn, nhãn phương tiện dễ nhầm và vật thể bị che khuất.

**Kết luận:** Đã hoàn thành quy trình thực hành AI-Assisted Labeling, từ Gold Manual → Model Pre-label → Human Review → SAM → Calibration → QC. Bộ dữ liệu đã được xuất và báo cáo đầy đủ, với quyết định chất lượng hiện tại là **REWORK**.

[Báo cáo chi tiết Day 08](submission/REPORT.md)