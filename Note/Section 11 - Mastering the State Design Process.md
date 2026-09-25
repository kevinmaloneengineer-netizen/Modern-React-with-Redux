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