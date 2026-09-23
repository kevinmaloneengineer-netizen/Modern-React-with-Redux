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

## Handling Form Submission
- Đặt `<input>` bên trong `<form>` là kỹ thuật HTML thuần (không phải React) để tận dụng hành vi có sẵn của trình duyệt: khi input đang focus và user nhấn Enter, trình duyệt tự động kích hoạt sự kiện `submit` trên form đó
- Lắng nghe sự kiện đó bằng prop `onSubmit` trên thẻ `<form>`, ví dụ: `<form onSubmit={handleFormSubmit}>`
- Dùng form + `onSubmit` thay vì gắn `onKeyDown` riêng cho input: chỉ cần 1 handler duy nhất là bắt được cả trường hợp nhấn Enter lẫn click nút submit trong form, đỡ phải viết 2 handler riêng biệt
- Hành vi mặc định (default behavior) của form trong trình duyệt: khi submit, trình duyệt tự gom toàn bộ giá trị input (dựa theo attribute `name` của từng input) thành query string, rồi tự động điều hướng/reload trang tới URL đó - hành vi này tồn tại để hỗ trợ form hoạt động kể cả không có JavaScript
- Trong ứng dụng React, hành vi mặc định này không cần thiết và gây phản tác dụng (trang bị reload, mất state) - phải chủ động tắt nó
- Cách tắt: gọi `event.preventDefault()` ngay trong hàm xử lý sự kiện `submit`, dùng đối tượng `event` mà React tự động truyền vào handler
- Pattern chuẩn khi xử lý form trong React: `const handleFormSubmit = (event) => { event.preventDefault(); ... }`

## Handling Input Elements

- KHÔNG BAO GIỜ đọc giá trị input bằng cách truy cập trực tiếp DOM (ví dụ `document.querySelector('input').value`) - dù code này chạy được, nhưng hoàn toàn sai cách trong React (là điều dễ bị loại ngay khi phỏng vấn nếu viết ra)
- React quản lý form control (input, checkbox, radio, select...) theo cách khác biệt hẳn so với JS/HTML thuần - phức tạp hơn cho việc đơn giản, nhưng thuận lợi hơn nhiều khi cần thêm tính năng nâng cao sau này (validation, transform text...)
- 5 bước chuẩn để làm việc với text input trong React ("controlled input"):
  1. Tạo 1 piece of state riêng cho input đó, đặt tên theo mục đích (VD: `term`), giá trị khởi tạo là chuỗi rỗng
  2. Tạo event handler lắng nghe sự kiện `onChange` của input - sự kiện này fire mỗi khi user gõ, xóa, dán, cắt... bất kỳ thay đổi nào trong input
  3. Trong handler đó, lấy giá trị hiện tại của input qua `event.target.value` (đây là cách đúng chuẩn để đọc giá trị input trong React, khác hoàn toàn với truy cập DOM trực tiếp)
  4. Dùng giá trị đó để cập nhật state: `setTerm(event.target.value)`
  5. Truyền state đó ngược lại vào input qua prop `value`: `<input value={term} onChange={handleChange} />`
- Vì mỗi keypress đều gọi setter function nên component sẽ re-render liên tục theo từng ký tự gõ vào - nghe có vẻ tốn kém nhưng thực tế không phải vấn đề lớn
- Input được setup theo cách này (state điều khiển giá trị hiển thị) gọi là "controlled input" - giá trị hiển thị trên input luôn đồng bộ với state trong component, không phải input tự quản lý giá trị của chính nó

## OK But Why?
- Prop `value` trên input sẽ ép buộc giá trị hiển thị của input phải luôn bằng đúng giá trị được truyền vào - nếu là chuỗi cứng, input không thể gõ/sửa được nữa; nếu là 1 piece of state, input sẽ hiển thị đúng theo state đó tại mọi thời điểm
- Vòng lặp xảy ra mỗi khi user gõ vào input đã set `value` + `onChange`:
  1. Browser tự vẽ ký tự user gõ vào input (hành vi mặc định của browser)
  2. Browser fire sự kiện change
  3. React gọi `onChange` handler mình đã gắn
  4. Trong handler, đọc `event.target.value` rồi gọi setter cập nhật state
  5. Component re-render với state mới
  6. JSX return, prop `value` của input được gán lại = state mới
  7. React ép browser ghi đè lại giá trị input bằng đúng state đó
- Điểm "kỳ lạ": ở bước cuối, React ghi đè input bằng giá trị mà input vốn đã có sẵn (vì user vừa gõ xong) - nhìn qua như thừa thãi, không thay đổi gì
- Mục đích thật sự: chuyển quyền kiểm soát giá trị input từ browser sang cho state system quản lý (gọi là "controlled input") - từ giờ, giá trị thật sự của input luôn = giá trị của piece of state tương ứng, không cần đọc trực tiếp DOM nữa
- Lợi ích cụ thể khi input đã "controlled" bởi state:
  - Set default value cho input chỉ cần đổi giá trị khởi tạo của `useState`, không cần set DOM
  - Muốn đổi giá trị input theo code chỉ cần gọi setter, không cần thao tác DOM
  - Vì mỗi keypress đều re-render, có thể dễ dàng hiển thị thông tin phụ thuộc vào giá trị đang gõ (ví dụ hiển thị lại text đang gõ ở chỗ khác, validate độ dài, show error message theo điều kiện)
  - Có thể chặn/lọc ký tự user nhập bằng cách xử lý giá trị trước khi đưa vào setter (ví dụ dùng regex loại bỏ ký tự không mong muốn) trong chính `onChange` handler - vì input hoàn toàn bị điều khiển bởi state, không phải giá trị thật của DOM
- Nguyên tắc chung: không bao giờ đọc giá trị trực tiếp từ input element (trừ đúng bên trong `onChange` handler qua `event.target.value`) - mọi thao tác đọc/ghi giá trị input đều đi qua state system

## Getting the List of Images
- Khi console.log kết quả của 1 hàm async mà nhận về 1 `Promise` object (thay vì data thật) - đó là dấu hiệu request chưa hoàn tất, đang cố đọc kết quả quá sớm trước khi response về
- Bất kỳ chỗ nào gọi 1 hàm `async` (như `searchImages()`) và cần dùng ngay kết quả trả về, phải dùng `await` trước lệnh gọi đó - áp dụng dây chuyền: hàm cha gọi hàm async con cũng phải tự đánh dấu `async` và dùng `await`
- `await` không chỉ dùng khi gọi trực tiếp axios/fetch, mà dùng ở BẤT KỲ đâu gọi 1 async function và cần chờ kết quả của nó, kể cả khi gọi hàm tự viết (không phải API call trực tiếp)

## Passing Data Down to ImageList
- Cập nhật nội dung trên màn hình luôn đồng nghĩa với việc phải dùng State System - dù có truyền dữ liệu xuống con qua props hay không, gốc rễ vẫn phải bắt đầu từ 1 piece of state
- Pattern chung khi cần đưa kết quả (từ API call hay tính toán bất kỳ) vào UI: tạo 1 piece of state ở component chứa logic đó, gán kết quả vào state (qua setter), rồi dùng state đó trong JSX (trực tiếp hoặc truyền xuống con qua props)
- Khi 1 component cha update state, KHÔNG chỉ chính nó re-render mà TOÀN BỘ component con của nó cũng tự động re-render theo, nhận props mới tương ứng - đây là hành vi mặc định của React
- Một piece of data có thể vừa là state (từ góc nhìn component tạo ra nó bằng useState) vừa là prop (từ góc nhìn component con nhận nó qua tham số) - đây chỉ là cách gọi tên khác nhau tùy góc nhìn, bản chất dữ liệu và hành vi không đổi
- Chu trình đầy đủ khi fetch data rồi hiển thị xuống component con: khởi tạo state rỗng ban đầu -> user tương tác (submit form) -> gọi API (có await) -> nhận kết quả -> setState với kết quả đó -> React re-render component cha và toàn bộ cây con -> component con nhận props mới chứa data thật

## Understanding the Key Prop
- Khi state (mảng data) thay đổi thứ tự/nội dung, React có 2 cách lý thuyết để cập nhật DOM: (A) xóa toàn bộ element cũ rồi tạo lại từ đầu, hoặc (B) so sánh với lần render trước để chỉ áp dụng thay đổi tối thiểu
- Cách A đơn giản nhưng tốn kém hiệu năng - với list càng dài, việc rebuild toàn bộ chỉ để đổi vài phần tử là rất lãng phí
- Cách B hiệu quả hơn nhiều, và đây chính là cách React thực sự dùng - nhưng để làm được cách B, React cần 1 "định danh" ổn định cho từng phần tử để so sánh giữa các lần render, đó chính là prop `key`
- `key` nên là 1 giá trị duy nhất, ổn định, gắn liền với từng item dữ liệu (ví dụ dùng ID có sẵn từ data, như `image.id`), không nên đổi qua các lần render
- Nhờ key, React so sánh được thứ tự/danh sách key giữa lần render cũ và mới, từ đó suy ra chính xác phần tử nào đã di chuyển, thêm, hay xoá - rồi chỉ áp dụng đúng thay đổi đó lên DOM, không động vào phần tử không đổi
- Toàn bộ việc so sánh key, tính toán thay đổi tối thiểu là do React tự làm bên trong - việc của developer chỉ là gán đúng prop `key` khi build list (thường trong bước map/list building)

## Notes on Keys (Requirements)
- Key luôn gắn vào phần tử JSX NGOÀI CÙNG (topmost) trong danh sách đang build - không phải cố định 1 element cụ thể, mà phụ thuộc cấu trúc code hiện tại
- Lỗi hay gặp khi refactor: bọc thêm 1 element bao ngoài (VD thêm `<div>` để style) mà quên dời `key` từ element cũ lên element bao ngoài mới - phải luôn để ý dời key theo đúng element ngoài cùng
- Giá trị của key chỉ được là string hoặc number - không được dùng object/array (nếu dùng sẽ tự bị convert thành string, không đúng ý muốn)
- Key chỉ cần duy nhất trong PHẠM VI 1 mảng đang render - nếu gộp nhiều mảng lại thành 1 mảng rồi render chung, các key trùng nhau giữa các mảng gốc sẽ gây lỗi (khác với trường hợp render 2 mảng riêng biệt cạnh nhau thì không sao)
- Key phải ỔN ĐỊNH qua các lần re-render - tuyệt đối không dùng `Math.random()` hay bất kỳ giá trị random nào làm key, vì mỗi lần render sẽ sinh ra key khác hoàn toàn, khiến React nghĩ toàn bộ list đã đổi và phải rebuild lại từ đầu, mất hết lợi ích của key
- Lựa chọn key tốt nhất: dùng ID có sẵn từ data (thường lấy từ database qua API) - ID luôn duy nhất và không đổi qua các lần render, là lựa chọn lý tưởng
- Nếu data không có ID/định danh duy nhất nào: 
  - Fallback 1: dùng index của phần tử trong mảng làm key - dễ nhưng dễ gây bug khi list bị sắp xếp lại hoặc component con có state riêng
  - Fallback 2: tự generate 1 ID duy nhất khi tạo record (phức tạp hơn, tùy cách app tạo dữ liệu)