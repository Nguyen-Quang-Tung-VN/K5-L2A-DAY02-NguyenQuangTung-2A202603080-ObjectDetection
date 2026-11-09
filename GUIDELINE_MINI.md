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

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_012.jpg` | Xe thùng kín cỡ trung đậu ven đường, không rõ là van chở hàng hay minibus chở khách vì góc chụp không thấy cửa hông. | Gán `van` (ID=3) | Thân xe dạng hộp kín, kích thước trung bình, không thấy hàng cửa sổ ngang liên tiếp đặc trưng của `bus`. Theo schema: `van` khi hình dạng thùng kín, không phải `bus` lớn nhiều cửa sổ. |
| `drive_024.jpg` | Xe bán tải (pickup) lớn, thùng sau lộ rõ nhưng cabin và tổng thể gần giống SUV cỡ lớn. | Gán `car` (ID=0) | Schema quy định rõ: xe bán tải/cabin nhỏ thuộc `car`, không phải `truck`. `Truck` là xe tải chở hàng thùng lớn hoặc container. Pickup không có thùng container → `car`. |
| `drive_031.jpg` | Phần đầu xe tải (đầu kéo) ló ra sau xe buýt, chỉ thấy khoảng 25–30% thân xe — phần cabin và một phần mui. | Gán `truck` (ID=1) | Phần nhìn thấy đủ để nhận diện: cabin xe đầu kéo rõ ràng, hình dạng đặc trưng (cao, vuông). Box bao sát phần lộ ra, cạnh chạm vào xe buýt che khuất. Quy tắc: bị che một phần vẫn gán nếu nhận diện được lớp. |
