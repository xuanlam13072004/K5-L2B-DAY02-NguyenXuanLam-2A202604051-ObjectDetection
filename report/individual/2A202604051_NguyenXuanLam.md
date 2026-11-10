# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Xuân Lâm
- MSSV: 2A202604051
- Email: xuanlam13072004@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/[CẦN ĐIỀN: username]/K5-L2B-DAY02-NguyenXuanLam-2A202604051-ObjectDetection
- Link Colab đã chạy: [CẦN ĐIỀN — chưa chạy, xem mục 5]

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
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: [CẦN ĐIỀN]
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: [CẦN ĐIỀN]
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

**Phân tích trước khi sửa.** Tôi gán 1095 box so với 653 box gold. Đối chiếu từng ảnh (vẽ box gold và box của tôi lên ảnh) cho thấy:

- Không có box trùng lặp (0 cặp box chồng nhau IoU > 0.7).
- 502 box không khớp box gold nào, chủ yếu là xe ở xa phía trên ảnh (cuối đường, bãi đỗ) mà gold không gán. Ví dụ rõ nhất: `drive_002`, `drive_003`, `drive_030`, `drive_031`.
- 117 box đúng vị trí nhưng khác lớp với gold. Các nhầm lẫn theo một vòng đều: tôi gán `bus` → gold `van` (53), `van` → gold `truck` (27), `truck` → gold `bus` (20). Khi mở ảnh, ví dụ `drive_002` có một xe khách cỡ lớn tôi gán `bus` nhưng gold ghi class 3 (`van`); `drive_009` các xe buýt lớn cũng bị gold ghi khác `bus`. Tôi nghi file gold dùng thứ tự class khác `data.yaml` nên đã báo giảng viên và **giữ nguyên lớp theo schema**, không đổi nhãn của mình cho khớp gold.
- Mean IoU của các box khớp là 0.85, nên độ bám sát box không phải vấn đề chính.

**Rework 1.** Xóa các box không khớp gold có chiều cao dưới 50 px (ảnh 640×640) vì là xe ở xa, nhỏ, khó xác định chắc lớp; giữ lại các box lớn, rõ dù không có trong gold. Tổng cộng xóa 437 box trên 46 ảnh (nhiều nhất `drive_002` 28, `drive_003` 27, `drive_030` 25, `drive_031` 21). Ngưỡng 50 px được tôi chọn sau khi xem số liệu đối chiếu với gold. Không thêm box và không đổi lớp trong vòng này.

**Chưa đạt ngưỡng.** F1 0.726 < 0.80. Phần còn lại chủ yếu đến từ 117 box khác lớp (nghi lỗi ID lớp của gold) và 59 xe tôi bỏ sót. Đã báo giảng viên: [CẦN ĐIỀN: thời điểm và phản hồi của giảng viên].

## 5. Kết quả YOLO

[CẦN ĐIỀN — chưa chạy notebook vì nhãn chưa đạt ngưỡng annotation; đang chờ giảng viên xác nhận có được train không.]

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Tạo task CVAT | Label tạo ban đầu sai thứ tự (`truck` đứng đầu) | Xóa và tạo lại label, export thử để kiểm thứ tự | Export đúng `car, truck, bus, van` |
| Viết guideline | Bắt đầu gán nhãn trước khi điền guideline | Quay lại điền định nghĩa lớp, quy tắc, ca mơ hồ | Đã có guideline |
| Gold QC | F1 0.545, FP rất cao | Vẽ box gold/box của tôi lên ảnh để tìm nguyên nhân | Rework 1 xóa box nhỏ ở xa, F1 0.726 |
| Gold QC | Nhầm lớp theo vòng bus→van, van→truck, truck→bus | Mở ảnh đối chiếu, thấy xe buýt lớn bị gold ghi `van` | Báo giảng viên, chưa đổi nhãn |
| Nộp bài | Quá hạn 11:59 ngày 10/11 | [CẦN ĐIỀN] | [CẦN ĐIỀN] |

## 7. Tự đánh giá theo rubric

- Định dạng YOLO (10): đủ 50 file, checker đạt.
- Đầy đủ vật thể (20): recall 0.729 so với gold, còn 59 xe bị sót chưa bổ sung.
- Độ bám sát box (20): mean IoU 0.85.
- Phân lớp (15): 117 box khác lớp với gold, đang chờ giảng viên xác nhận lỗi ID lớp của gold.
- Mini guideline (10): có định nghĩa 4 lớp, quy tắc box và 3 ca mơ hồ.
- Gold QC (15): chưa đạt ngưỡng; có 1 vòng rework với số liệu trước/sau và lý do.
- Pipeline YOLO (10): chưa chạy.

## 8. Công cụ AI và tài liệu tham khảo

- Claude (claude.ai): đọc và tóm tắt tài liệu repo; hướng dẫn thao tác CVAT và git; góp ý cách diễn đạt và gợi ý dấu hiệu phân biệt lớp trong `GUIDELINE_MINI.md`; viết script phân tích lỗi (đếm box trùng/thừa/sai lớp, vẽ box gold và box của tôi lên ảnh) và script xóa các box theo ngưỡng 50 px do tôi chọn; hỗ trợ soạn khung báo cáo này từ số liệu thực tế.
- Phần tôi tự làm/kiểm tra: gán nhãn 50 ảnh trên CVAT, chọn 3 ca mơ hồ và quyết định, chọn ngưỡng xóa, xem ảnh đối chiếu, chạy checker và evaluator.
- Nguồn ngoài repo: tài liệu CVAT.
