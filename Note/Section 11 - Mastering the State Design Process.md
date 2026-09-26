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