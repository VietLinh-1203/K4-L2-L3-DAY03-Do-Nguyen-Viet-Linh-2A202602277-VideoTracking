# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Đỗ Nguyễn Việt Linh`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Vẫn là cùng một xe, tránh ID switch và phân mảnh track.` |
| Xe bị che lâu hơn ngưỡng trên | `Tạo track mới / ID mới khi xe xuất hiện lại` | `Sau thời gian dài không đủ chắc chắn để nối đúng danh tính.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Xe đã biến mất khỏi vùng quan sát, không giả định xe quay lại là cùng object.` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ ID riêng của từng xe, theo vị trí trước–sau giao cắt, hướng chuyển động và đặc điểm xe, không đổi ID chỉ vì box chồng nhau.` | `Tránh gán nhầm hai xe và phát sinh ID switch.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `nhìn rõ thân xe và bbox rộng tối thiểu khoảng 20 px.` |
| Xe đang đỗ, không di chuyển | `Vẫn track nếu xe thuộc phạm vi cần gán nhãn, giữ một ID từ frame đầu đến frame cuối còn nhìn thấy.` |
| Keyframe đặt dày ở đâu | `Đặt dày tại lúc xe xuất hiện/rời ảnh, đổi tốc độ/hướng, thay đổi kích thước mạnh, bị che hoặc box bắt đầu trôi.` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 54 / ID 6`
- Tình huống: `Xe buýt mới xuất hiện ở rìa phải ảnh, chỉ thấy một phần nên dễ tạo bbox quá sớm.`
- Quyết định: `Bắt đầu track tại frame 54, không giữ bbox ở frame 51–53.`
- Lý do: `Frame 54 là mốc xe buýt xuất hiện theo ground truth.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 83–84 / ID 7  `
- Tình huống: `Bbox của xe bị trôi giữa các keyframe, IoU chỉ còn khoảng 0.52–0.56.`
- Quyết định: `Thêm keyframe ở frame 83 và 84, chỉnh bbox ôm sát phần xe nhìn thấy.`
- Lý do: `Xe di chuyển làm nội suy tuyến tính không còn đủ chính xác; thêm keyframe giúp tăng độ khít bbox.`

### Ca 3
- Clip / frame / ID: `clip_01 / frame 111–112 / ID 8`
- Tình huống: `Bbox xe bị lệch rõ ở giữa hai keyframe, IoU thấp khoảng 0.51–0.56.`
- Quyết định: `Thêm keyframe ở frame 111 và 112, giữ nguyên ID 8.`
- Lý do: `Đây vẫn là cùng một xe, chỉ cần hiệu chỉnh vị trí bbox, tạo ID mới sẽ làm tăng nguy cơ ID switch.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Khi bbox bị lệch giữa các keyframe, thêm keyframe để chỉnh bbox, không tạo ID mới nếu vẫn là cùng một xe.`
- `Khi xe bị che hoặc xuất hiện lại, cần dựa vào thời gian bị che và đặc điểm xe để quyết định giữ ID hay tạo ID mới.`
