## Sorting Objects
- Sort mảng object khác sort mảng number/string đơn thuần ở chỗ: cần XÁC ĐỊNH TRƯỚC criteria dùng để sort (name, cost, weight, hay 1 giá trị TÍNH TOÁN từ nhiều property như cost/weight ratio, hoặc độ dài string...)
- Điểm chung của mọi cách sort object: luôn cần 1 bước TRANSFORM - lấy 1 object, "chiết xuất" ra 1 giá trị đơn (number hoặc string) từ object đó, rồi dùng giá trị đơn này để so sánh/sắp xếp
- Một khi đã có được giá trị đơn (number/string) từ bước transform, việc sort tiếp theo hoàn toàn giống với sort mảng number/string thông thường đã học
- Cách tổng quát hoá: viết 1 function nhận vào 1 object, return về giá trị dùng để sort (VD `getSortValue = (item) => item.cost / item.weight`) - hàm này linh hoạt tuỳ tiêu chí sort mong muốn, đây chính là "chìa khoá" để sort bất kỳ mảng object nào theo bất kỳ tiêu chí nào

## Optional Sorting (Vấn đề thiết kế: Table sortable vs non-sortable)
- Vấn đề thiết kế: nếu nhồi thêm logic sort trực tiếp vào Table component (thêm state, click handler, icon, sort logic...) thì Table sẽ phình to, khó tái sử dụng cho trường hợp KHÔNG cần sort (VD bảng chỉ có 3 dòng, không cần sort)
- Nếu vừa muốn có bản sortable vừa có bản không sortable trong CÙNG 1 component, sẽ phải chèn rất nhiều `if` để bật/tắt từng phần logic - làm code rối, khó maintain
- Câu hỏi đặt ra: có cách nào tách biệt phần "chỉ hiển thị table thuần" và phần "logic sort" ra làm 2, thay vì gộp chung vào 1 component không?
- Đây là bước đặt vấn đề dẫn tới giải pháp: tạo 1 component RIÊNG BIỆT (SortableTable) bọc quanh Table gốc, chỉ thêm khả năng sort mà KHÔNG cần sửa gì bên trong Table gốc - Table gốc vẫn giữ nguyên đơn giản, dùng được cho cả 2 trường hợp

## React Fragments
- Khi cần trả về 1 JSX element có `key` (bắt buộc trong list) nhưng KHÔNG muốn tạo ra bất kỳ HTML element thật nào trong DOM (VD tránh việc lồng `<div>` hoặc `<th>` sai vị trí trong bảng `<table>`, gây lỗi HTML không hợp lệ) - dùng `Fragment`
- `Fragment` là component có sẵn của React, hoạt động như 1 "vỏ bọc vô hình": nhận `children`, trả về đúng `children` đó, nhưng KHÔNG tạo thêm bất kỳ tag HTML nào trong DOM thực tế
- Cách dùng: `import { Fragment } from 'react'`, rồi `<Fragment key={...}>{children}</Fragment>` - có thể gán `key` prop lên `Fragment` như 1 element bình thường
- Cú pháp rút gọn `<>...</>` cũng là Fragment, nhưng KHÔNG hỗ trợ truyền `key` prop - khi cần key (VD trong list), bắt buộc phải dùng dạng đầy đủ `<Fragment key={...}>`, không dùng được shorthand
- Use case điển hình: khi 1 function (VD `column.header()`) cần return JSX có key nhưng nằm trong ngữ cảnh HTML có cấu trúc nghiêm ngặt (như bên trong `<tr>`, chỉ chấp nhận `<th>`/`<td>`) - dùng Fragment để thoả điều kiện "có key" mà không phá vỡ cấu trúc HTML hợp lệ

## The Big Reveal (Kiến trúc SortableTable bọc Table)
- Giải pháp cuối cùng cho vấn đề "Table sortable vs non-sortable": tạo 1 component MỚI tên `SortableTable`, bên trong nó SỬ DỤNG LẠI nguyên vẹn component `Table` gốc - KHÔNG sửa bất kỳ dòng code nào trong `Table.js`
- `SortableTable` không tự vẽ giao diện phức tạp - nhiệm vụ của nó chỉ là: nhận `config` từ props, XỬ LÝ LẠI (transform) mảng `config` đó (thêm logic/props mới), rồi TRUYỀN TIẾP xuống `Table` gốc để hiển thị
- Cơ chế: mỗi config object (đại diện cho 1 cột) có thể có thêm property MỚI, tuỳ chọn, tên `sortValue` - đây là 1 function tương tự `render` nhưng dùng riêng để LẤY GIÁ TRỊ SẮP XẾP (không phải giá trị hiển thị) từ mỗi object dữ liệu
- Cột nào CÓ `sortValue` → được coi là SORTABLE; cột nào KHÔNG có → giữ nguyên, không sortable
- `SortableTable` duyệt qua `config`, với mỗi cột có `sortValue`, tự động GẮN THÊM 1 hàm `header` (để hiển thị header cell có thể click được, kèm icon chỉ hướng sort) - đây chính là nơi gắn click event handler để bắt sự kiện user click vào header
- Khi user click vào header 1 cột sortable, `SortableTable` tự thực hiện việc SORT dữ liệu (dùng đúng `sortValue` của cột đó), rồi truyền DATA ĐÃ SORT xuống `Table` gốc để hiển thị - `Table` gốc hoàn toàn không biết gì về việc sort, chỉ đơn giản nhận `data` mới và render ra
- Lợi ích lớn nhất của kiến trúc này: có được 2 component riêng biệt dùng cho 2 mục đích khác nhau (`Table` cho bảng đơn giản không cần sort, `SortableTable` cho bảng cần sort) mà KHÔNG PHẢI DUPLICATE code hiển thị bảng - đây là ví dụ điển hình của việc tái sử dụng component bằng kỹ thuật "composition" (bọc/kết hợp component) thay vì nhồi nhét logic vào 1 component duy nhất

## Adding SortableTable (Implementation chi tiết)
- Bước đầu tiên khi tạo `SortableTable`: chỉ đơn giản nhận TOÀN BỘ props và forward (spread) thẳng xuống `Table` (`<Table {...props} />`) - chưa làm gì thêm, chỉ để đảm bảo mọi thứ vẫn hoạt động y hệt trước khi bắt đầu thêm logic
- Nguyên tắc quan trọng: KHÔNG BAO GIỜ mutate (sửa trực tiếp) `props` hay các object bên trong `config` nhận được - phải dùng `map()` để tạo ra 1 MẢNG CONFIG MỚI, mỗi object mới được tạo bằng cách spread lại toàn bộ property cũ (`...column`) rồi thêm/ghi đè property mới cần bổ sung (VD thêm `header`)
```js
const updatedConfig = config.map(column => {
  if (!column.sortValue) return column;
  return {
    ...column,
    header: () => <Fragment key={column.label}><th>...</th></Fragment>,
  };
});
```
- Sau khi có `updatedConfig`, truyền nó xuống `Table` để GHI ĐÈ lên `config` gốc trong `props`: `<Table {...props} config={updatedConfig} />` - vì props đặt SAU trong JSX sẽ ghi đè props cùng tên đặt trước (`{...props}`)
- Dù `render` và `sortValue` có thể trông "giống hệt nhau" ở ví dụ đơn giản, vẫn nên tách 2 function riêng biệt - vì `render` có thể trả về JSX phức tạp (VD bọc `<h1>`, thêm style...) không dùng được trực tiếp làm giá trị để so sánh/sắp xếp, trong khi `sortValue` luôn phải trả về giá trị đơn thuần (number/string) để so sánh được

## Resetting Sort Order (Reset khi đổi cột sort)
- Vấn đề UX: nếu đang sort theo cột A (VD ở trạng thái ascending/descending), rồi click sang cột B khác - hành vi mong đợi là cột B phải LUÔN bắt đầu từ ascending, KHÔNG được tiếp tục chu trình cũ (ascending → descending → unsorted) của cột A
- Cách fix: trong hàm xử lý click, kiểm tra XEM cột vừa click có PHẢI LÀ cột đang được sort hiện tại hay không (so sánh label vừa click với state `sortBy` hiện tại)
- Nếu KHÁC cột đang sort (đang chuyển sang sort 1 cột mới): ép `sortOrder` về `ascending` ngay lập tức, cập nhật `sortBy` thành cột mới, rồi return sớm (không chạy tiếp phần logic toggle chu trình cũ)
- Chỉ khi click ĐÚNG VÀO cột đang sort hiện tại thì mới tiếp tục chạy chu trình toggle bình thường (ascending → descending → unsorted → lặp lại)
- Đây là pattern UX phổ biến cho bất kỳ bảng dữ liệu sortable nào trong thực tế - đảm bảo hành vi sort luôn dễ đoán, không gây bối rối cho người dùng