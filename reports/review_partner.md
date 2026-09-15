# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Lê Hải Nam` |
| Reviewer | `[điền tên partner]` |
| Pair ID | `clip_01` |
| CVAT version | `[điền]` |
| Thời điểm review | `2026-09-15` |

---

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

Nguồn evidence: `outputs/eval_vs_gold.json` (so sánh nhãn tác giả vs gold clip_01, 190 frame, IoU ngưỡng 0.5).
Tổng quan: FP 11 · FN 35 · IDSW 0 · HOTA 0.795 · IDF1 0.959 · MOTA 0.920 · MOTP 0.855.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 148 | 149 | 4 | BBOX TREO — xe đã rời khung | bbox ID 4 còn tồn tại frame 149–151 (3 frame) sau khi track gold 4 đã kết thúc. Vi phạm rule: `outside` phải bấm đúng frame xe khuất hoàn toàn khỏi khung | Bấm `outside` lại tại frame 149 (CVAT frame 148); xóa 3 frame bbox thừa | fixed |
| 2 | 168 | 169 | 8 | BBOX TREO — xe đã rời khung | bbox ID 8 còn tồn tại frame 169–171 (3 frame) sau khi track gold 8 đã kết thúc. Vi phạm rule: không để bbox "treo" sau khi xe rời khung | Bấm `outside` lại tại frame 169 (CVAT frame 168); xóa 3 frame bbox thừa | fixed |
| 3 | 167 | 168 | 8 | BBOX TRÔI — drift giữa keyframe | frame 168, track gold 8, IoU chỉ 0.57 — thấp hơn mức an toàn. Bbox interpolation bị kéo lệch khỏi xe do hai keyframe cách xa nhau. Rule: thêm keyframe ở đoạn xe đổi hướng/tốc độ | Thêm keyframe tại frame 168 để kẹp chặt interpolation | fixed |
| 4 | 117 | 118 | 6 | BBOX LỆCH — IoU thấp so với gold | frame 118, track gold 6, IoU 0.51 — gần ngưỡng fail. Bbox khoanh lệch, chưa sát với phần xe nhìn thấy. Rule: bbox ôm phần nhìn thấy, không đoán phần bị che | Thống nhất luật khoanh rồi điều chỉnh keyframe quanh frame 118 | needs-review |
| 5 | 137 | 138 | 3 | BBOX LỆCH — IoU thấp so với gold | frame 138, track gold 3, IoU 0.52. Tương tự F4 — bbox chưa bám sát xe ở đoạn xe di chuyển nhanh | Thêm keyframe tại hoặc ngay trước frame 138 | needs-review |
| 6 | 189 | 190 | 3 | BBOX LỆCH — IoU thấp so với gold | frame 190, track gold 3, IoU 0.52. Frame cuối clip, xe gần rìa ảnh, bbox kéo rộng ra | Điều chỉnh keyframe cuối của track 3 cho sát xe | needs-review |
| 7 | 169 | 170 | 8 | BBOX LỆCH — IoU thấp so với gold | frame 170, track gold 8, IoU 0.52. Xảy ra gần thời điểm bbox treo (finding #2) — có thể liên quan | Sau khi sửa outside (finding #2), kiểm lại bbox frame 170 | needs-review |
| 8 | 111 | 112 | 6 | BBOX LỆCH — IoU thấp so với gold | frame 112, track gold 6, IoU 0.54. Đoạn xe 6 đang quẹo/thay đổi hướng, cần keyframe dày hơn | Thêm keyframe quanh frame 112 | needs-review |
| 9 | 86 | 87 | 5 | BBOX LỆCH — IoU thấp so với gold | frame 87, track gold 5, IoU 0.55. Xe 5 mới xuất hiện và di chuyển nhanh, interpolation chưa bám kịp | Thêm keyframe tại frame 87 | needs-review |
| 10 | 87 | 88 | 5 | BBOX LỆCH — IoU thấp so với gold | frame 88, track gold 5, IoU 0.58. Tiếp diễn của F9 — drift tại frame 88 sau khi xe 5 đổi vận tốc | Điều chỉnh keyframe tại frame 87–88 cùng lúc với F9 | needs-review |

> Còn 2 finding loại BBOX LỆCH nữa — xem chi tiết trong `outputs/eval_vs_gold.json`.

---

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, khớp gold 8 track, không gán xe máy/người |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0 so với gold; không có hoán đổi ID |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | AssA 0.810, IDF1 0.959 — ID được giữ qua các điểm che khuất |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Frame 149–151 ID 4; frame 169–171 ID 8 — bbox treo 3 frame (finding #1, #2) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | 7 frame có IoU < 0.60 so với gold (finding #4–#10); cần chuẩn hóa luật khoanh |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Frame 168 ID 8 IoU 0.57 (finding #3); một số đoạn keyframe thưa gây drift |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py chạy 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Xem cột closure ở bảng finding trên |

---

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Không phát hiện ID switch; 8 track đều có entry/exit hợp lệ về mặt timeline |
| 2 — endpoint/scope | ĐÃ SỬA | Bbox treo ID 4 (frame 149–151) và ID 8 (frame 169–171) đã được tác giả fix sau finding #1, #2 |
| 3 — geometry/interpolation | NEEDS-REVIEW | 7 frame IoU < 0.60 (finding #4–#10) cần tác giả và reviewer thống nhất luật khoanh bbox trước khi đóng |

---

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Finding #1 và #2 — BBOX TREO. Rule: bấm outside tại frame đầu tiên xe khuất hoàn toàn khỏi khung, không phải frame sau đó. Đây là lỗi rõ ràng nhất vì bbox tồn tại ở vị trí không có xe thật.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `[điền nếu có ca tác giả không đồng ý sửa]`
3. Một rule cần Lab Coach làm rõ (nếu có): `Khi xe bị che một phần ở rìa khung, bbox có nên ôm trọn xe (ước lượng phần khuất) hay chỉ khoanh phần nhìn thấy? Một số finding BBOX LỆCH (IoU 0.51–0.58) có thể xuất phát từ hai cách hiểu khác nhau về rule này.`
