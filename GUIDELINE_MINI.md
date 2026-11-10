# Mini guideline — bài làm cá nhân

Phạm vi: task CVAT `Lab-Day02`, 50 ảnh `drive_001`–`drive_050` (640×640, camera giao thông nhìn từ trên cao), kiểu nhãn **bounding box chữ nhật**, 4 lớp. Xuất Ultralytics detector YOLO 1.0 theo thứ tự `obj.names`: 0 = `car`, 1 = `bus`, 2 = `van`, 3 = `truck`.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe chở người cỡ nhỏ: sedan, hatchback, SUV, taxi. | Mui xe thấp hơn hoặc ngang thân, nắp capo rõ, chiều dài ngắn. Ví dụ: phần lớn xe trong `drive_002` (taxi đỏ-vàng, SUV đen ở góc phải dưới). Đây là lớp mặc định cho xe con khi không đủ dấu hiệu của `van`. |
| `truck` | Xe chở hàng có cabin lái tách khỏi thùng hàng hoặc thùng kín/thùng hở phía sau. | Có khe hoặc bậc rõ giữa cabin và thùng. Ví dụ: xe tải thùng xanh lá giữa ảnh `drive_009`. Xe tải nhỏ chở hàng cũng tính là `truck`. |
| `bus` | Xe khách hoặc xe buýt cỡ lớn chở nhiều người. | Thân dài, liền một khối, nhiều cửa sổ chạy dọc thân. Ví dụ: xe buýt trắng giữa `drive_002`, xe buýt đỏ-trắng lớn ở `drive_009`. |
| `van` | Xe chở người hoặc hàng dạng hộp, mui cao, đầu xe ngắn: minivan, xe van, MPV dạng hộp. | Thân gần như hình hộp, mui cao hơn hẳn `car`, đầu xe ngắn hoặc gần như không có capo, thường có cửa trượt. Ví dụ: xe trắng mui cao bị cắt ở mép phải `drive_002`, hai xe van trắng phía trên `drive_009`. Ngắn hơn nhiều so với `bus`. |

## Quy tắc chung

- **Gán xe nào:** gán **mọi phương tiện nhìn thấy và nhận ra được** trong ảnh, gồm cả xe đậu ở bãi, lề đường hay đường phụ, không chỉ xe trên làn đường chính. Không gán người đi bộ, xe máy, xe đạp, vì không có trong schema.
- **Box bám sát vật thể nghĩa là:** cạnh box chạm điểm ngoài cùng của phần thân xe *nhìn thấy* (gồm gương và cản), không chừa nền. Không bao bóng xe trên mặt đường hay vệt phản chiếu.
- **Vật thể bị che khuất:**
  - Vẫn gán nếu còn nhận ra đó là xe và đúng lớp.
  - Box bao luôn phần bị che khuất ( theo ước lượng của người gán). Ví dụ: các xe buýt phía sau bị xe buýt khác che một phần ở `drive_009`.
  - Không dùng cờ `occluded` của CVAT.
- **Vật thể bị cắt bởi rìa ảnh:** vẫn gán nếu phần trong ảnh đủ nhận ra lớp, kéo box sát tới mép ảnh. Ví dụ: xe `van` ở mép phải `drive_002`.
- **Vật thể rất nhỏ/mơ hồ:**
  - Gán xe nhỏ ở xa nếu còn nhận ra là xe, khoảng từ 10 px trở lên.
  - Không gán cụm xe dày đặc ở hậu cảnh nếu không tách được từng chiếc, ví dụ bãi xe ở góc trên bên trái `drive_009`.
  - Nếu không chắc là `car` hay `van`, chọn `car`.


## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_015` | Xe tải nhỏ thùng hở chở hàng cồng kềnh phủ bạt; cabin nhỏ cỡ xe con. | Đổi sang `truck`, box bao cả cabin và hàng trên thùng. | Theo định nghĩa lớp, `truck` là xe có cabin tách thùng hàng, kể cả xe tải nhỏ. Kích thước cabin không quyết định lớp. Hàng chở trên xe là phần nhìn thấy của vật thể nên nằm trong box.|
| `drive_002` | Xe trắng mui cao ở mép phải bị rìa ảnh cắt một phần. Không chắc là `van` hay SUV. | Gán `van`, box kéo sát mép phải ảnh. | Phần nhìn thấy có thân dạng hộp, mui cao và cửa sổ hông lớn, đúng dấu hiệu của `van`. Xe bị cắt rìa vẫn gán khi còn nhận ra lớp. |
| `drive_009` | Nhiều xe buýt xếp chồng, xe sau bị xe trước che một phần. Góc trên bên trái có bãi xe dày đặc, xe nhỏ và dính vào nhau. | Gán từng xe buýt bị che, box bao cả phần không nhìn thấy (theo ước lượng của người gán). Không gán cụm xe dày đặc ở góc trên trái. | Xe bị che vẫn nhận ra là `bus` nên phải gán.|

**Cần giảng viên làm rõ:**
- Ranh giới giữa `van` và SUV/MPV.
- Kích thước tối thiểu bắt buộc phải gán.
