# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Hoàng Văn Đạt — 2A202602267
Ngày: 14/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (app.cvat.ai) |
| Thời gian gán `clip_02` (warm-up) | ~15 phút |
| Thời gian gán `clip_01` | ~30 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | ~6 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Hai xe đi song song, bbox chồng lên nhau (frame ~94, track 5 và 6)**: Khó xác định xe nào là xe nào sau khi tách ra. Xử lý bằng cách quan sát quỹ đạo và vị trí tương đối trước lúc chồng — xe bên trái giữ ID 5, xe bên phải giữ ID 6.
2. **Xe bị che khuất tạm thời (frame ~59, track 4)**: Xe đi qua vùng bị che bởi xe khác khoảng 8 frame. Xử lý: giữ nguyên ID vì thời gian che dưới 25 frame (2 giây), bbox ôm phần nhìn thấy trong lúc bị che một phần.
3. **Xe thoáng qua rìa khung hình (clip_02, track 2, frame 1–2)**: Xe chỉ xuất hiện 2 frame ở rìa ảnh. Xử lý: vẫn gán track vì xác định được là xe bốn bánh, bấm `outside` ngay ở frame 2.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Không phát hiện ID nhảy hay đổi số bất thường. Số ID hiển thị liên tục, nhất quán từ đầu đến cuối clip.
- Lượt 2: Phát hiện track 5 (clip_02) còn bbox treo sau khi xe rời khung (frame 57–60) — đã sửa bằng cách bấm `outside` đúng frame. Các track còn lại có frame đầu/cuối hợp lý.
- Lượt 3: Kiểm tra giữa các đoạn keyframe dài — bbox vẫn khít, không có trường hợp interpolation trôi quá xa.

Kiểm chéo với: *(chờ bạn cùng nhóm)*. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

*(Cập nhật sau khi kiểm chéo)*

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | — | — | — | — | — | — | — | — | — | — |
| Sau rework | — | — | — | — | — | — | — | — | — | — |

*(Cập nhật sau khi nhận gold từ giảng viên)*

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **chờ gold**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| *(Cập nhật sau rework)* | | | |

## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n.pt`, tracker `bytetrack.yaml`, conf `0.25`, imgsz `960`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | — | — | — | — | — | — | — | — | — | — |
| model vs gold | — | — | — | — | — | — | — | — | — | — |
| model vs bạn | 0.670 | 0.609 | 0.739 | 0.860 | 0.833 | 0.682 | 0.840 | 84 | 116 | 3 |

*(bạn vs gold và model vs gold: cập nhật sau khi nhận gold)*

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

*(Cập nhật sau khi có kết quả bạn vs gold)*

Từ kết quả model vs bạn: MOTA = 0.682 thấp hơn IDF1 = 0.833. Điều này cho thấy model bỏ sót và vẽ thừa bbox (FP=84, FN=116) nhiều hơn là mắc lỗi ID (IDSW=3). MOTA chỉ tính mỗi ID switch một lần, nên một xe bị tách thành hai track chỉ làm MOTA giảm 1 điểm — trong khi IDF1 phạt toàn bộ nửa quãng đời của track đó. Vì vậy MOTA có thể cao trong khi nhãn sai ID nghiêm trọng.

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

Model vs bạn: DetA = 0.609, AssA = 0.739. Lệch nhau 0.130 — DetA thấp hơn đáng kể. Chính DetA kéo HOTA (0.670) xuống, tức là model **không tìm ra xe** là vấn đề lớn hơn là đánh mất ID. FN = 116 (bỏ sót) lớn hơn nhiều so với IDSW = 3, xác nhận điều này. Model bỏ sót xe bị che hoặc xe ở rìa khung nhiều hơn là gán nhầm ID.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

Frame 106–109: bạn có bbox cho 2 xe mà model không nhận ra. Nhiều khả năng đây là xe bị che khuất một phần — model mất tự tin (conf thấp) nên không giữ track, trong khi bạn có thể nhận ra đó vẫn là xe cũ và giữ bbox theo quỹ đạo.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

Frame 106–109: đồng thời model cũng có 2 bbox mà bạn không có — có thể là xe nhỏ ở rìa khung hoặc xe vừa xuất hiện mà bạn chưa kịp vẽ track. Cần kiểm tra lại frame này trong CVAT.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

Loại nhiều nhất là **"chỉ model có"** và **"chỉ bạn có"** xuất hiện cùng nhau ở các frame 106–109 (mỗi bên 2 bbox). Điều này cho thấy clip_01 có nhiều đoạn xe bị che hoặc mật độ xe cao, khiến cả người và model đều bỏ sót những xe khác nhau. Model bỏ sót xe bị che (FN=116 lớn), còn bạn bỏ sót xe nhỏ ở rìa mà model phát hiện được.

## 6. Nếu phải gán thêm 10 clip nữa

Sẽ sửa trong `GUIDELINE_MINI.md`:
- Thêm ví dụ ảnh minh họa cho ca hai xe cắt nhau để người gán tiếp theo nhận ra nhanh hơn.
- Quy định rõ ngưỡng kích thước bbox tối thiểu (chiều rộng >= 20px) và viết thành luật cứng thay vì để mỗi người tự quyết.
- Thêm bước kiểm tra: sau khi gán xong mỗi track, replay lại đúng track đó (ẩn các track khác) để kiểm tra nhanh trước khi sang track tiếp theo.

Đổi trong quy trình: đặt keyframe thủ công ở frame ngay trước và sau khi xe bị che, thay vì dựa vào interpolation tự động của CVAT — giảm đáng kể lỗi bbox trôi.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json` *(chờ gold)*
- [x] `outputs/model_clip_01.txt`
- [ ] `outputs/eval_model_vs_gold.json` *(chờ gold)*
- [x] `outputs/eval_model_vs_me.json`
- [ ] `reports/review_partner.md` *(chờ kiểm chéo)*
- [x] `reports/REPORT.md` (file này)
