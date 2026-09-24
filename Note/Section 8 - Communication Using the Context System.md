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
- Quy tắc thực hành: application state thường NÊN đưa vào Context để mọi nơi trong app đều truy cập dễ dàng; component state thường KHÔNG cần đưa vào Context vì không ai khác cần dùng tới, giữ nguyên `useState` cục bộ trong component đó là đủ
- Ví dụ áp dụng vào app Reading List: `books` (ở App) → application state, nên đưa vào Context kèm các hàm thao tác (`createBook`, `editBookById`, `deleteBookById`...); `title` (ở BookCreate/BookEdit) và `showEdit` (ở BookShow) → component state, giữ nguyên cục bộ
- Khái niệm này sẽ trở nên rất quan trọng khi học tới Redux sau này (Redux về bản chất là 1 cách quản lý tập trung cho "application state")

## Refactoring to Use Context
- Chuyển 1 piece of state (đã xác định là application state) từ component cha sang Context KHÔNG chỉ đơn giản là "dời code" - phải làm thêm 1 bước quan trọng: xoá bỏ toàn bộ việc truyền state/function đó qua props ở MỌI tầng component đã từng nhận nó, và đổi các component đó sang lấy trực tiếp từ Context (`useContext`) thay vì nhận qua props
- Refactor này ảnh hưởng tới nhiều component cùng lúc: mỗi component từng nhận props liên quan (VD `books`, `createBook`, `editBookById`, `deleteBookById`) đều cần sửa lại - bỏ phần nhận qua props, thêm phần lấy qua `useContext`
- Đây là quá trình thực tế thường gặp trong dự án thật: hiếm khi thiết kế đúng ngay từ đầu, việc refactor lại cấu trúc data flow (chuyển từ props sang context, hoặc ngược lại) là chuyện bình thường và cần thiết khi app phát triển
- Cách tiếp cận khi refactor: làm từng bước nhỏ, sửa từng phần rồi test lại app vẫn chạy đúng, thay vì cố sửa toàn bộ 1 lần - tránh dồn quá nhiều thay đổi cùng lúc dễ gây lỗi khó tìm