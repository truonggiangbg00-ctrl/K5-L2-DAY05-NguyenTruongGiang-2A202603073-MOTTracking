# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Trường Giang
- MSSV: 2A202603073
- Email: truonggiangbg00@gmail.com
- Lớp: L2A
- Link repo bài làm: https://github.com/truonggiangbg00-ctrl/K5-L2A-DAY02-NguyenTruongGiang-2A202603073-ObjectDetection.git
- Link Colab đã chạy: https://colab.research.google.com/drive/16Bzq8f5pcBRRJqaSCaVfwxvJugsBFoCd?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 
- **Các ca mơ hồ đã quyết định:** Phân biệt `van` và `car` ở khoảng cách xa, xử lý xe bị che khuất (occlusion) trên 50%, và phân loại xe tải nhỏ (light truck) với xe tải lớn. 
- **Cách áp dụng schema:** Quy định vẽ bounding box khít với phần diện tích nhìn thấy của xe, bỏ qua các xe ở quá xa, mờ hoặc bị che khuất gần hết.
- **Phần cần giảng viên làm rõ:** Mức độ che khuất (occlusion) tối đa để quyết định bỏ qua không gán nhãn.

## 3. Annotation và self-QC

- Số file nhãn train: 50
- Kết quả checker và lỗi đã xử lý: Đã kiểm tra format YOLO (x_center, y_center, width, height) và khắc phục các box bị out of bounds hoặc lỗi label thừa.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg` đến `drive_010.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Sửa lại độ khít của một số bounding box bị vẽ quá rộng so với vật thể thực tế, bổ sung nhãn cho một số xe ở xa bị sót.
- Commit khóa nhãn ban đầu: `22b4d20`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | *tự điền từ json* | *tự điền từ json* |
| FP | *tự điền từ json* | *tự điền từ json* |
| FN | *tự điền từ json* | *tự điền từ json* |
| Precision | *tự điền từ json* | *tự điền từ json* |
| Recall | *tự điền từ json* | *tự điền từ json* |
| F1 | *tự điền từ json* | *tự điền từ json* |
| mean_iou_of_matches | *tự điền từ json* | *tự điền từ json* |
| passed | *tự điền từ json* | *tự điền từ json* |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Dữ liệu được giữ nguyên sau khi kiểm tra self-QC. Chưa thực hiện rework.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: CPU (AMD EPYC 7B12), Ultralytics 8.4.208, PyTorch 2.14.1+cpu.
- Các tham số thay đổi so với notebook nếu có và lý do: Không thay đổi. Giữ nguyên theo notebook (model `yolo26n.pt`, `epochs=50`, `imgsz=640`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.464 |
| Recall | 0.235 |
| mAP50 | 0.280 |
| mAP50–95 | 0.178 |

*Thư mục ảnh:* Đã trích xuất trong 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/`. Ví dụ minh họa: Trong ảnh `drive_052.jpg`, mô hình đã nhận diện class `car` và `bus` nhưng vẫn còn sót một số vật thể do Recall tổng thể thấp.

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   Lớp `bus` có recall thấp nhất (đạt 0.00 trên 2 instances). Có thể quan sát ở ảnh `drive_052.jpg` hoặc `drive_053.jpg` (ảnh chứa xe bus thực tế nhưng model không khoanh được box nào hoặc đoán sai).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   Cần kiểm tra độ khít (tightness) của bounding box. mAP50 chỉ yêu cầu độ chồng chập (IoU) 50%, nhưng mAP50-95 yêu cầu IoU từ 50% đến 95%. Điều này nghĩa là mô hình đoán trúng vị trí có vật thể, nhưng tọa độ khung viền vẽ ra lại quá rộng hoặc quá hẹp, chưa bám sát sát viền xe.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   Lớp `van` (Recall 0.083, Precision 0.204) và `truck` (Recall 0.200). Các ranh giới định nghĩa về hình dáng giữa xe tải nhẹ, xe bán tải, hoặc minivan thường mơ hồ. Dữ kiện hỗ trợ: Lớp `van` có số lượng 12 instances nhưng mô hình dự đoán rất kém do dễ nhầm với class `car` nếu góc chụp từ xa hoặc một nửa bị che khuất.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   Không dùng để train nhằm tránh hiện tượng rò rỉ dữ liệu (Data Leakage). Tập eval đóng vai trò là một tập kiểm thử độc lập (unseen data) giúp đánh giá khách quan khả năng tổng quát hóa của mô hình sau mỗi epoch, từ đó chọn ra checkpoint có hiệu năng thực tế tốt nhất (`best.pt`) mà không bị overfitting.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Train YOLO trên Colab | Chạy mất nhiều thời gian do sử dụng môi trường CPU | Chạy mặc định 50 epochs để khảo sát baseline | Đã chạy xong nhưng mAP50-95 và Recall tổng thể còn khá thấp (0.178) |
| Annotation | Phân vân class khi xe ở xa bị nhòe hoặc che khuất | Phóng to và đối chiếu bối cảnh | Vẫn có thể sai sót ở class `van` do thiếu quy định chi tiết |

## 7. Tự đánh giá theo rubric

- Đã hoàn thiện toàn bộ bước gán nhãn cho 50 ảnh train.
- Chạy thành công quy trình train trên Google Colab với Ultralytics YOLO26n.
- Trích xuất thành công file log, thông số metrics đánh giá (Precision, Recall, mAP).
- Trả lời đầy đủ 4 câu hỏi phân tích từ notebook dựa trên chỉ số thực tế.
- *(Cần bổ sung)*: Chạy script QC nội bộ để điền cụ thể các chỉ số TP/FP/FN cho mục 4.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng Gemini LLM để đọc, trích xuất dữ liệu log đào tạo tự động từ Google Colab và hỗ trợ định dạng khung báo cáo Markdown.
- Không sử dụng AI/Auto-labeling cho quy trình gán nhãn.

# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con, xe du lịch cá nhân (thường từ 4-7 chỗ). | Xe sedan, hatchback, SUV. Gầm thấp, mui xe bo viền, không có thùng chở hàng. |
| `truck` | Xe tải chở hàng trọng tải từ nhẹ đến nặng. | Phía sau buồng lái có thùng chở hàng rõ ràng (thùng kín, thùng bạt hoặc thùng hở), bánh xe thường lớn. Bao gồm cả xe bán tải (pickup). |
| `bus` | Xe buýt nội đô hoặc xe khách đường dài chở nhiều người. | Kích thước dài, dáng hình hộp chữ nhật, có nhiều cửa sổ kính sát nhau dọc theo thân xe. |
| `van` | Xe tải nhỏ (minivan), xe chở khách nhỏ (khoảng 16 chỗ) hoặc xe chở hàng cỡ nhỏ. | To và vuông vức hơn `car` nhưng nhỏ và không dài như `bus`. Thường có cửa lùa bên hông, cốp sau phẳng đứng. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Kéo các cạnh của bounding box (trên, dưới, trái, phải) chạm sát mép ngoài cùng của phần thân xe nhìn thấy được. Khung **không** bao lấy phần bóng râm (shadow) in trên mặt đường, ăng-ten quá nhỏ hoặc luồng khí thải.
- **Vật thể bị che khuất (occlusion):** Vẽ bounding box bao lấy **đúng phần diện tích xe còn nhìn thấy được** (không đoán định và vẽ lấn sang phần vật cản). Nếu xe bị che khuất quá 50% hoặc phần bị che lấp mất các đặc trưng phân biệt loại xe, **bỏ qua không gán nhãn**.
- **Vật thể bị cắt bởi rìa ảnh (truncation):** Bounding box dừng lại đúng ở sát mép viền của bức ảnh. Nếu phần xe nằm trong ảnh còn quá ít (không đủ nhận diện là xe gì), bỏ qua không gán.
- **Vật thể rất nhỏ/mơ hồ:** Các xe ở khoảng cách quá xa (gần đường chân trời), bị mờ nhòe do chuyển động hoặc kích thước tổng thể quá nhỏ (ví dụ: chiều ngang/dọc < 15 pixel), **không gán nhãn** để tránh tạo nhiễu (noise) cho mô hình (đánh dấu là background).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_015.jpg` | Xe SUV cỡ lớn (có thiết kế khá to và vuông vức phía đuôi) dễ nhầm với xe `van`. | Gán `car` | Vẫn thuộc dòng xe du lịch cá nhân (<9 chỗ), cấu trúc khung gầm và kính hậu mang đặc trưng của SUV/car chứ không chuyên dụng chở hàng/nhiều khách như van. |
| `drive_032.jpg` | Xe bán tải (pickup truck) có khoang chở người phía trước và thùng hở phía sau. | Gán `truck` | Tuy kích thước chỉ bằng SUV cỡ lớn nhưng có cấu trúc thùng chở hàng tách biệt, chức năng khớp với nhóm xe tải chuyên dụng theo định nghĩa của lớp truck. |
| `drive_052.jpg` | Một phần đầu của chiếc xe con lấp ló phía sau hông của một chiếc `bus`, bị che khuất khoảng 60-70%. | Bỏ qua (không gán nhãn) | Diện tích bị che khuất vượt mức cho phép (>50%), hình ảnh còn lại không đủ đặc trưng để máy học nhận diện chính xác, rủi ro làm nhiễu tập train. |