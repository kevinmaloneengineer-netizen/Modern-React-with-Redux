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