# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Hải Nam`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: Không gán xe đang đứng quá xa, chiếm diện tích bbox < ~10×10 px và không xác định được rõ là xe bốn bánh (nhìn giống blob). Bắt đầu track từ frame mà hình dạng xe bốn bánh xác định được.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Xe vẫn ở trong khung, chỉ bị xe khác hoặc vật thể che; ta nhận biết được cùng một xe khi nó hiện lại |
| Xe bị che lâu hơn 25 frame | tạo track mới nếu không đủ đặc điểm nhận dạng; giữ ID nếu vị trí + hướng di chuyển đủ rõ | Sau 2 giây, xe có thể đã di chuyển xa và nhầm với xe khác |
| Xe rời khung hình rồi quay lại | **track mới** — bấm `outside` khi xe khuất, tạo track mới khi xe vào lại | Không thể xác định chắc đó là cùng một xe sau khi đã ra ngoài tầm nhìn |
| Hai xe cắt nhau / chồng lên nhau | **KHÔNG đổi ID** — theo dõi màu sắc, kích thước và hướng di chuyển để phân biệt; đặt thêm keyframe ngay trước và sau điểm giao nhau | Đổi ID tại điểm cắt là lỗi phổ biến nhất; chi phí sửa rất cao |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa (x=0 hoặc x=width), không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**; không mở rộng để đoán phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **hình dạng 4 bánh nhận ra được, bbox tối thiểu ~20×15 px** |
| Xe đang đỗ, không di chuyển | vẫn gán bbox và track bình thường; chỉ đặt 1–2 keyframe nếu xe tuyệt đối không di chuyển; thêm keyframe nếu camera rung hoặc ánh sáng thay đổi |
| Keyframe đặt dày ở đâu | đặt keyframe **mỗi 10–15 frame** ở đoạn xe đổi hướng, tăng/giảm tốc, hoặc đi vào vùng bị che; đoạn thẳng + tốc độ đều có thể thưa hơn (mỗi 25–30 frame) |
| Bấm `outside` | bấm `outside` tại **frame đầu tiên xe khuất hoàn toàn** khỏi khung — không phải frame tiếp theo. Ví dụ: xe mất tại frame 148 → bấm outside tại frame 148 |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 149–151 / ID 4`
- Tình huống: Xe ID 4 rời khung ở rìa phải nhưng bbox vẫn còn tồn tại 3 frame sau khi xe đã khuất hoàn toàn. Không rõ xe khuất hẳn ở frame nào vì xe di chuyển từ từ ra rìa.
- Quyết định: Bấm lại `outside` tại frame 149 — frame đầu tiên xe không còn pixel nào trong khung.
- Lý do: Rule "outside tại frame đầu tiên xe khuất hoàn toàn" áp dụng chính xác ở đây. Bbox treo 3 frame sẽ tạo FP và bị eval trừ điểm MOTA.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 87–88 / ID 5`
- Tình huống: Xe ID 5 xuất hiện từ góc trên phải, di chuyển nhanh và đổi hướng trong khoảng frame 85–90. Keyframe đặt tại frame 79 và frame 95 khiến interpolation bị drift — bbox trôi ra khỏi xe tại frame 87–88 (IoU 0.55 và 0.58 so với gold).
- Quyết định: Thêm keyframe tại frame 87 để kẹp interpolation.
- Lý do: Xe đổi hướng + tốc độ trong đoạn ngắn → phải đặt keyframe dày hơn mức mặc định. Rule: keyframe mỗi 10–15 frame ở đoạn đổi hướng.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 104–116 / ID 6`
- Tình huống: Xe ID 6 đi vào vùng bị xe khác che khuất từ frame 104–116. Tại frame 112 và 118, IoU so với gold chỉ 0.54 và 0.51. Khó xác định phần nào của xe ID 6 thực sự nhìn thấy được vì xe che và xe bị che di chuyển gần nhau.
- Quyết định: Khoanh bbox ôm phần nhìn thấy của xe ID 6 (phần không bị xe khác che), dù bbox trông nhỏ hơn thực tế của xe. Thêm keyframe tại frame 104 và 116 để mark rõ điểm vào/ra vùng che.
- Lý do: Rule "bbox ôm phần nhìn thấy, không đoán phần bị che" — ưu tiên accuracy hơn là ước tính kích thước đầy đủ của xe. Hai người gán dễ quyết định khác nhau ở ca này → cần ghi vào guideline.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Luật `outside` thiếu mốc cụ thể**: Ban đầu chỉ ghi "bấm outside khi xe rời khung" mà không nói rõ frame nào. Sau khi eval phát hiện bbox treo 3 frame ở ID 4 và ID 8, đã bổ sung: *"bấm outside tại frame đầu tiên xe khuất hoàn toàn, không phải frame tiếp theo"*.
- **Luật keyframe chưa có ngưỡng số**: Trước đây chỉ ghi "đặt keyframe đủ dày" mà không có con số. Sau khi eval phát hiện bbox drift tại frame 168 (IoU 0.57) và frame 87–88 (IoU 0.55–0.58), đã cập nhật: *"mỗi 10–15 frame ở đoạn đổi hướng; mỗi 25–30 frame ở đoạn thẳng đều"*.
- **Luật bbox khi xe bị che một phần chưa giải quyết ca biên**: Khi xe bị che > 50% diện tích, hai người có thể gán bbox khác nhau hoàn toàn (khoanh phần thấy vs ước tính cả xe). Quy tắc chính thức: *luôn khoanh phần nhìn thấy được, kể cả khi chỉ còn < 50% diện tích xe*. Cần thêm ví dụ ảnh vào guideline nếu gán thêm clip.
