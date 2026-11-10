# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Huỳnh Nhật Quang
- MSSV: 2A202603113
- Email: 26ai.quanghn@vinuni.edu.vn
- Lớp: L2A
- Link repo bài làm: https://github.com/quanghunh/K5-L2A-DAY02-HuynhNhatQuang-2A202603113-ObjectDetection-.git
- Link Colab đã chạy: https://colab.research.google.com/drive/1EGKXPiMjJc4qVpoHGj3hc10KDjql3oH6#scrollTo=3e0508d8

## 2. Schema và mini guideline

Chi tiết: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file `.txt` (`drive_001.txt`–`drive_050.txt`), xuất Ultralytics detector YOLO 1.0 từ task CVAT `Lab-Day02`.
  
- **Kết quả checker và lỗi đã xử lý:**
  - `python3 tools/check_yolo_labels.py` cho ra: **"ĐẠT cấu trúc: 50/50 file nhãn hợp lệ"**. Checker không báo lỗi định dạng nào cần sửa.
  - Checker kiểm theo cấu trúc (5 cột, toạ độ normalized, class ID trong khoảng 0–3).
- **Danh sách ít nhất 10 ảnh đã rà bằng mắt:** 
`drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_0010`và ba ảnh `drive_001`, `drive_002`, `drive_009` đã được xem kèm box khi viết guideline.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**

  - Xe bị bỏ sót trong lần đầu gán và đã được thêm lần hai (24 box).
  - Box gán sai đã được xoá (8 box).
  - Nhãn đuọc thay đổi sau khi cân nhắc từ `car` sang `van`(3 box)
- **Commit khóa nhãn ban đầu:** 3f903fb

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 480 | — |
| FP | 620 | — |
| FN | 173 | — |
| Precision | 0.4364 | — |
| Recall | 0.7351 | — |
| F1 | 0.5476 | — |
| mean_iou_of_matches | 0.8487 | — |
| passed | false | — |

Kết quả: [outputs/annotation_evaluation.json](outputs/annotation_evaluation.json). F1 0.5476 > 0.5. Mean IoU 0.8487 > 0.7. Số box nộp từng ảnh trong file đánh giá khớp với bộ nhãn hiện tại trong CVAT.

**Nhận xét (dữ kiện đã kiểm):**
- Số box nộp (1100) gấp khoảng 1.7 lần số box gold (653). Precision thấp ở cả hai nhóm, recall tương đối ổn.

**Rework:** **không có rework được giữ lại**

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:**
  - Google Colab, GPU T4, torch-2.14.1+cu130.
  - Ultralytics **8.4.208**, lấy từ metadata của `best.pt`.
  - Thời gian train 50 epoch khoảng **78 giây** (`results.csv`).
- **Tham số train** (`args.yaml`):
  - Model và dữ liệu: `model: yolo26n.pt` (pretrained, `cls_remap: true`), `data: /content/Day2-Lab/data.yaml`.
  - So với notebook:không đổi tham số nào.
- **Split:**
  - Train gồm 50 ảnh `drive_001`–`drive_050`.
  - Eval/validation gồm 10 ảnh `drive_051`–`drive_060`, xem `assets/eval_predictions/visuals/`.
  - **Không dùng ảnh eval để train.**
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`. 
Đây là checkpoint của **epoch 1**, xem nhận xét bên dưới.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4071 |
| Recall | 0.2340 |
| mAP50 | 0.2737 |
| mAP50–95 | 0.2018 |

Nguồn: `model_eval_metrics.json`, [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/), [assets/eval_predictions/](assets/eval_predictions/), [assets/yolo26_runs/vehicle_annotation/](assets/yolo26_runs/vehicle_annotation/).

- **Một Vật cụ thể:** 
  - xe buýt to màu trắng ở giữa hình `drive_052` mang nhãn "bus".

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Theo confusion matrix, lớp "van" (tên theo `data.yaml`) có recall thấp nhất: **0/12** vật thể đúng lớp. 11 vật thể bị dự đoán thành "bus", 1 bị bỏ sót.
   - Minh hoạ: `drive_052` và `drive_053`. Các vật thể mang nhãn "van" ở đây thực chất là xe buýt, và model dự đoán "bus".
   - Recall thấp này chủ yếu do lệch class ID, không phải do model không thấy xe.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Khi đó có thể phát hiện vật thể box chưa khít. Cần kiểm:
     - box rộng/hẹp ở mép;
     - có bao bóng xe hay không;
     - xe nhỏ ở xa: lệch vài px đã giảm IoU mạnh;
     - quy ước box phần nhìn thấy so với toàn thân khi xe bị che;
     - xe bị cắt rìa.
   - Ở phiên này, mAP50–95 / mAP50 = 0.2018 / 0.2737 ≈ 0.74. Khoảng cách không quá lớn, nên vấn đề chính là phân lớp và bao phủ, không phải độ khít box.
   - Gold QC cũng cho thấy box khớp khá chặt (mean IoU 0.8487).

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - **Lỗi lớn nhất là lệch class ID khi xuất nhãn**, không phải do định nghĩa lớp. Dữ kiện: số lượng trong `labels.jpg` bị hoán vị đúng như trên, cộng với confusion "van" → "bus" 11 lần.
   - Liên quan guideline thật sự là ranh giới **`van` ↔ `car`**:
     - guideline chọn `car` khi không chắc;
     - 3 vật thể `car` thật trong eval bị dự đoán "van".
   - **Phạm vi gán xe đậu:** bộ nhãn train gán cả xe đậu (guideline). Ảnh eval như `drive_052`, `drive_056`, `drive_060` có hàng xe đậu bên trái mà nhãn eval không gán. Đây có thể là một nguồn khiến số FP nền rất lớn trong confusion matrix (867 dự đoán "car" trên nền).

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Nếu ảnh eval nằm trong tập train, model đã "thấy" chúng. Chỉ số đo trên đó bị thổi phồng và không phản ánh khả năng tổng quát.
   - Trong notebook, `drive_051`–`060` là tập validation (`split: val`), dùng để:
     - đo P/R/mAP sau mỗi epoch;
     - **chọn `best.pt`**: chỉ số trong `model_eval_metrics.json` trùng với epoch 1 của `results.csv`;
     - vẽ PR curve/confusion matrix.
   - Vì tập này đã được dùng để chọn checkpoint, kết quả trên nó vẫn hơi lạc quan so với một tập test hoàn toàn mới.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Xe nhỏ ở xa | Xe chỉ rộng 8–20 px, khó kéo box sát bằng tay; lệch 2–4 px đã làm IoU giảm mạnh (3 xe xa ở `drive_001` chỉ đạt IoU 0.66–0.74 với gold). | Phóng to ảnh trong CVAT khi vẽ; đặt ngưỡng trong guideline: gán từ khoảng 8 px nếu còn nhận ra là xe. | Mean IoU toàn bộ vẫn đạt 0.8487, nhưng box xe nhỏ là nguồn rủi ro khi tính ở IoU cao (mAP50–95). |
| Xe bị che khuất | Xe buýt và xe con chồng lên nhau (`drive_009`), không chắc nên vẽ box phần nhìn thấy hay ước lượng toàn thân. | Chọn quy ước box ước lượng toàn thân, ghi vào guideline mini.| `drive_009` chỉ khớp 6/21 box gold dù số box gần bằng (22) — chưa xác định do quy ước box hay do vị trí; cần rà bằng mắt. |
| Xuất nhãn từ CVAT | Không kiểm thứ tự class trong `obj.names` (`car, bus, van, truck`) so với `data.yaml` (`car, truck, bus, van`) trước khi train. | Phát hiện sau khi đọc `labels.jpg` và confusion matrix. | Checker vẫn ĐẠT (chỉ kiểm cấu trúc); cần remap ID và train lại. Bài học: đối chiếu thứ tự lớp ngay khi xuất. |
| Train YOLO | mAP tốt nhất ở epoch 1, giảm sau khi fine-tune; mAP50 của `bus` chỉ 0.014. | Đối chiếu `labels.jpg`, `val_batch0_labels.jpg` và confusion matrix. | Phát hiện lệch class ID: nhãn CVAT `car,bus,van,truck` khác `data.yaml` `car,truck,bus,van`, và nhãn eval khác ở bus/van. Còn thiếu: remap nhãn, train lại. |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md). 

| Tiêu chí (điểm) | Bằng chứng hiện có | Đánh giá |
|---|---|---|
<<<<<<< HEAD
| Hoàn thành và định dạng YOLO | `check_yolo_labels.py`: "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ". | Đạt về cấu trúc. **Phần "bốn class ID đúng"** nhưng thứ tự ID xuất từ CVAT (`car, bus, van, truck`) khác `data.yaml` (`car, truck, bus, van`). |
| Đầy đủ vật thể | Recall gold 0.735; gán cả xe bị che và xe bị cắt rìa (152 box chạm mép). | Còn 173 FN. Một số ảnh khớp ít như `drive_009` (6/21), `drive_042` (1/4), chưa rà nguyên nhân. |
| Độ bám sát và hình học box (20) | mean IoU of matches 0.8487 (≥ 0.70, đạt mức "xuất sắc" 0.80). | Đạt theo số liệu. Có box xe nhỏ ở xa lệch 2–4 px (IoU 0.66–0.74 ở `drive_001`). |
| Mini guideline và quyết định mơ hồ | `GUIDELINE_MINI.md` có định nghĩa lớp, quy tắc box và 3 ca mơ hồ có lý do. | Đủ yêu cầu. |
| Gold-set annotation QC | `annotation_evaluation.json`: F1 0.5476, mean IoU 0.8487. | **Vừa đủ**:  0.50 < F1 > 0.80 và chưa có vòng rework. |
| Pipeline YOLO và phân tích | Colab chạy xong 50 epoch; không dùng eval để train; có metrics, ảnh eval và phân tích lỗi (mục 5). | Có kết quả. Cần train lại sau khi sửa class ID để số liệu theo lớp có ý nghĩa. |
=======
| Hoàn thành và định dạng YOLO (10) | `check_yolo_labels.py`: "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ". | Đạt về cấu trúc. **Phần "bốn class ID đúng" có rủi ro:** thứ tự ID xuất từ CVAT (`car, bus, van, truck`) khác `data.yaml` (`car, truck, bus, van`). |
| Đầy đủ vật thể (20) | Recall gold 0.735; gán cả xe bị che và xe bị cắt rìa (152 box chạm mép). | Còn 173 FN. Một số ảnh khớp ít như `drive_009` (6/21), `drive_042` (1/4), chưa rà nguyên nhân. |
| Độ bám sát và hình học box (20) | mean IoU of matches 0.8487 (≥ 0.70, gần mức "xuất sắc" 0.80). | Đạt theo số liệu. Có box xe nhỏ ở xa lệch 2–4 px (IoU 0.66–0.74 ở `drive_001`). |
| Mini guideline và quyết định mơ hồ (10) | `GUIDELINE_MINI.md` có định nghĩa lớp, quy tắc box và 3 ca mơ hồ có lý do. | Đủ yêu cầu. |
| Gold-set annotation QC (15) | `annotation_evaluation.json`: F1 0.5476, mean IoU 0.8487. | **Vừa đủ**:  0.50 < F1 < 0.80 và chưa có vòng rework. |
| Pipeline YOLO và phân tích (10) | Colab chạy xong 50 epoch; không dùng eval để train; có metrics, ảnh eval và phân tích lỗi (mục 5). | Có kết quả. Cần train lại sau khi sửa class ID để số liệu theo lớp có ý nghĩa. |
>>>>>>> cd5a807f9ef2196f05fccda0fcfe60374352ff1d

**Điều kiện bắt buộc:**
- Có đủ 50 file nhãn.
- Có gold-set evaluation.
- Không dùng eval để train.
- Không thay đổi gold labels.
- Đã chạy qua `check_yolo_labels.py` (50/50 hợp lệ), nên không bị giới hạn 49 điểm.


## 8. Công cụ AI và tài liệu tham khảo

**Công cụ AI:** Claude (Anthropic), dùng ở các bước:
- **Soạn tài liệu:**
  - Soạn nháp `GUIDELINE_MINI.md` và báo cáo này, sau đó đã được kiểm tra và chỉnh sửa.
  - Tham khảo những phân tích số liệu theo nhóm ảnh và theo lớp từ CVAT và `annotation_evaluation.json`.
- **Phân tích output Colab:** đọc `results.csv`, `args.yaml`, `model_eval_metrics.json`, confusion matrix, PR curve và ảnh nhãn/dự đoán; phát hiện lệch class ID.


**Tài liệu ngoài repo:**
- Ultralytics YOLO26, trọng số `yolo26m.pt` (https://github.com/ultralytics/assets/releases, https://huggingface.co/Ultralytics/YOLO26).
- CVAT REST API documentation (https://docs.cvat.ai).
