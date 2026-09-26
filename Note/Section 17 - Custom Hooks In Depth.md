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