## useReducer in Action
- `useReducer` là hook THAY THẾ cho `useState`, cùng mục đích (tạo state, cập nhật state, tự động re-render khi state đổi) nhưng KHÁC CÁCH VIẾT LOGIC cập nhật - trong 1 component thường CHỈ dùng 1 trong 2, hiếm khi dùng chung cả 2
- Nên cân nhắc dùng `useReducer` khi: (1) có NHIỀU piece of state liên quan mật thiết tới nhau (thường được cập nhật cùng lúc), hoặc (2) giá trị MỚI của state phụ thuộc vào giá trị HIỆN TẠI của state
- Cú pháp: `const [state, dispatch] = useReducer(reducer, initialState)`
  - Argument 1: hàm `reducer` (tự định nghĩa) - chứa logic quyết định state mới
  - Argument 2: giá trị khởi tạo cho state
  - Trả về mảng 2 phần tử: phần tử 1 là `state` hiện tại (thường là 1 OBJECT gộp nhiều property), phần tử 2 là hàm `dispatch` (dùng để trigger cập nhật state, tương đương setter function của `useState`)
- Quy ước cộng đồng (không bắt buộc): với `useState` thường gọi NHIỀU LẦN, mỗi lần 1 piece of state đơn giản (number/string/boolean); với `useReducer` thường CHỈ gọi 1 LẦN DUY NHẤT, gộp TẤT CẢ state liên quan vào 1 object duy nhất
- Lợi ích thực tế: dễ debug hơn - chỉ cần `console.log(state)` là thấy TOÀN BỘ state của component, thay vì phải log riêng từng biến

## Rules of Reducer Functions
- Cách cập nhật state với `useReducer` khác hẳn `useState`: gọi `dispatch(...)` sẽ khiến React tự động chạy lại hàm `reducer` đã khai báo, TRUYỀN VÀO 2 argument: `state` (state hiện tại) và `action` (chính là giá trị mình truyền vào `dispatch`, quy ước gọi là `action`)
- `dispatch` chỉ nhận TỐI ĐA 1 argument - truyền nhiều hơn sẽ bị bỏ qua các argument thừa
- Giá trị RETURN từ hàm `reducer` chính là STATE MỚI của component - nếu không return gì (return `undefined`), state sẽ bị set thành `undefined`
- Quy tắc BẮT BUỘC cho hàm `reducer`:
  - KHÔNG được dùng `async`/`await`, không gọi API, không có Promise bên trong - reducer phải là hàm THUẦN (pure function)
  - Không nên tham chiếu tới biến bên ngoài (biến không nằm trong 2 argument `state`/`action`) - reducer chỉ nên hoạt động dựa trên đúng 2 argument nhận được
  - TUYỆT ĐỐI KHÔNG mutate trực tiếp object `state` nhận vào (không viết `state.count = ...`) - phải tạo OBJECT MỚI bằng kỹ thuật spread đã học (`{ ...state, count: state.count + 1 }`)
- Những quy tắc này (đặc biệt là pure function + không mutate) sẽ áp dụng NGUYÊN VẸN khi học Redux sau này - đây là bước đệm quan trọng

## Understanding Action Objects
- Vấn đề: khi có NHIỀU sự kiện khác nhau (click increment, click decrement, gõ input...) đều gọi `dispatch`, hàm `reducer` cần biết CHÍNH XÁC nó đang được gọi để làm gì, vì `reducer` chỉ nhận được đúng 2 argument `state` và `action`
- Giải pháp CHUẨN theo convention cộng đồng (không bắt buộc kỹ thuật, nhưng RẤT PHỔ BIẾN, đặc biệt quan trọng khi học Redux): mỗi lần `dispatch`, LUÔN truyền vào 1 OBJECT gọi là "action object", có cấu trúc:
```js
{ type: 'increment-count', payload: someValue }
```
- `type`: 1 string, dùng để `reducer` NHẬN DIỆN nó cần làm gì (VD tăng count, đổi giá trị input...) - giá trị chuỗi cụ thể KHÔNG có ý nghĩa đặc biệt gì với React, chỉ cần độc nhất và có ý nghĩa với người đọc code
- `payload` (tuỳ chọn): chứa DỮ LIỆU cần gửi kèm theo action đó (VD giá trị user vừa gõ vào input)
- Trong `reducer`, dùng chuỗi `if`/`else` (hoặc switch sau này) để so sánh `action.type`, từ đó biết chính xác cần cập nhật property nào của state, dùng `action.payload` nếu cần dữ liệu đi kèm
- Không có gì "đặc biệt" về mặt kỹ thuật với tên `type`/`payload` - hoàn toàn có thể đặt tên khác, nhưng đây là pattern cực kỳ phổ biến nên nên tuân theo để dễ hiểu, dễ làm quen với code người khác

## Constant Action Types
- Vấn đề thực tế: string `type` được viết TAY Ở 2 NƠI KHÁC NHAU (nơi gọi `dispatch` và nơi so sánh trong `reducer`) - nếu gõ sai chính tả dù chỉ 1 ký tự ở 1 trong 2 nơi, `reducer` sẽ KHÔNG NHẬN RA action đó, im lặng bỏ qua, không cập nhật state gì cả - RẤT khó phát hiện lỗi vì không có error message nào xuất hiện
- Giải pháp: định nghĩa các CONSTANT VARIABLE (thường ở đầu file, hoặc file riêng) để lưu các string action type, rồi dùng TÊN BIẾN đó thay vì gõ tay string trực tiếp ở cả 2 nơi:
```js
const INCREMENT_COUNT = 'increment';
const SET_VALUE_TO_ADD = 'set-value-to-add';
```
- Lợi ích: nếu gõ SAI TÊN BIẾN (thay vì sai string), JavaScript sẽ báo lỗi "variable not defined" NGAY LẬP TỨC, dễ phát hiện và fix hơn rất nhiều so với lỗi im lặng khi gõ sai string
- Quy ước đặt tên: thường viết HOA toàn bộ với dấu gạch dưới (`SCREAMING_SNAKE_CASE`, VD `INCREMENT_COUNT`) - đây là tín hiệu cho engineer khác biết đây là 1 hằng số action type, không bắt buộc về mặt kỹ thuật
- Giá trị chuỗi bên trong vẫn KHÔNG có ý nghĩa đặc biệt - chỉ cần DUY NHẤT trong phạm vi reducer đó, và tốt nhất nên có ý nghĩa dễ hiểu (để khi debug/console.log dễ đọc)

## A Few Design Considerations Around Reducers
- **Cân nhắc 1 - luôn spread `...state` trước khi ghi đè property, dù hiện tại có vẻ "thừa thãi":** nếu 1 case trong reducer CHỈ update đúng những property nó biết trước (không spread `...state`), khi sau này STATE MỞ RỘNG THÊM PROPERTY MỚI (không liên quan tới case đó), case cũ (thiếu `...state`) sẽ VÔ TÌNH XOÁ MẤT property mới đó khỏi state - luôn spread `...state` trước là cách "future-proof" (phòng ngừa trước cho tương lai), dù hiện tại có thể redundant
- **Cân nhắc 2 - nên đặt LOGIC TÍNH TOÁN bên trong `reducer`, KHÔNG đặt trong event handler nơi gọi `dispatch`:** nếu tính toán logic phức tạp ngay tại nơi dispatch (VD tự tính giá trị mới rồi nhét vào `payload`), và có NHIỀU nơi khác nhau trong app cần dispatch cùng 1 action type tương tự, rất dễ xảy ra tình trạng mỗi nơi viết code tính toán hơi khác nhau (dễ gõ nhầm dấu `+`/`-`, sai công thức) → bug rất khó phát hiện vì logic bị rải rác nhiều chỗ
- Nên giữ lệnh `dispatch` ở nơi gọi CÀNG ĐƠN GIẢN CÀNG TỐT (chỉ có `type`, may ra thêm `payload` là dữ liệu THÔ, không qua tính toán) - toàn bộ LOGIC NGHIỆP VỤ (business logic) nên tập trung DUY NHẤT bên trong `reducer` - giúp code dễ maintain, giảm khả năng lặp lại logic sai ở nhiều nơi, và có "1 nguồn sự thật" duy nhất cho việc state được thay đổi như thế nào

## Introducing Immer
- Immer: thư viện bên thứ 3 cho phép VIẾT reducer như đang MUTATE STATE TRỰC TIẾP (VD `state.count = state.count + 1`) - đi NGƯỢC LẠI hoàn toàn quy tắc "không bao giờ mutate state" đã học xuyên suốt từ đầu khoá
- Cài đặt: `npm install immer`
- Khi dùng Immer, reducer thay đổi 2 quy tắc quan trọng:
  1. Được phép trực tiếp sửa property của `state` nhận vào - Immer sẽ TỰ ĐỘNG theo dõi các thay đổi đó và tự tạo ra state mới đúng chuẩn immutable ở phía sau, không cần mình tự viết spread nữa
  2. KHÔNG BẮT BUỘC phải `return` giá trị từ hàm reducer nữa (return cũng không có tác dụng gì) - NHƯNG vẫn nên giữ `return` (return rỗng, không giá trị) ở CUỐI MỖI case trong switch statement - lý do KHÔNG liên quan tới Immer, mà là để tránh switch statement "rơi tiếp" (fall-through) xuống các case bên dưới (hành vi mặc định của JavaScript khi thiếu `break`/`return`)
- **CỰC KỲ QUAN TRỌNG:** không phải MỌI dự án React thực tế đều dùng Immer - thực tế PHẦN LỚN dự án KHÔNG dùng thư viện này - vẫn PHẢI nắm vững cách viết reducer immutable "thuần" (không có Immer) vì rất có thể bị hỏi/yêu cầu viết trong phỏng vấn xin việc, nơi công ty không dùng Immer