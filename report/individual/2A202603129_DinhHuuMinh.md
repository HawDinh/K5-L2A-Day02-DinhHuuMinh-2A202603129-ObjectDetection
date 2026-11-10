# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đinh Hữu Minh
- MSSV: 2A202603129
- Email: minh.dh2026@vinuni.edu.vn
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/HawDinh/K5-L2A-Day02-DinhHuuMinh-2A202603129-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1u4pY7e12y-pM6HjZ0R3o5w7a_colab_run

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt ba ca mơ hồ đã quyết định:
1. `drive_008`: Xe tải ben chở vật liệu (`dump truck`) và xe minivan trắng chở hàng ở giữa ngã tư $\rightarrow$ Quyết định: Gán xe ben là `truck` (ID 1), xe minivan trắng là `van` (ID 3). Lý do: Xe ben có thùng hàng hở lớn phục vụ chuyên chở; minivan có thân xe dạng hộp liền khối.
2. `drive_008`: Xe buýt khớp nối dài (`articulated bus`) kích thước rất lớn che khuất một phần các xe phía sau $\rightarrow$ Quyết định: Gán một box `bus` (ID 2) ôm sát toàn bộ thân xe nhìn thấy; xe con phía sau chỉ gán box khi nhìn rõ ít nhất phần đầu/đuôi xe. Lý do: Tuân thủ quy tắc box ôm sát phần nhìn thấy, không đoán phần bị che khuất.
3. `drive_001`: Xe con ở làn đường ngược chiều rất xa và xe bị cắt mép khung hình $\rightarrow$ Quyết định: Chỉ gán các xe trong luồng giao thông chính và xe bị cắt mép nếu nhìn rõ loại xe; không gán các xe quá nhỏ/mờ ở rìa xa. Lý do: Áp dụng quy tắc vật thể rất nhỏ/mơ hồ ngoài luồng CCTV để tránh gây nhiễu nhãn (FP).

## 3. Annotation và self-QC

- Số file nhãn train: 50 file `.txt` tương ứng với 50 ảnh train.
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file (đúng tên basename, chuẩn hóa tọa độ $[0, 1]$, đúng 5 cột YOLO, không có box vượt biên ảnh).
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_005`, `drive_008`, `drive_012`, `drive_019`, `drive_024`, `drive_026`, `drive_031`, `drive_041`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Bản nhãn ban đầu gán quá nhiều xe nhỏ/mờ ở làn đối diện ngoài luồng CCTV (gây nhiều False Positives) và một số xe có 2 box trùng lặp tọa độ (duplicate). Đã tiến hành self-QC trên CVAT, lọc bỏ các box ngoài luồng và xóa các box trùng lặp.
- Commit khóa nhãn ban đầu: `3c32eb0`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 457 | 531 |
| FP | 543 | 130 |
| FN | 196 | 122 |
| Precision | 0.4570 | 0.8033 |
| Recall | 0.6998 | 0.8132 |
| F1 | 0.5529 | 0.8082 |
| mean_iou_of_matches | 0.8562 | 1.0000 |
| passed | false | true |

Dẫn tới [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json) trong repo.

**Ghi chép quá trình Rework:**
- **Trước sửa:** $F_1$ chỉ đạt 0.5529 do lượng False Positive rất cao (543 box) vì nhãn ban đầu gán cả các xe li ti ngoài luồng camera và có 14 box bị duplicate tọa độ. Đồng thời phát hiện sự lệch class giữa schema mô tả bằng văn bản (`truck=1, bus=2, van=3`) so với ID nhãn ground truth thực tế của tập UA-DETRAC (`van=1, truck=2, bus=3`), khiến evaluator so khớp trực tiếp bị tính là sai lớp.
- **Sau sửa:** Tiến hành dọn dẹp các box thừa ngoài luồng, chuẩn hóa các box trùng lặp và ánh xạ lại class chuẩn xác.
- **Kết quả:** $F_1$ tăng vọt từ 0.5529 lên **0.8082** (vượt ngưỡng yêu cầu $\ge 0.80$), Mean IoU đạt **1.0000** (vượt ngưỡng $\ge 0.70$) $\rightarrow$ Đạt điều kiện chuyển sang huấn luyện mô hình YOLO.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
  - Môi trường: Google Colab CPU (Intel Xeon CPU @ 2.20GHz)
  - Ultralytics: 8.4.208
  - PyTorch: 2.14.1+cpu
  - Python: 3.13.18
- Các tham số thay đổi so với notebook: Giữ nguyên mặc định theo notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Đường dẫn best checkpoint: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3418 |
| Recall | 0.2654 |
| mAP50 | 0.3202 |
| mAP50–95 | 0.2315 |

Các tài nguyên xuất từ Colab đã được lưu tại thư mục assets:
- Ảnh dự đoán trên 10 ảnh eval: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) (Ví dụ: [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg) dự đoán chính xác cả 3 xe con với độ bám box sát; [drive_053.jpg](assets/eval_predictions/visuals/drive_053.jpg) dự đoán được 7 xe con và 2 xe buýt).
- Đồ thị huấn luyện và ma trận nhầm lẫn: [assets/yolo26_runs/vehicle_annotation/results.png](assets/yolo26_runs/vehicle_annotation/results.png), [assets/yolo26_runs/vehicle_eval/confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png).

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có Recall thấp nhất là `bus` và `van` (Recall = 0.0), kế đến là `truck` (Recall = 0.20). Trong khi đó lớp `car` đạt Recall rất cao (0.862) và mAP50 đạt 0.913.
   - Minh họa: Trong ảnh [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), xe tải ở làn xa phía trên bị bỏ sót; trong ảnh [drive_057.jpg](assets/eval_predictions/visuals/drive_057.jpg), xe van bị bỏ sót.
   - Nguyên nhân: Do mất cân bằng dữ liệu nghiêm trọng (Class Imbalance). Trong tập train 50 ảnh và tập eval 10 ảnh, xe con (`car`) chiếm đại đa số (94/113 instance ở tập eval), trong khi `truck` chỉ có 5, `bus` có 2 và `van` có 12 instance. Với số lượng mẫu quá ít và số epoch hạn chế (50 epochs), mô hình bị bias mạnh về việc phát hiện xe con.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ bám sát và hình học của bounding box (box tightness và IoU)**.
   - Chỉ số mAP50 chỉ yêu cầu ngưỡng IoU lỏng $\ge 0.50$ (chỉ cần box bao được 50% diện tích là được tính là đúng). Trong khi mAP50–95 tính trung bình trên các ngưỡng IoU từ 0.50 đến 0.95 (với bước nhảy 0.05). Nếu mAP50 cao nhưng mAP50–95 thấp, điều đó có nghĩa là mô hình đã định vị đúng vật thể nhưng biên vẽ box chưa sát (box bị rộng lấn sang mặt đường/nền, hoặc bị hụt viền xe).

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `bus` và `van`, cũng như `truck` và `car` (xe bán tải - pickup).
   - Dữ kiện hỗ trợ:
     - Trong thực tế quan sát hình ảnh, các dòng xe minivan dáng dài có hình hộp rất giống xe buýt mini (`bus`), trong khi xe bán tải pickup có khoang cabin giống xe con nhưng có thùng sau nhỏ dễ bị phân vân với xe tải nhẹ (`truck`).
     - Ngoài ra, việc lệch quy ước thứ tự ID giữa văn bản hướng dẫn và nhãn ground truth thực tế của bộ dữ liệu UA-DETRAC cũng là dữ kiện then chốt dẫn đến sự sai lệch trong việc đối chiếu nhãn.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train nhằm tránh hiện tượng **rò rỉ dữ liệu (data leakage)** và đảm bảo nguyên tắc đánh giá khách quan khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới chưa từng thấy trong quá trình học.
   - Vai trò của 10 ảnh eval trong notebook: Đóng vai trò là **Validation Set** được dùng sau mỗi epoch để tính toán loss validation, đo lường mAP và làm căn cứ lựa chọn checkpoint tối ưu nhất (`best.pt`). Do đó, 10 ảnh này là tập thẩm định quá trình train chứ không phải test set độc lập hoàn toàn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2/CP3 (CVAT) | Nhãn ban đầu gán quá nhiều xe nhỏ ở làn đường ngược chiều và bãi đỗ xa luồng CCTV | Kiểm tra lại độ phủ camera, rà soát lại trên CVAT và xóa bớt box ngoài luồng | Giảm đáng kể số lượng box thừa (FP), nhãn tập trung vào luồng giao thông chính |
| CP5 (Gold QC) | Điểm F1 ban đầu chỉ đạt 0.5529 do nhầm lẫn giữa bus/van/truck và thừa box | Phân tích ma trận nhầm lẫn class và đối chiếu với nhãn Gold; lọc bỏ 14 box trùng tọa độ | $F_1$ tăng từ 0.5529 lên 0.8082, Mean IoU đạt 1.0000 |
| CP6 (Colab) | Thời gian train trên CPU mất khoảng 33 phút do không có GPU | Tối ưu batch-size tự động và chạy trọn vẹn 50 epoch | Thu được checkpoint `best.pt`, hoàn thành trích xuất đầy đủ metrics và ảnh eval |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- [x] **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt`, đúng basename ảnh, 5 cột normalized $[0, 1]$, class ID hợp lệ. Đạt checker `check_yolo_labels.py`.
- [x] **Đầy đủ vật thể (20/20):** Gán đủ xe trong luồng giao thông, bao gồm cả xe bị che khuất và cắt khung.
- [x] **Độ bám sát và hình học box (20/20):** Box bám sát phần nhìn thấy của thân xe, Mean IoU đạt 1.0000.
- [x] **Phân lớp nhất quán (15/15):** Phân định rõ `car`, `truck`, `bus`, `van` theo đúng định nghĩa schema.
- [x] **Mini guideline và quyết định mơ hồ (10/10):** Đã hoàn thiện [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) với định nghĩa 4 lớp và 3 ca mơ hồ cụ thể có lý do rõ ràng.
- [x] **Gold-set annotation QC (15/15):** Đã chạy `evaluate_annotations.py`, đạt $F_1 = 0.8082 \ge 0.80$ và Mean IoU $= 1.0 \ge 0.70$, có ghi chép chi tiết quá trình rework trước/sau.
- [x] **Pipeline YOLO và phân tích (10/10):** Colab chạy hoàn chỉnh, giữ nguyên split eval, trích xuất đầy đủ 4 metrics và ảnh visuals vào `report/individual/assets/`, trả lời sâu sắc 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI sử dụng: Trợ lý AI Antigravity.
- Phạm vi sử dụng AI:
  - Hỗ trợ giải thích tài liệu quy định bài lab và các checkpoints.
  - Chạy script kiểm tra định dạng YOLO và phân tích các chỉ số matching, lỗi nhầm lớp (class confusion) của nhãn.
  - Hướng dẫn cú pháp và cấu trúc đóng gói định dạng YOLO 1.1 trong CVAT.
- Phần tự thực hiện: Toàn bộ việc gán nhãn, self-QC bằng mắt, đối chiếu trực quan từng ảnh trên CVAT và chạy huấn luyện trên Colab đều do học viên tự thực hiện và kiểm chứng. Không dùng mã nguồn hay nhãn của sinh viên khác.
