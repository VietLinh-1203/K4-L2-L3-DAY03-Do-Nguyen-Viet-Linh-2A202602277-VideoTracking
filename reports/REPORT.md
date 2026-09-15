# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `...`
Ngày: `...`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `...` phút |
| Thời gian gán `clip_01` | `...` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe buýt xuất hiện một phần ở frame 54: chỉ vẽ phần xe nhìn thấy.`
2. `Bbox trôi ở frame 83–84: thêm keyframe và chỉnh bbox.`
3. `SUV trắng đứng yên: vẫn track xuyên frame 1–190 vì là `vehicle`.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Kiểm ID; kết quả cuối `IDSW = 0`.`
- Lượt 2: `Phát hiện và xóa track thừa, kiểm mốc vào/ra ảnh.`
- Lượt 3: `Chỉnh bbox trôi ở frame 54, 83–84 và 106–112.`

Kiểm chéo với: `chưa thực hiện`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `chưa có`. Số lỗi bạn ấy tìm được trong bản của bạn: `chưa có`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Chưa có peer review. Sau khi chấm gold, bổ sung luật: xe đứng yên vẫn phải track và bật Outside ở frame đầu tiên xe biến mất.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `6ba44efd014d39b4032869e97fc26c5232a532591c632935ac042c330a53217f` |
| Thời điểm khóa | `2026-09-15T14:03:43.093379+00:00` |
| Số row / frame / track trước khi mở reference | `562 / 190 / 9` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.569 | 0.426 | 0.761 | 0.865 | 0.661 | 0.328 | 0.851 | 187 | 198 | 0 |
| Sau rework | 0.802 | 0.783 | 0.821 | 0.884 | 0.944 | 0.883 | 0.874 | 58 | 9 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Track thừa | 1–91, 1–36 | pre-gold 3, 4 | Xóa hai track thừa |
| Bỏ sót | 1–190 | 3 ở export cuối | Thêm track SUV trắng |
| Bbox trôi | 54, 83–84, 106–112 | 4–6 | Chỉnh mốc và thêm keyframe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack / BoT-SORT + ReID` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.802 | 0.783 | 0.821 | 0.884 | 0.944 | 0.883 | 0.874 | 58 | 9 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.786 | 0.727 | 0.851 | 0.916 | 0.879 | 0.760 | 0.910 | 81 | 65 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA 0.883 thấp hơn IDF1 0.944. ID được giữ tốt, lỗi còn lại chủ yếu là FP 58 và FN 9. MOTA chủ yếu phạt FP, FN và ID switch; IDF1 đo độ giữ identity rõ hơn.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; cả hai có 2 ID switch. ByteTrack đổi ID ở frame 94, ReID đổi ở frame 87. Đây không chứng minh riêng ReID gây ra khác biệt vì hai tracker khác implementation.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ReID tăng DetA 0.649 lên 0.711, giảm FN 54 xuống 26, nhưng FP tăng 88 lên 91. Detector và association đều còn lỗi.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Xe đỗ được giữ từ frame 1–190; ReID có bbox lỏng ở frame 190, IoU 0.589 với gold.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 104–115: ReID có bbox lỏng ở 104 và đổi ID ở 113. Đây là vùng cần xem lại keyframe thủ công.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Ghi rõ frame đầu/cuối của từng track, lưu số keyframe và thời lượng, rồi QC theo thứ tự: ID → endpoint → geometry → peer review.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
