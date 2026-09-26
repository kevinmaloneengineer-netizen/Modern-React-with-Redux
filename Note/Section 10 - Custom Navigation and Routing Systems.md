## Designing Button Props
- Khi thiết kế props cho 1 component tái sử dụng (VD Button), nên rà soát mockup/thiết kế UI trước để xác định các BIẾN THỂ (variant) cần hỗ trợ, rồi đặt tên prop tương ứng cho từng biến thể đó
- Với các thuộc tính có 2 trạng thái rõ ràng (VD bo góc hay vuông góc, outline hay solid), dùng prop kiểu Boolean là hợp lý: `rounded` (true/false), `outline` (true/false)
- Với các "loại"/purpose của component (VD primary, secondary, success, warning, danger), có 2 cách thiết kế:
  - Cách 1: dùng 1 prop string duy nhất (VD `variation="primary"`) - đơn giản, tự nhiên loại trừ lẫn nhau
  - Cách 2: dùng nhiều prop Boolean riêng biệt cho từng loại (VD `primary`, `success`, `danger`...) - JSX viết ra đọc rất tự nhiên, gọn (`<Button success rounded outline />`), nhưng có rủi ro: 2 prop có thể cùng `true` một lúc (VD vừa `primary` vừa `success`), gây xung đột styling không mong muốn, cần xử lý thêm để đảm bảo logic
- Nhắc lại cú pháp rút gọn JSX đã học: prop Boolean có thể viết tắt, chỉ ghi tên prop mà không cần `={true}` (VD `<Button primary />` tương đương `<Button primary={true} />`); bỏ hẳn prop đó thì giá trị sẽ là `undefined`, không phải `false`, nhưng JavaScript coi cả 2 là falsy nên xử lý logic vẫn coi như tương đương trong hầu hết trường hợp

## Introducing PropTypes
- Thư viện `prop-types`: dùng để validate props truyền vào component - kiểm tra prop có được cung cấp đủ không, và có đúng kiểu dữ liệu mong muốn không (string, number, boolean, function, object...)
- Thư viện này chỉ dùng cho mục đích DEVELOPMENT - nếu prop sai kiểu/thiếu, chỉ hiện WARNING trong console, KHÔNG throw error, KHÔNG làm crash app
- Từng rất phổ biến, hiện vẫn dùng nhiều nhưng đang giảm dần vì TypeScript làm được việc tương tự (và nhiều hơn thế)
- Cài đặt: `npm install prop-types`
- Cách dùng cơ bản: gán property `propTypes` (viết thường "p") lên chính component, giá trị là 1 object liệt kê tên từng prop kèm loại validate tương ứng:
```js
Card.propTypes = {
  title: PropTypes.string.isRequired,
  content: PropTypes.string,
  showImage: PropTypes.bool,
};
```
- Không thêm `.isRequired` → prop đó là OPTIONAL (không bắt buộc phải truyền, nhưng nếu có truyền thì phải đúng kiểu)
- Thêm `.isRequired` → prop đó BẮT BUỘC phải được truyền, đồng thời phải đúng kiểu dữ liệu
- Có nhiều kiểu kiểm tra sẵn: `PropTypes.array`, `PropTypes.bool`, `PropTypes.func`, `PropTypes.number`, `PropTypes.object`, `PropTypes.string`...
- Hỗ trợ VALIDATION TÙY CHỈNH (custom validation): thay vì gán 1 kiểu có sẵn, có thể gán 1 FUNCTION tự viết - function này nhận vào toàn bộ props object làm argument đầu tiên, cho phép kiểm tra logic phức tạp liên quan tới NHIỀU prop cùng lúc (VD kiểm tra chỉ được đúng 1 trong nhiều prop Boolean là `true` tại 1 thời điểm) - nếu phát hiện lỗi, return về 1 object `Error` để báo warning

## Introducing TailwindCSS
- Về bản chất, TailwindCSS vẫn là CSS library như Bulma  (đều cho sẵn 1 bộ className kèm style tương ứng), nhưng khác biệt cốt lõi: mỗi className của Tailwind chỉ gắn với ĐÚNG 1 rule CSS duy nhất (VD `block`, `mr-1`, `w-2`, `text-white`), thay vì 1 className gắn với nhiều rule cùng lúc như Bulma (`.card` có sẵn nhiều style gộp lại)
- Vì mỗi className chỉ làm 1 việc, muốn style 1 phần tử đầy đủ phải liệt kê rất nhiều className cùng lúc trên 1 element - gọi vui là "className soup" (súp className) - khiến JSX trông rối, khó đọc, phải nhớ/tra rất nhiều tên className riêng lẻ
- Một số CSS feature vẫn không làm tốt được với cách tiếp cận utility-class của Tailwind
- Lý do THỰC SỰ khoá học chọn dùng Tailwind: không phải vì bản thân Tailwind "hay" hơn, mà vì cách viết className dài dòng này VÔ TÌNH ép buộc developer phải tách nhỏ component ra thành các phần tái sử dụng được - nếu không tách nhỏ, code sẽ trở nên cực kỳ khó đọc/khó bảo trì
- Tóm gọn: Tailwind tự nó không có gì đặc biệt xuất sắc, nhưng THÓI QUEN mà nó ép buộc (viết component nhỏ, tái sử dụng cao) mới là giá trị thực sự mà khoá học muốn truyền đạt qua việc dùng nó

## How to use Tailwind
- Quy trình chung khi áp style bằng Tailwind:
    1) xác định rule CSS cần áp dụng (background, border, spacing...)
    2) vào docs chính thức tailwindcss.com/docs, dùng ô search (phím tắt Cmd/Ctrl + K) tìm đúng trang tài liệu cho rule đó
    3) tra bảng ví dụ để tìm đúng className tương ứng
    4) gán className đó vào `className` prop của element
- Mỗi rule CSS (background color, border, font...) có 1 trang docs riêng biệt trên Tailwind - không có 1 trang tổng hợp duy nhất
- className thường theo pattern viết tắt dễ đoán: `bg-` = background, `text-` = màu chữ/font color - đổi tiền tố này sẽ đổi loại rule đang áp dụng (VD `bg-red-500` → `text-red-500` để đổi từ nền đỏ sang chữ đỏ)
- Số cuối cùng trong className màu sắc (VD `-500`) biểu thị ĐỘ ĐẬM/NHẠT của màu, chạy theo đơn vị hàng trăm từ `50` đến `900` - số càng nhỏ màu càng nhạt, số càng lớn màu càng đậm
- Dù ban đầu phải tra cứu nhiều, nhưng vì className theo pattern lặp lại nhất quán, học Tailwind thực tế nhanh hơn tưởng tượng - nhiều rule lặp lại liên tục qua các component khác nhau trong cùng 1 project

## Issues with Event Handlers (in a Component Library)
- Vấn đề: khi wrap 1 component tuỳ chỉnh xung quanh 1 element gốc (VD Button custom bọc `<button>` gốc), các event prop chuẩn của HTML (`onClick`, `onMouseOver`...) sẽ KHÔNG tự động hoạt động - vì component custom nhận được prop đó nhưng không tự động chuyển tiếp (forward) nó xuống element gốc bên trong
- Cách fix "tạm ổn nhưng không bền vững": nhận từng prop event cụ thể ở component custom (VD `onClick`), rồi truyền lại y nguyên xuống element gốc bên trong (`<button onClick={onClick}>`)
- Hạn chế của cách fix trên: mỗi khi người dùng component cần thêm 1 loại event mới (`onMouseOver`, `onMouseEnter`, `onMouseLeave`, `onHover`...), phải quay lại sửa code component, thêm từng prop 1 cách thủ công - không mở rộng được, rất tốn công khi component library được nhiều người dùng thực tế
- Bản chất vấn đề: trong props nhận vào component custom, có 2 loại: (1) prop dành riêng cho LOGIC của component đó (VD `primary`, `outline`, `rounded`) - cần xử lý riêng bên trong; (2) mọi prop CÒN LẠI (đặc biệt các event handler chuẩn HTML) - nên được TỰ ĐỘNG chuyển tiếp xuống element gốc, không cần khai báo thủ công từng cái

## Passing Props Through (Rest/Spread Pattern)
- Giải pháp bền vững cho vấn đề chuyển tiếp props ở bài trước: dùng REST destructuring để tự động gom TOÀN BỘ props chưa được liệt kê tên cụ thể, rồi SPREAD toàn bộ chúng xuống element gốc - không cần khai báo thủ công từng prop 1
- Bước 1 - destructure các prop component thực sự cần dùng riêng, phần còn lại gom vào 1 biến (thường đặt tên `rest`):
```js
function Button({ primary, outline, rounded, children, ...rest }) {
```
- Bước 2 - spread biến `rest` đó ra element gốc bên trong:
```js
<button {...rest}>{children}</button>
```
- Không thể gán thủ công từng cái (VD `onClick={rest.onClick}`) vì `rest` là 1 object chứa BẤT KỲ số lượng prop nào không biết trước tên - phải dùng spread để tự động áp dụng toàn bộ, dù caller truyền `onClick`, `onMouseEnter`, `onMouseLeave`, hay bất kỳ prop HTML chuẩn nào khác
- Kết quả: người dùng component chỉ cần dùng Button custom y hệt như dùng `<button>` gốc bình thường (thêm bất kỳ event handler hay attribute chuẩn nào), không cần biết/quan tâm bên trong component đã wrap thêm logic gì - đây là pattern chuẩn khi xây dựng component library thực sự tái sử dụng được

## Handling the Special ClassName Case
- Bug tinh vi khi kết hợp spread `...rest` với className tự build bên trong component: nếu caller truyền `className` từ bên ngoài (VD muốn thêm margin), nó sẽ nằm trong object `rest`, nhưng khi render `<button className={classes} {...rest}>`, tuỳ THỨ TỰ ĐẶT PROP, prop nào đứng sau sẽ GHI ĐÈ prop đứng trước - nếu `classes` (do component tự build) đặt sau `{...rest}`, nó sẽ vô tình xoá mất `className` mà caller truyền vào, khiến style bên ngoài không có tác dụng
- `className` là 1 field ĐẶC BIỆT cần xử lý riêng, không thể để spread tự động xử lý như các prop khác (VD event handler) - vì nó cần được GỘP (merge) với các className nội bộ của component, không phải bị GHI ĐÈ hoàn toàn
- Cách fix: lấy `className` ra riêng từ object `rest` (destructure hoặc truy cập `rest.className`), rồi đưa nó vào làm 1 trong các argument của hàm `classNames()` cùng với các class nội bộ khác:
```js
const classes = classNames(baseClasses, { ... }, rest.className);
```
- Kết quả: className cuối cùng là sự KẾT HỢP giữa style nội bộ do component tự định nghĩa VÀ style bổ sung mà caller truyền vào từ ngoài - không cái nào ghi đè mất cái nào
- Bài học tổng quát: khi wrap 1 HTML element bằng component custom rồi forward toàn bộ props qua, hầu hết prop có thể spread trực tiếp an toàn, nhưng những prop có tính chất "gộp được" thay vì "thay thế hoàn toàn" (như `className`, đôi khi cả `style`) cần xử lý đặc biệt riêng, không thể phó mặc hoàn toàn cho spread tự động
- Pattern "component custom wrap 1 element gốc, forward props, thêm styling/behavior nhất quán" là pattern RẤT PHỔ BIẾN trong các dự án React chuyên nghiệp - sẽ gặp lại nhiều lần