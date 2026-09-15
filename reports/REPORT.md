# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Hoàng Văn Đạt — 2A202602267
Ngày: 15/09/2026

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

Kiểm chéo: tự kiểm dựa trên `eval_vs_gold.json` và `eval_reid_vs_me.json`. Chi tiết ở `reports/review_partner.md`.
Số lỗi phát hiện: 6 finding (5 bbox treo/sớm, 1 interpolation drift). Tất cả đã có closure.

Ca bất đồng: track 3 clip_01 frame 1–15 đứng im — check_mot_labels cảnh báo nhưng xe thực sự đỗ, đóng là `not-a-defect`. Luật còn thiếu trong GUIDELINE_MINI.md: quy tắc biên track (bấm `outside` chính xác) và keyframe dày hơn quanh vùng xe đổi hướng — đã bổ sung.

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.817 | 0.787 | 0.851 | 0.908 | 0.929 | 0.850 | 0.901 | 76 | 10 | 0 |
| Sau rework | — | — | — | — | — | — | — | — | — | — |

*(Sau rework: sửa 6 finding bbox treo và interpolation drift trong CVAT rồi export lại — chờ cập nhật)*

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT** — IDF1=0.929, MOTA=0.850, MOTP=0.901

Sau khi đọc danh sách lỗi, các lỗi cần sửa:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo trước khi xe xuất hiện | 54–78 | 5 | Bấm `outside` ở frame 53; track bắt đầu từ frame 79 |
| Bbox treo trước khi xe xuất hiện | 78–100 | 6 | Bấm `outside` ở frame 77; track bắt đầu từ frame 101 |
| Bbox treo sau khi xe rời khung | 149–151 | 4 | Bấm `outside` ở frame 148 |
| Bbox treo trước khi xe xuất hiện | 51–53 | 4 | Xóa keyframe frame 51–53 |
| Bbox treo trước khi xe xuất hiện | 103–105 | 7 | Bấm `outside` ở frame 102 |
| Bbox treo sau khi xe rời khung | 169–171 | 8 | Bấm `outside` ở frame 168 |
| Interpolation drift | 83–88 | 5 | Thêm keyframe ở frame 85 và 87 |

## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n.pt`, conf `0.25`, iou `0.70`, imgsz `960`, classes `[2,5,7]` (car/bus/truck), device `cpu`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.817 | 0.787 | 0.851 | 0.908 | 0.929 | 0.850 | 0.901 | 76 | 10 | 0 |
| ByteTrack vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.749 | 0.692 | 0.812 | 0.901 | 0.869 | 0.743 | 0.891 | 80 | 81 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Nhãn của bạn vs gold: MOTA = 0.850, IDF1 = 0.929 — MOTA **thấp hơn** IDF1. Điều này cho thấy lỗi chủ yếu là FP (76 bbox thừa — do bbox treo trước/sau biên track), không phải lỗi ID (IDSW = 0). MOTA tính lỗi ID switch chỉ một lần duy nhất tại frame xảy ra switch, trong khi IDF1 phạt toàn bộ quãng đời của track bị ảnh hưởng — nếu một xe bị đổi ID ở frame 50 thì IDF1 phạt từ frame đó đến hết clip, còn MOTA chỉ trừ 1 điểm. Vì vậy MOTA có thể cao trong khi lỗi ID nghiêm trọng, và IDF1 phản ánh trung thực hơn chất lượng giữ identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA, IDSW?**

| Metric | ByteTrack | ReID | Chênh |
| --- | ---: | ---: | ---: |
| IDF1 | 0.875 | 0.900 | +0.025 |
| AssA | 0.776 | 0.820 | +0.044 |
| IDSW | 2 | 2 | 0 |
| FN | 54 | 26 | -28 |

ReID treatment có IDF1 và AssA cao hơn ByteTrack, đồng thời FN giảm 28 — ReID bắt được nhiều xe bị che hơn. Cả hai đều có 2 IDSW, nhưng tại các frame khác nhau (ByteTrack: frame 59, 94; ReID: frame 87, 113). Sự khác biệt không chỉ do ReID — ByteTrack và BoT-SORT là hai implementation khác nhau (association algorithm, buffer logic), nên không thể quy toàn bộ chênh lệch cho appearance cue.

Frame evidence: track gold 5 (frame 87) — ByteTrack mất track lúc xe bị che nhẹ, tạo ID mới; ReID giữ được identity nhờ appearance embedding. Track gold 7 (frame 163–169) — ByteTrack bắt thiếu hơn ReID tại đoạn xe nhỏ ở rìa.

**3. DetA, FP và FN đổi thế nào? Có phải lỗi còn lại là detector, hay là association?**

ByteTrack: DetA=0.649, FP=88, FN=54 — bỏ sót 54 bbox (xe bị che, xe ở rìa), FP do bbox thừa từ track không khớp gold.
ReID: DetA=0.711, FP=91, FN=26 — FN giảm mạnh (bắt được thêm 28 bbox so với ByteTrack), nhưng FP tăng nhẹ (91 vs 88) vì ReID giữ track lâu hơn ở các đoạn xe khó phát hiện, đôi khi giữ nhầm.

Lỗi còn lại chủ yếu là **detector**: FN=26 ở ReID vẫn lớn so với nhãn tay (FN=10). IDSW=2 ở cả hai model cho thấy association khá tốt; vấn đề là detector bỏ sót xe bị che hoặc điểm thấp hơn ngưỡng conf=0.25.

**4. Một chỗ bạn đúng và ReID sai; một chỗ ReID đúng mà bạn cần xem lại:**

*Bạn đúng, ReID sai*: frame 16–116, track ReID ID 7 (BoT-SORT) không khớp bất kỳ track gold nào — 43 frame bbox thừa. Đây là xe tĩnh (hoặc vật thể tĩnh) mà ReID nhận nhầm là xe. Nhãn tay không có track tương ứng vì đã nhận ra đây không phải xe bốn bánh di chuyển thực sự.

*ReID đúng, bạn cần xem lại*: frame 106–121, ReID có track ID 27 với 16 frame mà nhãn tay không có. Đây khả năng là một xe nhỏ xuất hiện ở vùng mật độ cao xung quanh frame 106–109. Cần mở CVAT và kiểm tra xem có xe thật hay không — nếu có thì nhãn tay đang thiếu track ở đây.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

Loại nhiều nhất khi so ReID vs bạn: **"chỉ model có" (FP=80) và "chỉ bạn có" (FN=81)** — hai bên bỏ sót những xe khác nhau. Model bỏ sót 81 bbox mà bạn có (xe bị che, xe đỗ, xe rìa khung mà bạn đã gán keyframe thủ công). Ngược lại, bạn bỏ sót 80 bbox mà model phát hiện, nhiều trong số đó là FP thực sự (vật thể tĩnh, track thừa).

Điều này phản ánh đặc điểm của clip_01: mật độ xe trung bình, nhiều đoạn occlusion ngắn, và một số xe nhỏ ở rìa khung. Nhãn tay tốt hơn ở identity (IDSW=0 vs 2) và biên track theo ngữ cảnh; model tốt hơn ở việc phát hiện xe nhỏ điểm thấp mà mắt người dễ bỏ qua trong lần gán đầu.

## 6. Nếu phải gán thêm 10 clip nữa

Sẽ sửa trong `GUIDELINE_MINI.md`:
- Thêm luật cứng về biên track: bấm `outside` đúng frame xe rời khung hoàn toàn — không để sai 1 frame vì nó tạo FP lan ra nhiều frame liên tiếp.
- Quy định khoảng cách keyframe tối đa là 10 frame ở đoạn xe di chuyển nhanh hoặc đổi hướng — giảm interpolation drift.
- Thêm ngưỡng bbox tối thiểu cụ thể (≥ 15px) thay vì để mỗi người tự quyết.
- Thêm ví dụ ảnh minh họa cho ca hai xe cắt nhau để người gán tiếp theo nhận ra nhanh hơn.

Đổi trong quy trình: sau khi gán xong mỗi track, replay đúng track đó (ẩn các track khác) để kiểm tra nhanh biên và bbox trước khi sang track tiếp theo — phát hiện bbox treo sớm hơn, không phải đến lúc chạy validator mới biết.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `GUIDELINE_MINI.md` đã điền đầy đủ
- [x] `evidence/pre-gold/clip_01/gt.txt` + `manifest.json`
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`
- [x] `outputs/eval_reid_vs_gold.json`
- [x] `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
