# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

**Họ tên:** Nguyễn Quang Tùng  
**MSSV:** 2A202603080  
**Ngày:** 09/11/2026

---

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ / điểm phân biệt |
| --- | --- | --- |
| `car` (ID=0) | Xe cơ giới chở người cá nhân cỡ nhỏ, tối đa ~8 chỗ, thân thấp. Bao gồm sedan, hatchback, SUV, coupe, taxi, xe bán tải cabin nhỏ (pickup). | **Gán:** sedan, SUV, taxi, xe bán tải nhỏ. **Không gán:** xe buýt, xe tải thùng lớn, van chở hàng, mô tô. |
| `truck` (ID=1) | Xe cơ giới chở hàng hóa cỡ lớn, có thùng hoặc container rõ ràng phía sau, bao gồm xe đầu kéo (semi-truck). | **Gán:** xe tải thùng, xe container, xe đầu kéo. **Không gán:** xe bán tải nhỏ/pickup thuộc `car`, xe van chở hàng thuộc `van`. |
| `bus` (ID=2) | Xe chở khách công cộng hoặc liên tỉnh, thân dài và cao, nhiều cửa sổ hàng ngang liên tiếp. Bao gồm minibus chở khách đông người. | **Gán:** xe buýt thành phố, xe buýt liên tỉnh, minibus chở khách. **Không gán:** van chở người thuộc `van`, xe khách nhỏ ≤15 chỗ dạng thùng kín. |
| `van` (ID=3) | Xe thùng kín cỡ trung, dùng chở hàng hoặc chở người nhóm nhỏ, thân hộp đặc trưng. Bao gồm minivan và xe tải nhỏ kiểu van. | **Gán:** xe van chở hàng, minivan, xe chở người nhóm nhỏ dạng thùng kín. **Không gán:** xe buýt lớn nhiều hàng cửa sổ (`bus`), xe con thông thường (`car`). |

---

## Quy tắc chung

- **Box bám sát vật thể** nghĩa là: vẽ sát phần thân xe nhìn thấy được — không lấy thêm mặt đường phía dưới, bầu trời phía trên, hoặc phần thân xe bên cạnh. Cạnh box trùng hoặc sát pixel ngoài cùng của xe.
- **Vật thể bị che khuất:** vẽ box quanh phần nhìn thấy được nếu đủ để xác định lớp (nhìn thấy ít nhất ~30% thân xe và nhận diện được loại). Không đoán phần bị che; box chỉ bao phần lộ ra.
- **Vật thể bị cắt bởi rìa ảnh:** vẽ box đến đúng rìa ảnh — cạnh box trùng mép ảnh, không đoán phần nằm ngoài khung hình.
- **Vật thể rất nhỏ/mờ hồ:** nếu không xác định rõ lớp do kích thước quá nhỏ hoặc bị mờ — **bỏ qua, không gán**. Nếu kích thước đủ nhưng lớp mơ hồ (ví dụ van vs bus), ghi vào bảng quyết định dưới và hỏi giảng viên; tạm thời chọn lớp có khả năng cao nhất và đánh dấu để review.
- **Phản chiếu/bóng/biển quảng cáo có hình xe:** không gán. Chỉ gán xe thật hiện diện trong cảnh.
- **Xe đang đỗ:** gán bình thường theo loại xe, giống xe đang chạy.
- **Nhiều xe chồng lên nhau:** mỗi xe một box riêng, bao phần nhìn thấy của từng xe.

---

## Ghi lại quyết định cho các ca mơ hồ tiêu biểu (Rút ra từ ảnh thực tế)

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014.jpg` | Ở góc dưới màn hình (`y2 = 640`), xe con màu bạc đi sát phía trước camera chỉ lộ phần nắp cốp sau/mui xe, còn cản dưới và bánh xe bị cắt khỏi mép đáy ảnh. | Gán `car` (ID=0), vẽ box chạm mép đáy ảnh (`y2 = 1.0`). | Vật thể thuộc phạm vi camera góc rộng, phần nhìn thấy có đầy đủ đặc trưng đèn hậu và kính sau xe con (`car`). Box kéo sát đến pixel mép đáy của ảnh, không suy đoán phần gầm xe ngoài ảnh. |
| `drive_024.jpg` | Ở ngã tư phía xa có nhiều xe con chạy/đỗ ở làn đối diện với kích thước rất nhỏ (chỉ khoảng 26×26 pixel = 676 px²), độ phân giải thấp. | Gán `car` (ID=0) cho các xe thấy rõ hình dáng; bỏ qua các vệt mờ dưới 15px. | Vật thể kích thước nhỏ nhưng vẫn nhận diện được kết cấu mui xe và đèn xe; vẽ box vừa khít từng cụm pixel của xe, tránh để box quá rộng gây giảm độ bám sát (IoU). |
| `drive_002.jpg` | Ở sát mép phải ảnh, có một xe tải chở hàng (`truck`) lưu thông ở làn ngoài cùng, bị mép phải khung hình cắt ngang qua thân xe (`x2 = 640`), chỉ thấy phần thùng hàng và bánh sau bên trái. | Gán `truck` (ID=1), vẽ box chạm sát mép phải ảnh (`x2 = 1.0`). | Quy tắc vật thể bị cắt bởi rìa ảnh: chỉ vẽ box bao phần xuất hiện trong khung hình, không vẽ box suy đoán phần cabin đầu xe nằm ngoài ảnh. Thùng xe chở hàng lớn và bánh sau đủ để định danh chắc chắn là `truck`. |
| `drive_046.jpg` | Xe buýt lớn màu xanh/trắng (`bus`, box `[288, 307, 474, 477]`) chạy ngay chính giữa làn đường phía trước camera, thân xe to lớn chiếm trọn làn và có 2 gương chiếu hậu nhô sang hai bên. | Gán `bus` (ID=2), vẽ box ôm sát thân vỏ chính của xe buýt. | Quy tắc độ bám sát (tightness): bounding box phải ôm khít phần thân xe chính, không cố kéo rộng box sang hai bên để lấy tai gương chiếu hậu quá mỏng vì sẽ tạo khoảng trống thừa (background noise), làm giảm IoU. |
| `drive_001.jpg` | Trên đường cao tốc, phía bên trái có dải phân cách cứng bê tông. Phía bên kia dải phân cách có các xe đi ngược chiều ở làn đối diện bị dải phân cách che một phần bánh xe. | **Không gán nhãn** các xe ở làn đối diện bên kia dải phân cách cứng; chỉ gán các xe cùng chiều (5 xe). | Theo mục tiêu hệ thống ADAS (hỗ trợ người lái cảnh báo va chạm trên luồng di chuyển), các phương tiện ở chiều ngược lại đã được ngăn cách hoàn toàn bằng rào chắn cứng được loại bỏ để tránh gây nhiễu cho mô hình. |
