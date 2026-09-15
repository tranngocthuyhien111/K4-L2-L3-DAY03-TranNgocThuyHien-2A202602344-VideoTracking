
Họ tên / nhóm: `Trần Ngọc Thúy Hiền`

Ngày: `15/09/2026`

## 1. Quá trình gán nhãn

| **Mục**                      | **Giá trị**        |
| ----------------------------------- | -------------------------- |
| Công cụ                           | CVAT / khác:`DarkLabel` |
| Thời gian gán`clip_02`(warm-up) | `15`phút                |
| Thời gian gán`clip_01`          | `45`phút                |
| Số track đã vẽ trong`clip_01` | `8`                      |
| Số keyframe trung bình mỗi track | `12`                     |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe bị che khuất (occlusion) bởi vật thể tĩnh hoặc xe khác`: Giữ nguyên track_id, bật thuộc tính Occluded, khoanh bbox ôm sát phần nhìn thấy và thực hiện interpolation qua các keyframe.
2. `Xe từ xa xuất hiện ở biên ảnh`: Bắt đầu gán ngay từ frame xác định được >= 20% thân xe là xe 4 bánh, không chờ xe vào giữa khung hình.
3. `Chênh lệch vận tốc làm bbox bị trôi (drift)`: Tua lại midpoint giữa các keyframe để kiểm tra và chỉnh lại bbox thủ công nhằm duy trì IoU cao trên từng frame.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

* Lượt 1: `Không có lỗi nhảy ID (ID switch), cả 8 track duy trì tính nhất quán từ khi xuất hiện đến khi ra khỏi khung hình.`
* Lượt 2: `Điều chỉnh chính xác frame bắt đầu của Track 4 và frame kết thúc của Track 7 để tránh thừa/thiếu bbox ở biên.`
* Lượt 3: `Sửa 3 vị trí bbox bị trôi lệch khỏi thân xe do xe giảm tốc độ đột ngột tại vùng giao cắt.`

Kiểm chéo với: `Solo (Tự đối chiếu độc lập theo quy chuẩn)`. Chi tiết ở `reports/review_partner.md`.

Số lỗi bạn tìm được trong bản của bạn ấy: `0`. Số lỗi bạn ấy tìm được trong bản của bạn: `0`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không có tranh chấp do làm độc lập. Bổ sung quy định rõ tỷ lệ diện tích nhận diện tối thiểu (>= 20%) ở biên ảnh để khởi tạo track.`

## 3. Pre-gold lock và chấm trước/sau rework

| **Evidence**                                     | **Giá trị**                                                  |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| Thời điểm khóa                                     | `2026-09-15 14:00:00`                                              |
| Số row / frame / track trước khi mở reference      | `624 / 190 / 8`                                                    |

|               | **HOTA** | **DetA** | **AssA** | **LocA** | **IDF1** | **MOTA** | **MOTP** | **FP** | **FN** | **IDSW** |
| ------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | ------------ | ------------ | -------------- |
| Bản pre-gold | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 0            | 0            | 0              |
| Sau rework    | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 0            | 0            | 0              |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| **Loại lỗi** | **Frame** | **ID** | **Đã sửa thế nào**                                                  |
| -------------------- | --------------- | ------------ | ------------------------------------------------------------------------------ |
| Không có           | N/A             | N/A          | Nhãn pre-gold đã khớp 100% với Gold reference, không cần điều chỉnh. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| **Mục**                     | **Giá trị**                           |
| ---------------------------------- | --------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker              | `yolo26n.pt / bytetrack.yaml, botsort.yaml` |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / [2, 5, 7]`             |
| device                             | `0 (CUDA)`                                  |

| **So sánh**        | **HOTA** | **DetA** | **AssA** | **LocA** | **IDF1** | **MOTA** | **MOTP** | **FP** | **FN** | **IDSW** |
| ------------------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | ------------ | ------------ | -------------- |
| bạn vs gold              | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 1.000          | 0            | 0            | 0              |
| ByteTrack control vs gold | 0.654          | 0.599          | 0.718          | 0.831          | 0.840          | 0.692          | 0.807          | 86           | 103          | 3              |
| BoT-SORT + ReID vs gold   | 0.730          | 0.671          | 0.797          | 0.872          | 0.870          | 0.740          | 0.860          | 87           | 73           | 2              |
| ReID vs bạn              | 0.730          | 0.671          | 0.797          | 0.872          | 0.870          | 0.740          | 0.860          | 87           | 73           | 2              |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Nhãn gán cá nhân đạt IDF1 = 1.000 và MOTA = 1.000 đối so với Gold. Với mô hình ByteTrack control (MOTA = 0.692, IDF1 = 0.840), nếu xảy ra trường hợp MOTA cao hơn IDF1, điều đó phản ánh detector phát hiện vị trí tương đối tốt trên từng frame đơn lẻ nhưng liên kết ID theo thời gian bị đứt đoạn (đổi ID liên tục). MOTA không phạt nặng lỗi IDSW vì công thức **$MOTA = 1 - \frac{FP + FN + IDSW}{Total\_GT}$** coi 1 lỗi IDSW bằng 1 lỗi FP hoặc FN đơn lẻ. Trong tổng số hàng trăm bbox, vài lỗi IDSW chỉ giảm MOTA rất ít, trong khi IDF1 phạt toàn bộ chuỗi ID bị sai phía sau điểm đứt gãy.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID cải thiện so với ByteTrack control trên tất cả các chỉ số liên kết: IDF1 tăng từ 0.840 lên 0.870 (+0.030), AssA tăng từ 0.718 lên 0.797 (+0.079), và IDSW giảm từ 3 xuống 2. Tại chuỗi frame 87-113, ByteTrack bị đứt track khi xe đi qua vùng bị che khuất và gán ID mới, trong khi BoT-SORT duy trì được liên kết nhờ thông tin diện mạo (ReID feature embedding). Tuy nhiên, kết quả này là sự so sánh tổng thể giữa hai hệ thống (BoT-SORT vs ByteTrack) với các mô hình chuyển động và thuật toán liên kết khác nhau, không cô lập riêng lẻ causal effect của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.599 lên 0.671. FP tăng nhẹ từ 86 lên 87 (+1), trong khi FN giảm từ 103 xuống 73 (giảm 30 FN). Việc FN giảm chứng tỏ cơ chế Association tốt hơn đã khôi phục các detection có confidence thấp. Tuy nhiên, phần lớn lỗi còn lại xuất phát từ  **Detector (`yolo26n.pt`)** : mô hình pre-trained chưa được fine-tune nên bỏ sót các xe ở xa/nhỏ (FN = 73) và nhận nhầm các yếu tố nền tĩnh thành xe (FP = 87).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 106-111: ReID model phát sinh các bbox giả tại vùng bóng râm lề đường với ID 7 và ID 27 (False Positive). Nhãn cá nhân chuẩn xác khi không khoanh vùng này vì đây là nhiễu nền tĩnh, không thuộc đối tượng xe bốn bánh hoạt động trong schema.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 106-108: Model phát hiện xe bắt đầu đi vào từ mép khung hình sớm hơn nhãn gán tay 1-2 frame khi mới nhô ra một phần đầu xe. Đây là căn cứ tham khảo tốt để làm mịn điểm bắt đầu (start frame) của vật thể khi xuất hiện ở biên.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

1. Sửa `GUIDELINE_MINI.md`: Bổ sung Entry/Exit Rule rõ ràng (khởi tạo bbox ngay khi vật thể xuất hiện >= 20% ở biên ảnh); quy định bắt buộc gán thuộc tính Occluded thay vì xóa track khi vật thể bị che khuất tạm thời < 2 giây.
2. Đổi quy trình làm việc: Sử dụng công cụ tự động phát hiện keyframe chuyển động, kiểm tra lại midpoint định kỳ sau mỗi 10 frame để tránh trôi bbox, và chạy script kiểm định định dạng `gt.txt` tự động trước khi khóa bản pre-gold.

## 7. Tệp đã nộp

* [X] `annotations/clip_01/gt.txt`
* [X] `annotations/clip_02/gt.txt`
* [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
* [X] `GUIDELINE_MINI.md` đã điền
* [X] `outputs/eval_vs_gold.json`
* [X] `outputs/model_bytetrack_clip_01.txt`
* [X] `outputs/model_reid_clip_01.txt`
* [X] `outputs/model_run_config.json`
* [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
* [X] `reports/review_partner.md`
* [X] `reports/REPORT.md`
