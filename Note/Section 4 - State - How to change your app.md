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