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