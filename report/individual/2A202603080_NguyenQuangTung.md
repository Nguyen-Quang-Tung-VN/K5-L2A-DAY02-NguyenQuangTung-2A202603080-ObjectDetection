# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Quang Tùng
- MSSV: 2A202603080
- Email: nguyenquangtung753@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/Nguyen-Quang-Tung-VN/K5-L2A-DAY02-NguyenQuangTung-2A202603080-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/Nguyen-Quang-Tung-VN/K5-L2A-DAY02-NguyenQuangTung-2A202603080-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb#scrollTo=3e0508d8

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt các ca mơ hồ tiêu biểu đã quyết định (chi tiết trong GUIDELINE_MINI.md):
1. `drive_027.jpg` (Xe bị cắt rìa ảnh): Xe buýt to sát góc ảnh -> Quyết định gán `bus` (ID=2) và vẽ box bám sát pixel lộ diện chạm mép ảnh [x=0, y=H], không đoán phần ngoài khung hình.
2. `drive_012.jpg` (Van vs Bus/Minibus): Xe thùng kín cỡ trung đậu ven đường -> Quyết định gán `van` (ID=3) do thân xe dạng hộp kín, không có hàng cửa sổ ngang liên tiếp đặc trưng của `bus`.
3. `drive_014.jpg` (Hai xe chồng chéo / Occlusion): `truck` và `bus` đỗ sát nhau che khuất nhau -> Quyết định vẽ 2 box riêng biệt cho `truck` (ID=1) và `bus` (ID=2), bao quanh phần nhìn thấy của từng xe, không gộp chung.
4. `drive_039.jpg` (Xe quá nhỏ ở hậu cảnh): Xe con ở xa tít chỉ có kích thước 17×8 px (148 px²) -> Quyết định bỏ qua không gán nhãn vì dưới ngưỡng nhận diện tin cậy (< 20×20 px), tránh tạo False Positive.
5. `drive_002.jpg` (Phân biệt Truck vs Bus cùng khung hình): Cùng có kích thước to lớn -> Phân loại rạch ròi `truck` (có khoang chở hàng riêng biệt) và `bus` (thân liền khối nhiều cửa sổ ngang hành khách).
6. `drive_024.jpg` (Bán tải pickup lớn): Thùng hàng hở phía sau nhưng cabin 4-5 chỗ giống SUV -> Quyết định gán `car` (ID=0) theo schema bán tải cabin nhỏ thuộc `car`.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` ĐẠT 50/50 file. Đã xử lý lỗi lồng thư mục xuất từ CVAT (`labels/train/labels/train/`) và loại bỏ các file thừa (`train.txt`, `data.yaml`).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_005.jpg`, `drive_012.jpg`, `drive_014.jpg`, `drive_024.jpg`, `drive_027.jpg`, `drive_031.jpg`, `drive_039.jpg`, `drive_048.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện một số box bị lỏng ở cạnh mép ảnh; xe ở rất xa bị mô hình auto-label detect nhầm thành car; xe bán tải bị gán nhầm sang truck. Đã điều chỉnh box bám sát pixel mép xe và chuẩn hóa theo schema.
- Commit khóa nhãn ban đầu: `5caef93`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 476 | 631 |
| FP | 354 | 42 |
| FN | 177 | 22 |
| Precision | 0.5735 | 0.9376 |
| Recall | 0.7289 | 0.9663 |
| F1 | 0.6419 | 0.9517 |
| mean_iou_of_matches | 0.8508 | 0.8761 |
| passed | False | True |

Dẫn tới file kết quả: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

**Giải trình Rework:**
- Trước sửa, F1 đạt 0.6419 chưa qua ngưỡng 0.80 do số lượng False Positives cao (354 box dư) vì mô hình gán cả xe quá nhỏ ở hậu cảnh (diện tích < 0.002) và 50 box bị nhầm class (chủ yếu giữa car vs truck, van vs bus).
- Sau khi nhận Gold set, tiến hành rà soát sửa nhãn: giữ nguyên tọa độ box đã vẽ để đảm bảo mean IoU đạt 0.8761 (tự nhiên, không copy đè), sửa đúng 50 box nhầm class ID, lọc bỏ các box rác ở xa và bổ sung các xe bị thiếu. Kết quả sau sửa đạt F1 = 0.9517 và Mean IoU = 0.8761, đạt mức Xuất sắc theo Rubric.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab (Tesla T4 GPU), Ultralytics 8.4.x, PyTorch CUDA.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định theo notebook (50 epochs, imgsz=640, model `yolo26n.pt`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.6300 (63.00%) |
| Recall | 0.6044 (60.44%) |
| mAP50 | 0.6415 (64.15%) |
| mAP50–95 | 0.4173 (41.73%) |

Hình ảnh và log đánh giá được giải nén từ `day2_colab_outputs.zip` và lưu tại:
- [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/): chứa 10 ảnh eval kèm bounding box dự đoán (ví dụ [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) minh họa cảnh giao thông mật độ cao và [drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg)).
- [assets/yolo26_runs/](assets/yolo26_runs/): chứa kết quả training `results.csv`, `confusion_matrix.png`, và các đồ thị huấn luyện.

**Trả lời bốn câu hỏi cuối notebook:**

1. *Lớp nào recall thấp nhất? Ảnh nào minh họa?*
   - Lớp `bus` và `truck` có recall thấp nhất do hiện tượng mất cân bằng dữ liệu nghiêm trọng (class imbalance). Trong 10 ảnh eval, `car` chiếm tới 94 mẫu, trong khi `truck` chỉ có 5 mẫu và `bus` chỉ có 2 mẫu. Minh họa tại ảnh [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), mô hình nhận diện tốt hầu hết xe con (`car`) nhưng gặp khó khăn khi phát hiện hoặc phân loại chính xác các xe lớn có góc nhìn chéo hoặc bị che khuất một phần.
2. *Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?*
   - Cần kiểm tra độ bám sát mép và kích thước biên của bounding box (box tightness). Chỉ số mAP50 chỉ đòi hỏi IoU đạt tối thiểu 0.50 nên box vẽ hơi rộng hoặc lỏng vẫn được tính là True Positive. Tuy nhiên, mAP50–95 tính trung bình trên các ngưỡng IoU từ 0.50 đến 0.95 (với bước nhảy 0.05); nếu box bị thừa lề đường, lấy thêm bầu trời hoặc lấn sang xe bên cạnh, IoU chỉ đạt 0.60–0.70 khiến mAP ở các ngưỡng cao sụt giảm nghiêm trọng.
3. *Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?*
   - Sự nhầm lẫn giữa `van` và `car` (minivan/SUV cỡ lớn) hoặc `van` và `bus` (xe khách 16 chỗ/minibus). Dữ kiện hỗ trợ là ma trận nhầm lẫn (`confusion_matrix.png`) trong `assets/yolo26_runs/vehicle_eval/`, cho thấy một số đối tượng `van` bị phân loại thành `car` do góc chụp từ xa không thấy cửa trượt hông hoặc hình dáng thùng vuông không lộ rõ, phản ánh sự cần thiết của guideline chi tiết hơn về góc nhìn và kích thước thân xe.
4. *Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?*
   - Không đưa 10 ảnh `eval` vào tập train để tránh hiện tượng rò rỉ dữ liệu (data leakage) và đánh giá thiên vị (overfitting), đảm bảo đánh giá khách quan khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy. Trong notebook, 10 ảnh này đóng vai trò tập validation giúp theo dõi các chỉ số đánh giá qua từng epoch và chọn ra checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Export CVAT | Giải nén tạo thư mục lồng `labels/train/labels/train/` và file thừa `train.txt` | Di chuyển 50 file `.txt` ra đúng thư mục cha và xóa file thừa | Chạy `check_yolo_labels.py` ĐẠT 50/50 |
| Chấm nhãn Gold | F1 ban đầu chỉ đạt 0.64 do nhiều FP xe ở xa và nhầm class | Rework: sửa 50 box nhầm class, lọc box rác và bù các box sót | F1 tăng lên 0.9517, Mean IoU 0.8761 đạt chuẩn Xuất sắc |
| Colab GPU | Cần clone tự động qua URL repo GitHub | Push nhãn trước khi chạy Colab và dán REPO_URL chính xác | Pipeline chạy hoàn chỉnh 50 epochs |

## 7. Tự đánh giá theo rubric

- Hoàn thành và định dạng YOLO (10đ): ĐẠT (50 file hợp lệ, đúng 5 cột).
- Đầy đủ vật thể (20đ): ĐẠT (đã gán đầy đủ, kể cả vật thể che khuất/cắt khung).
- Độ bám sát và hình học box (20đ): ĐẠT (Mean IoU đạt 0.8761).
- Phân lớp nhất quán (15đ): ĐẠT (tuân thủ đúng 4 lớp trong schema).
- Mini guideline (10đ): ĐẠT (đã nêu chi tiết quy tắc và 3 ca mơ hồ).
- Gold QC (15đ): ĐẠT (F1 = 0.9517 >= 0.80, có vòng rework giải thích rõ).
- Pipeline YOLO và phân tích (10đ): ĐẠT (Colab chạy đủ 50 epochs, có đầy đủ metrics, ảnh eval và trả lời chi tiết 4 câu hỏi phân tích).

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng trợ lý AI để hỗ trợ kiểm tra định dạng YOLO, phân tích ma trận lỗi FP/FN giữa nhãn học viên và tập Gold, và gợi ý cấu trúc giải trình rework.
- Toàn bộ nhãn và phân tích đã được tự kiểm tra và xác nhận thủ công.
