# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

**Họ tên:** Nguyễn Quang Tùng  
**MSSV:** 2A202603080  
**Ngày:** 09/11/2026

---

## 1. Định nghĩa lớp

| Lớp | ID | Định nghĩa chuẩn | Ví dụ / Điểm phân biệt |
| :--- | :---: | :--- | :--- |
| `car` | 0 | Xe cơ giới chở người cá nhân cỡ nhỏ, tối đa ~8 chỗ, thân thấp. Bao gồm sedan, hatchback, SUV, crossover, coupe, taxi và xe bán tải (pickup). | **Gán:** sedan, SUV, taxi, xe bán tải nhỏ/pickup.<br>**Không gán:** xe buýt, xe tải thùng lớn, van chở hàng, mô tô/xe gắn máy. |
| `truck` | 1 | Xe cơ giới chở hàng hóa cỡ trung và lớn, có thùng chở hàng, container hoặc rơ-moóc rõ ràng phía sau, bao gồm xe đầu kéo (semi-truck). | **Gán:** xe tải thùng, xe tải ben, xe container, xe đầu kéo.<br>**Không gán:** xe bán tải nhỏ/pickup thuộc `car`, xe van chở hàng thùng liền thuộc `van`. |
| `bus` | 2 | Xe chở khách công cộng hoặc du lịch liên tỉnh, thân xe dài và cao, có nhiều hàng cửa sổ ngang liên tiếp dành cho hành khách. | **Gán:** xe buýt thành phố, xe khách du lịch, minibus chở khách đông người.<br>**Không gán:** xe van chở người gia đình thuộc `van`, xe khách nhỏ ≤15 chỗ dạng thùng kín. |
| `van` | 3 | Xe thùng kín cỡ trung, thân hộp đặc trưng với vách đứng, dùng chở hàng bưu chính hoặc chở nhóm người (minivan/panel van). | **Gán:** xe van chở hàng, minivan, xe cứu thương cỡ nhỏ.<br>**Không gán:** xe buýt lớn nhiều hàng cửa sổ (`bus`), xe con thông thường/SUV (`car`). |

---

## 2. Quy tắc chung

- **Box bám sát vật thể (Tightness):** Vẽ sát pixel ngoài cùng của phần thân xe nhìn thấy được. Tuyệt đối không lấy thêm mặt đường phía dưới, bầu trời phía trên hoặc lấn sang xe bên cạnh.
- **Vật thể bị che khuất (Occlusion):** Nếu phần nhìn thấy được $\ge 30\%$ thân xe và đủ đặc trưng để nhận diện lớp, vẽ box bao quanh **phần nhìn thấy được**, không suy đoán phần bị che khuất. Nếu bị che khuất quá nặng ($< 30\%$) không thể xác định lớp $\rightarrow$ bỏ qua.
- **Vật thể bị cắt bởi rìa ảnh (Truncation):** Vẽ box đến sát đúng mép ảnh (cạnh box trùng rìa ảnh $x=0$, $x=W$, $y=0$ hoặc $y=H$). Không suy đoán phần thân xe nằm ngoài khung hình.
- **Ngưỡng kích thước xe nhỏ ở hậu cảnh:** Xe ở quá xa có diện tích quá nhỏ ($< 20 \times 20\text{ px}$ hoặc diện tích chuẩn hóa $< 0.002$) bị mờ và không thể nhận diện đặc trưng lớp chắc chắn $\rightarrow$ **bỏ qua không gán nhãn** để tránh tạo False Positives gây nhiễu cho mô hình.
- **Vật thể không phải xe thật:** Hình ảnh phản chiếu qua kính, bóng đổ trên mặt đường, biển quảng cáo có hình xe $\rightarrow$ **không gán**.
- **Xe đang đỗ / dừng đỗ:** Gán bình thường theo đúng chủng loại xe như xe đang di chuyển.
- **Nhiều xe xếp hàng hoặc chồng lấn:** Mỗi xe phải có một box riêng biệt, bao trọn phần lộ diện của xe đó; tuyệt đối không gộp nhiều xe vào một bounding box.

---

## 3. Ghi lại quyết định cho các ca mơ hồ tiêu biểu (Rút ra từ ảnh thực tế)

| Ảnh | Tình huống mơ hồ | Quyết định cuối | Lý do theo schema |
| :--- | :--- | :--- | :--- |
| `drive_027.jpg` | **Xe bị cắt bởi rìa ảnh:** Xe buýt lớn xuất hiện sát góc dưới bên trái, bị cắt mép trái và mép dưới khung hình. | Gán `bus` (ID=2), vẽ box chạm sát mép ảnh `[x=0, y=H]`. | Cạnh box bám sát pixel ngoài cùng còn nhìn thấy được; cạnh trái trùng mép trái và cạnh dưới chạm mép dưới ảnh. Tuyệt đối không tự suy đoán phần thân xe nằm ngoài khung hình. |
| `drive_012.jpg` | **Van vs Bus/Minibus:** Xe thùng kín cỡ trung đậu ven đường, không rõ là xe van chở hàng hay minibus chở khách vì góc chụp từ phía đuôi xe. | Gán `van` (ID=3) | Thân xe dạng khối hộp kín, tỷ lệ dài/cao trung bình, không có dãy cửa sổ hành khách ngang liên tiếp đặc trưng của `bus`. Theo schema: thân hộp kín không phải bus lớn thì thuộc `van`. |
| `drive_014.jpg` | **Hai xe chồng chéo / che khuất (Occlusion):** Xe tải chở hàng (`truck`) và xe buýt (`bus`) đỗ rất sát nhau, phần thân che khuất nhau đáng kể. | Gán 2 box riêng biệt cho `truck` (ID=1) và `bus` (ID=2). | Cả hai xe đều lộ trên 30% thân xe và nhận diện rõ lớp. Vẽ hai box riêng bám sát phần nhìn thấy của từng xe, không gộp và không vẽ lấn sang nhau. |
| `drive_039.jpg` | **Xe quá nhỏ ở hậu cảnh (Kích thước biên):** Xe con ở rất xa tít chân trời, diện tích box chỉ khoảng 17×8 px (148 px²), mờ và mất chi tiết nhận dạng. | **Bỏ qua, không gán nhãn.** | Kích thước dưới ngưỡng nhận diện tin cậy (< 20×20 px), không thể xác định rõ chi tiết phân loại và dễ gây nhiễu False Positive cho detector YOLO. |
| `drive_002.jpg` | **Truck vs Bus cùng xuất hiện trong một cảnh:** Cùng có kích thước to lớn và góc nhìn chéo tương đồng. | Phân loại rạch ròi `truck` (ID=1) và `bus` (ID=2). | Nhận diện dựa trên cấu trúc chức năng: `truck` có cabin rời và khoang/thùng chở hàng riêng biệt phía sau; `bus` có thân liền khối với nhiều hàng cửa sổ kính dành cho hành khách. |
| `drive_024.jpg` | **Xe bán tải (Pickup) cỡ lớn:** Thùng hàng hở phía sau nhưng cabin 4-5 chỗ giống SUV. | Gán `car` (ID=0) | Schema quy định rõ: xe bán tải/cabin nhỏ thuộc `car`, không phải `truck`. `Truck` chỉ dành cho xe tải chở hàng lớn, xe container hoặc đầu kéo. |
