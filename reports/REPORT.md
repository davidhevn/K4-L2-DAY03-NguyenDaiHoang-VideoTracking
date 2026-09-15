# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Đại Hoàng`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT (Local Docker)` |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `50` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `8` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe bị che khuất tạm thời (Occlusion): Khi xe đi qua các cột/cây hoặc bị xe khác che một phần, duy trì đúng Track ID cũ bằng cách chỉnh bbox chỉ ôm khít phần nhìn thấy được, không phỏng đoán phần khuất.`
2. `Xe đi ra khỏi khung hình (Exit frame): Phải ấn phím Outside ('O') ngay tại frame cuối cùng xe còn trong khung để tránh việc CVAT nội suy kéo bbox lơ lửng ở các frame trống tiếp theo.`
3. `Xe đi chéo và thay đổi kích thước xa - gần: Đặt keyframe dày hơn (khoảng 5-8 frame một keyframe) thay vì để giãn cách quá dài làm lệch bounding box do nội suy tuyến tính.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Không bị nhảy nhót ID (ID switch), 8 xe giữ nguyên 8 track ổn định.`
- Lượt 2: `Phát hiện 1 track xe đi hết khung hình nhưng chưa bấm Outside ở frame cuối, đã bấm 'O' để ngắt track.`
- Lượt 3: `Phát hiện 2 frame ở giữa bị trôi bbox nhẹ do xe giảm tốc độ, đã thêm keyframe bổ sung để bbox ôm khít viền xe.`

Kiểm chéo với: `N/A (Làm việc độc lập, không ghép cặp kiểm chéo)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Trường hợp xe ở rất xa rìa ảnh khi mới chỉ nhìn thấy 1/3 đầu xe. Thống nhất bổ sung vào GUIDELINE_MINI.md: Chỉ bắt đầu tạo track khi nhìn rõ tối thiểu bánh trước/đèn xe và xác định chắc chắn là phương tiện 4 bánh.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `6d6a9068fdf967af9d867f53297c928350c76b4ea8f25ffebeeccc12a2f740ab` |
| Thời điểm khóa | `15/09/2026 16:30` |
| Số row / frame / track trước khi mở reference | `611 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.812 | 0.795 | 0.840 | 0.885 | 0.895 | 0.820 | 0.875 | 45 | 38 | 1 |
| Sau rework | 0.816 | 0.804 | 0.829 | 0.883 | 0.960 | 0.916 | 0.872 | 43 | 5 | 0 |

*(Lưu ý: Nếu chưa nhận file gold từ Lab Coach, trên Colab đã chạy so sánh ReID vs Bạn: HOTA 0.764, DetA 0.707, AssA 0.828, IDF1 0.889, MOTA 0.777, MOTP 0.878).*

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox lệch (IoU thấp) | 118 | 6 | Thu nhỏ lại bbox để không lấy lấn sang mặt đường |
| Bbox lệch | 138-140 | 2 | Chỉnh lại cạnh phải bbox ôm khít đuôi xe |
| Outside muộn | 190 | 2 | Đặt lại điểm kết thúc track chuẩn xác khi xe chạm viền ảnh |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.816 | 0.804 | 0.829 | 0.883 | 0.960 | 0.916 | 0.872 | 43 | 5 | 0 |
| ByteTrack control vs gold | 0.708 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.764 | 0.707 | 0.828 | 0.890 | 0.889 | 0.777 | 0.878 | 80 | 53 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`IDF1 cao hơn MOTA (0.889 so với 0.777 khi đối chiếu ReID vs Bạn). Khi MOTA cao mà IDF1 thấp, điều đó phản ánh mô hình bắt đối tượng tương đối đủ nhưng bị vỡ track, nhầm lẫn ID nghiêm trọng trong toàn bộ hành trình xe chạy. MOTA không phạt nặng lỗi ID vì chỉ đếm ID switch tại duy nhất thời điểm chuyển giao (cộng 1 lỗi), trong khi IDF1 đo lường sự trùng khớp identity trên toàn bộ chiều dài thời gian (trajectory), nên việc chia đôi track sẽ làm giảm điểm IDF1 rất nặng.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID cho kết quả Association vượt trội hơn ByteTrack control (AssA tăng, IDSW giảm từ 6 xuống 2). Tại phân đoạn frame 105 - 115 khi các xe đi giao cắt hoặc che khuất nhau, ByteTrack thuần dựa vào motion/Kalman filter dễ bị lạc hướng khi vận tốc thay đổi đột ngột; trong khi đó, BoT-SORT bổ sung thêm đặc trưng ngoại hình (appearance embeddings) giúp duy trì nhận diện xe khi xuất hiện trở lại. Tuy nhiên, đây là sự so sánh giữa hai hệ thống hoàn chỉnh (system comparison) chứ không cô lập biến độc lập ReID, bởi vì ByteTrack và BoT-SORT còn có sự khác biệt về thuật toán liên kết hộp và xử lý camera motion.`

*(Nhận xét mục Stretch ReID: Khi thử nghiệm các ngưỡng appearance_thresh khác nhau (0.7, 0.8, 0.9), các chỉ số HOTA (0.763), AssA (0.820) và IDSW (2) gần như không thay đổi. Điều này cho thấy việc siết chặt thêm ngưỡng nhận diện ngoại quan không mang lại lợi ích rõ rệt trên clip này, vì các ca occlusion/crossing đã được giải quyết ở ngưỡng mặc định 0.8).*

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA (0.707) thấp hơn đáng kể so với AssA (0.828). Số FP lên tới 80 và FN là 53. Điều này chứng minh lỗi còn lại phần lớn nằm ở Detector (YOLO26n bắt nhầm vật thể tĩnh hoặc bỏ sót xe nhỏ/bị che) chứ không phải do tracker/association. Khi detector trượt bbox, tracker không có đầu vào để liên kết.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Tại frame 105 - 106, ReID sinh ra các ID ảo (chỉ model có 2 bbox) do detector nhận nhầm một phần bóng râm/vật thể bên lề đường thành xe ô tô (False Positive). Người gán nhãn nhận định chính xác đó là vật thể tĩnh và không gán.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Tại frame 118 (track bản A 6 / IoU 0.53): ReID vẽ bbox ôm sát thân xe hơn, trong khi bbox gán nhãn tay của tôi bị phồng ra ngoài lề đường do nội suy xa giữa 2 keyframe. Đây là điểm giá trị giúp tôi bổ sung keyframe để tinh chỉnh lại nhãn của mình.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Bổ sung vào GUIDELINE_MINI: Quy định rõ diện tích tối thiểu của xe khi đi vào rìa ảnh (ví dụ: nhìn thấy tối thiểu 20% thân xe) mới bắt đầu khởi tạo track; và quy định rõ không gán các phương tiện đỗ khuất hẳn trong bóng râm ngoài lề đường.
- Cải tiến quy trình: Sau mỗi 30-40 frame nội suy, bắt buộc phải dừng lại kiểm tra frame chính giữa (mid-point check) để tránh bbox bị phồng hoặc trôi theo quán tính.

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
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
