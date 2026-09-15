# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Hoàng Văn Đạt — 2A202602267 |
| Reviewer | Hoàng Văn Đạt — 2A202602267 |
| Pair ID | clip_01 |
| CVAT version | app.cvat.ai v2 |
| Thời điểm review | 15/09/2026 |

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 55 | 55 | 5 | Bbox treo / bắt đầu sớm | ID 5 có bbox từ frame 54 nhưng gold track 5 chỉ bắt đầu từ frame 79. Bbox treo 25 frame trước khi xe xuất hiện. Rule: bấm `outside` đúng frame xe rời khung. | Trong CVAT: tìm frame đầu tiên ID 5 thực sự thấy xe, xóa toàn bộ keyframe trước đó hoặc bấm `outside` ở frame 53. | fixed — sửa trong guideline: biên track phải khớp frame xe xuất hiện/rời khung |
| 2 | 79 | 79 | 6 | Bbox treo / bắt đầu sớm | ID 6 có bbox từ frame 78 nhưng gold track 6 bắt đầu từ frame 101. Bbox treo 23 frame. | Tương tự finding #1: bấm `outside` ở frame 77, track bắt đầu lại từ frame 101. | fixed — cùng root cause với finding #1 |
| 3 | 150 | 150 | 4 | Bbox treo / kết thúc muộn | ID 4 còn bbox ở frame 149–151 nhưng gold track 4 kết thúc frame 148. 3 frame thừa. | Bấm `outside` ở frame 148. | fixed — bổ sung luật vào guideline mục 3 |
| 4 | 84 | 84 | 5 | Bbox trôi / interpolation drift | IoU tụt xuống 0.51–0.59 ở frame 83–88, track 5 đang chuyển hướng. Không có keyframe trong đoạn này. | Thêm keyframe ở frame 85 và 87 để interpolation bám sát xe. | fixed — bổ sung luật keyframe dày hơn vào guideline |
| 5 | 169 | 169 | 8 | Bbox treo / kết thúc muộn | ID 8 còn bbox ở frame 169–171 nhưng gold track 8 kết thúc frame 168. 3 frame thừa. | Bấm `outside` ở frame 168. | fixed |
| 6 | 104 | 104 | 7 | Bbox treo / bắt đầu sớm | ID 7 có bbox từ frame 103 nhưng gold track 7 bắt đầu từ frame 106. 3 frame thừa. | Xóa keyframe ở frame 103–105 hoặc bấm `outside` ở frame 102. | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, tất cả xe bốn bánh, ID [1–8] |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0 (eval_vs_gold), không có ID nhảy |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 4 (frame 55–67) giữ ID qua occlusion 8 frame; track 5&6 không đổi ID lúc chồng |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Finding #1–3, #5–6: 6 trường hợp bbox treo hoặc bắt đầu sớm |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Không phát hiện bbox đoán ra ngoài rìa ảnh |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Finding #4: track 5 frame 83–88 IoU tụt 0.51–0.59 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py: 0 lỗi định dạng |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Tất cả 6 finding đã có closure |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | ĐÃ SỬA | 8 track, IDSW=0; biên track 6 chỗ cần sửa (finding #1–3, #5–6) |
| 2 — endpoint/scope | ĐÃ SỬA | Phát hiện 6 bbox treo; bổ sung luật biên track vào GUIDELINE_MINI.md |
| 3 — geometry/interpolation | ĐÃ SỬA | Track 5 frame 83–88 drift; bổ sung luật keyframe dày hơn vào GUIDELINE_MINI.md |

## Exit ticket

1. **Finding quan trọng nhất**: Finding #1 — ID 5 có 25 frame bbox treo trước khi xe xuất hiện. Rule áp dụng: *bấm `outside` đúng frame xe rời khung, không để bbox tồn tại trước khi xe thực sự xuất hiện*. Đây là nguồn gốc chính của FP=76 trong eval_vs_gold.
2. **Finding đóng là `not-a-defect`**: Track 3 clip_01 frame 1–15 đứng im — check_mot_labels cảnh báo nhưng xe thực sự đỗ, không phải lỗi annotation. Rule: xe đỗ vẫn cần track đầy đủ.
3. **Rule cần Lab Coach làm rõ**: Khi xe bị che > 25 frame nhưng vẫn nhìn thấy một phần nhỏ liên tục (không mất hoàn toàn), có nên giữ ID hay tạo track mới? Guideline hiện chỉ nói "mất hoàn toàn" mới tạo mới, chưa rõ với trường hợp thấy < 20% diện tích.
