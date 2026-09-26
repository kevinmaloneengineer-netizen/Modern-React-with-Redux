## Dynamic Table Headers
- Thêm prop `config` (mảng object) bên cạnh `data` cho component Table - mỗi object trong `config` đại diện cho 1 CỘT
- Số lượng object trong `config` quyết định số cột hiển thị, không còn phụ thuộc số property của object dữ liệu
- Bước đầu: mỗi config object có `label` để hiển thị trong `<th>`, dùng map qua `config` để tạo header động

## Done! But It's Not Reusable
- Vấn đề: Table hardcode theo đúng property của 1 loại object cụ thể → không tái dùng được cho dữ liệu khác
- Checklist thiết kế component tái sử dụng: số row linh hoạt, số column linh hoạt (không ràng buộc theo số property), 1 số cột sort được/1 số không, hỗ trợ sort nhiều kiểu dữ liệu, giá trị cell có thể TÍNH TOÁN từ nhiều property, cell hiển thị được bất kỳ nội dung nào (không chỉ text)
- Viết rõ requirement trước khi code là bước quan trọng để định hình API của component

## Rendering Individual Cells
- Thêm property `render` (1 function) vào mỗi config object: nhận vào 1 object dữ liệu, trả về giá trị/JSX cần hiển thị cho cell đó
- Bản refactor trung gian: gọi `config[i].render(item)` theo index cố định - chứng minh ý tưởng hoạt động nhưng vẫn giả định cứng số cột

## Nested Maps
- Giải pháp cuối: MAP LỒNG MAP - map ngoài duyệt `data` tạo row, map trong (với mỗi row) duyệt `config` tạo từng `<td>` bằng `column.render(item)`
- Nhờ đó số `<td>` luôn tự động khớp số object trong `config`, không cần sửa code Table khi thêm/bớt cột
- `key` cho `<td>` nên dùng field ổn định của config (VD `column.label`)
- Đây là kỹ thuật cốt lõi để có component Table thực sự reusable: thêm/bớt cột chỉ cần sửa mảng `config` ở nơi dùng, không đụng code bên trong Table