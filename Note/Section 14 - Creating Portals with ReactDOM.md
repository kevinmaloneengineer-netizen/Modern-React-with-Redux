## We're Lucky it Works At All!
- 2 className Tailwind then chốt cách modal hiện tại hoạt động: `absolute` (áp dụng `position: absolute`) và `inset-0` (áp dụng `top:0; left:0; right:0; bottom:0`)
- Quy tắc CSS `position: absolute`: element sẽ được định vị theo PARENT GẦN NHẤT có `position` khác `static` (tức có `relative`, `absolute`, `fixed`, hoặc `sticky`) - nếu KHÔNG có parent nào như vậy, nó sẽ định vị theo TOÀN BỘ document (HTML gốc)
- Kết hợp `absolute` + `inset-0`: element sẽ TRẢI RỘNG lấp đầy toàn bộ chiều cao/rộng của parent định vị gần nhất đó (hoặc toàn màn hình nếu không có parent nào được định vị)
- Vấn đề: modal hiện tại "chạy đúng" hoàn toàn chỉ vì MAY MẮN - không có bất kỳ component cha nào của nó đang set `position` khác `static`, nên nó mặc định lấp đầy toàn bộ trang
- Rủi ro thực tế: chỉ cần 1 component cha bất kỳ (App, ModalPage...) thêm `className="relative"` (rất phổ biến trong dự án thật, ví dụ Google search results có tới 25 element dùng position khác static) là modal sẽ NGAY LẬP TỨC bị lỗi - chỉ lấp đầy đúng phạm vi của component cha đó, không còn che phủ toàn màn hình nữa
- Bài học: không nên thiết kế modal/component nào đó phụ thuộc vào "may mắn không có parent bị position" - cần giải pháp đảm bảo hoạt động đúng TRONG MỌI TRƯỜNG HỢP, không phụ thuộc vào cấu trúc CSS của các component cha

## Fixing the Modal with Portals
- Portal: tính năng của React cho phép RENDER 1 component ra vị trí KHÁC trong cây DOM thực tế, khác với vị trí nó được viết trong JSX (không còn là con trực tiếp của parent component gọi nó nữa)
- Giải pháp cho vấn đề ở bài trước: thay vì để HTML của Modal nằm lồng bên trong cấu trúc DOM của ModalPage (dễ bị ảnh hưởng bởi `position` của các component cha), dùng Portal để đẩy HTML của Modal ra hẳn 1 vị trí RIÊNG BIỆT ngay dưới `<body>` - đảm bảo Modal KHÔNG BAO GIỜ có parent bị position, luôn định vị theo toàn document
- Cú pháp: `ReactDOM.createPortal(jsxContent, domElement)` - import `ReactDOM` từ `react-dom`
  - Argument 1: JSX bình thường muốn hiển thị (y hệt nội dung modal cũ)
  - Argument 2: tham chiếu tới 1 DOM element THẬT đã tồn tại sẵn (thường lấy qua `document.querySelector('.modal-container')`)
- Bước chuẩn bị: cần tự thêm 1 element trống (VD `<div class="modal-container"></div>`) vào file `index.html`, đặt ngay trước thẻ đóng `</body>` - đây chính là "cửa" để Portal render nội dung vào
- Return trực tiếp `ReactDOM.createPortal(...)` từ trong component thay vì return JSX thông thường - component vẫn hoạt động, nhận props, quản lý state như bình thường, chỉ khác chỗ HTML thực tế xuất hiện ở đâu trong DOM
- Kết quả: dù component cha có thêm `position: relative` hay bất kỳ CSS nào khác, Modal vẫn LUÔN hiển thị đúng, lấp đầy toàn màn hình - vì nó không còn là "con cháu DOM" của các component cha đó nữa, mà được gắn thẳng vào `<body>`
- Đây là pattern CHUẨN, bắt buộc phải biết khi xây dựng bất kỳ Modal, Tooltip, Dropdown menu, hay Toast notification nào trong dự án React thực tế - vì vấn đề `overflow: hidden` hoặc `position` của component cha là cực kỳ phổ biến