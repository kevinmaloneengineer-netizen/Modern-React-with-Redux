# Section 3 - Building with reusable components

# Props trong React
- Props là cách truyền dữ liệu từ Parent Component xuống Child Component
- Cách truyền: viết attribute trên JSX element, ví dụ `<Child color="red" />`
- React gom toàn bộ attribute đó lại thành 1 object gọi là Props Object, ví dụ `{ color: 'red' }`
- Props Object tự động được truyền vào làm argument đầu tiên của function component con: `function Child(props) { ... }`
- Trong component con, dùng `props.tenThuocTinh` để lấy giá trị ra, ví dụ `props.color`
- Có thể truyền nhiều component con giống nhau nhưng khác data qua props, mỗi component nhận props riêng của nó (ví dụ 3 `ProfileCard` với `title`/`handle` khác nhau)
- Mỗi lần gọi component là một props object riêng biệt, không bị lẫn giữa các instance
- Khi in giá trị props trong JSX, phải bọc curly braces: {props.title}

## Destructuring
- Có thể destructure props ngay trong tham số hàm thay vì viết props.x nhiều lần: function Child({ title, handle }) { ... }
- Cách này tương đương const { title, handle } = props nhưng gọn hơn, hay dùng trong dự án thực tế

## Images
- Import ảnh vào component phải giữ đuôi file (.png), khác import JS component (không đuôi)
- Ảnh <9.7kb được inline thành base64 string, ảnh lớn hơn được serve như file riêng (path)
- Hiển thị ảnh: <img src={bienDaImport} /> — dùng {} vì đang truyền biến JS
- Ảnh từ API/nguồn ngoài thì dùng URL trực tiếp, không cần import
- Truyền biến ảnh đã import xuống làm prop: <ProfileCard image={alexaImage} />
- Prop thừa không dùng tới ở component con thì không lỗi, chỉ đơn giản không hiển thị gì

## Handling Image Accessibility
- Luôn thêm alt="..." vào thẻ <img> để hỗ trợ screen reader (accessibility), không bắt buộc nhưng React sẽ cảnh báo nếu thiếu

## Review on how CSS Works
- CSS selector .ten-class sẽ tìm và áp dụng rule cho mọi element có className tương ứng trong JSX
- CSS library (Bulma...) yêu cầu viết đúng cấu trúc HTML + class name theo docs để có giao diện đẹp sẵn
- 2 cách thêm CSS: dùng CSS library có sẵn, hoặc viết custom CSS (nhiều phương pháp: public/src, CDN, NPM, SASS, CSS-in-JS, inline style)
- Cách phổ biến nhất trên dự án thực tế: cài CSS library qua NPM rồi import vào project

## A Big Pile of HTML!
- Dùng CSS library (Bulma) giống như ký "hợp đồng": phải viết đúng cấu trúc HTML + class name theo docs thì mới có style đẹp tự động
- Trong JSX phải dùng `className` thay vì `class` (dù docs library viết bằng HTML thuần dùng `class`)
- Phải thêm nhiều div/element lồng nhau với đúng tên class quy định (`card`, `card-image`, `card-content`, `media-content`...) để Bulma CSS nhận diện và style đúng
- Chỉ viết đúng HTML/class theo docs vẫn chưa đủ để layout đẹp ngay - thường phải thêm cấu trúc bổ sung (container, section, columns, column) để các phần tử được canh chỉnh đúng kích thước, vị trí
- Đây là vấn đề chung của mọi CSS library, không riêng gì Bulma
- Sau khi thêm đủ cấu trúc, style sẽ tự động áp dụng đúng như mong đợi

## Last Bit of Styling
- Style tiêu đề trang bằng Bulma hero component: section (className "hero is-primary") > div (className "hero-body") > p (className "title") chứa text
- Thêm prop mới (description) cho component giống cách đã làm với title/handle: truyền string ở component cha, nhận qua destructuring ở component con, dùng {description} trong JSX
- Tên prop truyền xuống và tên nhận ở component con phải khớp chính xác từng chữ, nếu không data sẽ không hiển thị đúng
- Một số lỗi layout nhỏ (card không đều chiều cao) là do CSS của thư viện (Bulma), không liên quan đến React - có thể tạm chấp nhận, không phải lúc nào cũng cần fix ngay


---

# Section 4 - State - How to change your app

## Event System
- 2 khái niệm quan trọng: Event System (phát hiện tương tác người dùng) và State System (cập nhật nội dung trên màn hình)
- Event System: cách React nhận biết người dùng đang làm gì (click, kéo thả, gõ phím...)
- Cách wire 1 hàm xử lý sự kiện vào element: `onClick={handleClick}` - chỉ viết tên hàm, không thêm dấu ngoặc () sau tên hàm
- Định nghĩa hàm xử lý bằng arrow function: `const handleClick = () => { ... }`
- Khi save và click nút, hàm sẽ chạy (kiểm tra qua console.log trong DevTools)

## Events in Detail
- 6 bước để dùng Event System: 
   - (1) quyết định lắng nghe event nào
   - (2) tạo function (gọi là event handler/callback function)
   - (3) đặt tên theo convention handle + EventName (không bắt buộc, chỉ là quy ước cộng đồng)
   - (4) truyền function đó làm prop cho plain element (div, button...)
   - (5) tên prop phải đúng chính xác tên event hợp lệ (onClick, onMouseEnter...)
   - (6) truyền reference của function, không gọi nó (không thêm dấu ngoặc `()`)
- 2 event dùng nhiều nhất (~90% trường hợp): click event và change event (khi user gõ/thay đổi input, textarea, radio...)
- Xem danh sách đầy đủ các event có thể lắng nghe trong React docs (mục Supported Events)
- Có thể đổi loại event đang lắng nghe chỉ bằng cách đổi tên prop, ví dụ từ `onClick` sang `onMouseMove` - mỗi lần di chuột dù chỉ 1 pixel cũng gọi lại handler liên tục
- "Event handler"/"callback function" chỉ là thuật ngữ quy ước giữa developer, bản chất vẫn là function JS bình thường, không có gì đặc biệt về mặt kỹ thuật

## Variations on Event Handlers
- Function có dấu ngoặc `()` sau tên sẽ được gọi (invoke) ngay lập tức; không có dấu ngoặc thì chỉ là reference tới function, chưa gọi
- Truyền event handler vào prop phải KHÔNG có dấu ngoặc `()` - vì muốn button tự gọi function đó trong tương lai (khi user click), không phải gọi ngay lúc component render
- Nếu lỡ thêm `()` (ví dụ `onClick={handleClick()}`), function sẽ chạy ngay khi component render lần đầu, không đợi user click, và button sẽ mất reference để gọi lại sau này
- Có nhiều cách viết event handler, tất cả đều hoạt động như nhau:
  - Định nghĩa function riêng ở trên, rồi truyền reference vào prop (`onClick={handleClick}`)
  - Viết trực tiếp arrow function ngay trong prop: `onClick={() => console.log(...)}`
- Viết inline (trực tiếp trong prop) thường dùng khi logic ngắn gọn - giúp đọc code dễ hơn vì không phải nhảy qua lại tìm định nghĩa function riêng
- Trên dự án thực tế sẽ gặp cả 2 cách, tùy theo lượng code trong callback nhiều hay ít

## State System
- Event System chỉ cho biết user vừa làm gì (click...), nhưng để cập nhật nội dung hiển thị trên màn hình phải dùng State System
- Cần 1 biến để lưu trạng thái (ví dụ `count`), bắt đầu từ giá trị 0
- Khi user click, tăng biến đó lên 1
- Khi biến thay đổi, nội dung trên màn hình phải tự cập nhật theo - quá trình này gọi là **re-render**
- Cách dùng State System cơ bản:
  - Import `useState` từ React
  - Khai báo: `const [count, setCount] = useState(0)` - trả về 1 mảng gồm giá trị hiện tại và hàm để cập nhật giá trị đó
  - Cập nhật giá trị bằng hàm setter, ví dụ `setCount(count + 1)`
  - Hiển thị giá trị trong JSX: `{count}`
- Khi gọi hàm setter, React tự động re-render lại phần UI liên quan để hiển thị giá trị mới

## More on State
- Định nghĩa State: dữ liệu bên trong component sẽ thay đổi theo thời gian khi user tương tác (click, gõ phím...)
- Khi state thay đổi, React tự động cập nhật (re-render) nội dung trên màn hình - đây là CÁCH DUY NHẤT để thay đổi nội dung hiển thị trong React
- Dấu hiệu cần dùng state: bất cứ khi nào muốn "nếu user làm gì đó thì hiển thị/thay đổi cái gì đó trên màn hình"
- `const [count, setCount] = useState(0)` dùng array destructuring - lấy ra 2 giá trị: piece of state (`count`) và setter function (`setCount`)
- Argument truyền vào `useState()` là giá trị khởi tạo mặc định (starting value) cho piece of state đó
- Một component có thể gọi `useState` nhiều lần (0, 1, 2, 3...), thường không quá 4 lần - nếu nhiều hơn 4 là dấu hiệu nên tách nhỏ component ra
- Piece of state có thể là bất kỳ kiểu dữ liệu nào (number, string, array, object...) tùy nhu cầu
- KHÔNG BAO GIỜ gán trực tiếp vào state (không viết `count = 1`) - luôn phải gọi setter function để cập nhật (`setCount(1)`)
- 4 bước dùng State System:
    1) định nghĩa state bằng `useState`
    2) truyền giá trị khởi tạo cho `useState`
    3) dùng state đó trong component (thường là hiển thị trong JSX)
    4) khi user tương tác thì gọi setter function để cập nhật state, khiến React re-render component

## The Re-Rendering Process
- Mỗi lần gọi setter function (`setCount`), toàn bộ component function sẽ được React gọi lại từ đầu (không phải chỉ update 1 phần) - đây gọi là re-render
- Lần render đầu tiên (khi mới load trang): React lấy giá trị mặc định truyền vào `useState(0)` gán cho `count`, giá trị mặc định này chỉ dùng đúng 1 lần duy nhất rồi bỏ đi, không bao giờ dùng lại
- Từ lần render thứ 2 trở đi: `count` sẽ lấy đúng giá trị mà mình vừa truyền vào setter function ở lần gọi trước đó, không còn liên quan gì đến giá trị mặc định ban đầu
- Luồng cụ thể: user click -> gọi `handleClick` -> gọi `setCount(count + 1)` -> React nhận thấy state thay đổi -> tự động re-render (gọi lại toàn bộ function component) -> lần này `count` = giá trị mới vừa set -> JSX trả về hiển thị giá trị mới trên màn hình
- Quá trình này lặp lại liên tục mỗi khi setter function được gọi - phải hình dung component được gọi đi gọi lại nhiều lần, không phải chỉ chạy 1 lần duy nhất
- Có thể tùy chỉnh mức tăng mỗi lần bằng cách đổi giá trị truyền vào setter, ví dụ `setCount(count + 5)` thay vì `+ 1`

**5 điều quan trọng cần nhớ về State:**
1. Dùng State System khi cần cập nhật nội dung hiển thị trên màn hình
2. Gọi `useState` mỗi khi muốn định nghĩa 1 piece of state mới trong component
3. Argument đầu tiên truyền vào `useState` là giá trị khởi tạo (initial value)
4. Cách DUY NHẤT để cập nhật state là gọi setter function
5. Mỗi lần gọi setter function, React sẽ re-render (gọi lại) component đó

## Why Array Destructuring?
- Array Destructuring là feature của JavaScript thuần, không riêng gì React
- Cách viết dài (không dùng destructuring): lấy cả mảng ra rồi truy cập từng phần tử qua index, ví dụ `myArray[0]`, `myArray[1]`
- Cách viết ngắn (dùng destructuring): `const [firstElement, secondElement] = makeArray()` - hoàn toàn tương đương nhưng gọn hơn
- Dấu ngoặc vuông vế trái KHÔNG tạo ra 1 mảng mới - nó chỉ báo JS: coi bên phải là 1 mảng, rồi lấy lần lượt từng phần tử gán vào các biến mới theo đúng thứ tự
- `useState(...)` thực chất trả về 1 MẢNG gồm 2 phần tử: phần tử đầu là giá trị hiện tại của state, phần tử sau là setter function
- Lý do useState trả về mảng (thay vì object): vì với mảng, có thể tự do đặt tên biến bất kỳ khi destructure (`const [count, setCount]`, hay `const [value, setValue]`...) mà không cần viết thêm gì
- Nếu useState trả về object thay vì mảng, sẽ phải viết dài hơn nhiều để đổi tên field (phải dùng object destructuring với cú pháp `oldName: newName`, hoặc truy cập qua `.property`) - bất tiện hơn hẳn khi cần khai báo nhiều piece of state với tên khác nhau
- Đây là câu hỏi hay gặp trong phỏng vấn xin việc: "Tại sao useState dùng array destructuring?"

## Random
- Tạo hàm `getRandomAnimal()`: có 1 mảng chứa các animal string cố định (bird, cat, cow, dog, gator, horse), lấy ngẫu nhiên 1 phần tử bằng `Math.floor(Math.random() * array.length)`
- `Math.random()` trả về số thập phân từ 0 đến gần 1, nhân với `array.length` rồi làm tròn xuống (`Math.floor`) để ra index hợp lệ trong mảng
- Piece of state mới: `const [animals, setAnimals] = useState([])` - khởi tạo là mảng rỗng, dùng để lưu danh sách animal đã thêm
- KHÔNG BAO GIỜ update state trực tiếp (không viết `animals.push(...)` hay `animals = ...`) - vì React không biết bạn vừa đổi state nếu không gọi qua setter function
- Cách thêm phần tử mới vào mảng state đúng chuẩn: `setAnimals([...animals, getRandomAnimal()])` - dùng spread operator (`...animals`) để copy toàn bộ phần tử cũ sang mảng MỚI, rồi thêm phần tử mới vào cuối
- Lý do không dùng `.push()`: vì `.push()` sẽ thay đổi (mutate) trực tiếp mảng state cũ thay vì tạo ra mảng mới - vi phạm nguyên tắc không được mutate state trực tiếp (sẽ học chi tiết hơn ở bài sau)
- Khi return 1 mảng string trực tiếp trong JSX (`{animals}`), React chỉ in nối các string lại với nhau, không xuống dòng/không có style riêng - cần custom component sau này để hiển thị đẹp hơn

## List Building in React
- Nhận prop bằng object destructuring: `function AnimalShow({ type })` - cách phổ biến hơn so với nhận toàn bộ `props` object rồi truy cập `props.type`
- Import component con vào file cha: `import AnimalShow from './AnimalShow'` - dùng `./` vì cùng thư mục với file đang import
- Dùng `map()` (hàm built-in của JavaScript, không riêng React) để biến 1 mảng dữ liệu thành 1 mảng component
- Cách viết: `animals.map((animal, index) => { return <AnimalShow type={animal} key={index} /> })` - tham số đầu là từng phần tử trong mảng gốc, tham số thứ 2 là index
- `map()` chạy hàm transform cho từng phần tử trong mảng gốc, lấy giá trị return của hàm đó gộp vào 1 mảng mới cùng độ dài - không thay đổi mảng gốc
- Biến kết quả (`renderedAnimals`) là 1 mảng chứa các component JSX, in ra trong JSX bằng `{renderedAnimals}`
- Bắt buộc phải truyền prop `key` (thường dùng `index`) khi render list bằng map - đây là cách React

## Loading and Showing SVGs
- Import từng file SVG như import ảnh thông thường (phải giữ đuôi `.svg`): `import bird from './svg/bird.svg'`
- Tạo object `svgMap` để map từ type string (như "cat", "dog") sang biến SVG tương ứng đã import
- Dùng shorthand property của JS khi key và value cùng tên: viết `{ bird, cat, cow, dog, gator, horse }` tương đương với `{ bird: bird, cat: cat, ... }`
- Lấy đúng SVG cần hiển thị bằng cách tra cứu động qua bracket notation: `svgMap[type]` - `type` là string nhận từ prop (`"cat"`, `"dog"`...), dùng để lookup value tương ứng trong object
- Hiển thị SVG lấy được vào thẻ `<img src={svgMap[type]} alt="animal" />`
- Kỹ thuật "map string sang giá trị tương ứng qua object" là JavaScript thuần, không riêng gì React - dùng khi cần chọn 1 trong nhiều giá trị dựa theo 1 chuỗi động

## Increasing Image Size
- Một event handler duy nhất có thể xử lý click cho nhiều phần tử con: gắn `onClick` lên phần tử cha (div bao ngoài) thay vì gắn riêng cho từng img bên trong
- Style động dựa trên state: `style={{ width: 10 + 10 * clicks }}` - object style có thể chứa biểu thức tính toán từ state
- CSS numeric value bắt buộc phải có đơn vị (px, %, em...) khi ở dạng string; nếu truyền số nguyên không có đơn vị, style sẽ không hoạt động
- Inline style trong JSX luôn có 2 lớp `{{ }}`: lớp ngoài là JS expression, lớp trong là object literal chứa các CSS property (viết theo camelCase, không phải kebab-case)

## Adding Custom CSS
- Tổ chức CSS theo module: mỗi component có file CSS riêng cùng tên (App.js <-> App.css, AnimalShow.js <-> AnimalShow.css) - dễ quản lý hơn khi project lớn dần, thay vì gộp chung 1 file
- File CSS chỉ có hiệu lực khi được import vào file JS tương ứng: `import './App.css'` - chỉ cần import, không cần gán vào biến vì mục đích là load CSS vào trình duyệt, không lấy giá trị gì từ nó
- Import CSS phải đặt ở component sẽ dùng CSS đó, mỗi component tự import file CSS riêng của mình
- Quy trình viết CSS thủ công luôn theo 3 bước:
    1) thêm `className` cho phần tử JSX cần style,
    2) viết selector khớp className đó trong file CSS,
    3) viết rule style bên trong selector

## Finalizing Styling
- Selector không có dấu chấm (ví dụ `button { ... }`) sẽ áp dụng cho MỌI phần tử cùng loại thẻ trong toàn app, không cần className
- Flexbox layout để sắp xếp danh sách phần tử: `display: flex; flex-direction: row; flex-wrap: wrap; justify-content: center;` - cho phép các phần tử tự xuống dòng khi không đủ chỗ và canh giữa
- Một className cha có thể không thấy hiệu ứng rõ ràng ngay, chỉ phát huy tác dụng khi kết hợp với style của các phần tử con bên trong nó
- Để định vị 1 phần tử con dựa theo phần tử cha: cha đặt `position: relative`, con đặt `position: absolute` kèm toạ độ (`bottom`, `right`...) - con sẽ định vị theo cha thay vì theo toàn trang


---

# Section 5 - Using an API with React

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


---

# Section 6 - How to Handle Forms

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
- Kỹ thuật này cũng dùng được để chèn vào đầu hoặc cuối mảng, nhưng phức tạp hơn hẳn so với cách spread đơn giản  (`[newItem, ...array]` hoặc `[...array, newItem]`) - nên chỉ cần dùng `slice` khi thực sự cần chèn vào giữa

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
- key khi map list dùng `book.id` (ID đã có sẵn từ data) - đúng theo yêu cầu key phải duy nhất và ổn định 
- Đây là ví dụ thực tế hoàn chỉnh của luồng: App (state) → List (nhận props, map) → Show (nhận props, hiển thị) - luồng dữ liệu 1 chiều cha xuống con qua nhiều cấp component


---

# Section 7 - Data Persistence with API Requests

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


---

# Section 8 - Communication Using the Context System

## Introducing the Context System
- Vấn đề với props hiện tại: truyền callback/data xuyên qua nhiều tầng component (prop drilling) - không sai, chỉ tẻ nhạt và dễ nhầm lẫn khi số tầng component tăng lên
- Context: cơ chế chia sẻ dữ liệu (số, string, mảng, object, function...) cho nhiều component cùng lúc mà KHÔG cần truyền qua từng tầng props trung gian - các component có thể "lấy trực tiếp" dữ liệu dù không có quan hệ cha-con trực tiếp
- Context KHÔNG thay thế props: vẫn dùng props song song để customize từng component riêng lẻ; cũng KHÔNG thay thế state management library như Redux - Context chỉ là "kênh truyền tải" dữ liệu, không quan tâm cách tổ chức dữ liệu (đó là việc của Redux)
- 3 bước setup Context:
  1. **Tạo context**: `const BookContext = createContext()` (import `createContext` từ React) - trả về 1 object có 2 property là `Provider` và `Consumer` (component `Consumer` ít dùng)
  2. **Cung cấp dữ liệu (Provider)**: bọc các component cần chia sẻ dữ liệu bên trong `<BookContext.Provider value={...}>...</BookContext.Provider>` - prop `value` chứa đúng dữ liệu muốn share, có thể là bất kỳ kiểu dữ liệu nào; chỉ những component nằm BÊN TRONG Provider mới truy cập được dữ liệu đó. Pattern phổ biến: đặt Provider bao trọn App component ở tầng ngoài cùng để toàn bộ app đều truy cập được
  3. **Sử dụng dữ liệu (Consume)**: trong component cần lấy dữ liệu, import `useContext` từ React và chính context object đã tạo, gọi `const value = useContext(BookContext)` - trả về đúng giá trị đã truyền vào prop `value` ở bước 2

## Practicing Context - Sharing a Simple Value
- Quy ước tổ chức: tạo thư mục riêng (VD `src/context/`) chứa các file định nghĩa context object, mỗi context 1 file riêng
- Bước 1 - tạo context object trong file riêng: `import { createContext } from 'react'; const BooksContext = createContext(); export default BooksContext;`
- Bước 2 - bọc Provider ở tầng cao nhất của app (thường ở `index.js`, nơi render component App): thay vì render trực tiếp `<App />`, bọc nó bằng `<BooksContext.Provider value={...}><App /></BooksContext.Provider>` - chỉ những component nằm BÊN TRONG cặp thẻ Provider mới truy cập được giá trị đó, component nằm ngoài phạm vi Provider không truy cập được
- Bước 3 - lấy dữ liệu ở bất kỳ component con nào (dù ở tầng sâu bao nhiêu, không cần truyền qua props trung gian): `import { useContext } from 'react'; import BooksContext from '../context/books'; const value = useContext(BooksContext);`
- Hạn chế của cách làm cơ bản này: giá trị `value` truyền vào Provider đang bị HARDCODE cố định - muốn đổi giá trị chia sẻ phải tự tay sửa code, chưa có cách nào để giá trị đó thay đổi động (sẽ cần kết hợp thêm state ở bài sau để giải quyết vấn đề này)

## Implementing the Dynamic Context (Full Working Example)
- File context (VD `BooksContext.js`) có 2 export riêng biệt: default export là context object (dùng cho `useContext`), named export là component `Provider` tự viết (dùng để bọc app) - import cả 2 cùng lúc: `import BooksContext, { Provider } from './context/books'`
- Component `Provider` tự viết: khai báo state (`useState`), khai báo function thay đổi state đó, gom cả 2 vào 1 object (VD `valueToShare`), rồi render Provider gốc với `value={valueToShare}` và hiển thị `children` bên trong (để render đúng những gì được đặt lồng bên trong nó)
- Ở `index.js` (hoặc nơi root app): chỉ cần import và dùng component `Provider` tự viết, không cần import context object hay set `value` thủ công nữa - vì `Provider` tự viết đã tự lo phần đó bên trong
- Ở component con cần dùng dữ liệu: `const { count, incrementCount } = useContext(BooksContext)` - dùng object destructuring để lấy trực tiếp từng property cần thiết từ object trả về, thay vì lấy nguyên object rồi truy cập qua dấu chấm
- Luồng hoạt động đầy đủ: component con gọi function lấy từ context (VD `incrementCount()`) → function đó gọi setter cập nhật state NẰM BÊN TRONG Provider tự viết (ở tầng cao nhất) → Provider re-render → giá trị mới được truyền lại vào `value` của Provider

## Application State vs Component State
- 2 khái niệm dùng để PHÂN LOẠI state theo mức độ quan trọng/phạm vi ảnh hưởng - không phải tính năng mới của React, không thay đổi cách viết/gọi state, chỉ là cách tư duy để quyết định state nào nên đưa vào Context
- **Application state**: state là "trọng tâm chính" của cả ứng dụng, được nhiều component khác nhau cần dùng đến (VD mảng `books` trong app quản lý sách - gần như mọi component đều quan tâm)
- **Component state** (còn gọi là local state): state chỉ phục vụ 1 component (hoặc rất ít component liên quan trực tiếp), không component nào khác cần biết đến (VD `title` đang gõ trong form tạo/sửa sách, `showEdit` để toggle form ẩn/hiện)
- Cách phân loại không có quy tắc tuyệt đối, mang tính chủ quan - các developer khác nhau có thể phân loại khác nhau cho cùng 1 trường hợp, đây là điều bình thường
- Quy tắc thực hành: application state thường NÊN đưa vào Context để mọi nơi trong app đều .truy cập dễ dàng; component state thường KHÔNG cần đưa vào Context vì không ai khác cần dùng tới, giữ nguyên `useState` cục bộ trong component đó là đủ
- Ví dụ áp dụng vào app Reading List: `books` (ở App) → application state, nên đưa vào Context kèm các hàm thao tác (`createBook`, `editBookById`, `deleteBookById`...); `title` (ở BookCreate/BookEdit) và `showEdit` (ở BookShow) → component state, giữ nguyên cục bộ
- Khái niệm này sẽ trở nên rất quan trọng khi học tới Redux sau này (Redux về bản chất là 1 cách quản lý tập trung cho "application state")

## Refactoring to Use Context
- Chuyển 1 piece of state (đã xác định là application state) từ component cha sang Context KHÔNG chỉ đơn giản là "dời code" - phải làm thêm 1 bước quan trọng: xoá bỏ toàn bộ việc truyền state/function đó qua props ở MỌI tầng component đã từng nhận nó, và đổi các component đó sang lấy trực tiếp từ Context (`useContext`) thay vì nhận qua props
- Refactor này ảnh hưởng tới nhiều component cùng lúc: mỗi component từng nhận props liên quan (VD `books`, `createBook`, `editBookById`, `deleteBookById`) đều cần sửa lại - bỏ phần nhận qua props, thêm phần lấy qua `useContext`
- Đây là quá trình thực tế thường gặp trong dự án thật: hiếm khi thiết kế đúng ngay từ đầu, việc refactor lại cấu trúc data flow (chuyển từ props sang context, hoặc ngược lại) là chuyện bình thường và cần thiết khi app phát triển
- Cách tiếp cận khi refactor: làm từng bước nhỏ, sửa từng phần rồi test lại app vẫn chạy đúng, thay vì cố sửa toàn bộ 1 lần - tránh dồn quá nhiều thay đổi cùng lúc dễ gây lỗi khó tìm

## A Small Taste of Reusable Hooks
- Hook: function do React cung cấp, thêm khả năng đặc biệt cho component, LUÔN bắt đầu bằng tiền tố `use` (VD `useState`, `useEffect`, `useContext`) - đây là quy ước nhận diện hook, không phải chỉ là tên gọi ngẫu nhiên
- Custom hook: hook TỰ VIẾT (không phải React cung cấp sẵn), dùng để gói gọn 1 đoạn logic có thể tái sử dụng ở nhiều component - đây là 1 trong những cách chính để tái sử dụng code trong React (bên cạnh tái sử dụng qua component)
- Custom hook đơn giản nhất chỉ là 1 function bọc quanh 1 hoặc nhiều hook có sẵn, ẩn bớt phần lặp lại (boilerplate) - VD gộp `useContext(BooksContext)` thành `useBooksContext()`
- Quy trình tạo custom hook: viết 1 function nhỏ gọi hook có sẵn bên trong, return kết quả ra ngoài, rồi tách function đó ra 1 file riêng (thường đặt trong thư mục `src/hooks/`, đặt tên file khớp tên hook)
- Lợi ích cụ thể: những nơi cần dùng context không phải viết lặp lại 2 dòng import (`useContext` + context object) ở mọi file, chỉ cần import đúng 1 custom hook và gọi nó - giảm boilerplate, dễ nhớ, dễ đọc code hơn
- Custom hook có thể đơn giản (chỉ vài dòng như ví dụ này) hoặc phức tạp (chứa nhiều logic hơn) - độ phức tạp tuỳ vào nhu cầu tái sử dụng thực tế của dự án


---

# Section 9 - Deeer Dive into Hooks

## Return to useEffect - The Stale Reference Bug
- useEffect có 3 khía cạnh "khó nhằn" cần hiểu sâu:
  1) khi nào arrow function được gọi
  2) có thể return gì từ arrow function đó
  3) khái niệm "stale variable reference" (biến bị "cũ", không cập nhật) - đây là chủ đề hay gây bug thực tế và hay bị hỏi khi phỏng vấn xin việc React
- Demo bug: đặt event listener trực tiếp lên `document.body` (thay vì lên 1 JSX element cụ thể) bên trong `useEffect(() => {...}, [])` để bắt click ở bất kỳ đâu trên trang - đây là cách làm KHÔNG chuẩn trong React (chỉ để minh hoạ), nhưng giúp lộ ra vấn đề rõ ràng
- Hiện tượng lỗi: khi click tăng counter lên rồi click chỗ khác trên trang, console.log bên trong handler đó vẫn in ra `0` (giá trị counter lúc `useEffect` chạy lần đầu tiên) chứ không phải giá trị hiện tại thực sự đang hiển thị trên màn hình
- Nguyên nhân sẽ được giải thích ở bài tiếp theo - đây là dấu hiệu của "stale reference": function bên trong `useEffect` (khi dùng mảng dependency rỗng, chỉ chạy 1 lần) đã "chụp" (capture) giá trị biến tại đúng thời điểm nó được tạo ra lần đầu, và không tự cập nhật theo các lần re-render sau đó dù biến đó đã đổi giá trị trong state thực tế

## Understanding & Fixing the Stale Reference Bug
- Bug "stale variable reference" có thể xảy ra bất cứ khi nào arrow function trong `useEffect` chứa (hoặc TẠO RA) 1 function khác có tham chiếu tới 1 biến ngoài - không quan trọng function đó được định nghĩa trực tiếp bên trong `useEffect` hay được định nghĩa ở ngoài rồi mới gán vào trong (VD `document.body.onclick = onClick`) - cả 2 trường hợp đều bị lỗi giống nhau
- Đây là bug CỰC KỲ phổ biến với `useEffect`, ai cũng gặp - nên Create React App tích hợp sẵn công cụ ESLint để cảnh báo (dòng gạch chân vàng dưới dependency array) khi phát hiện code có nguy cơ dính bug này
- KHÔNG NÊN làm theo cảnh báo ESLint 1 cách mù quáng - phải hiểu rõ nguyên nhân, vì đôi khi làm đúng theo gợi ý của ESLint lại gây ra bug khác (sẽ minh hoạ ở bài sau)
- Cách fix trong ví dụ này: thêm biến bị "stale" (VD `counter`) vào dependency array của `useEffect` -> `useEffect(() => {...}, [counter])`
- Lý do cách fix này đúng: khi thêm biến vào dependency array, mỗi lần biến đó đổi giá trị (dẫn tới re-render), arrow function bên trong `useEffect` sẽ CHẠY LẠI TỪ ĐẦU - tạo ra 1 function MỚI hoàn toàn (dù nhìn code giống hệt lần trước), và function mới này sẽ tham chiếu tới giá trị MỚI NHẤT của biến, không còn "kẹt" ở giá trị cũ từ lần chạy đầu tiên nữa
- Bản chất: mỗi lần `useEffect` chạy lại, toàn bộ code + closure bên trong nó được tạo mới hoàn toàn trong bộ nhớ - không phải "cập nhật" function cũ, mà là tạo hẳn 1 function mới với tham chiếu đúng tới giá trị hiện tại

## ESLint is Good, but be Careful!
- ESLint KHÔNG SAI khi cảnh báo thêm biến vào dependency array, nhưng nó không "nhìn thấy" toàn bộ data flow thực tế trong app - làm theo cảnh báo mù quáng có thể tạo ra bug mới, không phải fix bug
- Case cụ thể: thêm `fetchBooks` vào dependency array của `useEffect` trong App component -> warning biến mất, nhưng app rơi vào vòng lặp vô hạn gọi API liên tục (kiểm tra được qua Network tab)
- Nguyên nhân gốc: MỖI LẦN component Provider re-render, function `fetchBooks` được ĐỊNH NGHĨA LẠI TỪ ĐẦU - dù cùng tên biến, cùng logic bên trong, nhưng về mặt kỹ thuật đây là 1 function HOÀN TOÀN MỚI trong bộ nhớ (địa chỉ tham chiếu khác), không phải cùng 1 function được tái sử dụng
- Vì `fetchBooks` nằm trong dependency array của `useEffect`, và mỗi lần Provider re-render lại tạo ra 1 `fetchBooks` mới về reference -> React coi đây là "giá trị đã thay đổi" -> chạy lại arrow function trong `useEffect` -> gọi lại `fetchBooks` -> gọi API -> update state -> Provider re-render lại -> lặp lại chu trình -> vòng lặp vô hạn
- Bài học cốt lõi: khi 1 FUNCTION được tạo mới ở mỗi lần render (không phải giá trị nguyên thuỷ như number/string), đưa function đó vào dependency array rất dễ gây ra vòng lặp vô hạn re-render, vì reference của nó luôn "đổi" theo mỗi lần component chạy lại dù logic bên trong y hệt
- Hướng giải quyết đúng đắn: KHÔNG bỏ `fetchBooks` ra khỏi dependency array (vì làm vậy sẽ khiến app "hoạt động tạm ổn" hiện tại nhưng dễ vỡ về sau nếu implementation của `fetchBooks` thay đổi) - mà cần tìm kỹ thuật khác để đảm bảo `fetchBooks` KHÔNG bị tạo mới ở mỗi lần render (sẽ học ở bài tiếp theo, liên quan tới `useCallback`)

## Stable References with useCallback
- `useCallback` là 1 hook có nhiệm vụ DUY NHẤT: cung cấp 1 "stable reference" (tham chiếu ổn định, không đổi) cho 1 function qua các lần render - không tự chạy function đó, không thêm tính năng gì mới, chỉ khắc phục vấn đề "function bị tạo mới ở mỗi lần render" đã gây ra bug infinite loop ở bài trước
- Cú pháp: `const stableFn = useCallback(myFunction, [])` - argument 1 là function, argument 2 là dependency array BẮT BUỘC phải có (khác `useEffect`, không được bỏ trống)
- Hành vi ở LẦN RENDER ĐẦU TIÊN: `useCallback` chỉ đơn giản trả về đúng function vừa truyền vào, không làm gì đặc biệt
- Hành vi ở CÁC LẦN RE-RENDER SAU: nếu dependency array là mảng RỖNG `[]`, `useCallback` sẽ BỎ QUA function mới vừa được tạo lại ở lần render này, và trả về NGUYÊN function đã lưu từ lần render đầu tiên - function mới tạo ra coi như bị "vứt bỏ", không được dùng
- Kết quả: dù component re-render bao nhiêu lần, biến nhận từ `useCallback` (với dependency `[]`) luôn trỏ tới ĐÚNG 1 function duy nhất trong bộ nhớ (từ lần render đầu tiên) - reference không bao giờ đổi
- Cách áp dụng để fix bug infinite loop: bọc `fetchBooks` bằng `useCallback(fetchBooks, [])` ở component tạo ra nó (Provider), rồi share/dùng phiên bản "stable" đó thay vì function gốc - khi đưa vào dependency array của `useEffect` ở nơi khác, React sẽ thấy reference không đổi qua các lần render -> không kích hoạt chạy lại -> hết vòng lặp vô hạn

## useEffect Cleanup Functions - Cú pháp & Cơ chế
- `useEffect` chỉ được phép return đúng 1 kiểu duy nhất: 1 FUNCTION (gọi là cleanup function) - không được return number, string, hay bất kỳ giá trị nào khác
- Không được dùng `async`/`await` trực tiếp trên function truyền vào `useEffect`, vì hàm async luôn tự động return 1 Promise (do JavaScript quy định), mà `useEffect` chỉ chấp nhận return function hoặc không return gì cả
- Cú pháp: `useEffect(() => { ...code...; return () => { ...cleanup code... }; }, [deps])`
- Cleanup function được React tự động gọi vào đúng 1 thời điểm: NGAY TRƯỚC KHI arrow function chính của `useEffect` chuẩn bị chạy lại ở lần re-render tiếp theo
- Cleanup function CHỈ được gọi khi arrow function chính thực sự sắp chạy lại - nếu dependency array khiến arrow function không bao giờ chạy lại (VD mảng rỗng `[]`), cleanup function cũng sẽ không bao giờ được gọi
- Nếu bỏ hẳn dependency array (không truyền argument 2), arrow function chạy lại ở MỌI lần re-render -> cleanup function cũng sẽ được gọi trước MỖI lần re-render tương ứng
- Mục đích thực tế của cleanup function: dọn dẹp/huỷ bỏ những gì effect trước đó đã thiết lập (VD event listener, subscription, timer...) trước khi effect mới được thiết lập lại - tránh rò rỉ tài nguyên (memory leak) hoặc side effect bị nhân đôi qua các lần render

## The Purpose of Cleanup Functions (Ứng dụng thực tế)
- Cách gắn event listener lên `body` an toàn hơn `document.body.onclick = fn`: dùng `document.addEventListener('click', listener)` - cho phép nhiều listener cùng tồn tại đồng thời trên cùng 1 phần tử (khác với gán trực tiếp `.onclick`, chỉ giữ được 1 listener tại 1 thời điểm)
- Vấn đề khi dùng `addEventListener` bên trong `useEffect` mà KHÔNG có cleanup: mỗi lần component re-render, `useEffect` chạy lại và tạo thêm 1 listener MỚI, các listener cũ vẫn còn tồn tại (không tự mất đi) - dẫn tới hàng chục, hàng trăm listener chồng chất, mỗi lần click sẽ log ra nhiều dòng trùng lặp
- Cách fix bằng cleanup function: return 1 function gọi `document.body.removeEventListener('click', listener)` từ bên trong `useEffect` - function này sẽ được React tự động gọi để gỡ bỏ listener CŨ ngay trước khi `useEffect` chạy lại và tạo listener MỚI ở lần re-render tiếp theo
- Kết quả: mỗi lần re-render, luôn đảm bảo gỡ đúng listener cũ trước khi thêm listener mới → tại mọi thời điểm chỉ có đúng 1 listener đang hoạt động, không bị nhân bản
- Trên thực tế thường KHÔNG cần đặt tên riêng cho cleanup function (VD không cần biến trung gian `cleanUp`) - chỉ cần return trực tiếp function xử lý dọn dẹp ngay tại chỗ, ngắn gọn hơn
- Use case phổ biến nhất của cleanup function: bất cứ khi nào set up 1 side effect thủ công trên DOM (event listener thủ công, subscription...) mà không thể làm qua JSX props thông thường (VD `onClick`) - thường gặp khi build các component như dropdown, modal (sẽ gặp lại kỹ thuật này ở các app sau trong khoá học)


---

# Section 10 - Custom Navigation and Routing Systems

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


---

# Section 11 - Mastering the State Design Process

## State Design Process Overview (8 bước thiết kế state)
Quy trình 8 bước để xác định: cần bao nhiêu piece of state, tên/kiểu dữ liệu của nó, và nên đặt ở component nào - kèm xác định luôn các event handler cần thiết.

**Phần 1 - Xác định cần state hay event handler gì (bước 1-3):**
1. Viết ra bằng lời văn: user tương tác gì (click, gõ...) và nội dung màn hình thay đổi ra sao sau mỗi tương tác, dựa theo mockup
2. Phân loại từng dòng vừa viết: hành động của user → cần EVENT HANDLER; thay đổi hiển thị trên màn hình → cần STATE
3. Gom nhóm các bước giống/tương tự nhau, loại bỏ trùng lặp, khái quát hoá mô tả còn lại cho gọn và tổng quát hơn

**Phần 2 - Xác định tên và KIỂU DỮ LIỆU của state (bước 4-7):**
- Nguyên tắc ưu tiên: nên chọn state là number, boolean, hoặc string - TRÁNH dùng array/object cho state nếu có thể, vì array/object khó thao tác đúng cách hơn (nhớ lại rule không mutate)
4. Từ mockup, loại bỏ phần KHÔNG thay đổi qua các tương tác (chỉ giữ lại phần thực sự biến động)
5. Thay các phần còn lại bằng mô tả text tối giản nhất có thể (VD "expanded"/"collapsed" thay vì mô tả chi tiết UI)
6. Lặp lại bước 4-5 với 1 biến thể khác của mockup (VD đổi item nào đang active) để có thêm 1 bộ kết quả so sánh
7. Từ các bộ kết quả text đã thu được ở bước 5-6, "làm ngược" (giống giải phương trình): tưởng tượng viết 1 function nhận props + 1 argument phụ, sao cho argument đó đủ để function trả đúng các kết quả text đã ghi - kiểu dữ liệu và tên gọi hợp lý của argument đó chính là gợi ý cho state cần tạo (VD 1 con số `expandedIndex` cho biết vị trí item đang mở)

**Phần 3 - Xác định VỊ TRÍ đặt state (bước 8):**
- Câu hỏi cốt lõi: có component nào KHÁC (ngoài component đang thiết kế) có NHU CẦU HỢP LÝ cần biết giá trị state này không?
  - CÓ → đặt state ở component CHA chung, rồi truyền xuống các con qua props (vì sibling components không giao tiếp trực tiếp được với nhau)
  - KHÔNG → đặt state ngay bên trong chính component đó (local state), không cần đẩy lên cao hơn
- Event handler nên được ĐỊNH NGHĨA (không phải "dùng") ở CÙNG component với state mà nó thay đổi - dù có thể truyền xuống dùng ở component con khác qua props
- Quy trình này đặc biệt hữu ích khi thiết kế component PHỨC TẠP - với component đơn giản có thể thấy hơi rườm rà, nhưng giá trị thực sự phát huy khi bài toán khó hơn

## Finding the Expanded Item
- Dùng useState(0) cho expandedIndex, khởi tạo item đầu tiên mở sẵn
- Trong map((item, index) => ...), so sánh index === expandedIndex để biết item hiện tại có đang expanded hay không
- Nên tạo biến trung gian isExpanded = index === expandedIndex để tránh lặp lại phép so sánh nhiều lần trong cùng 1 lần map, dùng chung cho cả việc show/hide content lẫn hiển thị icon

## Conditional Rendering
- React KHÔNG render Boolean, null, hay undefined ra màn hình - trả về các giá trị này trong JSX sẽ không hiển thị gì cả (khác với string/number, luôn hiển thị)
- Nhắc lại quy tắc short-circuit của JS: || trả về giá trị TRUTHY đầu tiên; && trả về giá trị FALSY đầu tiên (nếu có) hoặc giá trị TRUTHY cuối cùng
- Kết hợp 2 quy tắc trên để ẩn/hiện HOÀN TOÀN 1 phần JSX (không chỉ dùng CSS ẩn mà là không render luôn): {isExpanded && <div>{item.content}</div>}
  - Nếu isExpanded là true → trả về div (giá trị truthy cuối)
  - Nếu isExpanded là false → trả về false (giá trị falsy đầu) → React không hiển thị gì
- Đây gọi là "conditional rendering" - kỹ thuật cực kỳ phổ biến, sẽ dùng liên tục trong mọi component React

## Understanding the Issue (Stale State Bug khi Click Nhanh)
- Bug minh hoạ: click 2 lần rất nhanh (mô phỏng bằng $0.click(); $0.click(); trong console) vào cùng accordion header có thể khiến panel bị đóng sai, không mở lại như mong đợi
- Nguyên nhân gốc: React KHÔNG update state ngay lập tức khi gọi setter function - có 1 độ trễ nhỏ (gọi là batching, giúp gom nhiều lần update lại xử lý cùng lúc để tối ưu hiệu năng)
- Nếu click lần 2 xảy ra TRƯỚC KHI React kịp xử lý xong update của lần click 1, thì trong lần click 2, biến state đọc được vẫn là giá trị CŨ (chưa cập nhật) - dẫn tới logic tính toán bị sai vì dựa trên dữ liệu đã "cũ" (stale)
- Có thể verify bug này bằng cách thêm console.log giá trị state ngay đầu event handler - sẽ thấy nó không đổi dù đã gọi setter trước đó

## Applying the Fix (Functional State Updates)
- 2 hướng giải quyết bug stale state: (1) ép React update ngay lập tức (không nên, làm mất lợi ích tối ưu của batching), (2) dùng functional update - đây là cách được chọn
- Cú pháp thông thường (dễ dính bug khi update dựa trên giá trị cũ): setCounter(counter + 1)
- Cú pháp functional update: setCounter(currentCounter => currentCounter + 1) - truyền vào 1 FUNCTION thay vì giá trị trực tiếp
- Quy tắc quan trọng: argument đầu tiên của function này LUÔN LUÔN là giá trị mới nhất, cập nhật nhất của state đó (kể cả khi có nhiều lần gọi setter xếp hàng chờ xử lý) - không bị stale
- Chỉ cần RETURN giá trị mới mong muốn từ trong function đó, không cần gọi lại setter thêm lần nào khác
- Khi nào nên dùng functional update: chỉ cần cân nhắc khi giá trị state MỚI phụ thuộc vào giá trị state CŨ (VD tăng/giảm dựa trên giá trị hiện tại, so sánh điều kiện dựa trên giá trị hiện tại) - đây chính là trường hợp dễ dính bug stale value
- Trên thực tế, bug này RẤT HIẾM xảy ra với user thật (vì không ai click nhanh tới mức đó) - nên phần lớn code thực tế vẫn dùng cách đơn giản; chỉ cần cân nhắc functional update khi app phức tạp, có nhiều state tương tác với nhau


---

# Section 12 - Practicing Props and State Design

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


---

# Section 13 - Making Navigation Reusable

## Theory of Navigation in React
- Chia navigation thành 2 kịch bản riêng biệt:
    1) user LẦN ĐẦU vào app (gõ URL/click link từ site khác)
    2) user ĐÃ Ở TRONG app rồi điều hướng tiếp (click link nội bộ, back/forward, hoặc điều hướng bằng code)
- Kịch bản 1: server LUÔN trả về cùng 1 file index.html bất kể URL nào được yêu cầu - file HTML này load bundle.js, React app khởi động, TỰ ĐỌC route hiện tại từ address bar, rồi áp dụng routing rules để quyết định hiển thị component nào
- Kịch bản 2: khi user click 1 link nội bộ, phải CHẶN hành vi điều hướng mặc định của trình duyệt (không cho nó tự động request 1 HTML document mới) - thay vào đó, tự xử lý bằng JavaScript: cập nhật address bar thủ công + áp dụng lại routing rules để đổi component hiển thị, hoàn toàn KHÔNG reload trang
- Lợi ích cốt lõi của cách làm này: vì JavaScript environment KHÔNG bị reset khi điều hướng nội bộ, state (kể cả data đã fetch từ API trước đó) vẫn được GIỮ NGUYÊN khi user quay lại 1 trang đã xem trước đó (VD bấm back) - giúp hiển thị lại ngay lập tức, không cần fetch lại API
- Điều kiện để lợi ích trên hoạt động đúng: state phải được đặt ở component CHA (hoặc trong Context) chứ không phải trong chính component bị ẩn/hiện đó - vì khi 1 component bị gỡ khỏi màn hình, state cục bộ bên trong nó sẽ MẤT HẲN theo mặc định
- Mọi thư viện routing thực tế (React Router và tương tự) đều hoạt động theo đúng nguyên lý này bên dưới - không có gì thần bí, chỉ là áp dụng các kỹ thuật React đã biết (event handler, useEffect, Context) theo đúng pattern

## The PushState Function
- KHÔNG dùng `window.location = url` để đổi URL trong app kiểu SPA - cách này LUÔN gây full page refresh (reload toàn bộ trang, mất hết JS state)
- Dùng `window.history.pushState(state, title, path)` thay thế: đổi được URL trên address bar NHƯNG KHÔNG gây reload trang - `state` thường truyền object rỗng `{}`, `title` thường truyền chuỗi rỗng `''`, `path` là đường dẫn mới muốn chuyển tới (chỉ cần path, không cần domain đầy đủ)
- `pushState` tự động khiến nút Back của trình duyệt hoạt động đúng như mong đợi (quay lại URL trước đó), mà vẫn không gây reload trang
- Đây chính là công cụ nền tảng để tự xây dựng bất kỳ hệ thống routing nào cho SPA từ đầu bằng JavaScript thuần

## Handling Back/Forward Buttons
- Khi navigate bằng `pushState`, việc bấm nút Back/Forward của trình duyệt KHÔNG gây full page refresh - đây là hành vi có sẵn của browser, không cần tự code thêm gì để ngăn refresh
- Nhưng bấm Back/Forward chỉ đổi URL trên address bar, KHÔNG tự động cập nhật nội dung hiển thị trên trang - cần tự lắng nghe sự kiện để biết user vừa back/forward
- Sự kiện cần lắng nghe: `popstate`, gắn trên `window`: `window.addEventListener('popstate', handler)`
- `popstate` CHỈ được trigger khi user back/forward tới 1 URL đã được thêm vào history bằng `pushState` trước đó - nếu URL đó là do user tự gõ vào address bar (không qua pushState), sẽ không có popstate, thay vào đó xảy ra full page refresh như bình thường
- Trong handler của `popstate`, dùng `window.location.pathname` để lấy đúng path hiện tại, từ đó cập nhật lại state (VD `currentPath`) để trigger re-render đúng nội dung tương ứng

## Programmatic Navigation
- Phân biệt 2 loại điều hướng: **manual/user-triggered navigation** (user tự click link, bấm back/forward) vs **programmatic navigation** (code TỰ điều hướng user, không do user trực tiếp thao tác - VD tự động đăng xuất và chuyển hướng sau khi hết thời gian không hoạt động)
- Hàm `navigate(to)` tự viết cần làm ĐỦ 2 việc: (1) gọi `pushState` để cập nhật address bar, (2) TỰ TAY cập nhật state `currentPath` bằng setter - vì gọi `pushState` KHÔNG tự động trigger sự kiện `popstate`, nên nếu không tự cập nhật state, component sẽ không re-render dù URL đã đổi
- Đây là điểm khác biệt quan trọng với back/forward: back/forward tự trigger `popstate` (nên cập nhật state trong handler của popstate là đủ), nhưng gọi `pushState` trực tiếp từ code thì KHÔNG - phải tự chủ động cập nhật state song song
- Function `navigate` này nên được share qua Context để bất kỳ component nào trong app cũng gọi được khi cần điều hướng bằng code

## A Link Component
- Component `Link` tự viết dùng để THAY THẾ thẻ `<a>` thường cho MỌI liên kết điều hướng NỘI BỘ trong app - chỉ dùng `<a>` thường khi link trỏ ra ngoài domain khác
- Bên trong vẫn render ra `<a>` thật (để giữ đúng ngữ nghĩa HTML, hỗ trợ accessibility, right-click "mở tab mới"...), nhưng gắn `onClick` để can thiệp hành vi
- Trong handler `onClick`: bắt buộc gọi `event.preventDefault()` ĐẦU TIÊN để chặn hành vi điều hướng/reload mặc định của thẻ `<a>`, sau đó gọi hàm `navigate(to)` lấy từ Context để điều hướng bằng code thay thế
- Đây chính là pattern thực tế mà `<Link>` của React Router (và các thư viện router khác) áp dụng bên dưới - hiểu được cách tự làm giúp hiểu rõ bản chất khi dùng thư viện có sẵn

## A Route Component
- Component `Route` tự viết nhận 2 prop: `path` (đường dẫn cần khớp) và `children` (nội dung sẽ hiển thị nếu khớp)
- Logic bên trong CỰC KỲ đơn giản: lấy `currentPath` từ Context, so sánh với prop `path` - nếu KHỚP thì `return children`, nếu KHÔNG khớp thì `return null` (không hiển thị gì)
- Cách dùng: đặt nhiều `<Route path="...">...</Route>` cạnh nhau trong component cha (thường là App) - tại 1 thời điểm chỉ đúng 1 Route có path khớp `currentPath` sẽ thực sự render nội dung, các Route còn lại tự động render null
- Đây chính là nguyên lý nền tảng bên dưới `<Route>` của React Router - hiểu được giúp không còn thấy routing declarative (`<Route path>`) là "phép màu", mà chỉ là conditional rendering dựa trên so sánh string đơn giản

## Highlighting the Active Link
- Thêm prop `activeClassName` vào component `Link` tự viết - đây là className CHỈ áp dụng khi link đó đang là link "đang active" (trỏ đúng tới trang hiện tại)
- Logic xác định active: so sánh `currentPath` (lấy từ Context) với prop `to` của chính Link đó - nếu 2 giá trị bằng nhau thì thêm `activeClassName` vào danh sách class hiện tại, dùng lại kỹ thuật `classNames()` (thư viện) 
- Pattern tổng quát: `classNames(baseClassName, className, { [activeClassName]: currentPath === to })` hoặc tương tự - style động dựa theo điều kiện so sánh path hiện tại
- Đây là pattern RẤT PHỔ BIẾN trong thực tế cho bất kỳ thanh điều hướng nào (sidebar, navbar, breadcrumb...) cần highlight link đang được xem - áp dụng được với cả router tự viết lẫn khi dùng React Router thật (React Router có sẵn `NavLink` làm y hệt việc này)


---

# Section 14 - Creating Portals with ReactDOM

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


---

# Section 15 - Make a Feature - Full Data Table !

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


---

# Section 16 - Getting Clever with Data Sorting

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


---

# Section 17 - Custom Hooks In Depth

## Custom Hook Creation
- Trigger để nhận biết "code này có thể tách thành custom hook": tìm 1 piece of state trong component, rồi xác định TẤT CẢ các block code liên quan mật thiết tới chính piece of state đó (khai báo state, useEffect liên quan, event handler liên quan) - CHỈ trừ những phần có chứa JSX (luôn giữ lại JSX trong component gốc, không đưa vào hook)
- Custom hook không cần phải "thật sự dùng lại được ở nhiều nơi" mới đáng tách - chỉ cần TÁCH ĐÚNG LOGIC LIÊN QUAN tới 1 state đã giúp component gốc gọn gàng, dễ đọc hơn, đó đã là lợi ích thực tế

## Hook Creation Process in Depth (Quy trình 9 bước tạo Custom Hook)
1. Tạo 1 function mới trong file component, đặt tên tạm `useSomething`
2. Tìm các block code KHÔNG chứa JSX, chỉ liên quan tới 1-2 piece of state cụ thể (khai báo state, useEffect, event handler liên quan)
3. CẮT các block đó ra khỏi component, DÁN vào bên trong function `useSomething`
4. Sau khi cắt dán, sẽ xuất hiện lỗi "not defined" ở CẢ 2 PHÍA - tìm hết các biến bị lỗi "not defined" TRONG COMPONENT (những biến hook cần trả VỀ cho component dùng)
5. Trong hook, RETURN 1 OBJECT chứa đúng các biến/function mà component đang cần (dùng shorthand nếu tên key = tên value)
6. Trong component, GỌI hook đó và DESTRUCTURE object trả về để lấy đúng các biến cần dùng
7. Tìm tiếp các biến bị lỗi "not defined" CÒN LẠI BÊN TRONG HOOK (những giá trị hook cần NHẬN VÀO từ bên ngoài) - đây chính là ARGUMENT cần thêm vào phần khai báo của hook function
8. Đặt tên LẠI cho chính hook function - đổi từ tên tạm `useSomething` thành tên mô tả đúng chức năng, LUÔN bắt đầu bằng tiền tố `use`
9. Đặt tên LẠI cho các property được return từ hook (nếu tên gốc như `handleClick` không đủ rõ nghĩa trong ngữ cảnh hook mới) - nên đổi thành tên mô tả rõ hành động, dễ hiểu cho engineer khác đọc code
- (Bước bổ sung không đánh số): tách hẳn hook function ra 1 FILE RIÊNG (thường đặt trong thư mục `src/hooks/`), import lại vào component - hoàn tất việc reusable hoá

## Making a Reusable Sorting Hook (Áp dụng quy trình vào SortableTable)
- Dấu hiệu nhận biết cần tách hook: khi phát hiện 2 component KHÁC NHAU về UI (VD 1 bảng sortable, 1 list sortable) nhưng có LOGIC RẤT GIỐNG NHAU bên dưới (cùng cần `sortOrder`, `sortBy` state, cùng cần hàm sort dữ liệu dựa trên 1 property) - đây chính là cơ hội để tách phần logic dùng chung ra thành 1 custom hook độc lập, tránh duplicate code khi phải viết lại UI khác cho cùng 1 tính năng
- Áp dụng đúng quy trình 9 bước ở trên vào `SortableTable`: tách 2 state (`sortOrder`, `sortBy`) + hàm xử lý click (chỉ liên quan tới sort, không dính JSX) + toàn bộ logic sort dữ liệu → gộp vào 1 hook mới tên `useSort`
- Hook `useSort` nhận vào 2 argument: `data` (mảng dữ liệu gốc) và `config` (mảng cấu hình cột, cần có `label` và hàm lấy giá trị sort) - trả về object gồm `sortOrder`, `sortBy`, `sortedData` (dữ liệu đã sort xong), và hàm đổi cột đang sort
- Đặt lại tên hàm trả về cho rõ nghĩa: từ `handleClick` (tên mơ hồ, không rõ trong ngữ cảnh hook) đổi thành tên mô tả đúng hành động (VD `setSortColumn`) - giúp engineer khác dùng hook dễ hiểu ngay ý nghĩa mà không cần đọc code bên trong
- Kết quả cuối: `SortableTable` được đơn giản hoá đáng kể (chỉ còn gọi hook + xử lý phần JSX/config header), còn `useSort` trở thành 1 hook ĐỘC LẬP, có thể tái sử dụng cho BẤT KỲ component nào khác cần sort dữ liệu (không chỉ riêng cho bảng) - miễn truyền đúng `data` và `config` theo đúng cấu trúc mong đợi


---

# Section 18 - Into the World of Reducers

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


---

# Section 19 - Dive Into Redux Toolkit

## Into the World of Redux
- Redux: thư viện JS quản lý state cho toàn bộ ứng dụng, dùng CHUNG kỹ thuật với `useReducer` (dispatch, action, reducer) - về bản chất là cùng 1 tư duy, chỉ khác quy mô
- Khác biệt 1 - vị trí state: `useReducer` tạo state SỐNG BÊN TRONG 1 component (và children của nó); Redux tạo 1 object riêng biệt gọi là **Redux Store**, nằm HOÀN TOÀN BÊN NGOÀI cây component - mọi component cần state phải tự "kết nối" tới store
- Thư viện **React Redux**: giúp kết nối Redux Store với React app dễ dàng hơn - 100% optional về kỹ thuật nhưng gần như MỌI dự án thực tế đều dùng. Bản chất bên dưới nó cũng chỉ dùng Context, không có gì "phép màu"
- Khác biệt 2 - số lượng reducer: `useReducer` thường chỉ có 1 reducer duy nhất quản lý hết; Redux thường có NHIỀU reducer khác nhau, mỗi cái phụ trách 1 phần riêng của state (VD reducer cho users, reducer cho videos, reducer cho messages) - nếu dự án chỉ cần đúng 1 reducer, đó là dấu hiệu có thể chưa cần dùng tới Redux
- Khác biệt 3 - độ phức tạp của state object: state trong Redux thường có NHIỀU property con (VD `{ users, videos, messages }`), mỗi property được tạo ra bởi 1 reducer riêng - giúp tách trách nhiệm quản lý state, tránh 1 reducer khổng lồ ôm hết logic

## Redux vs Redux Toolkit
- Lý do pattern dispatch/action/reducer phổ biến: khi app có RẤT NHIỀU component, dispatch function đóng vai trò là "điểm thay đổi trung tâm" duy nhất - giúp engineer dễ hiểu VÌ SAO state đang đổi, DỄ DEBUG (VD chỉ cần console.log mọi action được dispatch là thấy hết lịch sử thay đổi state)
- Nhược điểm của pattern này (Redux thuần/cổ điển): phải viết RẤT NHIỀU boilerplate - định nghĩa constant action types, export/import chúng giữa nhiều file, viết switch statement dài trong reducer - toàn bộ chỉ để "báo" cho reducer biết nó đang được gọi vì lý do gì
- **Redux Toolkit (RTK)**: là 1 THƯ VIỆN BỌC (wrapper) quanh Redux gốc - dùng RTK vẫn là đang dùng Redux, chỉ đơn giản hoá quá trình tạo action type + giảm code boilerplate + tự động sinh switch statement bên dưới
- RTK là cách làm ĐƯỢC KHUYẾN NGHỊ hiện nay cho mọi dự án Redux mới - khoá học từ đây trở đi khi nói "Redux" thường ngầm hiểu là "Redux Toolkit"

## Understanding the Store
- Store: 1 object DUY NHẤT (hầu như mọi dự án chỉ có 1 store) chứa TOÀN BỘ state của ứng dụng
- Thông thường KHÔNG tương tác trực tiếp với store (việc đó do React Redux lo) - chỉ thao tác thủ công khi debug: `store.dispatch(actionObject)` để đổi state, `store.getState()` để xem toàn bộ state hiện tại
- Cấu trúc state object trong store: các KEY ở tầng ngoài cùng được định nghĩa NGAY KHI TẠO STORE (`configureStore({ reducer: { songs: songsSlice.reducer } })`) - key nào xuất hiện trong object `reducer` truyền vào `configureStore`, key đó sẽ xuất hiện trên state object; GIÁ TRỊ của từng key do đúng REDUCER tương ứng sinh ra và cập nhật theo thời gian
- Muốn đổi tên/cấu trúc của state object tầng ngoài cùng → sửa ở chính nơi gọi `configureStore`, không sửa ở đâu khác

## Understanding Slices
- `createSlice`: hàm của Redux Toolkit, nhận vào 1 object cấu hình gồm `name`, `initialState`, và `reducers` (object chứa các "mini-reducer" function) - trả về 1 object gọi là "slice"
- Mỗi function trong `reducers` object có thể hình dung như 1 CASE riêng lẻ trong 1 switch statement lớn - `createSlice` tự động GOM tất cả các mini-reducer này lại thành 1 reducer TỔNG (combined reducer), nằm ở property `slice.reducer` (số ít, khác với `reducers` truyền vào lúc đầu)
- Combined reducer chính là thứ được đưa vào `configureStore` - nó tự biết cách "định tuyến" 1 action object tới đúng mini-reducer tương ứng
- Cách combined reducer xác định NÊN CHẠY mini-reducer nào: dựa vào TYPE của action, theo pattern cố định `tênSlice/tênMiniReducer` (VD slice tên `song`, mini-reducer tên `addSong` → type tương ứng là `song/addSong`) - không cần tự nhớ pattern này vì có công cụ tự động tạo (action creators, học ở bài sau)
- Bên trong mỗi mini-reducer: TỰ ĐỘNG được dùng kèm thư viện Immer (được phép mutate state trực tiếp), KHÔNG cần return state mới - chỉ cần áp dụng đúng thay đổi lên state là đủ
- Điểm CỰC KỲ QUAN TRỌNG (nhắc lại nhiều lần xuyên suốt khoá): trong mỗi mini-reducer, biến `state` KHÔNG PHẢI toàn bộ state object của store - nó CHỈ là phần state riêng do đúng slice đó quản lý (VD với songsSlice, `state` chỉ là mảng songs, không phải `{ songs, movies, ... }`)

## Understanding Action Creators
- Action creator: 1 FUNCTION được `createSlice` TỰ ĐỘNG tạo ra cho MỖI mini-reducer đã định nghĩa - gọi function này sẽ trả về 1 action object sẵn sàng để dispatch, KHÔNG cần tự viết tay `{ type: '...', payload: ... }`
- Vị trí truy cập: nằm trên property `slice.actions` (số nhiều - LƯU Ý dễ nhầm với `slice.reducer` số ít ở bài trước) - property này bị đặt tên hơi gây nhầm lẫn (giảng viên nhận xét lẽ ra nên gọi là "actionCreators" mới đúng bản chất)
- Cách dùng: `slice.actions.addSong('tên bài hát')` → trả về `{ type: 'song/addSong', payload: 'tên bài hát' }` - đối số ĐẦU TIÊN (và duy nhất) truyền vào action creator chính là `payload`
- Mục đích DUY NHẤT của action creator: tránh phải tự tay gõ đúng string type (dễ gõ sai chính tả) và tự tạo object thủ công - hoàn toàn không có gì phức tạp hơn thế, không cần "nghĩ quá nhiều" về khái niệm này

## Updating State from a Component (6 bước cập nhật state)
1. Vào slice, thêm 1 mini-reducer mới xử lý đúng thay đổi state mong muốn
2. Ở cuối file slice, EXPORT action creator tương ứng (`export const addSong = songsSlice.actions.addSong`)
3. Xác định component cần dispatch (nơi có event handler liên quan)
4. Import action creator vừa export + import hook `useDispatch` từ `react-redux`
5. Gọi hook: `const dispatch = useDispatch()` - hook này dùng Context bên dưới để lấy đúng hàm `dispatch` của store
6. Trong event handler: gọi action creator để tạo action object, rồi gọi `dispatch(actionObject)` - thường viết gọn thành 1 dòng: `dispatch(addSong(song))`
- Không cần tạo mini-reducer MỚI mỗi lần muốn update state theo cùng 1 cách - có thể tái dùng lại action creator đã có ở bất kỳ đâu trong app

## Accessing State in a Component (4 bước đọc state)
1. Xác định component cần đọc state
2. Import hook `useSelector` từ `react-redux`
3. Gọi hook, truyền vào 1 "selector function": `useSelector(state => state.songs)` - selector nhận vào TOÀN BỘ state object của store, CHỈ return đúng phần component thực sự cần (tránh component phải nhận cả state không liên quan)
4. Dùng giá trị trả về từ hook để render UI
- **Điểm gây nhầm lẫn quan trọng (nhắc đi nhắc lại):** chữ "state" có 2 NGHĨA KHÁC NHAU tuỳ ngữ cảnh - BÊN TRONG 1 slice (trong mini-reducer), "state" chỉ là phần state riêng của slice đó; BÊN NGOÀI slice (VD trong `useSelector`), "state" luôn là TOÀN BỘ state object của store - nhầm lẫn giữa 2 nghĩa này là nguồn gốc bug rất phổ biến khi làm việc với Redux

## Resetting State (Vấn đề reset về mảng rỗng với Immer)
- Với hầu hết update, có thể mutate trực tiếp `state` (VD `state.push(...)`) nhờ Immer tự xử lý phía sau
- Riêng trường hợp muốn RESET state về giá trị hoàn toàn mới (VD mảng rỗng `[]`): viết `state = []` KHÔNG hoạt động - đây chỉ là GÁN LẠI biến cục bộ `state`, không phải MUTATE nó, nên Immer không nhận diện được đây là thay đổi cần áp dụng
- Cách đúng trong trường hợp đặc biệt này: RETURN giá trị mới mong muốn (`return []`) - Immer/Redux Toolkit hiểu rằng nếu reducer return 1 giá trị, đó chính là state mới, bất kể có mutate gì trước đó hay không

## Watching for Other Actions (extraReducers)
- Sự thật quan trọng về cơ chế dispatch: khi gọi `dispatch(action)`, action đó được gửi tới **MỌI combined reducer** trong store (không chỉ 1 slice cụ thể) - hành vi này không thể tắt, và cũng không cần tắt
- Mỗi combined reducer mặc định CHỈ PHẢN ỨNG với đúng những action type khớp với các mini-reducer nó tự định nghĩa (VD `song/addSong`, `song/removeSong`) - nếu nhận action type lạ, nó im lặng bỏ qua, không đổi gì
- `extraReducers`: 1 property BỔ SUNG có thể thêm vào cấu hình `createSlice`, dùng để dạy 1 slice "để ý" tới các action type KHÔNG PHẢI do chính nó tạo ra (VD dạy songsSlice phản ứng với action `movie/reset` vốn thuộc về movieSlice)
- Cú pháp: `extraReducers: (builder) => { builder.addCase(actionTypeOrCreator, (state, action) => {...}) }` - hàm xử lý bên trong `addCase` có cấu trúc/hành vi giống hệt 1 mini-reducer bình thường (cùng quy tắc Immer, cùng cách xử lý `state`)
- Nhược điểm cách làm này: tạo ra sự PHỤ THUỘC ngầm giữa 2 slice - nếu sau này đổi tên/xoá reducer gốc bên slice kia, slice đang "nghe ké" sẽ ngừng hoạt động đúng mà không có cảnh báo rõ ràng gì

## Manual Action Creation (createAction - giải pháp tách phụ thuộc)
- Giải pháp khắc phục nhược điểm ở trên: tạo ra 1 ACTION TYPE ĐỘC LẬP, KHÔNG gắn với bất kỳ slice cụ thể nào, rồi cho NHIỀU slice khác nhau cùng lắng nghe type đó qua `extraReducers`
- Dùng `createAction` (import từ Redux Toolkit) để tự tay tạo 1 action creator, không thông qua `createSlice`: `const reset = createAction('app/reset')`
- `reset` tạo ra vẫn là 1 action creator bình thường - gọi `reset()` sẽ trả về action object `{ type: 'app/reset' }`, dùng được y hệt các action creator tự động khác
- String truyền vào `createAction` không có ý nghĩa đặc biệt về mặt kỹ thuật, chỉ cần là 1 string DUY NHẤT, có ý nghĩa dễ hiểu (convention: `domain/hànhĐộng`)
- Cả 2 (hoặc nhiều) slice liên quan đều dùng `extraReducers` + `builder.addCase(reset, handler)` để cùng lắng nghe ĐÚNG 1 action type độc lập này - loại bỏ hoàn toàn sự phụ thuộc trực tiếp giữa các slice với nhau, mỗi slice chỉ phụ thuộc vào 1 action type dùng chung, không phụ thuộc vào cấu trúc nội bộ của slice khác


---

# Section 20 - Managing Multiple Slices with Redux Toolkit

## Thinking About Derived State
- Quy trình xác định state cho ứng dụng Redux: (1) liệt kê hết những gì thay đổi trên màn hình, (2) xác định user thao tác gì gây ra thay đổi đó, (3) gom nhóm các state liên quan, tạo 1 slice riêng cho từng nhóm - về bản chất y hệt State Design Process đã học trước đây, chỉ khác chỗ chứa (Redux store thay vì component)
- **Derived state**: KHÔNG phải 1 tính năng đặc biệt của React/Redux, mà chỉ là 1 KHÁI NIỆM - dữ liệu hiển thị trên màn hình mà thay vì LƯU riêng thành 1 piece of state, có thể TÍNH TOÁN ra được từ những state đã có sẵn, bằng JavaScript thuần
- Dấu hiệu nhận biết derived state: khi thấy nội dung trên màn hình thay đổi, đừng vội kết luận "cần thêm state mới" - hãy tự hỏi: giá trị này có thể tính ra được từ state đã tồn tại không? (VD tổng chi phí = duyệt qua mảng xe, cộng dồn cost của từng xe → không cần lưu riêng piece of state "totalCost")
- Lợi ích của việc nhận diện đúng derived state: giảm số lượng state cần quản lý, tránh dữ liệu bị TRÙNG LẶP/KHÔNG ĐỒNG BỘ (VD nếu lưu riêng `totalCost` mà quên cập nhật khi `cars` đổi, 2 giá trị sẽ lệch nhau) - đặc biệt quan trọng khi app lớn dần, càng ít state trực tiếp càng dễ maintain

## Maintaining a Collection with a Slice (Quản lý danh sách bằng Slice)
- Khi 1 slice cần quản lý danh sách (mảng object, VD danh sách xe), thường thiết kế `initialState` là 1 OBJECT gồm nhiều property phụ trợ (VD `{ searchTerm: '', cars: [] }`), KHÔNG chỉ đơn thuần là 1 mảng trần
- Mỗi item trong mảng nên có 1 ID DUY NHẤT để phân biệt (ngay cả khi 2 item có dữ liệu giống hệt nhau, VD 2 xe cùng tên cùng giá) - dùng `nanoid` (import từ `@reduxjs/toolkit`) để tự động sinh ID ngẫu nhiên, đây là tiện ích có sẵn của Redux Toolkit, không có gì đặc biệt về mặt kỹ thuật so với các cách generate ID khác
- Vấn đề "giả định ngầm" giữa các slice: khi 1 mini-reducer (VD `addCar` trong carsSlice) cần dữ liệu đang được QUẢN LÝ BỞI SLICE KHÁC (VD tên/giá xe đang nằm trong formSlice) - vì 1 slice KHÔNG BAO GIỜ được phép đọc trực tiếp state của slice khác, reducer đó buộc phải GIẢ ĐỊNH rằng `action.payload` sẽ có đúng cấu trúc cần thiết (VD `{ name, cost }`) do nơi gọi `dispatch` tự đóng gói sẵn và truyền vào
- Đây là TRÁCH NHIỆM của người viết code ở phía gọi `dispatch` - phải đảm bảo LUÔN truyền đúng payload theo đúng cấu trúc mà reducer mong đợi, vì reducer không có cách nào tự kiểm tra hay lấy dữ liệu từ nơi khác
- Thao tác XOÁ 1 item khỏi mảng theo ID: dùng `.filter()` để tạo mảng mới, giữ lại các phần tử có ID KHÁC với ID cần xoá (`car.id !== action.payload`), rồi gán mảng mới đó cho property tương ứng trong state

## Awkward Double Keys (Đặt tên key trùng lặp gây rối)
- Vấn đề: khi tên KEY của slice trong `configureStore` (VD `cars: carsSlice.reducer`) TRÙNG với tên PROPERTY bên trong `initialState` của chính slice đó (VD state có `{ cars: [...] }`), sẽ dẫn tới cách truy cập bị lặp từ chữ khó đọc: `state.cars.cars` - đây KHÔNG PHẢI bug, chỉ là hệ quả tự nhiên khi đặt tên trùng nhau không cẩn thận
- Cách khắc phục: đổi tên 1 trong 2 chỗ (thường đổi tên property BÊN TRONG slice, VD đổi `cars` thành `data` hoặc `list` hoặc `entities`) - để tránh việc truy cập bị lặp chữ gây khó hiểu cho người đọc code khác (dễ tưởng là lỗi đánh máy)
- Bài học tổng quát: cần CHÚ Ý trước khi đặt tên key ở `configureStore` VÀ tên property trong `initialState` của từng slice - vì khi 2 tên này trùng nhau, code truy cập state cuối cùng sẽ luôn có dạng lặp chữ khó đọc

## Reminder on ExtraReducers
- Nhắc lại + minh hoạ thêm 1 use case thực tế cho `extraReducers`: 1 slice (VD `formSlice`) có thể lắng nghe action creator ĐƯỢC EXPORT từ 1 SLICE KHÁC (VD `addCar` từ `carsSlice`) mà không cần biết action đó thuộc slice nào về mặt kỹ thuật
- Nên IMPORT trực tiếp action creator function (VD `import { addCar } from './carsSlice'`) rồi truyền THẲNG function đó vào `builder.addCase(addCar, handler)` - KHÔNG nên tự gõ tay string action type (VD `'cars/addCar'`) để tránh rủi ro gõ sai chính tả, dù cả 2 cách đều hoạt động được về mặt kỹ thuật
- Use case cụ thể: khi 1 form (formSlice) cần TỰ RESET lại các trường nhập liệu (`name`, `cost`) ngay sau khi 1 slice khác (carsSlice) xử lý xong action `addCar` - đây là ví dụ điển hình của việc 1 slice "phản ứng" theo hành động xảy ra ở slice khác, không cần phải tự dispatch thêm 1 action riêng nào khác cho việc reset form

## Derived State in useSelector (Đào sâu: nơi nào NÊN và KHÔNG NÊN tính derived state)
- Áp dụng thực tế derived state: tính năng LỌC (filter) danh sách xe theo `searchTerm` - đây là derived state điển hình, tính toán từ 2 piece of state có sẵn (`data` và `searchTerm`), không cần lưu riêng 1 state mới cho "danh sách đã lọc"
- Vị trí lý tưởng để đặt logic tính derived state: NGAY BÊN TRONG hàm selector truyền vào `useSelector` - giúp phần còn lại của component chỉ làm việc với dữ liệu ĐÃ ĐƯỢC XỬ LÝ SẴN, không cần biết gì về dữ liệu gốc chưa lọc
- Pattern hay dùng để code selector dễ đọc hơn: DESTRUCTURE trực tiếp trên argument của selector function để lấy đúng phần state cần dùng, tránh viết lặp đường dẫn dài (VD `({ cars: { data, searchTerm } }) => ...` thay vì lặp lại `state.cars.data`, `state.cars.searchTerm` nhiều lần)
- **NGOẠI LỆ quan trọng - KHÔNG PHẢI derived state nào cũng nên tính trong `useSelector`:** với tính năng "in đậm (bold) tên xe nếu trùng khớp với tên đang gõ trong form" - KHÔNG nên gắn thêm property tuỳ ý (VD `bold: true`) vào chính OBJECT DỮ LIỆU (car object) chỉ để phục vụ UI
- Lý do: object dữ liệu (car) nên đại diện đúng cho MÔ HÌNH DỮ LIỆU thuần tuý (chỉ có `id`, `name`, `cost`) - không nên trộn lẫn KHÁI NIỆM UI (trạng thái hiển thị, như có bold hay không) vào TRONG chính data model, vì 2 khái niệm này (dữ liệu vs trạng thái hiển thị) nên được xem là 2 THỰC THỂ TÁCH BIỆT
- Giải pháp đúng: lấy CẢ 2 nguồn dữ liệu cần thiết ra từ `useSelector` riêng biệt (VD danh sách xe VÀ tên đang gõ trong form, dù 2 cái này nằm ở 2 slice khác nhau) - rồi tính toán "có nên bold hay không" NGAY TẠI THỜI ĐIỂM RENDER, bên trong phần logic hiển thị JSX của component, KHÔNG tính sẵn trong selector


---

# Section 21 - Interfacing with API's Using Async Thunks

## Data Fetching Techniques
- Trong Redux Toolkit, có 2 cách chính để fetch data từ API: **Async Thunk Functions** hoặc **Redux Toolkit Query (RTK Query)** - thường 1 dự án chỉ chọn DÙNG 1 TRONG 2, hiếm khi dùng chung cả 2 (khoá học dùng cả 2 chỉ để minh hoạ, không phải thực hành chuẩn)
- QUY TẮC TUYỆT ĐỐI: KHÔNG BAO GIỜ thực hiện network request (gọi API, dùng `async`/`await`, Promise...) trực tiếp bên trong 1 REDUCER function - reducer PHẢI luôn là hàm ĐỒNG BỘ (synchronous) 100%, chỉ nhận `state`/`action` và trả về state mới
- Mọi logic gọi API PHẢI đi qua 1 trong 2 kỹ thuật trên (thunk hoặc RTK Query), không có ngoại lệ nào khác

## Adding State for Data Loading
- Khi implement tính năng fetch data (hiển thị loading → data hoặc error), cần bổ sung 3 PIECE OF STATE chuẩn vào slice, bên cạnh state dữ liệu chính (`data`):
  - `isLoading` (Boolean): `true` khi đang trong quá trình fetch, `false` khi không
  - `error` (object hoặc `null`): `null` nếu không có lỗi, chứa error object nếu request thất bại
- Trong VÒNG ĐỜI của 1 request, cần dispatch NHIỀU HƠN 1 action để cập nhật state đúng lúc: (1) khi BẮT ĐẦU request → set `isLoading: true`; (2a) khi THÀNH CÔNG → set `isLoading: false` + cập nhật `data`; (2b) khi LỖI → set `isLoading: false` + cập nhật `error`
- Đây chính là lý do cần 1 cơ chế tự động hoá việc dispatch nhiều action theo từng giai đoạn của request - dẫn tới khái niệm Async Thunk ở bài sau

## Understanding Async Thunks
- Async Thunk: 1 function ĐẶC BIỆT tự động DISPATCH SẴN các action tương ứng với TỪNG GIAI ĐOẠN của 1 network request, không cần tự tay dispatch từng action riêng lẻ
- 3 giai đoạn action tự động được dispatch: `pending` (khi request vừa bắt đầu), `fulfilled` (khi request thành công), `rejected` (khi request thất bại)
- Cần định nghĩa trong slice 3 case tương ứng (thường qua `extraReducers`) để lắng nghe đúng 3 loại action này, từ đó cập nhật `isLoading`, `data`, `error` đúng thời điểm
- Đây là giải pháp chuẩn cho vấn đề "cần dispatch nhiều action theo từng bước của 1 request" đã đặt ra ở bài trước - thunk tự động lo phần dispatch, mình chỉ cần viết logic xử lý từng loại action trong reducer

## Steps for Adding a Thunk
- Quy trình tạo 1 async thunk:
  1. Tạo file riêng cho thunk (thường trong thư mục `store/thunks/`), đặt tên theo MỤC ĐÍCH của request (VD `fetchUsers.js`)
  2. Gọi `createAsyncThunk(baseType, asyncFunction)` - import từ `@reduxjs/toolkit`
     - `baseType`: 1 string mô tả mục đích request (VD `'users/fetch'`) - dùng làm TIỀN TỐ để tự động sinh 3 action type (`users/fetch/pending`, `users/fetch/fulfilled`, `users/fetch/rejected`) - giá trị chuỗi không có ý nghĩa đặc biệt, chỉ cần dễ hiểu khi debug
     - `asyncFunction`: hàm `async` chứa logic gọi API thực tế (dùng `axios` hoặc tương tự), RETURN về dữ liệu cần dùng trong reducer (VD `return response.data`)
  3. Export thunk function để dùng ở nơi khác
  4. Trong slice, dùng `extraReducers` với `builder.addCase(thunk.pending, ...)`, `.addCase(thunk.fulfilled, ...)`, `.addCase(thunk.rejected, ...)` để xử lý từng giai đoạn
  5. Ở component, dispatch thunk y hệt dispatch 1 action bình thường: `dispatch(fetchUsers())`
- Giá trị RETURN từ hàm async bên trong thunk chính là `action.payload` nhận được ở case `fulfilled`

## Unexpected Loading State
- Bug thực tế: dùng CHUNG 1 piece of state `isLoading` cho NHIỀU loại thao tác khác nhau (VD fetch danh sách users VÀ tạo user mới) sẽ dẫn tới hành vi UI SAI - khi tạo user mới (chỉ ảnh hưởng 1 nút bấm), toàn bộ danh sách user bị ẩn đi và hiện skeleton loading, dù dữ liệu cũ vẫn còn nguyên và không cần load lại
- Nguyên nhân: reducer xử lý case `pending` của MỌI thunk đều set chung 1 `isLoading` - không phân biệt được đang loading vì lý do gì
- Đây là dấu hiệu cho thấy cần THIẾT KẾ LOADING STATE CHI TIẾT HƠN (fine-grained) thay vì dùng 1 cờ Boolean chung cho tất cả loại request

## Strategies for Fine-Grained Loading State
- "Fine-grained loading state": có STATE RIÊNG BIỆT cho TỪNG LOẠI request khác nhau (VD `isLoadingUsers`, `isCreatingUser`, và với xoá theo từng item cụ thể cần biết ID nào đang bị xoá) thay vì dùng chung 1 cờ `isLoading` cho tất cả
- Cách tệ (không nên làm): thêm ngày càng nhiều Boolean/array riêng lẻ để track từng loại thao tác - dễ phình to, khó quản lý khi số loại request tăng
- **Option 1 (đơn giản, phù hợp app nhỏ):** đưa loading/error state RA KHỎI Redux store, đặt LOCAL ngay trong từng component bằng `useState` bình thường - vẫn dùng Redux cho state chính (data), nhưng trạng thái loading/error của TỪNG THAO TÁC RIÊNG LẺ do component tự quản lý cục bộ. Có state trong component khi dùng Redux là HOÀN TOÀN BÌNH THƯỜNG, không phải sai (VD trạng thái open/close của 1 dropdown không cần đưa vào Redux)
- **Option 2 (phức tạp hơn, scale tốt hơn cho app lớn):** vẫn giữ mọi thứ trong Redux store, tận dụng `requestId` mà mỗi lần `dispatch(thunk())` tự động trả về (nằm trong Promise) - lưu 1 mảng các request đang diễn ra kèm trạng thái (pending/fulfilled/rejected) trong 1 slice riêng, mỗi component chỉ cần lưu lại `requestId` của mình để tra cứu trạng thái đúng request đó trong store. Đây là ý tưởng NỀN TẢNG mà module RTK Query (sẽ học sau) tự động hoá sẵn cho mình
- Nguyên tắc chọn: app nhỏ → Option 1 đơn giản, đủ dùng; app lớn, cần khả năng mở rộng và tra cứu trạng thái request dễ dàng từ nhiều nơi → Option 2 (hoặc dùng RTK Query có sẵn)

## Creating a Reusable Thunk Hook
- Vấn đề: mỗi lần cần dispatch 1 thunk theo Option 1 (fine-grained loading ở component), phải viết lặp lại: tạo state loading, tạo state error, dispatch, xử lý catch/finally - rất tốn công nếu lặp lại nhiều lần trong nhiều component
- Giải pháp: đóng gói toàn bộ pattern đó vào 1 CUSTOM HOOK dùng chung, gọi là `useThunk`
- Cách dùng mong muốn: `const [doFetchUsers, isLoadingUsers, loadingUsersError] = useThunk(fetchUsers)` - trả về mảng 3 phần tử: (1) function để TRIGGER chạy thunk, (2) cờ loading, (3) error nếu có
- Implementation bên trong hook: tạo `isLoading`/`error` bằng `useState`, viết 1 hàm `runThunk` gọi `dispatch(thunk(arg))`, dùng `.unwrap()` để lấy Promise gốc rồi `.catch()` để bắt lỗi cập nhật `error`, `.finally()` để set `isLoading` về `false`
- Điểm KỸ THUẬT QUAN TRỌNG cần chú ý: hàm `runThunk` trả về từ hook PHẢI được bọc bằng `useCallback` (dependency là `[dispatch, thunk]`) - để đảm bảo REFERENCE ỔN ĐỊNH qua các lần render, tránh gây vòng lặp vô hạn nếu function này được đưa vào dependency array của `useEffect` ở nơi sử dụng (nhắc lại đúng vấn đề đã học ở phần `useCallback` trước đây)
- Hook này có thể nhận thêm argument để truyền vào thunk khi cần (VD ID của user muốn xoá) - giúp hook tái sử dụng được cho cả trường hợp thunk cần tham số đầu vào

## Fixing a Delete Error (Trả về đúng data cần thiết từ thunk, không phải response.data mù quáng)
- Bug thực tế: sau khi xoá user thành công (DELETE request), reducer cần biết CHÍNH XÁC user nào vừa bị xoá để loại bỏ khỏi mảng `data` cục bộ - nhưng response của DELETE request từ server thường trả về OBJECT RỖNG, không chứa thông tin gì hữu ích
- Bài học quan trọng: KHÔNG PHẢI lúc nào cũng nên `return response.data` từ thunk 1 cách máy móc - cần RETURN ĐÚNG DỮ LIỆU MÀ REDUCER THỰC SỰ CẦN để xử lý logic, có thể không liên quan trực tiếp tới response từ server
- Cách fix: bên trong thunk, thay vì return `response.data` (rỗng), return lại chính ĐỐI TƯỢNG USER đã được truyền vào làm argument lúc gọi thunk (dữ liệu này mình đã có sẵn trước khi gửi request, không cần chờ response trả về)
- Nhờ đó, trong reducer, `action.payload` giờ chứa đúng thông tin user cần xoá (có `id`), cho phép dùng `.filter()` để loại đúng user đó ra khỏi mảng `data`
- Bài học tổng quát: khi viết thunk, luôn tự hỏi "reducer ở phía sau THỰC SỰ CẦN gì từ action.payload" - không mặc định luôn trả `response.data`, đặc biệt với các request như DELETE thường có response rỗng hoặc không đầy đủ thông tin


---

# Section 22 - Modern Async with Redux Toolkit Query

## Introducing Redux Toolkit Query (RTK Query) - Tổng quan
- RTK Query là 1 MODULE nằm sẵn trong Redux Toolkit - dùng để tạo ra thứ gọi là "API" (LƯU Ý: đây KHÔNG PHẢI backend server, mà là code phía CLIENT/React, cung cấp giao diện dễ dùng để fetch/thay đổi data)
- Tạo API bằng `createApi(config)` - kết quả trả về chứa nhiều "nguyên liệu" Redux quen thuộc (slice, thunk...) nhưng ĐIỀU THỰC SỰ QUAN TRỌNG là 1 bộ HOOK được TỰ ĐỘNG SINH RA - các hook này lo hết việc fetch data, hiện loading, xử lý lỗi... mà không cần tự viết tay
- Trong config có mục `endpoints` - mô tả từng loại request cần làm (VD `fetchAlbums`, `addAlbum`, `removeAlbum`) - tên các key này quyết định TÊN HOOK tự động sinh ra (VD `fetchAlbums` → `useFetchAlbumsQuery`)
- 2 thuật ngữ cốt lõi: **query** = request ĐỌC dữ liệu (GET); **mutation** = request THAY ĐỔI dữ liệu (POST/PUT/DELETE) - tên hook tương ứng sẽ có hậu tố `Query` hoặc `Mutation`
- Hook query khi gọi trả về object có `data`, `error`, `isLoading` - dùng trực tiếp trong component để render loading/error/data mà không cần tự quản lý state thủ công như cách làm với thunk
- RTK Query là thư viện MẠNH, xử lý gần như MỌI khía cạnh của data fetching (loading, error, cache, refetch...) - nên độ phức tạp bên trong khá cao, cần chấp nhận rằng ngay cả với thư viện tốt, bản thân data fetching vẫn là 1 bài toán khó

## Creating a RTK Query API
- Quy trình bắt đầu: (1) Nhận diện và GOM NHÓM các loại request theo LOẠI DỮ LIỆU chúng thao tác (VD nhóm request về users, nhóm về albums, nhóm về photos) - mỗi nhóm tạo 1 API riêng
- Tạo file riêng cho mỗi API (VD `store/apis/albumsApi.js`), import `createApi` từ `@reduxjs/toolkit/query/react` (LƯU Ý đường dẫn import khác với `createSlice` thông thường)
- 3 property BẮT BUỘC trong config truyền vào `createApi`:
  - **`reducerPath`**: 1 string xác định KEY lưu trữ toàn bộ state của API này trong big state object của store (tương tự key trong `configureStore`) - phải là string DUY NHẤT, không trùng với key khác đã dùng
  - **`baseQuery`**: cấu hình cách gửi request (bài sau)
  - **`endpoints`**: mô tả từng loại request cụ thể (bài sau)

## Creating an Endpoint
- `baseQuery`: dùng `fetchBaseQuery({ baseUrl: '...' })` (import từ RTK) để cấu hình phiên bản `fetch` có sẵn (built-in của browser, không phải Axios) - chỉ cần cung cấp `baseUrl` (URL gốc của server API)
- `endpoints`: 1 FUNCTION nhận argument `builder`, RETURN về 1 object mô tả từng loại request
- Với mỗi request cần trả lời trước các câu hỏi: mục đích request, tên ngắn gọn, là query hay mutation, path tương đối so với baseUrl, có query string không, method gì (GET/POST/DELETE...), có body không - trả lời rõ các câu hỏi này giúp việc viết code sau đó cực kỳ đơn giản, gần như "điền vào chỗ trống"
- Cú pháp định nghĩa 1 endpoint dạng QUERY:
```js
fetchAlbums: builder.query({
  query: (user) => ({
    url: '/albums',
    params: { userId: user.id },
    method: 'GET',
  }),
}),
```
- `query` (tên field, dễ gây nhầm) là 1 function nhận vào ĐÚNG argument mà component sẽ truyền vào khi gọi hook - return về object mô tả chi tiết request (url, params, method, body)
- KEY đặt tên cho endpoint (VD `fetchAlbums`) quyết định tên hook tự động sinh ra theo pattern `use` + TênEndpoint + `Query`/`Mutation`

## Using the Generated Hook
- Sau khi định nghĩa endpoint, cần export hook tự động sinh ra: `export const { useFetchAlbumsQuery } = albumsApi`
- Cần "kết nối" API vào store - PHỨC TẠP HƠN so với kết nối 1 slice thông thường, gồm nhiều bước: (1) thêm combined reducer của API vào `configureStore` (dùng `[albumsApi.reducerPath]: albumsApi.reducer` để tránh gõ tay string dễ sai), (2) thêm `middleware` bắt buộc (`getDefaultMiddleware().concat(albumsApi.middleware)`), (3) gọi `setupListeners(store.dispatch)` 1 LẦN DUY NHẤT (dùng chung cho mọi API, không lặp lại cho từng API)
- Cách dùng hook trong component: gọi TRỰC TIẾP, KHÔNG cần bọc trong `useEffect` hay event handler - hook TỰ ĐỘNG fetch data ngay khi component render lần đầu:
```js
const { data, error, isLoading } = useFetchAlbumsQuery(user);
```
- Argument truyền vào hook chính là argument sẽ được chuyển tới function `query` đã định nghĩa trong endpoint

## Changing Data with Mutations
- Định nghĩa 1 endpoint dạng MUTATION dùng `builder.mutation` thay vì `builder.query` - cấu trúc bên trong (field `query`) giống hệt cách viết của query, chỉ khác method (POST/DELETE...) và có thể có `body`
- Hook mutation tự động sinh có TÊN KHÁC CHÚT ít so với query - không có "s" ở cuối tên endpoint gốc khi tạo tên hook (VD endpoint `addAlbum` → hook `useAddAlbumMutation`)
- **Cách gọi hook mutation KHÁC HẲN hook query** - trả về 1 MẢNG 2 phần tử: (1) 1 FUNCTION dùng để TỰ TAY TRIGGER thực thi mutation khi cần (VD lúc user click nút), (2) object `results` chứa trạng thái (tương tự `data`/`error`/`isLoading` của query)
- Mutation KHÔNG tự động chạy khi component render (khác hẳn query) - phải chủ động gọi function trigger đó trong event handler:
```js
const [addAlbum, results] = useAddAlbumMutation();
// ...
const handleAddAlbum = () => addAlbum(user);
```

## Differences Between Queries and Mutations
- **Query hook**: TỰ ĐỘNG chạy fetch ngay khi component hiển thị lần đầu (hành vi mặc định, có thể tuỳ chỉnh trì hoãn) - trả về TRỰC TIẾP 1 object (`data`, `error`, `isLoading`...)
- **Mutation hook**: KHÔNG tự động chạy - trả về 1 MẢNG gồm (function để trigger, object kết quả) - vì thay đổi dữ liệu thường cần xảy ra ĐÚNG LÚC user thao tác (click save, submit form...), không nên tự động chạy ngay khi component mount
- `results` object của mutation có thêm property `status` (VD `'uninitialized'` khi chưa từng gọi, chuyển trạng thái sau khi gọi) - cung cấp thông tin chi tiết hơn về trạng thái của lần gọi mutation gần nhất

## Refetching with Tags
- Vấn đề cốt lõi: sau khi chạy 1 MUTATION (VD thêm album mới), dữ liệu trên SERVER đã đổi, nhưng dữ liệu đã CACHE SẴN trong Redux store (từ lần QUERY trước đó) vẫn CŨ, không tự động cập nhật - gây ra tình trạng UI không hiển thị dữ liệu mới dù request đã thành công
- **Hệ thống TAG**: giải pháp của RTK Query để TỰ ĐỘNG refetch lại đúng những query đã bị dữ liệu ảnh hưởng, sau khi 1 mutation chạy xong
- Cách hoạt động cơ bản: gắn `providesTags: ['Album']` vào 1 endpoint QUERY (đánh dấu: "query này liên quan tới tag Album") - gắn `invalidatesTags: ['Album']` vào 1 endpoint MUTATION (đánh dấu: "khi mutation này chạy xong, mọi query có tag 'Album' cần bị coi là LỖI THỜI")
- Khi mutation chạy xong và tag bị invalidate, RTK Query TỰ ĐỘNG chạy lại (refetch) TẤT CẢ query có tag khớp - không cần tự tay gọi lại hook hay dispatch gì thêm
- Giá trị chuỗi tag không có ý nghĩa đặc biệt về mặt kỹ thuật, chỉ cần KHỚP CHÍNH XÁC giữa `providesTags` và `invalidatesTags` - convention: đặt tên singular, viết hoa chữ cái đầu (VD `'Album'`)
- **Vấn đề thực tế phát sinh khi dùng tag đơn giản (string cố định):** nếu nhiều component cùng gọi 1 query (VD nhiều user panel cùng mở, mỗi cái tự fetch album riêng), TẤT CẢ đều mang CHUNG 1 tag string → khi chỉ 1 mutation ảnh hưởng tới 1 user cụ thể, TOÀN BỘ các query khác (của user không liên quan) cũng bị coi là stale và bị refetch KHÔNG CẦN THIẾT - gây lãng phí request

## Fine-Grained Tag Validation
- Giải pháp cho vấn đề "refetch thừa": thay vì tag là STRING đơn giản, dùng OBJECT có 2 property `{ type, id }` - cho phép TAG HOÁ CHI TIẾT theo từng bản ghi cụ thể (VD theo ID của user), không chỉ theo "loại" dữ liệu chung chung
- `providesTags` khi cần tag động: viết dưới dạng FUNCTION (không phải mảng cố định), nhận vào `(result, error, arg)` - trong đó `arg` chính là argument đã truyền vào hook lúc gọi (VD user object) - return về mảng tag object động dựa trên dữ liệu thực tế: `[{ type: 'Album', id: user.id }]`
- `invalidatesTags` tương tự cũng viết dưới dạng FUNCTION `(result, error, arg)`, với `arg` là argument đã truyền vào khi gọi MUTATION - return về đúng tag object cần vô hiệu hoá, khớp CHÍNH XÁC với tag của query cụ thể cần refetch
- Kết quả: chỉ ĐÚNG query liên quan tới user cụ thể bị đánh dấu lỗi thời và refetch lại, các query của user khác không bị ảnh hưởng - giải quyết triệt để vấn đề refetch thừa
- Bài học tổng quát quan trọng: việc thiết kế tag KHÔNG có công thức cố định, y hệt hoàn toàn giữa mọi trường hợp - luôn phải TỰ PHÂN TÍCH xem 1 mutation cụ thể sẽ ẢNH HƯỞNG tới NHỮNG QUERY NÀO, rồi thiết kế tag sao cho khớp đúng ý đồ đó, không có 1 pattern universal áp dụng máy móc cho mọi tình huống

## Getting Clever with Cache Tags (Kỹ thuật tag nâng cao khi endpoint không có sẵn dữ liệu cần thiết)
- Vấn đề nâng cao: đôi khi 1 mutation (VD xoá album) CHỈ CÓ trong tay 1 phần dữ liệu hạn chế (VD chỉ có album ID, KHÔNG có sẵn user ID) - nhưng cần invalidate đúng tag đang gắn theo user ID → không thể áp dụng công thức đơn giản như bài trước
- Giải pháp KHÔNG NÊN làm: cố tình sửa cách TRUYỀN PROPS giữa các component (VD truyền thêm user vào nơi không cần) chỉ để phục vụ nhu cầu của Redux - nên GIỮ NGUYÊN cấu trúc component, tìm giải pháp khác ở tầng Redux
- Giải pháp ĐÚNG - "trở nên khôn khéo hơn" với tag: 1 endpoint QUERY có thể trả về NHIỀU TAG CÙNG LÚC (không chỉ 1), bao gồm: (1) 1 tag RIÊNG cho TỪNG bản ghi (VD mỗi album 1 tag `{ type: 'Album', id: album.id }`), VÀ (2) 1 tag TỔNG QUÁT hơn đại diện cho "toàn bộ nhóm" (VD `{ type: 'UsersAlbums', id: user.id }`)
- Nhờ có NHIỀU tag gắn vào CÙNG 1 query, các mutation KHÁC NHAU (có dữ liệu sẵn có khác nhau) đều có thể tìm được ĐÚNG loại tag phù hợp với thông tin chúng đang có để invalidate: mutation XOÁ album (chỉ có album ID) → invalidate theo tag `{ type: 'Album', id: albumId }`; mutation THÊM album (có sẵn user object) → invalidate theo tag `{ type: 'UsersAlbums', id: user.id }` - cả 2 đều trỏ tới ĐÚNG 1 query, dù xuất phát từ 2 loại dữ liệu khác nhau
- Bài học tổng quát: khi thiết kế tag, luôn cân nhắc gắn NHIỀU LOẠI TAG khác nhau cho cùng 1 query nếu cần - để các mutation với dữ liệu đầu vào khác nhau đều có "cửa" phù hợp để invalidate đúng, tránh phải thay đổi cấu trúc data/props chỉ để phục vụ nhu cầu của caching system

## Adding Automatic Data Refetching (Áp dụng thực chiến cho Photos)
- Áp dụng lại đúng chiến lược tag "kết hợp nhiều loại" (từ bài trước) cho feature Photos: mỗi ảnh có 1 tag riêng theo ID của chính nó (`{ type: 'Photo', id: photo.id }`), CỘNG THÊM 1 tag đại diện cho CẢ ALBUM chứa những ảnh đó (`{ type: 'AlbumPhoto', id: album.id }`)
- Mutation XOÁ ảnh (chỉ có sẵn photo object, có photo ID) → invalidate theo tag `{ type: 'Photo', id: photo.id }` - khớp đúng tag riêng của từng ảnh
- Mutation THÊM ảnh (chỉ có sẵn album object, có album ID) → invalidate theo tag `{ type: 'AlbumPhoto', id: album.id }` - khớp đúng tag đại diện cho cả album
- Đây là minh chứng thực tế cho nguyên tắc: khi các mutation khác nhau có QUYỀN TRUY CẬP DỮ LIỆU khác nhau (cái thì chỉ biết ID bản ghi, cái thì chỉ biết ID nhóm cha), thiết kế NHIỀU TẦNG TAG trên cùng 1 query giúp TẤT CẢ các mutation đều tìm được đúng "chìa khoá" để trigger refetch chính xác, mà không cần thay đổi cấu trúc dữ liệu hay props truyền giữa các component
- Hệ thống TAG được xác nhận là "TÍNH NĂNG QUAN TRỌNG NHẤT" (killer feature) của toàn bộ RTK Query - đáng đầu tư thời gian hiểu sâu vì sẽ áp dụng lặp lại liên tục trong bất kỳ dự án thực tế nào dùng RTK Query


---

# Section 23 - Diving Into TypeScript

## Why Use TypeScript?
- Mục đích CHÍNH của TypeScript: giúp phát hiện lỗi NGAY TRONG LÚC VIẾT CODE (trong editor), thay vì phải chạy code rồi mới thấy lỗi - đây là công cụ hỗ trợ DEVELOPMENT, không phải tính năng runtime
- Lợi ích phụ: đóng vai trò như TÀI LIỆU tự nhiên cho component - engineer khác nhìn vào type/interface là hiểu ngay component nhận props gì, kiểu dữ liệu ra sao
- TypeScript được COMPILE (biên dịch) thành JavaScript thuần trước khi chạy trong trình duyệt - browser hoàn toàn không hiểu TypeScript, quá trình biên dịch diễn ra tự động phía sau
- TypeScript KHÔNG cải thiện hiệu năng runtime như các ngôn ngữ compiled có kiểu tĩnh khác - nó thuần túy là công cụ hỗ trợ phát triển
- Ví dụ minh hoạ với React: định nghĩa `interface` mô tả props component cần nhận (tên + kiểu dữ liệu), gán interface đó vào phần destructure props bằng dấu `:` - TypeScript sẽ tự động báo lỗi NGAY khi thiếu prop bắt buộc hoặc truyền sai kiểu dữ liệu

## Basic Types and Type Annotations
- Type annotation: cú pháp thêm THÔNG TIN VỀ KIỂU DỮ LIỆU vào code, có thể đặt ở nhiều vị trí - khai báo biến, tham số hàm, kiểu trả về của hàm, thuộc tính trong class
- Cú pháp cơ bản: `const tênBiến: kiểu = giá trị` (VD `const color: string = 'red'`)
- Các kiểu cơ bản: `string`, `number`, `boolean`, và dạng mảng của chúng (`string[]`, `number[]`, `boolean[]`)
- Khi biên dịch sang JavaScript thuần, TOÀN BỘ type annotation sẽ bị LOẠI BỎ HOÀN TOÀN - JavaScript kết quả không còn dấu vết gì của TypeScript
- Nếu cố gán sai kiểu dữ liệu vào biến đã có annotation (VD thêm số vào mảng string), TypeScript báo lỗi NGAY LẬP TỨC trong editor, trước khi code được chạy

## Describing Objects with Interfaces
- Để mô tả kiểu dữ liệu cho OBJECT, có 2 cách: annotation TRỰC TIẾP (viết ngay tại chỗ dùng, dạng `{ prop: kiểu, ... }`) hoặc định nghĩa `interface` RIÊNG rồi tái sử dụng ở nhiều nơi
- Cú pháp `interface`: `interface TenInterface { thuocTinh1: kieu1; thuocTinh2: kieu2; }` - quy ước đặt tên interface viết hoa chữ cái đầu (VD `Car`)
- Sau khi định nghĩa, interface trở thành 1 KIỂU DỮ LIỆU MỚI, dùng được ở bất kỳ đâu cần khai báo type (thay thế annotation trực tiếp dài dòng)
- Interface CỰC KỲ PHỔ BIẾN trong dự án React - gần như MỌI component đều có 1 interface đi kèm để mô tả các props mà nó nhận vào
- Nếu object thực tế có THÊM property không được khai báo trong interface, hoặc THIẾU property bắt buộc, TypeScript sẽ báo lỗi ngay
- Interface cũng có thể mô tả cả FUNCTION là 1 property của object (không chỉ dữ liệu thuần) - cú pháp: `tenHam: (thamSo: kieu) => kieuTraVe` (dùng `void` nếu function không trả về gì) - đây là kỹ thuật RẤT hay dùng khi mô tả các callback prop (VD `onClick`, `onChange`) trong interface props của React component

## Type Unions
- Type Union: kết hợp NHIỀU KIỂU DỮ LIỆU khác nhau thành 1 KIỂU MỚI, cho phép 1 biến/tham số nhận MỘT TRONG SỐ các kiểu đó
- Cú pháp: dùng ký tự `|` (pipe, không phải chữ "l" hay "I") để nối các kiểu: `string | number | string[] | Image`
- Dùng khi 1 function/biến cần chấp nhận NHIỀU LOẠI dữ liệu khác nhau (thay vì phải viết nhiều function riêng biệt cho từng kiểu) - phản ánh đúng thực tế lập trình thường gặp hơn so với việc viết hàm chỉ nhận đúng 1 kiểu cố định
- Nếu truyền vào 1 kiểu KHÔNG nằm trong danh sách union đã khai báo, TypeScript sẽ báo lỗi ngay

## Type Narrowing with Type Guards
- Vấn đề: khi 1 biến có kiểu là TYPE UNION (VD `string | number | string[] | Image`), TRƯỚC KHI thao tác với biến đó theo cách riêng của TỪNG kiểu cụ thể (VD gọi `.toUpperCase()` chỉ có ở string), cần XÁC ĐỊNH CHÍNH XÁC biến đang thực sự là kiểu nào tại thời điểm đó - quá trình này gọi là "type narrowing" (thu hẹp kiểu dữ liệu)
- Công cụ để thực hiện type narrowing gọi là "type guard" - thường là 1 `if` statement dùng toán tử `typeof` (có sẵn trong JavaScript thuần, không phải cú pháp riêng của TypeScript) để kiểm tra kiểu thực tế của biến
- Cú pháp: `if (typeof value === 'string') { ... }` - BÊN TRONG khối `if` này, TypeScript TỰ ĐỘNG hiểu rằng `value` chắc chắn là `string`, cho phép gọi an toàn các method chỉ có ở string (VD `.toUpperCase()`) mà không báo lỗi
- Ngoài phạm vi khối `if` đó (trước hoặc sau), TypeScript vẫn coi `value` là TOÀN BỘ type union ban đầu, không được "thu hẹp" - type guard chỉ có tác dụng CỤC BỘ trong đúng nhánh code mà nó bảo vệ

## The "Any" and "Unknown" Types
- `any`: 1 kiểu ĐẶC BIỆT, TẮT HOÀN TOÀN việc kiểm tra kiểu dữ liệu cho biến đó - TypeScript sẽ KHÔNG báo lỗi dù truy cập property không tồn tại hay gán sai kiểu
- NÊN TRÁNH dùng `any` càng nhiều càng tốt vì nó ĐI NGƯỢC LẠI mục đích cốt lõi của TypeScript - nhiều công cụ lint code (ESLint) sẽ tự động cảnh báo khi phát hiện dùng `any`
- Trường hợp thường gặp `any` trong thực tế: khi dùng thư viện bên thứ 3 hoặc gọi API - VD kết quả của `res.json()` (từ hàm `fetch` có sẵn) mặc định có kiểu `any`
- Khi biết CHẮC CHẮN kiểu thực tế của 1 biến `any`, dùng "type assertion" để ép kiểu: `data as Book` - báo cho TypeScript "hãy tin tôi, đây chắc chắn là kiểu Book", dù bản thân TypeScript không thể tự kiểm chứng điều đó
- `unknown`: PHIÊN BẢN NGHIÊM NGẶT HƠN của `any` - báo cho TypeScript biết biến CÓ THỂ là bất kỳ kiểu gì, nhưng KHÔNG cho phép truy cập BẤT KỲ property nào của biến đó cho tới khi đã thực hiện type narrowing (dùng type guard) để xác minh chắc chắn kiểu thực tế
- Nên ưu tiên dùng `unknown` thay vì `any` khi thực sự không chắc chắn về kiểu dữ liệu (VD dữ liệu trả về từ API bên ngoài, không kiểm soát được) - buộc phải viết type guard đầy đủ trước khi dùng, an toàn hơn `any` rất nhiều dù tốn công viết code kiểm tra hơn

## Introduction to Function Generics
- Generics: cơ chế cho phép TRUYỀN KIỂU DỮ LIỆU vào function như 1 dạng "tham số đặc biệt" - tương tự cách truyền giá trị bình thường vào tham số hàm, nhưng đây là truyền KIỂU thay vì giá trị
- Cú pháp khai báo: đặt dấu `<TenGeneric>` NGAY SAU tên hàm, trước dấu ngoặc tham số: `function wrapInArray<TypeToWrap>(value: TypeToWrap): TypeToWrap[] { ... }`
- Khi GỌI hàm, có thể chỉ định rõ kiểu generic bằng cú pháp tương tự: `wrapInArray<string>('hi')` - kiểu `string` sẽ "thay thế" mọi chỗ xuất hiện `TypeToWrap` bên trong định nghĩa hàm, y hệt cách 1 argument giá trị thay thế cho tên tham số
- Tên của generic type hoàn toàn tự đặt (giống tên tham số hàm) - quy ước RẤT PHỔ BIẾN trong thực tế là dùng chữ cái đơn `T` (viết tắt của "Type") làm tên generic, dù không bắt buộc

## Generics with Fetch (Ứng dụng thực tế loại bỏ trùng lặp code)
- Vấn đề thực tế khi KHÔNG dùng generic: mỗi loại request tới API khác nhau (fetch user, fetch message, fetch image) phải viết RIÊNG 1 function gần như GIỐNG HỆT NHAU, chỉ khác đường dẫn URL và kiểu dữ liệu ép về (`data as User`, `data as Message`...) - dẫn tới lặp code không cần thiết
- Giải pháp bằng generic: gộp thành 1 function DUY NHẤT `fetchData<T>(path: string): Promise<T>` - nhận vào `path` (string, giá trị bình thường) VÀ `T` (kiểu generic, xác định kiểu dữ liệu mong muốn trả về)
- Cú pháp kiểu trả về khi hàm là `async`: `Promise<T>` - biểu thị hàm trả về 1 Promise mà khi resolve sẽ cho ra giá trị kiểu `T`
- Cách gọi: `fetchData<User>('/users')`, `fetchData<Message>('/messages')` - mỗi lần gọi chỉ cần chỉ định đúng kiểu mong muốn, không cần viết lại toàn bộ logic hàm
- Đây là ví dụ THỰC TẾ rất điển hình cho thấy generic giải quyết đúng vấn đề: tránh lặp code khi có nhiều hàm chỉ khác nhau ở KIỂU DỮ LIỆU xử lý, không khác về LOGIC

## Generic Type Constraints
- Generic Type Constraint: đặt RÀNG BUỘC/YÊU CẦU lên 1 generic type, giới hạn kiểu dữ liệu nào được phép truyền vào (không cho phép "bất kỳ kiểu gì" một cách tuỳ tiện)
- Cú pháp: `<T extends KieuRangBuoc>` - VD `<T extends object>` nghĩa là kiểu `T` BẮT BUỘC phải là 1 object (không được là number, string...)
- Từ khoá `extends` ở đây nên hiểu theo nghĩa **"phải là"** (must be) thay vì nghĩa "kế thừa" thông thường - cách hiểu này giúp bớt gây nhầm lẫn với khái niệm kế thừa class/interface
- Áp dụng ràng buộc cho CẢ NHIỀU generic type cùng lúc trong 1 hàm: `function merge<T extends object, U extends object>(objA: T, objB: U)` - đảm bảo cả 2 tham số đều phải là object trước khi cho phép merge (dùng spread) chúng lại với nhau
- Nếu truyền vào 1 giá trị KHÔNG THOẢ MÃN ràng buộc (VD truyền number hoặc string cho tham số yêu cầu `extends object`), TypeScript sẽ báo lỗi ngay tại lời gọi hàm
- Type constraint giúp function generic vừa LINH HOẠT (chấp nhận nhiều kiểu khác nhau) vừa AN TOÀN (không chấp nhận kiểu vô nghĩa/không phù hợp với logic thực tế của hàm)


---

# Section 24 - Build a Google Maps Clone with Typescript



---

# Section 25 - Navigation and Data Fetching with React Router



---

# Section 26 - Legacy Version of Modern React with Redux Course

