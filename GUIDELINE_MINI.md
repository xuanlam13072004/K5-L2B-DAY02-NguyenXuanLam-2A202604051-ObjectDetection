# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con chở người. | Sedan, SUV, taxi, xe bán tải. Thân thấp, có capo rõ ở đầu xe; khác `van`: van cao và vuông như hộp, đầu xe ngắn. |
| `truck` | Xe có kích thước lớn, chủ yếu dùng để vận chuyển hàng hóa, vật liệu hoặc thiết bị. | Xe tải chở hàng, xe đầu kéo. Cabin đầu xe tách rời thùng hàng, thân cao và dài; khác bán tải (`car`): bán tải nhỏ, cao ngang xe con; khác `bus`: bus là một khối liền có dãy cửa sổ. |
| `bus` | Xe lớn dùng để vận chuyển nhiều hành khách. | Xe buýt, minibus chở khách. Thân rất dài, cao, một khối hộp liền với dãy cửa sổ hành khách đều nhau; khác `van`: van chỉ dài hơn xe con một chút, ít cửa sổ. |
| `van` | Xe dáng hộp, nhỏ gọn hơn xe tải và xe buýt, dùng để chở hàng hoặc chở người. | Xe van, minivan chở hàng hoặc chở người. Dáng hộp, mui cao, đầu xe ngắn, cỡ gần bằng xe con nhưng cao hơn; khác `car` (SUV): SUV có capo rõ; khác `bus`: van ngắn hơn nhiều. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: các cạnh box chạm đúng điểm cao nhất, thấp nhất, trái nhất và phải nhất của phần thân xe nhìn thấy được. Không để khoảng trống thừa, không lấy thêm mặt đường hay xe bên cạnh, và không cắt lẹm vào thân xe.
- Vật thể bị che khuất: vẽ box quanh phần thân xe nhìn thấy được nếu vẫn nhận diện được lớp; không đoán phần bị che. Hai xe chồng nhau thì vẽ hai box riêng.
- Vật thể bị cắt bởi rìa ảnh: vẽ box ôm sát phần xe còn trong khung hình, cạnh box chạm đúng rìa ảnh; không kéo box ra ngoài biên ảnh.
- Vật thể rất nhỏ/mơ hồ: không gán nếu quá nhỏ hoặc mờ do chuyển động đến mức không xác định chắc lớp; ghi tên ảnh vào mục ca mơ hồ và hỏi giảng viên.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_025` | Vật thể màu tối bị che phần lớn thân dưới, màu gần giống mặt đường, không thấy rõ đường viền xe. | Không gán nhãn; ghi lại để hỏi giảng viên. | Không xác định chắc là xe hay thuộc lớp nào; guideline yêu Xe tải chở trên thùng một vật trông giống xe con. | Gán `truck` cho xe tải; không gán vật trên thùng. Ghi lại để hỏi giảng viên. | Vật trên thùng là hàng hóa đang được chở, không phải phương tiện đang lưu thông, và không nhìn rõ để xác định lớp. |
| `drive_041` | Xe con màu đen bị xe tải che phần đuôi. | Gán `car` cho phần nhìn thấy. | Vẫn thấy rõ đường viền đầu xe và cabin nên xác định được là xe con; schema yêu cầu gán phần nhìn thấy của xe bị che. |
