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