## The Path Forward
- React KHÔNG có tools/objects/functions built-in nào để gọi HTTP request - đây không phải trách nhiệm của React
- Phạm vi của React chỉ gói gọn trong 2 việc: hiển thị content lên màn hình, và xử lý user events (click, type...)
- Mọi logic fetch data đều phải tự viết bằng JavaScript thuần, hoàn toàn tách biệt khỏi React - có thể viết và test logic đó độc lập trước khi gắn vào component
- Chiến lược tiếp cận: viết xong business logic + data fetching thuần JS trước, sau đó mới tích hợp vào React component

## HTTP Requests
- HTTP Request và Response đều có cấu trúc 3 phần: dòng đầu tiên (request line / status line), headers, và body (optional)
- Request line chứa Method + URL, ví dụ: `GET https://api.unsplash.com/images/search HTTP/1.1`
- Response status line chứa mã trạng thái, ví dụ: `HTTP/1.1 200 OK`
- Headers dùng để cung cấp thêm thông tin (như authentication) cho request/response, không nằm trong body
- Body chứa dữ liệu thực sự cần trao đổi (ví dụ JSON data trong response)
- Trong trình duyệt/JS, request và response được thao tác qua object (interface đẹp), không phải chuỗi text thô, dù bản chất bên dưới vẫn là text
- Các HTTP method phổ biến: GET (lấy dữ liệu), POST (tạo record mới), PUT (cập nhật toàn bộ record), PATCH (cập nhật một phần record), DELETE (xóa record)
- Status code nhóm theo chữ số đầu: 2xx = thành công, 3xx = bị redirect sang nơi khác, 4xx = lỗi do phía request (client), 5xx = lỗi do phía server
- Quy tắc nhanh: status code ≥ 400 nghĩa là có gì đó không thành công
- Có thể xem toàn bộ HTTP request thực tế của trang web qua tab Network trong DevTools trình duyệt - mỗi resource load lên (HTML, JS...) đều là 1 request riêng
- Trong tab Network, tab "Preview" hiển thị kết quả đã qua diễn giải (interpreted), tab "Response" hiển thị dữ liệu thô (raw) trả về
- Gọi HTTP request luôn tốn thời gian (network latency) - không xảy ra tức thì
- JavaScript mặc định KHÔNG tự động dừng lại chờ response trả về - nó chạy tiếp dòng code kế tiếp ngay lập tức, dù request chưa hoàn tất
- Nếu không xử lý đúng cách, code chạy ngay sau khi gọi request (như đọc kết quả) sẽ nhận giá trị chưa đúng vì response chưa kịp về - cần có cơ chế đợi (asynchronous handling) trước khi dùng tới response

## The Unsplash API
- Để dùng API bên thứ 3, quy trình chuẩn: đăng ký tài khoản -> tạo 1 app trên platform đó -> được cấp Access Key (hoặc API Key) để định danh request của mình
- Access Key phải được đính kèm trong mọi request gửi tới API, thường qua header `Authorization`
- Cách tìm hiểu 1 API mới: đọc documentation, xác định 3 thứ chính - root URL (endpoint gốc), cách xác thực (auth header), và route/params cụ thể cho tác vụ cần làm (ví dụ search cần query string)
- Query string dùng để truyền tham số cho GET request, format: `?tenTruong=giaTri`, nối vào sau URL
- Request tổng hợp đầy đủ gồm: method (GET) + root URL + route (`/search/photos`) + query string (`?query=...`) + header xác thực (`Authorization: Client-ID <access_key>`)

## Making an HTTP Request
- React không có sẵn tool cho HTTP request - phải dùng thư viện ngoài như Axios, hoặc hàm `fetch` có sẵn trong trình duyệt
- Axios được ưa chuộng hơn `fetch` vì cú pháp gọn hơn, ít code hơn để xử lý cùng 1 việc
- Cài thư viện bên ngoài qua npm: `npm install <package-name>` - npm (Node Package Manager) tải package đó về từ registry công khai, lưu vào thư mục `node_modules`
- Cú pháp gọi request với Axios: `axios.get(url, { headers: {...}, params: {...} })`
  - `headers`: object chứa các header cần đính kèm (như authorization)
  - `params`: object chứa key-value pairs, Axios tự động chuyển thành query string và gắn vào URL
- Logic fetch data nên tách riêng khỏi React component - viết thành file/function độc lập, test riêng lẻ trước khi tích hợp vào app
- Vì request bất đồng bộ (không trả về ngay), function chứa request phải đánh dấu `async`, và đặt từ khóa `await` trước lệnh gọi request để chờ kết quả trả về trước khi chạy tiếp
- Dữ liệu response trả về nằm trong property `.data` của response object; nếu API trả list, thường có thêm 1 lớp property (ví dụ `.data.results`) chứa mảng item thực tế

## Using Async/Await
- Vấn đề gốc: JavaScript mặc định chạy mỗi dòng code càng nhanh càng tốt, không tự dừng lại đợi request hoàn tất - nên gọi request rồi dùng ngay kết quả ở dòng kế tiếp sẽ không có dữ liệu đúng
- `await` là từ khóa báo cho JS: dừng lại tại dòng này, chờ cho đến khi có response trả về rồi mới tiếp tục chạy các dòng phía sau
- `async` là điều kiện bắt buộc đi kèm: bất kỳ function nào dùng `await` bên trong đều phải được đánh dấu `async` ở phần khai báo function
- Cú pháp: `const fetchData = async () => { const response = await axios.get(...); ... }`
- Có khoảng "thời gian trễ" thực tế giữa dòng gọi request (`await`) và dòng code chạy tiếp theo - vì phải chờ network round-trip hoàn tất

## Data Fetching Cleanup
- Free tier của 1 API bên thứ 3 thường có giới hạn quota (ví dụ 50 requests/giờ) - vượt quá sẽ bị lỗi, cần cẩn thận số lần gọi khi develop/test
- App/API key có thể bị nhà cung cấp chủ động vô hiệu hóa (disabled) - nếu gặp lỗi bất thường từ API, nên kiểm tra lại trạng thái app trên dashboard của họ trước khi debug code
- Không nên hardcode giá trị cố định trong function fetch data - nên nhận giá trị động qua tham số (parameter), ví dụ `searchImages(term)` thay vì hardcode `query: 'cars'`
- Function fetch data nên return đúng phần dữ liệu cần dùng, không trả nguyên response object đầy đủ - lọc lấy đúng nhánh dữ liệu cần thiết (ví dụ `response.data.results`) trước khi return, để nơi gọi function không phải tự đào sâu vào structure response
- Code test/debug tạm thời (console.log, gọi thử function ở entry file...) nên dọn dẹp sau khi xác nhận logic hoạt động đúng, trước khi tích hợp vào phần chính của ứng dụng

## About Data Flow
- Component hierarchy trong app: App -> (SearchBar, ImageList) -> ImageList -> nhiều ImageShow
- Trước khi viết code, có thể suy luận trước data thuộc về component nào dựa vào nơi nó được tạo ra và nơi nó cần hiển thị - không cần code React trước mới biết
- Sibling components (2 component cùng là con của 1 parent) KHÔNG THỂ giao tiếp trực tiếp với nhau - không có cách nào 1 component gọi thẳng data sang component anh em của nó
- Muốn chia sẻ thông tin giữa 2 sibling component, bắt buộc phải đi qua component cha chung của chúng - đây là nguyên tắc bất biến trong React
- Chiều Cha -> Con: dùng props như bình thường (đã biết) - cha gọi function, có data, truyền data đó xuống con qua props
- Chiều Con -> Cha: đây là chiều NGƯỢC với props thông thường - cần 1 kỹ thuật khác để con báo lên cho cha biết 1 sự kiện vừa xảy ra kèm dữ liệu liên quan (kỹ thuật này sẽ học ở bài tiếp theo, thường gọi là "callback prop" hay "lifting state up")
- Chiến lược tổng quát khi thiết kế data flow trong React: xác định data cần đi từ đâu đến đâu trong cây component, rồi áp đúng kỹ thuật cho từng chiều (props cho cha->con, callback cho con->cha)

## Child to Parent Communication
- Quan sát lại event handler bình thường (VD: onClick trên button) dưới góc nhìn khác: parent định nghĩa function, truyền xuống con qua props (VD prop `onClick`); khi sự kiện xảy ra, con tự gọi function đó (lấy từ props) và truyền vào 1 event object - dữ liệu (event object) thực chất đang "chảy ngược" từ con lên cha
- Đây chính là cách để làm "child to parent communication": coi việc giao tiếp ngược như 1 event handler tự định nghĩa, không phải built-in event của DOM (click, change...) mà là custom callback do mình tự đặt tên và tự gọi
- Cách làm:
    1) Parent định nghĩa 1 callback function,
    2) Truyền function đó xuống Child qua props (dùng tên tùy chọn, thường đặt kiểu `onSubmit`, `onXxx`),
    3) Trong Child, khi sự kiện cần báo lên xảy ra, gọi `props.onXxx(data)` truyền kèm dữ liệu cần gửi lên,
    4) Function ở Parent (đã truyền xuống) sẽ chạy với dữ liệu đó, coi như dữ liệu đã "đi lên" tới Parent
- Bản chất, props vẫn chỉ truyền 1 chiều Cha -> Con (chính là function) - nhưng chiều dữ liệu thực sự "trồi lên" là do Con chủ động GỌI function đó và truyền tham số vào, chứ không phải props tự đảo chiều
- Pattern đặt tên phổ biến cho prop kiểu callback này: tiền tố `on` + tên sự kiện (ví dụ `onSubmit`, `onClick`) dù đó là component tự định nghĩa, không phải native DOM event