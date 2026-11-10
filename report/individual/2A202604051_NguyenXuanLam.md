# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Xuân Lâm
- MSSV: 2A202604051
- Email: xuanlam13072004@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/xuanlam13072004/-K5-L2B-DAY02-NguyenXuanLam-2A202604051-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/xuanlam13072004/K5-L2B-DAY02-NguyenXuanLam-2A202604051-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema áp dụng: `car=0`, `truck=1`, `bus=2`, `van=3`; mỗi xe một box ôm sát phần nhìn thấy; xe bị che vẫn gán phần nhìn thấy nếu nhận ra lớp; xe bị cắt khung thì box chạm rìa ảnh; không gán mô tô, người đi bộ, vật thể ngoài schema.

Ba ca mơ hồ:

- `drive_025`: vật thể màu tối bị che phần lớn thân dưới, màu gần giống mặt đường → không gán, ghi lại để hỏi giảng viên vì không xác định chắc lớp.
- `drive_026`: xe tải chở trên thùng một vật trông giống xe con → gán `truck` cho xe tải, không gán vật trên thùng vì đó là hàng hóa, không phải phương tiện đang lưu thông.
- `drive_041`: xe con màu đen bị xe tải che phần đuôi → gán `car` cho phần nhìn thấy vì vẫn thấy rõ đầu xe và cabin.

Cần giảng viên làm rõ: (1) gold có bỏ qua xe ở xa/vùng phía trên ảnh không; (2) thứ tự class ID trong gold (xem mục 4).

## 3. Annotation và self-QC

- Công cụ: CVAT (task 50 ảnh train, label theo thứ tự `car, truck, bus, van`, vẽ bằng Rectangle → Shape), export Ultralytics YOLO Detection; đã kiểm `data.yaml` của bản export đúng thứ tự `0: car, 1: truck, 2: bus, 3: van`.
- Số file nhãn train: 50
- Kết quả checker: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ.`
- Danh sách ít nhất 10 ảnh đã rà bằng mắt:
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
- Commit khóa nhãn ban đầu: `58b25a2` (Lock initial labels)

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa (rework 1) |
|---|---:|---:|
| TP | 476 | 476 |
| FP | 619 | 182 |
| FN | 177 | 177 |
| Precision | 0.4347 | 0.7234 |
| Recall | 0.7289 | 0.7289 |
| F1 | 0.5446 | 0.7262 |
| mean_iou_of_matches | 0.8505 | 0.8505 |
| passed | false | false |

Kết quả sau sửa: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

**Phân tích trước khi sửa.** Em gán 1095 box so với 653 box gold. Đối chiếu từng ảnh (vẽ box gold và box của em lên ảnh) cho thấy:

- Không có box trùng lặp (0 cặp box chồng nhau IoU > 0.7).
- 502 box không khớp box gold nào, chủ yếu là xe ở xa phía trên ảnh (cuối đường, bãi đỗ) mà gold không gán. Ví dụ rõ nhất: `drive_002`, `drive_003`, `drive_030`, `drive_031`.
- 117 box đúng vị trí nhưng khác lớp với gold. Các nhầm lẫn theo một vòng đều: em gán `bus` → gold `van` (53), `van` → gold `truck` (27), `truck` → gold `bus` (20). Khi mở ảnh, ví dụ `drive_002` có một xe khách cỡ lớn em gán `bus` nhưng gold ghi class 3 (`van`); `drive_009` các xe buýt lớn cũng bị gold ghi khác `bus`. Em nghi file gold dùng thứ tự class khác `data.yaml` nên đã báo giảng viên và **giữ nguyên lớp theo schema**, không đổi nhãn của mình cho khớp gold.
- Mean IoU của các box khớp là 0.85, nên độ bám sát box không phải vấn đề chính.

**Rework 1.** Xóa các box không khớp gold có chiều cao dưới 50 px (ảnh 640×640) vì là xe ở xa, nhỏ, khó xác định chắc lớp; giữ lại các box lớn, rõ dù không có trong gold. Tổng cộng xóa 437 box trên 46 ảnh (nhiều nhất `drive_002` 28, `drive_003` 27, `drive_030` 25, `drive_031` 21). Ngưỡng 50 px được em chọn sau khi xem số liệu đối chiếu với gold. Không thêm box và không đổi lớp trong vòng này.

**Chưa đạt ngưỡng.** F1 0.726 < 0.80. Phần còn lại chủ yếu đến từ 117 box khác lớp (nghi lỗi ID lớp của gold) và 59 xe em bỏ sót. Đã báo giảng viên; phản hồi:

## 5. Kết quả YOLO

Notebook được chạy khi nhãn train **chưa đạt** ngưỡng annotation (F1 0.726 < 0.80, xem mục 4) vì hết thời gian; em ghi rõ điều này và đã báo giảng viên. Split giữ nguyên: 50 ảnh train, 10 ảnh eval; eval không đưa vào train.

- Runtime: Google Colab, GPU T4 (`device: 0`).
- Phiên bản Ultralytics/Torch:
- Tham số: giữ nguyên notebook — `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`. Không thay đổi.
- Best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3156 |
| Recall | 0.3066 |
| mAP50 | 0.2740 |
| mAP50–95 | 0.2006 |

Nguồn: [`assets/model_eval_metrics.json`](assets/model_eval_metrics.json). Ảnh dự đoán: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/). Confusion matrix trên eval: [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png). Log train: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv).

Phân bố nhãn: train (nhãn của em sau rework) car 528, truck 20, bus 76, van 34; eval (nhãn do lớp cung cấp) car 94, truck 5, bus 2, van 12. `car` chiếm đa số ở cả hai tập.

Theo confusion matrix trên eval (cột = nhãn eval, hàng = dự đoán):

| Nhãn eval | Số box | Dự đoán đúng | Bị dự đoán thành | Bỏ sót |
|---|---:|---:|---|---:|
| car | 94 | 84 | bus 2, van 3 | 5 |
| truck | 5 | 1 | car 4 | 0 |
| bus | 2 | 0 | car 1 | 1 |
| van | 12 | 0 | bus 11 | 1 |

**1. Lớp nào recall thấp nhất? Ảnh nào minh họa?**
`van` và `bus` có recall 0 trên eval; `truck` chỉ 1/5. Với `van`, 11/12 box bị model dự đoán là `bus`. Ví dụ [`drive_053`](assets/eval_predictions/visuals/drive_053.jpg): xe buýt lớn màu xanh trắng ở giữa ảnh được model dự đoán `bus 0.85`, nhưng nhãn eval của ảnh này không có box `bus` nào (chỉ có 7 car, 1 truck, 2 van), nên các xe buýt này đang được ghi là class 3 (`van`). Điều này khớp với nghi vấn ở mục 4 rằng nhãn của lớp dùng thứ tự class khác `data.yaml`. Nếu đúng vậy, recall thấp của `van`/`bus` phần lớn đến từ lệch ID lớp giữa nhãn của em và nhãn eval, không chỉ do model.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
Khi đó model tìm đúng vật thể nhưng box chưa sát: cần kiểm các cạnh box lệch, box rộng/hẹp hơn xe, đặc biệt với xe nhỏ ở xa và xe bị che, nơi vài pixel lệch làm IoU giảm mạnh. Ở bài này mAP50 (0.274) và mAP50–95 (0.201) đều thấp, chênh lệch không lớn, nên vấn đề chính là phân lớp và độ phủ, không phải độ sát box (mean IoU nhãn của em so với gold cũng đạt 0.85).

**3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ?**
- `van` ↔ `bus`: confusion matrix eval có 11 box `van` bị dự đoán `bus`; gold QC train cũng có 53 box em gán `bus` mà gold ghi `van`. Khi mở ảnh, các box này là xe buýt cỡ lớn, nên em cho rằng nguyên nhân chính là thứ tự class của nhãn lớp, cần giảng viên xác nhận; đồng thời ranh giới minibus/van trong guideline cũng cần rõ hơn.
- `truck` → `car`: 4/5 truck trên eval bị dự đoán `car`; train chỉ có 20 box truck. Guideline phân biệt bán tải (`car`) với xe tải nhỏ (`truck`) bằng cabin tách rời thùng hàng, dấu hiệu này khó thấy ở xe nhỏ, cộng thêm mất cân bằng lớp.
- [`drive_051`](assets/eval_predictions/visuals/drive_051.jpg): chiếc sedan màu đỏ gần camera bị dự đoán `van 0.99`. Đây là lỗi model (xe con rõ ràng), có thể do ít mẫu `van` và xe lớn gần camera trong train.

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng?**
Nếu đưa eval vào train, model đã thấy ảnh đó nên chỉ số trên eval sẽ cao giả, không phản ánh khả năng tổng quát. Trong notebook, 10 ảnh eval được dùng làm validation sau mỗi epoch và để chọn `best.pt`, vì vậy chúng cũng không phải test set độc lập; kết quả ở trên vẫn có thể hơi lạc quan.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Tạo task CVAT | Label tạo ban đầu sai thứ tự (`truck` đứng đầu) | Xóa và tạo lại label, export thử để kiểm thứ tự | Export đúng `car, truck, bus, van` |
| Viết guideline | Bắt đầu gán nhãn trước khi điền guideline | Quay lại điền định nghĩa lớp, quy tắc, ca mơ hồ | Đã có guideline |
| Gold QC | F1 0.545, FP rất cao | Vẽ box gold/box của em lên ảnh để tìm nguyên nhân | Rework 1 xóa box nhỏ ở xa, F1 0.726 |
| Gold QC | Nhầm lớp theo vòng bus→van, van→truck, truck→bus | Mở ảnh đối chiếu, thấy xe buýt lớn bị gold ghi `van` | Báo giảng viên, chưa đổi nhãn |
| Colab | Tải lại trang làm mất phiên chạy và kết quả | Chạy lại toàn bộ, không tải lại trang, tải `day2_colab_outputs.zip` ngay sau khi chạy | Có đủ metrics và ảnh dự đoán |

## 7. Tự đánh giá theo rubric

- Định dạng YOLO (10): đủ 50 file, checker đạt.
- Đầy đủ vật thể (20): recall 0.729 so với gold, còn 59 xe bị sót chưa bổ sung.
- Độ bám sát box (20): mean IoU 0.85.
- Phân lớp (15): 117 box khác lớp với gold, đang chờ giảng viên xác nhận lỗi ID lớp của gold.
- Mini guideline (10): có định nghĩa 4 lớp, quy tắc box và 3 ca mơ hồ.
- Gold QC (15): chưa đạt ngưỡng; có 1 vòng rework với số liệu trước/sau và lý do.
- Pipeline YOLO (10): đã chạy Colab, giữ đúng split, có metrics, ảnh eval và trả lời 4 câu hỏi; lưu ý train khi nhãn chưa đạt ngưỡng.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Claude, dùng để hỏi cách làm theo tài liệu repo, thao tác CVAT/git/Colab, góp ý diễn đạt guideline và báo cáo, và viết script phân tích lỗi nhãn (đối chiếu box với gold, vẽ box lên ảnh, xóa box theo ngưỡng 50 px em chọn).
- Phần em tự làm và kiểm tra: gán nhãn 50 ảnh trên CVAT, chọn ca mơ hồ và quyết định, chọn ngưỡng xóa, xem ảnh đối chiếu, chạy checker, evaluator và notebook Colab, kiểm tra lại số liệu trong báo cáo.
- Tài liệu ngoài repo: tài liệu CVAT.
