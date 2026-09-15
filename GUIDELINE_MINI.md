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

Bổ sung: gán xe ngay từ frame đầu tiên xác định được đủ hình dạng xe bốn bánh —
không chờ xe đi vào giữa khung mới bắt đầu track.

---

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps) | Xe vẫn là xe cũ nếu thời gian che ngắn; thay ID làm tăng IDSW |
| Xe bị che lâu hơn 25 frame | Tạo track mới; bấm `outside` ở frame cuối thấy, track mới bắt đầu frame đầu thấy lại | Quá lâu không đảm bảo nhận dạng đúng |
| Xe rời khung hình rồi quay lại | **Track mới** — bấm `outside` đúng frame xe ra, tạo track mới khi xe vào lại | Không thể đảm bảo cùng xe; tránh nhầm ID |
| Hai xe cắt nhau / chồng lên nhau | Quan sát quỹ đạo trước khi chồng; xe bên nào ở phía nào giữ nguyên ID đó | IoU thấp trong lúc overlap không có nghĩa đổi ID |
| Xe đứng yên suốt clip | Vẫn gán track đầy đủ; không bấm `outside` chỉ vì xe không di chuyển | Xe đỗ vẫn là xe bốn bánh cần gán |

---

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm **phần nhìn thấy được**; không vẽ bbox bao cả phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng: chiều rộng hoặc chiều cao ≥ 15px |
| Xe đang đỗ, không di chuyển | Đặt keyframe ở frame đầu và frame cuối; thêm 1 keyframe giữa nếu ánh sáng thay đổi |
| Keyframe đặt dày ở đâu | Mỗi lần xe đổi hướng, tăng tốc, hoặc bị che một phần — đặt keyframe ngay trước và sau sự kiện đó |
| Frame đầu / cuối của track | Bấm `outside` chính xác ở frame xe rời khung hoàn toàn, không để bbox treo thêm frame |

---

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1 — Xe đứng yên nhiều frame liên tiếp (clip_01, frame 1–15, ID 3)
- **Clip / frame / ID**: clip_01, frame 1–15, track 3
- **Tình huống**: Xe đỗ sát lề, không di chuyển suốt 15 frame đầu. CVAT interpolation không dịch chuyển bbox, check_mot_labels cảnh báo "bbox gần như đứng im".
- **Quyết định**: Giữ nguyên track, không bấm `outside`.
- **Lý do**: Xe đỗ là xe thật, nằm trong phạm vi gán. Đứng im không phải lỗi; cảnh báo chỉ nhắc kiểm tra chứ không bắt sửa.

### Ca 2 — Hai xe đi song song, bbox chồng lên nhau (clip_01, frame ~94, ID 5 và 6)
- **Clip / frame / ID**: clip_01, frame 90–97, track 5 và 6
- **Tình huống**: Hai xe chạy cùng chiều, bbox chồng lên nhau khoảng 7 frame, khó phân biệt xe nào là xe nào sau khi tách ra.
- **Quyết định**: Quan sát vị trí tương đối trước frame 90 — xe bên trái giữ ID 5, xe bên phải giữ ID 6. Không đổi ID.
- **Lý do**: Quỹ đạo trước và sau lúc chồng nhất quán; đổi ID ở đây sẽ làm tăng IDSW mà không phản ánh thực tế.

### Ca 3 — Xe thoáng qua rìa khung hình chỉ 2 frame (clip_02, frame 1–2, ID 2)
- **Clip / frame / ID**: clip_02, frame 1–2, track 2
- **Tình huống**: Xe xuất hiện ở rìa trái ảnh, chỉ nhìn thấy khoảng 1/3 thân xe, và biến mất ngay frame 3.
- **Quyết định**: Vẫn gán track 2 frame vì xác định được là xe bốn bánh. Bấm `outside` ở frame 2.
- **Lý do**: Luật gán từ frame đầu xác định được. Track chỉ 2 frame là bình thường nếu xe thật sự chỉ xuất hiện 2 frame.

### Ca 4 — Xe bị che khuất tạm thời, bbox ôm phần nào (clip_01, frame ~59, ID 4)
- **Clip / frame / ID**: clip_01, frame 55–67, track 4
- **Tình huống**: Xe đi qua vùng bị che bởi xe khác khoảng 8 frame, chỉ thấy một phần nhỏ của xe.
- **Quyết định**: Giữ nguyên ID 4 (dưới ngưỡng 25 frame). Bbox ôm phần nhìn thấy trong lúc bị che.
- **Lý do**: 8 frame < 25 frame ngưỡng giữ ID. Tạo track mới ở đây sẽ là lỗi tách track.

---

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Sau khi chạy `evaluate_tracking.py` và đọc `eval_vs_gold.json`, phát hiện các luật còn mơ hồ:

- **Luật biên track cần chính xác hơn**: Gold cho thấy ID 5, 6, 4, 7, 8 có bbox xuất hiện sai frame so với gold (bbox treo hoặc bắt đầu sớm). Cần ghi rõ: *bấm `outside` đúng frame cuối thấy xe, không phải frame sau đó*.
- **Keyframe quanh vùng xe chuyển hướng nhanh**: Track 5 (frame 83–88) có IoU tụt xuống 0.51–0.59 do interpolation drift. Cần quy định: đặt keyframe mỗi khi xe thay đổi hướng hoặc tốc độ đột ngột, khoảng cách tối đa giữa 2 keyframe là 10 frame trong đoạn xe di chuyển nhanh.
- **Bbox tối thiểu cần viết thành số cụ thể**: Trước chỉ ghi "xác định được là xe bốn bánh" — nay thêm ngưỡng cứng: chiều rộng ≥ 15px hoặc chiều cao ≥ 15px.
