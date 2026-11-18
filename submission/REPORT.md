# Báo cáo Day 08 — 2A202603082 - Hoàng Quảng Quyền

**Bản nhãn chính thức:** [`batch_reviewed.json`](labels/batch_reviewed.json), gồm **24 ảnh Batch và 409 annotation**. Người kiểm tra và chốt phiên bản dữ liệu: **Hoàng Quảng Quyền**. Tên phiên bản: **Final Rework — Reviewed & Approved by Hoàng Quảng Quyền**. Công cụ AI có tham gia hỗ trợ Rework; số đo từ lượt đánh giá mù đầu tiên được giữ riêng và không thay bằng kết quả sau phản hồi.


## 1. Model và recipe (Part 2–3)

| Nhóm lớp | Model | Ngưỡng gộp | Cơ sở lựa chọn |
|---|---|---:|---|
| `person`, `bicycle`, `motorbike`, `bus`, `truck` | YOLO26-n | 0,30 | Detector cho các lớp thông dụng; giữ ngưỡng mặc định khi bằng chứng Gold theo lớp còn ít. |
| `car` | YOLO26-n | 0,35 | Ngưỡng được đề xuất từ `threshold_sweep` trên 4 ảnh Gold. |
| `three_wheeler` | YOLOE-s | 0,15 | Bổ sung khả năng phát hiện xe ba bánh, là lớp YOLO26 có thể nhầm với lớp xe khác. |

**Đối chiếu các lần chạy trong [`model_log.csv`](model_log.csv):**

- **r01 — YOLO26-n, conf 0,25:** 148 box/8 ảnh Pilot, trung bình 1,60 giây/ảnh. Có các dự đoán thô ngoài taxonomy như ô, đèn giao thông và ba lô.
- **r02 — YOLO26-n, conf 0,50:** 81 box/8 ảnh Pilot, trung bình 0,66 giây/ảnh. So với r01, số box giảm **67/148 = 45,3%**; chỉ từ số lượng box chưa thể kết luận recall giảm bao nhiêu.
- **r03 — YOLOE-s, conf 0,25:** 8 box `three_wheeler`/8 ảnh Pilot, trung bình 0,91 giây/ảnh. Chưa có precision/recall riêng của lượt Pilot.
- **r04 — YOLO26-n + SAM 2.1 Tiny:** thử trên `img_004.jpg` và `img_013.jpg`, tạo **8 polygon** (7 `person`, 1 `car`, không có `motorbike`), trung bình **9,29 giây/ảnh**. Ba dự đoán `bottle` ngoài taxonomy bị loại. Đây là thử nghiệm **SAM nhận box đề xuất**, khác với thao tác điểm click ở Part 5.
- **r05 — YOLO26-n, conf 0,05:** xuất **1.525 box** sau ánh xạ trên 36 ảnh, 0,36 giây/ảnh; dùng làm nguồn cho bước chọn ngưỡng và gộp.
- **r06 — YOLOE-s, conf 0,05:** xuất **125 box `three_wheeler`** trên 36 ảnh, 0,43 giây/ảnh; sau gộp còn 52 box của lớp này trong kết quả pre-label đã lưu.

Trường hợp **điểm tin cậy cao nhưng sai lớp**: YOLO26-n gán `truck` với độ tin cậy **0,6788** cho chiếc tuk-tuk ghế vàng trong `img_017.jpg`. Vì vậy không thể chỉ dùng confidence để chấp nhận box; cần kiểm tra loại phương tiện theo taxonomy. Nhật ký chưa cung cấp một ví dụ được xác minh cho trường hợp điểm thấp nhưng đúng, nên không khẳng định điều này đã quan sát được.

Lệnh gộp đã sử dụng:

```bash
python scripts/merge_prelabels.py \
  work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json \
  --iou 0.6 --conf 0.3 --thr "car=0.35,three_wheeler=0.15" \
  --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Chọn **SAM 2.1 Tiny (`sam2.1_t.pt`)** vì cấu hình Mac M1 RAM 8 GB cần model gọn; thí nghiệm cho thấy SAM có thể sinh mask từ box hoặc point prompt nhưng vẫn có lỗi dính người, thiếu vùng và không trả mask trong một số lần thử.

**Giới hạn của Part 3:** Ngưỡng được chọn và số đo máy được tính trên cùng 4 ảnh Gold, nên kết quả có thể lạc quan khi áp dụng sang ảnh chưa thấy. Lớp hiếm có mẫu Gold ít không đủ căn cứ để tối ưu ngưỡng riêng. Xem [`threshold_sweep.md`](qc/threshold_sweep.md).

## 2. AI error profile (Part 4)

Theo [`review_stats.md`](qc/review_stats.md), sau khi loại các đối tượng nằm quanh bẫy theo cách tính của script:

| Trạng thái | Số đo | Mẫu số |
|---|---:|---|
| Accepted | **223/335 = 66,6%** | 335 box máy |
| Edited | **54/335 = 16,1%** | 335 box máy |
| Deleted | **58/335 = 17,3%** | 335 box máy |
| Added | **120/397 = 30,2%** | 397 đối tượng sau duyệt được tính |

Accepted dùng ngưỡng **IoU ≥ 0,9**. Số `added` lớn cho thấy nhiều đối tượng cuối không ghép được với pre-label; **không được coi toàn bộ 120 là vật máy chắc chắn bỏ sót**, vì thuật toán còn có thể tính thay lớp/sai vị trí như deleted + added. Script ghi nhận 8 trường hợp đổi lớp, nên **120 − 8 = 112** chỉ là ước lượng theo quy tắc ghép của script, không phải ground truth độc lập.

Ở lớp `three_wheeler`, có 9 accepted, 10 edited, **24 deleted**, 4 added. Điều này phản ánh khó khăn khi nhận diện xe ba bánh và xe đẩy, cũng như các box bị loại khi review. Hai lớp có số `added` đáng chú ý là `person` (48) và `motorbike` (45); cần ưu tiên rà soát vật thể bị che và các cụm phương tiện đông.

Trên **4 ảnh Gold, 57 đối tượng**, máy có **TP = 34, FP = 9, FN = 23**: precision **34/43 = 79,1%**, recall **34/57 = 59,6%**. Vì hiệu chỉnh ngưỡng trên chính tập Gold này, đây là số đo in-sample có giới hạn. Bộ Gold không nằm trong nhãn sau duyệt; vì thế không tính residual/new error của người duyệt trên Gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

### Initial Blind Review

Lượt review đầu tiên trên CVAT: **2/12 = 16,7%** bẫy được phát hiện, **chưa đạt** ngưỡng 80%. Bản xuất ban đầu có 24 ảnh và **314 annotation**; được giữ riêng tại [`batch_reviewed_initial.json`](labels/batch_reviewed_initial.json). Đây là lượt dùng để đánh giá review mù trước khi xem đáp án/nhận phản hồi.

### Final Rework — Hoàng Quảng Quyền

Sau khi xem phản hồi và có AI hỗ trợ rà soát, bản nhãn cuối có 24 ảnh, **409 annotation**. Kết quả chấm lại: **10/12 = 83,3%**, **đạt ngưỡng 80% ở lượt Rework**, nhưng **không phải đánh giá mù độc lập**. Hai trường hợp vẫn lọt là `img_029.jpg` (**loose**) và `img_035.jpg` (**wrong_class**).

Kết quả đầu thấp cho thấy quy trình review ban đầu chưa phát hiện đủ lỗi được cài vào nhãn máy. Những nguyên nhân có thể góp phần gồm tập trung vào box hiện hữu, quét sót đối tượng và tin tưởng quá mức vào nhãn AI. **Đây là các giả thuyết cần người thực hiện xác nhận bằng trải nghiệm của chính mình, không phải nguyên nhân đã được đo trực tiếp.**

Trong lần review kế tiếp, cần ẩn box máy để quét vật thể trước, kiểm từng lớp/box sau đó và dành một lượt riêng tìm box thừa/trùng. Tỷ lệ bắt bẫy sau Rework là chỉ dấu cải thiện quy trình sửa lỗi, không thay thế QC độc lập.

## 4. Hiệu quả thời gian và ba quy tắc giảm thiên kiến

Theo [`time_log.csv`](time_log.csv):

| Bước | Ảnh | Tổng phút | Phút/ảnh |
|---|---:|---:|---:|
| Gold gán tay (`gold_manual`) | 4 | 30 | **7,5** |
| Batch review (`batch_review`) | 24 | 120 | **5,0** |

Lượt review Batch mất ít hơn **2,5 phút/ảnh**, tương đương khoảng **33,3% thời gian mỗi ảnh** so với Gold trong nhật ký. Tuy nhiên hai tập có số lượng và độ khó ảnh khác nhau; thời gian Batch ở đây không phản ánh đầy đủ công sức Rework, và không thể suy ra hiệu quả nhân quả chỉ từ hai số trung bình.

**Đánh giá dựa trên số đo:** AI Pre-label giúp tạo trước nhiều box để người gán nhãn tập trung vào sửa và bổ sung. Tuy vậy, tốc độ của hai tập ảnh không thể so sánh như một thí nghiệm đối chứng; số `added` lớn và lỗi QC còn tồn tại cho thấy không thể chỉ dựa vào thời gian ngắn hơn để khẳng định quy trình đã hiệu quả toàn diện.

Ba quy tắc giảm thiên kiến đề xuất áp dụng lần sau:

1. **Quét ảnh không có nhãn máy trước:** đánh dấu danh sách vật cần có, sau đó mới bật pre-label để so sánh, nhằm phát hiện missing.
2. **Kiểm riêng các box dễ nhầm hoặc chồng lấn:** rà `three_wheeler` so với `motorbike`/`car`/`truck`; kiểm người lái và box `person` trùng nhau; ghi nhận số box loại/sửa theo lớp.
3. **QC mẫu sau khi chốt, không chỉ dựa vào bẫy:** kiểm các ảnh ca khó và ảnh ngẫu nhiên, tính cả tỷ lệ ảnh lỗi với mẫu số, dừng Release nếu lỗi vượt ngưỡng hoặc kết quả chưa được xác nhận.

## 5. SAM Mask cho ba vật (Part 5)

Phần thử nghiệm r04 sử dụng **SAM 2.1 Tiny** nhận box của YOLO26-n, tạo 8 polygon trên hai ảnh (`img_004`, `img_013`) với thời gian trung bình 9,29 giây/ảnh. Riêng Part 5 sử dụng **point prompt** (click dương/âm, và tùy chọn box), lưu ba ảnh sau:

| Ảnh | Đối tượng | Prompt thực hiện | Quan sát trực quan | Bằng chứng |
|---|---|---|---|---|
| `img_013.jpg` | Xe ba bánh có người ngồi | **2 dương + 1 âm** (3 click) | SAM giữ phần lớn mái/thân/bánh nhưng mask còn dính người và xuất hiện các vùng thủng, rách; chưa đạt phân tách sạch. Lần thử tiếp với **6 dương + 7 âm** không trả mask nên không dùng làm ảnh kết quả. | [`mask_01_img013.jpg`](masks/mask_01_img013.jpg) |
| `img_014.jpg` | Người phụ nữ gánh hàng | **2 dương + 2 âm** (4 click) | Mask bám được phần lớn hình người và nón lá, đồng thời không phủ hai giỏ hàng. Ở vùng bàn tay, mép cơ thể và phần sát đòn gánh vẫn cần kiểm tra kỹ khi phóng lớn. | [`mask_02_img014.jpg`](masks/mask_02_img014.jpg) |
| `img_018.jpg` | Người đi bộ nhỏ giữa giao lộ | **2 dương + 1 âm + 1 box** (3 click và 1 box) | Script trả **520 pixel, 0,1% ảnh, 1 polygon**; vùng mask nhỏ nằm quanh người được chọn. Do đối tượng ở xa và ảnh xem nhỏ, chưa thể xác nhận sai số đường biên pixel-level. | [`mask_03_img018.jpg`](masks/mask_03_img018.jpg) |

**Đối chiếu guideline v1.0 mục 4.2:** mặt nạ xe không được gộp người lái/hành khách, không lấy nền, không lấp các khoảng trống lớn nhìn xuyên qua khung. Trường hợp `img_013` cho thấy SAM không tự bảo đảm các quy tắc này; click âm cần chọn đúng vùng và quá nhiều điểm không nhất thiết cải thiện kết quả. Với vật thể nhỏ (`img_018`), cần phóng lớn trước khi đánh giá biên.

## 6. Calibration (Part 6)

Đã ghi **3 quyết định** trong [`calibration_decisions.csv`](qc/calibration_decisions.csv) và phản ánh tại [`guideline_v1.1_calibrated.md`](guidelines/guideline_v1.1_calibrated.md):

| Case | Nguồn | Ảnh | Nội dung chốt / làm rõ |
|---|---|---|---|
| **CAL-01** | `cau_hoi_mo` | `img_013.jpg` | Phân tách người khỏi xe ba bánh trong mask; chỉ giữ bộ phận thuộc xe và loại khoảng trống lớn. |
| **CAL-02** | `loi_may` | `img_009.jpg` | Xe máy hai bánh chở hàng vẫn là `motorbike`, không tự suy thành `three_wheeler` vì có nhiều hàng. |
| **CAL-03** | `cau_hoi_mo` | `img_014.jpg` | Box `person` bao phần cơ thể nhìn thấy, không mở rộng ôm cả hai giỏ và đòn gánh. |

Nguyên nhân gồm **nhầm lớp của pre-label** và **ranh giới đối tượng chưa rõ khi người, xe và hàng hóa chồng nhau**. Guideline v1.1 chỉ làm rõ và thêm ví dụ, không thay đổi taxonomy hay ngưỡng chất lượng của v1.0. Các trường hợp mơ hồ khác vẫn cần `can_review` khi chưa có chứng cứ đủ mạnh.

## 7. QC và quyết định (Part 6)

Bộ nhãn chính thức đã được **xuất kỹ thuật** thành [`final_det.coco.json`](release/final_det.coco.json), **24 ảnh, 409 annotation**; Gold, Batch và bản dựng vượt qua kiểm tra định dạng. Việc có thư mục `release/` không có nghĩa đã được chấp thuận phát hành dữ liệu. Xem [`release_manifest.json`](release/release_manifest.json).

Mẫu QC gồm **8/24 ảnh**, với **3 ca khó + 5 ảnh chọn ngẫu nhiên** theo seed 42, tổng **151 đối tượng**. Theo phiếu [`qc_review.csv`](qc/qc_review.csv) được rà soát với AI hỗ trợ và **được người thực hiện đánh dấu xác nhận**:

| Chỉ số theo phiếu hiện tại | Kết quả | Mẫu số |
|---|---:|---|
| Lỗi theo đối tượng | **6/151 = 4,0%** | 151 đối tượng thuộc 8 ảnh mẫu |
| Ảnh có ít nhất một lỗi | **4/8 = 50,0%** | 8 ảnh mẫu |
| Bẫy được phát hiện ở Final Rework | **10/12 = 83,3%** | 12 bẫy |

Cả **6 dòng lỗi** được ghi nhận với loại `thua`: **`img_003`** (ID 21: box `person` trùng với người lái đã có ID 2), **`img_004`** (ID 33: box `car` chồng lên xe tuk-tuk đã có `three_wheeler` ID 26), **`img_008`** (ID 80, 81, 82: ba xe đẩy hàng đang mang nhãn `three_wheeler`, cần làm rõ cấu trúc bánh xe khi sửa), và **`img_017`** (ID 126: box người chỉ có phần rất nhỏ nhìn thấy). Hai lỗi đầu có bằng chứng trực quan rõ; bốn ca sau có độ khó cao do đặc điểm xe đẩy và che khuất. Các ghi chú gốc ở bốn ca này vẫn còn từ ngữ *nghi vấn*; khi Rework cần xác minh lại trước khi xóa/đổi nhãn. Phiếu QC không phải phép gán nhãn độc lập trên toàn bộ 24 ảnh.

Script phát hiện kiểu lỗi `thua` lặp lại và gợi ý **REWORK**. Theo [`qc_report_auto.md`](qc/qc_report_auto.md), với 4/8 ảnh được đánh dấu lỗi, **cận trên Wilson 95% khoảng 78%**; mẫu chỉ 8 ảnh không đủ để chứng minh ngưỡng ảnh lỗi tối đa **2%**.

**Quyết định QC: REWORK (18/11/2026).** Tỷ lệ ảnh có lỗi **4/8 = 50,0%** vượt ngưỡng tối đa 2%; đồng thời lỗi `thua` xuất hiện lặp lại **6/6 dòng lỗi**. Dù đã đạt ngưỡng bẫy ở lượt sửa có phản hồi, bộ dữ liệu chưa đủ căn cứ để Release. Cần xác minh lại các ca khó ID 80/81/82 và 126, sửa các lỗi được xác nhận trong bộ nhãn, dựng lại release, rồi kiểm mẫu QC mới. `check_submission.py` báo sạch lỗi chỉ cho biết hồ sơ nộp đủ/đúng định dạng.

## 8. Ghi chú cá nhân

**Công cụ hỗ trợ:** CVAT chạy qua Docker để gán nhãn; Python và các script của repo để chạy model, kiểm COCO và thống kê; ChatGPT và Codex hỗ trợ giải thích quy trình, gợi ý click SAM, rà soát annotation và chuẩn bị phiếu QC. Kết quả **Initial Blind Review 2/12** được giữ riêng; lượt **Final Rework 10/12** có phản hồi và AI hỗ trợ được báo cáo đúng bản chất, không giới thiệu là lần review mù.

**Môi trường:** MacBook Pro Apple Silicon M1, RAM 8 GB, môi trường `.venv`, mô hình YOLO26-n, YOLOE-s và SAM 2.1 Tiny (MPS theo khả năng thiết bị). Các lượt thử thực hiện được ghi trong `model_log.csv` (r01–r06); không có bằng chứng trong các log này về việc phải thay bằng cache. Nếu có dùng cache ở bước khác thì cần ghi bổ sung đúng thực tế.

**Khó khăn thực tế:** Tích hợp CVAT/COCO, gán đúng nhóm phương tiện ba bánh, xử lý các box trùng và vật bị che, cũng như prompt SAM chưa ổn định. Với tuk-tuk `img_013`, một lần thử nhiều điểm không trả mask; với ảnh `img_018`, mask rất nhỏ nên cần xem kỹ hơn.

**Bài học rút ra từ kết quả:** Pre-label giúp tạo nhãn ban đầu nhanh hơn trong lượt ghi thời gian này nhưng không bảo đảm chính xác. Ngay cả khi số bẫy phát hiện ở Rework đạt 10/12, mẫu QC vẫn ghi nhận lỗi thừa. Lần sau cần giữ riêng số đo trước phản hồi, quét ảnh không nhãn máy, kiểm đối tượng chồng lấn và thực hiện QC độc lập trước khi phát hành.

**Phạm vi và tính minh bạch:** Giữ nguyên taxonomy, các ngưỡng đánh giá và script chấm; phân biệt rõ Gold gán thủ công, review mù ban đầu, Rework có AI hỗ trợ và kiểm tra QC. Mọi thay đổi nhãn sau đợt QC này cần được ghi ở vòng sửa kế tiếp, không ghi đè lịch sử ban đầu.
