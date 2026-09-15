# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: Hoàng Văn Đạt — 2A202602267
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

Bổ sung: xe thoáng qua chỉ xuất hiện 1–2 frame vẫn gán nếu xác định được là xe bốn bánh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps) | Xe vẫn là cùng một thực thể, chỉ tạm khuất sau vật cản |
| Xe bị che lâu hơn 25 frame | Mở track mới | Không còn đủ tự tin đây là cùng một xe |
| Xe rời khung hình rồi quay lại | **Track mới** — bấm `outside` ngay khi xe ra khỏi rìa ảnh | Đã ra khỏi khung là kết thúc track, không đoán xe quay lại |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID từng xe, bbox ôm phần nhìn thấy của từng xe riêng | Mỗi xe vẫn là thực thể độc lập dù bbox chồng lấp |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm đúng phần **nhìn thấy được**, không vẽ rộng ra phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên có thể xác định rõ đó là xe bốn bánh; ngưỡng chọn: chiều rộng bbox >= 20px |
| Xe đang đỗ, không di chuyển | Vẫn gán và track suốt thời gian xe còn trong khung |
| Keyframe đặt dày ở đâu | Đặt dày khi xe rẽ, phanh, bị che, hoặc đổi tốc độ; thưa hơn khi xe đi thẳng đều |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_02` / frame 1–2 / ID 2
- Tình huống: Xe xuất hiện ở rìa ảnh chỉ trong 2 frame rồi biến mất hoàn toàn. Không rõ là xe vào khung hay chỉ thoáng qua do camera rung.
- Quyết định: Gán track, bấm `outside` ở frame 2.
- Lý do: Vẫn xác định được là xe bốn bánh, dù chỉ 2 frame. Luật lab: gán từ frame đầu tiên xác định được.

### Ca 2
- Clip / frame / ID: `clip_01` / frame 59 / ID 4
- Tình huống: Xe đi qua vùng bị che bởi xe khác trong khoảng 8 frame. Sau khi hiện lại, hình dạng và vị trí vẫn nhất quán với xe cũ.
- Quyết định: Giữ nguyên ID 4 — thời gian bị che dưới 25 frame.
- Lý do: Theo luật lab: che dưới 2 giây thì giữ ID cũ. Xe hiện lại đúng quỹ đạo dự kiến.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 94 / ID 5
- Tình huống: Hai xe đi song song, bbox gần như chồng lên nhau trong khoảng 10 frame. Khó phân biệt xe nào là xe nào sau khi tách ra.
- Quyết định: Quan sát hướng di chuyển trước khi chồng — xe bên trái vẫn là ID 5, xe bên phải vẫn là ID 6.
- Lý do: Dùng quỹ đạo và vị trí tương đối để xác định lại ID sau khi tách, không đổi ID.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Cần ghi rõ hơn ngưỡng "xe quá nhỏ": hiện tại chọn 20px chiều rộng nhưng chưa thống nhất với cả nhóm.
- Luật hai xe cắt nhau cần thêm ví dụ cụ thể: nếu cả hai xe đều bị che hoàn toàn trong lúc chồng thì xử lý thế nào (tạm thời dùng quỹ đạo Kalman để quyết định).
- Bổ sung luật: xe đứng yên nhiều frame liên tiếp vẫn đặt keyframe mỗi 15–20 frame để tránh bbox trôi khi nền thay đổi.
