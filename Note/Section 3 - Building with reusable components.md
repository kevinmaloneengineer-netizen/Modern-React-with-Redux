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