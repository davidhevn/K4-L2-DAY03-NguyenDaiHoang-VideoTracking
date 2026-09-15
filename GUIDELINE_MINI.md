# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Đại Hoàng`
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

Bổ sung của nhóm (nếu có): `Không gán bóng râm/vật thể cố định ven đường kể cả khi detector/ReID bắt nhầm (như ID 26 tại frame 105). Không gán các loại xe ba bánh hoặc phương tiện thô sơ.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Quãng ngắt ngắn vận tốc và hướng di chuyển có tính liên tục cao, giữ nguyên ID để tối ưu chỉ số AssA và IDF1. |
| Xe bị che lâu hơn ngưỡng trên | **Mở track mới** | Quá 2 giây (~25 frame), quỹ đạo di chuyển và đặc trưng hình ảnh không còn tính dự đoán chắc chắn, tránh rủi ro gộp nhầm 2 xe khác nhau. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Quy tắc một chiều của MOT 1.1: khi một phương tiện đã thoát hoàn toàn ra khỏi góc máy (exit frame) thì quỹ đạo trước đó kết thúc vĩnh viễn. |
| Hai xe cắt nhau / chồng lên nhau | **Duy trì độc lập hai ID riêng biệt**; xe ở trước giữ bbox bình thường, xe ở sau chỉ khoanh phần hở ra | Tránh lỗi ID Switch (nhảy nhầm ID) khi giao cắt đường đi. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `Nhìn rõ tối thiểu 20% thân xe hoặc thấy rõ cụm đèn/bánh xe trước.` |
| Xe đang đỗ, không di chuyển | Vẫn gán nhãn và duy trì track ID xuyên suốt toàn bộ thời gian xe nằm trong khung hình. |
| Keyframe đặt dày ở đâu | Đặt dày (3–5 frame/keyframe) khi xe bắt đầu rẽ, giảm tốc hoặc đi chéo; các đoạn thẳng đều duy trì 15–20 frame/keyframe. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 118 / ID 6`
- Tình huống: `Xe ô tô di chuyển theo đường chéo, khoảng cách giữa 2 keyframe quá xa khiến nội suy tuyến tính làm bbox bị phồng sang mặt đường, IoU tụt xuống 0.53.`
- Quyết định: `Chèn thêm một keyframe trung gian ngay tại frame 118 và kéo mép phải bbox ôm sát viền vỏ xe.`
- Lý do: `Bbox bắt buộc phải bám khít phần nhìn thấy được để đảm bảo tiêu chuẩn LocA và MOTP >= 0.70.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 140 / ID 2`
- Tình huống: `Xe chạy dưới trời nắng gắt tạo bóng đổ đen dài ở mặt đường (IoU chỉ đạt 0.51 do bbox ôm cả bóng).`
- Quyết định: `Nâng cạnh đáy của bounding box lên đúng vị trí tiếp xúc của lốp xe với mặt đường, loại bỏ hoàn toàn phần bóng râm.`
- Lý do: `Quy chuẩn schema chỉ đo lường thân xe thực tế, gán kèm bóng đổ sẽ làm phình kích thước và gây lệch tâm IoU.`

### Ca 3
- Clip / frame / ID: `clip_01 / frame 105 / ID 26 (từ model ReID)`
- Tình huống: `Model BoT-SORT + ReID sinh ra bbox ID 26 chỉ tồn tại trong 1 frame tại lề đường.`
- Quyết định: `Không tạo track theo model, giữ nguyên nhãn tay không gán.`
- Lý do: `Kiểm tra ảnh gốc tại frame 105 cho thấy đó chỉ là góc bốt kỹ thuật và bóng râm tĩnh ven đường bị detector nhận nhầm (False Positive).`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Quy định về Exit frame: Khi xe di chuyển ra mép ảnh, ngay tại frame cuối cùng còn nhìn thấy thân xe phải bấm phím tắt 'O' (Outside) để ngắt track ngay lập tức, không để lại box rỗng trôi dạt ở các frame sau.`
- `Quy định kiểm soát Mid-point: Với mọi track có khoảng cách giữa 2 keyframe lớn hơn 20 frame, bắt buộc phải kiểm tra frame chính giữa (mid-point check) để phát hiện hiện tượng trôi lệch hình học (interpolation drift).`
