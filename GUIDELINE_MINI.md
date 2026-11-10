# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con dùng để chở người, bao gồm sedan, hatchback, SUV, taxi và pickup. | Pickup được xếp vào `car`; không nhầm với xe tải chở hàng. |
| `truck` | Xe tải dùng chủ yếu để vận chuyển hàng hóa, bao gồm xe tải thùng và xe đầu kéo. | Không xếp pickup hoặc xe cabin nhỏ kiểu xe con vào `truck`. |
| `bus` | Xe có thiết kế chở nhiều hành khách, bao gồm xe buýt và minibus chở khách. | Phân biệt với `van` dựa vào mục đích và hình dáng xe chở khách công cộng. |
| `van` | Xe van hoặc minivan dùng để chở người hoặc hàng hóa, có thân xe dạng hộp liền khối. | Không xếp xe buýt lớn vào `van`; không xếp sedan/SUV thông thường vào `van`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: box bao quanh sát phần thân xe thực sự nhìn thấy, hạn chế lấy nền, mặt đường hoặc vật thể bên cạnh.
- Vật thể bị che khuất: vẫn gán nhãn nếu phần nhìn thấy đủ để xác định loại xe; box chỉ bao phần nhìn thấy và không đoán phần bị che.
- Vật thể bị cắt bởi rìa ảnh: vẫn gán nếu nhận diện được lớp; box chạm đúng rìa ảnh và không mở rộng ra ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: chỉ gán khi có thể xác định lớp một cách hợp lý; nếu không chắc thì không tự đoán, ghi lại ảnh và hỏi giảng viên.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008` | Xe tải ben chở vật liệu (`dump truck`) và xe minivan trắng chở hàng ở giữa ngã tư | Gán xe tải ben là `truck`, xe minivan trắng là `van` | Xe ben có thùng hàng hở lớn phục vụ chuyên chở; minivan có thân xe dạng hộp liền khối. |
| `drive_008` | Xe buýt khớp nối dài (`articulated bus`) kích thước lớn che khuất một phần các xe phía sau | Gán một box `bus` ôm sát toàn bộ thân xe nhìn thấy; xe con phía sau chỉ gán nếu thấy rõ phần thân | Tuân thủ quy tắc box ôm sát phần nhìn thấy, không đoán phần bị che khuất. |
| `drive_001` | Xe con ở xa phía rìa ngoài và xe bị cắt mép ảnh | Chỉ gán xe trong luồng giao thông chính; xe cắt mép chỉ gán khi nhìn rõ loại xe | Vật thể quá xa, nhỏ hoặc mơ hồ ở lề ngoài không gán để tránh gây nhiễu nhãn (FP). |
