## Building the Dropdown - State Design (Áp dụng lại quy trình 8 bước)
- Áp dụng lại quy trình State Design Process đã học (lần này nhanh hơn) cho component Dropdown
- Phase 1 xác định: cần 2 event handler (click vào dropdown để mở, click vào 1 option để chọn) và 2 piece of state (menu đang mở/đóng, item nào đang được chọn)
- Phase 2 (bước 4-7) tìm ra: piece of state "menu mở/đóng" nên là Boolean (VD isOpen), piece of state "item được chọn" nên là 1 object option hoặc null (không phải index/string đơn thuần) vì cần chứa đủ label + value để hiển thị
- Bước 8 (vị trí đặt state): dùng ví dụ mở rộng (shopping cart có CartTotal cần biết giá trị đã chọn) để minh hoạ nguyên tắc "có component khác cần biết state này không" - kết luận: `selected` (item được chọn) nên đặt ở PARENT vì component khác ngoài Dropdown có lý do hợp lý cần biết giá trị này; còn `isOpen` (trạng thái mở/đóng) chỉ Dropdown tự quan tâm nên đặt LOCAL ngay trong Dropdown

## Dropdown as a Controlled Component
- Dropdown được thiết kế theo đúng pattern "controlled component" đã học từ text input trước đây: piece of state (`selected`) sống ở PARENT, truyền xuống Dropdown qua props để hiển thị giá trị hiện tại, kèm 1 callback (`onSelect`) để Dropdown báo ngược lên khi user chọn option mới
- Luồng dữ liệu: user click option trong Dropdown → Dropdown gọi callback nhận từ props, truyền lên item vừa chọn → Parent cập nhật state → Parent re-render → giá trị mới chảy xuống lại Dropdown qua props → Dropdown hiển thị đúng lựa chọn mới
- Lợi ích giống hệt lý do dùng controlled input trước đây: Parent luôn biết chính xác giá trị hiện tại của Dropdown (chỉ cần đọc state), và có thể ép Dropdown hiển thị giá trị bất kỳ (chỉ cần đổi state) - không cần thao tác trực tiếp vào DOM của Dropdown

## Existence Check Helper (Optional Chaining)
- Optional chaining `?.`: kiểm tra 1 biến có phải null/undefined trước khi truy cập property của nó - nếu biến là null/undefined, cả biểu thức trả về `undefined` thay vì ném lỗi "Cannot read properties of null/undefined"
- Cú pháp: `variable?.property` thay vì phải viết dài dòng `variable ? variable.property : undefined`
- Kết hợp optional chaining với toán tử `||` để có giá trị fallback gọn gàng: `selection?.label || 'Select...'`
  - Nếu `selection` là null → `selection?.label` = `undefined` → `||` trả về `'Select...'`
  - Nếu `selection` có giá trị → `selection?.label` = label thật → `||` trả về label đó (vì là truthy đầu tiên)
- Kỹ thuật này rất hay dùng để rút gọn logic hiển thị fallback/default value khi dữ liệu có thể null/undefined

## Community Convention with Props Names
- Với các component dạng "form control" (input, dropdown, checkbox, radio, slider...), có 1 QUY ƯỚC PHỔ BIẾN trong cộng đồng (không bắt buộc về mặt kỹ thuật) về cách đặt tên 2 prop chính:
  - Prop chứa giá trị hiện tại: luôn đặt tên là `value`
  - Prop callback khi giá trị thay đổi: luôn đặt tên là `onChange`
- Lý do tuân theo convention này: khi có nhiều loại form control khác nhau trong 1 app (dropdown, input, checkbox...), nếu mỗi loại tự đặt tên prop riêng (`selection`/`onSelect`, `text`/`onTextChange`, `checked`/`onCheck`...) sẽ rất khó nhớ - dùng chung `value`/`onChange` cho MỌI form control giúp code nhất quán, dễ đoán, dễ nhớ hơn nhiều
- Đây chỉ ảnh hưởng tên PROP truyền vào component con - không ảnh hưởng gì đến tên biến state/event handler bên trong component cha (parent vẫn có thể tự đặt tên state/handler tuỳ ý)

## Document-Wide Click Handlers
- Cách lắng nghe click XẢY RA Ở BẤT KỲ ĐÂU trên trang: `document.addEventListener('click', handleClick)` - không gắn vào 1 element JSX cụ thể mà gắn thẳng lên `document`
- Trong handler, `event.target` cho biết chính xác element nào vừa bị click (dù là bất kỳ phần tử nào trên trang, kể cả các phần tử con lồng sâu bên trong)
- Kỹ thuật này là nền tảng để giải quyết bài toán "phát hiện click ra ngoài 1 component" (click-outside detection) - rất phổ biến khi cần tự động đóng dropdown/modal/menu khi user click ra vùng khác ngoài nó

## Event Capture and Bubbling

- Khi user click, trình duyệt xử lý sự kiện qua 3 GIAI ĐOẠN theo đúng thứ tự: **Capture phase** → **Target phase** → **Bubble phase**
- **Capture phase**: bắt đầu từ element CHA NGOÀI CÙNG (thường là `document`/`body`), đi DẦN XUỐNG tới đúng element vừa bị click, kiểm tra từng element trên đường đi xem có handler đăng ký ở capture phase không
- **Target phase**: kiểm tra đúng element vừa bị click, gọi handler thông thường (handler set up qua React onClick chính là chạy ở giai đoạn này)
- **Bubble phase**: từ element vừa bị click, đi NGƯỢC LÊN qua từng element cha, kiểm tra handler đăng ký ở bubble phase (đây là giai đoạn MẶC ĐỊNH của hầu hết event handler thông thường)
- Cú pháp `addEventListener(event, handler, useCapture)`: tham số thứ 3 là Boolean - `false` hoặc bỏ trống (mặc định) = lắng nghe ở target + bubble phase (như bình thường); `true` = lắng nghe ở CAPTURE phase
- 90-99% trường hợp thực tế, developer chỉ quan tâm target + bubble phase (không cần capture) - nhưng với bài toán dropdown, cần dùng capture phase để giải quyết vấn đề thứ tự thực thi

## Why a Capture Phase Handler?
- Vấn đề thực tế khi KHÔNG dùng capture phase (`false`/mặc định): custom click-outside handler (đặt trên `document`, chạy ở bubble phase) bị chạy SAU handler của React (đặt trực tiếp trên option, chạy ở target phase) - vì thứ tự target luôn chạy trước bubble
- Do React trì hoãn việc xử lý state update (batching) NHƯNG quá trình duyệt qua bubble phase của browser diễn ra CÒN CHẬM HƠN thế - nên thực tế, đến lúc custom handler (bubble) chạy, component ĐÃ re-render xong và element đã bị xoá khỏi DOM mất rồi → check "click có nằm trong dropdown không" luôn sai (vì element không còn tồn tại để kiểm tra)
- Cách fix: đặt custom handler chạy ở CAPTURE phase (`addEventListener('click', handler, true)`) - vì capture phase LUÔN chạy TRƯỚC target phase, nên custom handler chắc chắn chạy trước khi React kịp xử lý bất kỳ điều gì, đảm bảo lúc kiểm tra, dropdown vẫn còn nguyên trạng thái cũ (chưa bị đóng/xoá)
- Cách debug thực tế: dùng `performance.now()` (đo thời gian tính bằng mili-giây, độ chính xác cao) đặt tại nhiều điểm trong code (trước/sau khi update state, trong custom handler) để so sánh thứ tự và khoảng cách thời gian thực thi giữa các đoạn code - từ đó xác nhận chính xác đoạn nào chạy trước/sau
- Bài học tổng quát: khi cần đảm bảo 1 custom handler LUÔN chạy TRƯỚC handler khác trên cùng 1 sự kiện (bất kể handler kia thuộc React hay JS thường), đặt custom handler ở capture phase là giải pháp đáng tin cậy

## useRef in Action
- `useRef` là 1 hook, mục đích chính (chiếm phần lớn use case): lấy tham chiếu trực tiếp tới 1 DOM element mà component tự tạo ra, để có thể thao tác/kiểm tra nó ngoài luồng JSX thông thường
- Cách dùng: `const divEl = useRef();` sau đó gắn vào element cần tham chiếu qua prop đặc biệt `ref`: `<div ref={divEl}>...</div>`
- Điểm cần nhớ (dễ nhầm): `useRef` KHÔNG trả về trực tiếp tham chiếu tới element - nó trả về 1 OBJECT có property `current`, giá trị thật của tham chiếu nằm ở `divEl.current`, không phải `divEl` trực tiếp
- Lý do thiết kế theo dạng object với `.current` (thay vì trả thẳng giá trị): liên quan tới cách React so sánh reference giữa các lần render (tương tự lý do dependency array hoạt động dựa trên so sánh reference đã học ở `useCallback`)
- Đây chính là công cụ để hoàn thiện bài toán click-outside: lấy được tham chiếu DOM thật của Dropdown qua `useRef`, rồi trong custom click handler (chạy ở capture phase), so sánh xem `event.target` có nằm bên trong `divEl.current` hay không, để quyết định đóng dropdown hay giữ nguyên