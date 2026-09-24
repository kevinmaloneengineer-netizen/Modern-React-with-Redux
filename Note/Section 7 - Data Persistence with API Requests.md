## Adding Data Persistence
- Vấn đề của app hiện tại: state chỉ sống trong bộ nhớ trình duyệt - user reload trang hoặc mở tab mới là mất sạch dữ liệu, không có tính persistence (lưu trữ lâu dài)
- Giải pháp: tách phần lưu trữ dữ liệu thật sự ra 1 API server riêng (có database), React app chỉ giữ state tạm để hiển thị UI, còn "nguồn sự thật" (source of truth) của data nằm ở server
- Dùng JSON-Server (thư viện mã nguồn mở) để dựng nhanh 1 API server cho mục đích học tập/development - lưu data vào 1 file JSON đơn giản, dễ xem trực tiếp trong code editor, nhưng cách tương tác với nó vẫn rất sát với 1 API thật ngoài production
- 3 việc cần làm để tích hợp API vào app đã có sẵn:
  1. Dựng API server, hiểu cách nó hoạt động
  2. Khi app khởi động, gọi request lấy danh sách sách hiện có từ API để khởi tạo state
  3. Với 3 hàm sửa đổi dữ liệu đã có (create/edit/delete), sửa lại logic: gọi API trước để thực hiện thay đổi thật trên server, nhận phản hồi thành công rồi MỚI update local state (thay vì update local state trực tiếp như trước)
- Vì đã tổ chức tốt từ đầu (toàn bộ business logic về books tập trung ở component App, không rải rác khắp nơi), việc thêm API vào không cần sửa đổi nhiều - đây là lợi ích của việc centralize logic đúng chỗ ngay từ đầu thiết kế

## How the API Works
- JSON Server lưu data vào 1 file JSON đơn giản (VD `db.json`), mỗi key trong file đó (VD `books`) tự động trở thành 1 route/resource riêng biệt để tương tác
- JSON Server tự cung cấp sẵn 4 route CRUD tiêu chuẩn theo REST convention, không cần tự code backend:
  - **Tạo mới**: `POST /books` với body chứa object cần tạo (VD `{ title: '...' }`) → JSON Server tự thêm object vào mảng, TỰ SINH ID cho record đó, trả về object đầy đủ (kèm ID) trong response
  - **Lấy toàn bộ danh sách**: `GET /books` → trả về mảng toàn bộ object hiện có
  - **Sửa 1 record**: `PUT /books/:id` với body chứa property mới cần cập nhật → JSON Server tìm đúng object theo ID, cập nhật, trả về object đã update
  - **Xoá 1 record**: `DELETE /books/:id` → JSON Server tìm và xoá object đó, trả về object vừa bị xoá trong response
- Route URL luôn theo pattern `/tenKeyTrongDbJson` (số nhiều) cho danh sách, và `/tenKeyTrongDbJson/:id` cho 1 record cụ thể - nếu đổi tên key trong file (VD từ `books` sang `photos`) thì route cũng đổi tương ứng
- Đây là REST API convention chuẩn (method HTTP tương ứng với hành động: POST=tạo, GET=đọc, PUT=sửa, DELETE=xoá) - áp dụng tương tự khi làm việc với API thật ngoài production, không riêng gì JSON Server

## The REST Client
- Standalone API client là 1 công cụ độc lập để gửi request thử tới API server (chạy local hoặc online) mà không cần viết code app thật - dùng để test/khám phá API trước khi tích hợp vào code
- REST Client là 1 extension của VS Code, cho phép viết request ngay trong 1 file text và gửi thử trực tiếp từ editor
- Cách dùng: tạo file có đuôi `.http` (VD `api.http`) trong project, viết cấu hình request theo cú pháp REST Client (method + URL + headers...), extension sẽ tự hiện nút "Send Request" ngay phía trên đoạn cấu hình đó để gửi thử và xem response ngay bên cạnh
- Lợi ích: file `.http` này trở thành tài liệu sống (documentation) cho cả team - bất kỳ ai làm việc trên project sau này đều có thể mở file đó, thấy sẵn các mẫu request (create/get/update/delete...) để hiểu cách API hoạt động mà không cần đọc code
- Đây chỉ là công cụ hỗ trợ development/test, không dùng để thao tác thay đổi dữ liệu thật trong production hay chỉnh sửa logic chương trình

## Creating a New Record
- React chỉ lo phần hiển thị UI, không tự làm network request - phải cài thư viện ngoài (Axios) để gửi request thật tới API server
- Cài đặt: `npm install axios`, import vào file cần dùng: `import axios from 'axios'`
- Gọi POST request tạo record mới với Axios: `await axios.post('http://localhost:3001/books', { title })` - argument thứ 2 là body của request
- Vì network request là bất đồng bộ, hàm chứa request phải đánh dấu `async`, và dùng `await` để chờ response trả về trước khi dùng tới nó (áp dụng lại kiến thức async/await đã học)
- Response trả về từ Axios có cấu trúc `{ data: ..., status: ..., ... }` - dữ liệu thật sự cần dùng (record vừa tạo, đã có ID do server tự sinh) nằm trong `response.data`
- Luồng cập nhật state sau khi tạo thành công: gửi request tạo record → nhận `response.data` (record đầy đủ kèm ID từ server) → dùng spread để thêm vào state local: `setBooks([...books, response.data])`
- Không cần tự sinh ID ngẫu nhiên nữa (khác với cách làm trước đó khi chưa có backend) - vì giờ ID được server (JSON Server) tự động gán khi tạo record thành công
- Có thể kiểm tra request thành công qua 2 cách: xem Network tab trong DevTools (status code 201 = created), hoặc mở trực tiếp file `db.json` để xác nhận record đã thực sự được lưu vào database

## What Just Happened? (Infinite Loop Bug)
- Hàm fetch data lúc app khởi động cũng theo pattern quen thuộc: `async` function, `await axios.get(url)`, lấy `response.data` rồi `setBooks(response.data)`
- KHÔNG BAO GIỜ gọi trực tiếp 1 hàm có chứa lệnh gọi API + update state ngay trong thân function component (VD gọi thẳng `fetchBooks()` ở phần đầu component, không đặt trong bất kỳ điều kiện/lifecycle nào)
- Lý do gây lỗi nghiêm trọng: mỗi lần component render, hàm đó bị gọi lại → gửi request → nhận response → gọi setState → setState kích hoạt re-render → component render lại → hàm đó lại bị gọi lại từ đầu → lặp vô hạn (infinite loop)
- Hậu quả thực tế: gửi request tới API server liên tục hàng trăm/hàng nghìn lần mỗi giây (kiểm tra được qua tab Network trong DevTools, filter Fetch/XHR để thấy số lượng request tăng vọt bất thường)
- Vấn đề cốt lõi cần giải quyết: làm sao để gọi 1 hàm fetch data CHỈ ĐÚNG 1 LẦN, đúng vào thời điểm component vừa render lần đầu tiên - không phải mỗi lần re-render đều gọi lại

## Introducing useEffect
- `useEffect` là hàm import từ React, dùng để chạy code tại những thời điểm cụ thể trong vòng đời component (lần render đầu tiên, và/hoặc mỗi lần re-render sau đó) - giải quyết đúng vấn đề "chỉ muốn chạy 1 lần khi component mount" từ bài trước
- Cú pháp: `useEffect(() => { ... }, [dependencyArray])`
  - Argument 1: arrow function chứa code cần chạy
  - Argument 2: 1 mảng, quyết định KHI NÀO function đó được gọi lại
- Truyền mảng RỖNG `[]` làm argument 2 → function chỉ chạy đúng 1 lần duy nhất, ngay khi component được render lần đầu tiên (initial render), không chạy lại ở các lần re-render sau
- Cách dùng để fix lỗi infinite loop ở bài trước: đặt lệnh gọi `fetchBooks()` bên trong `useEffect(() => { fetchBooks(); }, [])` - đảm bảo chỉ fetch data đúng 1 lần lúc app khởi động, không bị gọi lại mỗi lần re-render
- `useEffect` còn nhiều chi tiết/quy tắc quan trọng hơn (cách hoạt động của dependency array, khi nào nên/không nên dùng...) sẽ được giải thích sâu ở các bài tiếp theo

## How the API Works (useEffect Deep Dive)
- Function truyền vào `useEffect` LUÔN được gọi 1 lần ngay sau initial render (lần render đầu tiên của component) - điều này áp dụng cho mọi trường hợp, không phụ thuộc argument thứ 2
- Argument thứ 2 chỉ quyết định function đó có được gọi lại ở CÁC LẦN RE-RENDER SAU hay không - có 3 biến thể:
  1. **Mảng rỗng `[]`**: chỉ chạy đúng 1 lần sau initial render, không bao giờ chạy lại nữa dù component re-render bao nhiêu lần
  2. **Không truyền argument thứ 2 (bỏ trống)**: chạy sau initial render VÀ chạy lại sau MỌI lần re-render tiếp theo
  3. **Mảng có chứa 1 hoặc nhiều giá trị (dependency)**: chạy sau initial render, và chạy lại sau các lần re-render CHỈ KHI ít nhất 1 giá trị trong mảng đó đã thay đổi so với lần render trước (các giá trị này thường là state hoặc props)
- Việc dùng mảng rỗng `[]` (biến thể 1) chính là cách đã áp dụng ở bài trước để fetch data đúng 1 lần khi app khởi động, tránh vòng lặp vô hạn