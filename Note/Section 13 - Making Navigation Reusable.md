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