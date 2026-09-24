## Reminder on Event Handlers
- Pattern quen thuộc lặp lại: khi 1 component con cần "báo lên" cho parent 1 sự kiện + dữ liệu (VD user submit form tạo mới), dùng đúng kỹ thuật callback prop (child to parent communication) - không có kỹ thuật gì mới, chỉ áp dụng lại
- Callback function (VD `createBook`) được định nghĩa ở component cha (nơi giữ state cần sửa), rồi truyền xuống con qua props
- Con gọi callback đó, truyền kèm dữ liệu cần thiết (VD title vừa nhập) khi sự kiện xảy ra (submit form)
- Cha nhận dữ liệu trong callback, dùng nó để update state (thêm object mới vào mảng, sinh ID ngẫu nhiên cho object đó...)
- Khi state cập nhật, toàn bộ cây con re-render theo, nhận props mới phản ánh đúng dữ liệu vừa thay đổi
- Với ứng dụng CRUD nhiều thao tác (create, edit, delete), thường sẽ có nhiều callback function riêng biệt tương ứng từng hành động, đều định nghĩa ở component cha rồi truyền xuống các con phù hợp qua props

## Implementing Create Flow
- Event handler nhận dữ liệu dạng argument (không chỉ qua event object): callback truyền từ cha xuống con (VD `createBook`) có thể được định nghĩa để nhận trực tiếp giá trị cần dùng làm tham số (VD `createBook(title)`), thay vì phải tự đào bới object phức tạp
- Component chứa form tạo mới (BookCreate) tự quản lý state riêng cho input của chính nó (VD `title`) - đây là state cục bộ (local state), độc lập với books state ở component cha; input này vẫn theo đúng pattern controlled input (state + `onChange` + `value`)
- Trong `handleSubmit` của form: gọi `event.preventDefault()` trước, sau đó gọi callback nhận từ props (VD `onCreate(title)`) để đẩy dữ liệu lên component cha
- Sau khi submit thành công, nên reset lại input về rỗng bằng cách gọi setter với chuỗi rỗng (`setTitle('')`) - đây chính là lợi ích thực tế của controlled input: xoá/reset giá trị input chỉ cần đổi state, không cần đụng vào DOM
- Đặt tên prop callback tùy ý miễn rõ nghĩa cho người đọc code khác (VD `onCreate`, `onSubmit` đều được) - không có quy tắc bắt buộc, chỉ cần thể hiện đúng ý nghĩa hành động

## Updating State (Arrays/Objects) - The Mutation Problem
- Update state kiểu number/string/boolean đơn giản, nhưng update state là array hoặc object phức tạp hơn - cần hiểu tại sao và làm sao cho đúng
- Sai lầm phổ biến: dùng `array.push(item)` rồi gọi setter (`setBooks(books)`) - code chạy, mảng thực sự có thêm phần tử, nhưng component KHÔNG re-render, UI không cập nhật gì cả
- Lý do: `push()` là mutating method - nó sửa trực tiếp mảng CŨ trong bộ nhớ (memory), không tạo mảng mới. Khi gọi setter với chính mảng đó, React so sánh reference (địa chỉ tham chiếu trong bộ nhớ) của state cũ và state "mới" - vì cùng trỏ tới 1 mảng y hệt trong memory, React kết luận "không có gì thay đổi" và bỏ qua re-render (đây là 1 optimization của React)
- Cách đúng: phải tạo ra 1 MẢNG MỚI hoàn toàn trong bộ nhớ, không sửa mảng cũ - dùng spread operator: `setBooks([...books, newBookObject])`
  - `[...books]` copy toàn bộ phần tử từ mảng cũ sang 1 mảng mới (copy, không phải cắt-dán)
  - Thêm phần tử mới vào cuối mảng mới đó
  - Toàn bộ kết quả là 1 array hoàn toàn khác về reference so với `books` cũ
- Vì reference mới khác reference cũ, React nhận biết đúng có thay đổi và tiến hành re-render như bình thường
- Quy tắc cốt lõi: khi update state là array/object, luôn tạo bản sao mới (immutable update), không bao giờ gọi các method làm thay đổi trực tiếp dữ liệu gốc (mutating methods như `push`, `splice`, gán trực tiếp property...)
- React chỉ áp dụng optimization so sánh reference này với object/array; với number/string/boolean/null/undefined thì không gặp vấn đề này (vì mỗi lần gán giá trị mới tự động tạo ra 1 giá trị hoàn toàn khác để so sánh)

## Don't Mutate That State!
- Quy tắc chỉ áp dụng khi array/object đó đang được quản lý bởi state system - object/array thường (không phải state) vẫn có thể mutate thoải mái như bình thường, không có vấn đề gì
- Các thao tác KHÔNG BAO GIỜ được làm trực tiếp lên state là array/object (vì đều là mutating operations - sửa trực tiếp dữ liệu gốc, không tạo bản sao mới):
  - `array.push(item)` - thêm phần tử vào cuối
  - `array[index] = value` - sửa phần tử theo index
  - `object.property = value` - gán trực tiếp property của object
- Bất kỳ thao tác nào sửa trực tiếp state array/object hiện có (thay vì tạo bản sao mới) đều khiến React không nhận ra sự thay đổi, dẫn tới không re-render
- Chủ đề "update array/object trong state đúng cách" (thêm đầu/cuối/giữa mảng, xoá phần tử, sửa phần tử, và tương tự với object) là kiến thức nền tảng quan trọng, dùng nhiều cả trong React lẫn các thư viện quản lý state khác như Redux

## Adding Elements to the Start or End
- Thêm phần tử vào ĐẦU mảng: `setColors([newColor, ...colors])` - tạo mảng mới, đặt phần tử mới trước, rồi spread toàn bộ phần tử cũ theo sau
- Thêm phần tử vào CUỐI mảng: `setColors([...colors, newColor])` - chỉ cần đổi thứ tự, spread phần tử cũ trước, phần tử mới đặt sau cùng
- Cả 2 cách đều tạo ra 1 mảng hoàn toàn mới trong bộ nhớ (không mutate mảng gốc), đúng nguyên tắc update state

## Adding Elements to the Middle
- `array.slice(start, end)` trả về các phần tử từ index `start` cho đến trước index `end` (không bao gồm `end`) - không mutate mảng gốc, chỉ trả về 1 mảng mới chứa phần được "cắt"
- Nếu chỉ truyền 1 argument `array.slice(start)`: lấy toàn bộ phần tử từ index `start` đến hết mảng
- Công thức chèn phần tử vào GIỮA mảng tại vị trí `index` mà không mutate: `[...array.slice(0, index), newItem, ...array.slice(index)]`
  - Phần đầu: lấy các phần tử trước vị trí cần chèn
  - Chèn phần tử mới vào giữa
  - Phần sau: lấy các phần tử từ vị trí đó trở đi
- Kỹ thuật này cũng dùng được để chèn vào đầu hoặc cuối mảng, nhưng phức tạp hơn hẳn so với cách spread đơn giản đã học trước đó (`[newItem, ...array]` hoặc `[...array, newItem]`) - nên chỉ cần dùng `slice` khi thực sự cần chèn vào giữa

## Removing Elements
- `array.filter(callback)` trả về 1 mảng MỚI, chỉ giữ lại những phần tử mà callback trả về truthy - không mutate mảng gốc
- Mnemonic dễ nhớ: **FKT - Filter Keeps True** - callback return `true` (hoặc giá trị truthy) thì giữ phần tử đó lại, return `false` thì loại bỏ
- Callback của filter nhận 2 tham số: phần tử hiện tại và index của nó
- Xoá phần tử theo GIÁ TRỊ: `array.filter(item => item !== valueToRemove)`
- Xoá phần tử theo INDEX: `array.filter((item, index) => index !== indexToRemove)`
- Xoá phần tử theo THUỘC TÍNH (VD ID trong object): `array.filter(item => item.id !== idToRemove)`
- Cả 3 cách đều cùng 1 nguyên lý: viết điều kiện "giữ lại nếu KHÔNG khớp với thứ cần xoá" - phần tử khớp điều kiện xoá sẽ trả về false và bị loại khỏi mảng kết quả

## Modifying Elements by Property
- Cập nhật 1 property của 1 object nằm bên trong 1 array trong state: kết hợp `map()` (duyệt qua mảng) với spread operator (tạo object mới)
- Dùng `array.map(item => ...)` duyệt qua từng phần tử: nếu KHÔNG phải phần tử cần sửa, return nguyên phần tử đó không đổi; nếu ĐÚNG phần tử cần sửa, return 1 object mới thay thế nó
- `map()` luôn trả về 1 mảng mới hoàn toàn (đúng nguyên tắc immutable update), khác với việc sửa trực tiếp phần tử trong mảng gốc
- Cách tạo object mới có property được cập nhật mà không mutate object gốc: `{ ...book, title: newTitle }`
  - `{ ...book }` copy toàn bộ key-value từ object gốc sang object mới
  - Field viết sau (VD `title: newTitle`) sẽ ghi đè lên field cùng tên đã copy trước đó, vì JavaScript object không cho phép trùng key - key sau luôn thắng
- Công thức tổng quát để update 1 record trong mảng theo điều kiện (VD theo ID):
```js
array.map(item => item.id === targetId ? { ...item, updatedField: newValue } : item)
```
- Nguyên tắc xuyên suốt: không bao giờ sửa trực tiếp object/array gốc đang nằm trong state - luôn tạo bản sao mới ở mọi cấp (cả mảng lẫn từng object bên trong)

## Why Recreate Objects Instead of Mutating Them?
- Reference so sánh của React (khi quyết định re-render) chỉ áp dụng ở CẤP MẢNG - React chỉ quan tâm mảng cha có phải mảng mới không, không tự động kiểm tra từng object bên trong mảng đó
- Vì vậy, kỹ thuật "tạo mảng mới nhưng vẫn sửa trực tiếp object cũ bên trong" (VD `book.title = newTitle` rồi đưa vào mảng mới) VẪN hoạt động bình thường ở hiện tại - app vẫn re-render đúng, không lỗi ngay lập tức
- Vấn đề chỉ lộ ra khi áp dụng thêm 1 optimization phổ biến ở tầng component con: chỉ re-render component đó nếu props nhận vào là 1 reference khác so với lần trước (so sánh reference của chính object, không so sánh nội dung bên trong)
- Nếu object bên trong bị mutate trực tiếp (không được tạo mới), nó vẫn là CÙNG 1 reference trong bộ nhớ giữa lần render cũ và mới - dù nội dung (property) đã đổi, optimization ở component con sẽ nhầm tưởng "props không đổi" và bỏ qua re-render, khiến UI không cập nhật dù state cha đã thay đổi đúng
- Đây chính là lý do dù cấp mảng đã handle đúng, vẫn PHẢI recreate cả object bên trong (không chỉ mutate) khi update - để đảm bảo tính đúng đắn khi sau này áp dụng các kỹ thuật tối ưu performance dựa trên so sánh reference
- Nguyên tắc rút ra: luôn tạo object/array mới ở MỌI CẤP bị thay đổi (không chỉ cấp ngoài cùng), để đảm bảo an toàn về lâu dài, dù hiện tại có vẻ như mutate trực tiếp vẫn "chạy được"

## Random
- Vì React thường không tự sinh ID (ID thường do backend/database sinh), nhưng nếu chưa có backend, cần tự tạo ID tạm thời để đảm bảo mỗi record là duy nhất
- Cách tạo số ngẫu nhiên trong 1 khoảng: `Math.round(Math.random() * maxNumber)`
  - `Math.random()` trả về số thập phân ngẫu nhiên từ 0 đến gần 1
  - Nhân với `maxNumber` để mở rộng khoảng giá trị (VD nhân 999 sẽ ra số từ 0 đến ~999)
  - `Math.round()` làm tròn về số nguyên cho dễ đọc, dễ dùng làm ID
- Cách này KHÔNG đảm bảo tuyệt đối duy nhất (có thể trùng số nếu chạy nhiều lần liên tiếp), nhưng xác suất trùng rất thấp, đủ dùng cho app nhỏ/demo/chưa có backend thật

## Rendering the Books List
- Pattern quen thuộc: parent (App) giữ state mảng (`books`), truyền xuống List component qua props (`<BookList books={books} />`), List component `map()` qua từng phần tử để tạo component con cho từng item (`<BookShow key={book.id} book={book} />`), component con nhận đúng 1 object và hiển thị field cần thiết (`book.title`)
- Đặt tên prop: theo nội dung dữ liệu đang truyền, số ít cho 1 item (VD `book`), số nhiều cho cả mảng (VD `books`) - giúp code dễ đọc, phân biệt rõ cấp component nào đang xử lý 1 item hay cả danh sách
- key khi map list dùng `book.id` (ID đã có sẵn từ data) - đúng theo yêu cầu key phải duy nhất và ổn định đã học trước đó
- Đây là ví dụ thực tế hoàn chỉnh của luồng: App (state) → List (nhận props, map) → Show (nhận props, hiển thị) - luồng dữ liệu 1 chiều cha xuống con qua nhiều cấp component