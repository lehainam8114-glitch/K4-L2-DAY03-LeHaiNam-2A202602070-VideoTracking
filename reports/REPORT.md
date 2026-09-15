# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Lê Hải Nam`
Ngày: `15-09-2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `20` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `4` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất bởi xe khác** — Giữ nguyên ID và thêm keyframe ngay trước/sau điểm bị che để interpolation không bị lệch. Đối chiếu hướng di chuyển và màu xe để xác nhận đúng ID sau khi hiện lại.
2. **Hai xe đi gần nhau, bbox gần chồng lên nhau** — Khoanh sát phần nhìn thấy của từng xe, không mở rộng bbox ra ngoài vùng bị che. Kiểm tra frame tiếp theo để đảm bảo ID không bị hoán đổi.
3. **Xe xuất hiện/rời khung ở rìa ảnh** — Bấm `outside` chính xác frame xe vừa khuất hoàn toàn; không để bbox "treo" vài frame sau khi xe đã rời khung (ID 4 frame 149-151, ID 8 frame 169-171 là lỗi đã phát hiện qua eval).

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Kiểm ID switch — không phát hiện trường hợp nào đổi ID giữa chừng (IDSW = 0 so với gold).
- Lượt 2: Kiểm frame đầu/cuối track — phát hiện ID 4 và ID 8 còn bbox sau khi xe rời khung (3 frame mỗi ID); đã ghi nhận để sửa.
- Lượt 3: Kiểm bbox giữa track — phát hiện frame 168 track gold 8 IoU chỉ 0.57; cần thêm keyframe ở đó.

Kiểm chéo với: `...` ← điền tên bạn review. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Cần bổ sung quy tắc "bbox treo" — xe đã rời khung bao nhiêu frame thì nhất định bấm `outside`; hiện tại guideline chưa nêu rõ tolerance. Ngoài ra cần quy tắc khoanh bbox khi xe bị che một phần: chỉ khoanh phần nhìn thấy hay vẫn ước lượng cả xe?

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` ← xem file manifest |
| Thời điểm khóa | `...` ← xem file manifest |
| Số row / frame / track trước khi mở reference | `549 bbox / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `0.785` | `0.775` | `0.793` | `0.884` | `0.960` | `0.932` | `0.844` | 13 | 35 | 0 |
| Sau rework | **0.795** | **0.783** | **0.810** | **0.868** | **0.959** | **0.920** | **0.855** | 11 | 35 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT** — IDF1 0.959 / MOTA 0.920 / MOTP 0.855

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TREO — xe đã rời khung | 149–151 | 4 | Bấm `outside` đúng frame xe rời khung, xóa 3 frame thừa |
| BBOX TREO — xe đã rời khung | 169–171 | 8 | Bấm `outside` đúng frame xe rời khung, xóa 3 frame thừa |
| BBOX TRÔI — IoU thấp giữa keyframe | 168 | 8 | Thêm keyframe tại frame 168 để interpolation bám sát xe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `...` ← xem model_run_config.json |
| weights / hai tracker | `yolo26n.pt` / ByteTrack và BoT-SORT + ReID |
| conf / IoU / imgsz / classes | `...` ← xem model_run_config.json |
| device | `...` ← xem model_run_config.json |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.795 | 0.783 | 0.810 | 0.868 | 0.959 | 0.920 | 0.855 | 11 | 35 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.778 | 0.720 | 0.845 | 0.883 | 0.903 | 0.790 | 0.872 | 102 | 13 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Bản annotation của tôi: MOTA = 0.920, IDF1 = 0.959 — cả hai đều cao, nhưng IDF1 > MOTA. Đây là chiều ngược thường thấy: MOTA phạt FP + FN + IDSW nhưng không tính chất lượng giữ ID liên tục qua thời gian. Nếu MOTA cao mà IDF1 thấp (ví dụ ByteTrack: MOTA 0.749 vs IDF1 0.875), có nghĩa model detect được phần lớn xe (ít FP/FN) nhưng hay đổi ID — mỗi lần đổi ID chỉ bị tính một lần IDSW trong MOTA, trong khi IDF1 trừ điểm trên toàn bộ đoạn track bị gán sai. MOTA không phạt nặng vì IDSW chỉ đếm số lần chuyển ID, không nhân với độ dài đoạn track sau đó.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

> Lưu ý: Đây không phải causal ablation của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau — sự khác biệt có thể đến từ thuật toán matching, không chỉ từ ReID.

| Metric | ByteTrack | BoT-SORT + ReID | Delta |
|--------|----------:|----------------:|------:|
| IDF1   | 0.875     | 0.900           | +0.025 |
| AssA   | 0.776     | 0.820           | +0.044 |
| IDSW   | 2         | 2               | 0 |

BoT-SORT + ReID có AssA cao hơn đáng kể (+0.044): xe mất rồi xuất hiện lại giữ ID tốt hơn. Ví dụ cụ thể: track gold 5 (60 frame) — ByteTrack chia thành ID [32, 23] với IDSW tại frame 94, trong khi BoT-SORT + ReID chia thành [18, 17] với IDSW tại frame 87. Cả hai đều có IDSW = 2, nhưng FN của ReID (26) thấp hơn ByteTrack (54) rõ rệt, cho thấy BoT-SORT hồi phục track sau occlusion tốt hơn — ít bỏ sót xe khi xe hiện lại.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

| Metric | ByteTrack | BoT-SORT + ReID |
|--------|----------:|----------------:|
| DetA   | 0.649     | 0.711           |
| FP     | 88        | 91              |
| FN     | 54        | 26              |

ReID cải thiện đáng kể FN (54 → 26), tức là bắt được nhiều xe hơn trong các frame mà ByteTrack bỏ sót. FP tăng nhẹ (88 → 91) do model detect thêm object không thuộc gold (ID 38 xuất hiện ở ReID không có ở ByteTrack). Lỗi còn lại chủ yếu là **detector**: cả hai tracker đều có DetA < 0.72 và nhiều BBOX LỆCH (30 trường hợp ở ByteTrack, 9 trường hợp ở ReID), trong khi bản annotation của tôi đạt DetA 0.783. Model YOLO zero-shot bỏ sót xe nhỏ/xa và khoanh bbox không sát. Lỗi association (IDSW = 2) vẫn tồn tại ở cả hai, chưa được giải quyết.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Frame 16–116, ID 7 (ReID)**: BoT-SORT + ReID tạo một track ID 7 kéo dài 43 frame không khớp với bất kỳ track gold nào — đây là bbox thừa. Annotation của tôi không có track này, nghĩa là tôi đúng khi không gán object này (khả năng cao là xe máy hoặc object nền bị detector YOLO nhầm). ReID sai vì detector detect nhầm và tracker giữ ID đó suốt 43 frame liên tục thay vì loại bỏ do độ tin cậy thấp.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Frame 168, track gold 8**: Báo cáo eval_vs_gold chỉ ra IoU chỉ 0.57 tại frame này — đây là ca BBOX TRÔI trong annotation của tôi. Khi đối chiếu với kết quả ReID (LocA 0.872 > annotation LocA 0.868 ở một số đoạn), tôi xem lại frame 168 và xác nhận bbox của tôi bị trôi ra khỏi xe do khoảng cách quá xa giữa hai keyframe. Đây là ca model gợi ý tôi cần thêm keyframe, không phải model sai. Evidence: `outputs/eval_vs_gold.json` mục BBOX TRÔI frame 168.

## 6. Nếu phải gán thêm 10 clip nữa

Sẽ sửa trong `GUIDELINE_MINI.md`:
1. **Quy tắc `outside` rõ hơn**: ghi rõ "bấm outside tại frame đầu tiên xe khuất hoàn toàn khỏi khung, không phải frame tiếp theo" để tránh bbox treo 1–3 frame như ID 4 và ID 8.
2. **Mật độ keyframe tối thiểu**: thêm quy tắc "đặt keyframe ít nhất mỗi 15 frame ở đoạn xe chuyển hướng hoặc thay đổi tốc độ" để tránh bbox trôi giữa hai keyframe xa.
3. **Phân biệt xe 4 bánh nhỏ vs xe máy**: thêm ví dụ ảnh cho ca mơ hồ (xe tải nhỏ từ xa trông giống xe máy) để giảm FP do nhầm loại.

Sẽ đổi trong quy trình làm việc:
- Chạy `eval_vs_gold` sớm hơn (sau khi gán xong draft lần đầu) để biết lỗi bbox treo ngay, không chờ đến cuối.
- Dùng frame sequence gợi ý từ báo cáo lỗi (ví dụ BBOX LỆCH) để ưu tiên thêm keyframe đúng chỗ thay vì thêm đều.

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
- [ ] `reports/review_partner.md` ← điền sau khi kiểm chéo
- [x] `reports/REPORT.md` (file này)
