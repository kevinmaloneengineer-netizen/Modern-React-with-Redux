## Introducing Redux Toolkit Query (RTK Query) - Tổng quan
- RTK Query là 1 MODULE nằm sẵn trong Redux Toolkit - dùng để tạo ra thứ gọi là "API" (LƯU Ý: đây KHÔNG PHẢI backend server, mà là code phía CLIENT/React, cung cấp giao diện dễ dùng để fetch/thay đổi data)
- Tạo API bằng `createApi(config)` - kết quả trả về chứa nhiều "nguyên liệu" Redux quen thuộc (slice, thunk...) nhưng ĐIỀU THỰC SỰ QUAN TRỌNG là 1 bộ HOOK được TỰ ĐỘNG SINH RA - các hook này lo hết việc fetch data, hiện loading, xử lý lỗi... mà không cần tự viết tay
- Trong config có mục `endpoints` - mô tả từng loại request cần làm (VD `fetchAlbums`, `addAlbum`, `removeAlbum`) - tên các key này quyết định TÊN HOOK tự động sinh ra (VD `fetchAlbums` → `useFetchAlbumsQuery`)
- 2 thuật ngữ cốt lõi: **query** = request ĐỌC dữ liệu (GET); **mutation** = request THAY ĐỔI dữ liệu (POST/PUT/DELETE) - tên hook tương ứng sẽ có hậu tố `Query` hoặc `Mutation`
- Hook query khi gọi trả về object có `data`, `error`, `isLoading` - dùng trực tiếp trong component để render loading/error/data mà không cần tự quản lý state thủ công như cách làm với thunk
- RTK Query là thư viện MẠNH, xử lý gần như MỌI khía cạnh của data fetching (loading, error, cache, refetch...) - nên độ phức tạp bên trong khá cao, cần chấp nhận rằng ngay cả với thư viện tốt, bản thân data fetching vẫn là 1 bài toán khó

## Creating a RTK Query API
- Quy trình bắt đầu: (1) Nhận diện và GOM NHÓM các loại request theo LOẠI DỮ LIỆU chúng thao tác (VD nhóm request về users, nhóm về albums, nhóm về photos) - mỗi nhóm tạo 1 API riêng
- Tạo file riêng cho mỗi API (VD `store/apis/albumsApi.js`), import `createApi` từ `@reduxjs/toolkit/query/react` (LƯU Ý đường dẫn import khác với `createSlice` thông thường)
- 3 property BẮT BUỘC trong config truyền vào `createApi`:
  - **`reducerPath`**: 1 string xác định KEY lưu trữ toàn bộ state của API này trong big state object của store (tương tự key trong `configureStore`) - phải là string DUY NHẤT, không trùng với key khác đã dùng
  - **`baseQuery`**: cấu hình cách gửi request (bài sau)
  - **`endpoints`**: mô tả từng loại request cụ thể (bài sau)

## Creating an Endpoint
- `baseQuery`: dùng `fetchBaseQuery({ baseUrl: '...' })` (import từ RTK) để cấu hình phiên bản `fetch` có sẵn (built-in của browser, không phải Axios) - chỉ cần cung cấp `baseUrl` (URL gốc của server API)
- `endpoints`: 1 FUNCTION nhận argument `builder`, RETURN về 1 object mô tả từng loại request
- Với mỗi request cần trả lời trước các câu hỏi: mục đích request, tên ngắn gọn, là query hay mutation, path tương đối so với baseUrl, có query string không, method gì (GET/POST/DELETE...), có body không - trả lời rõ các câu hỏi này giúp việc viết code sau đó cực kỳ đơn giản, gần như "điền vào chỗ trống"
- Cú pháp định nghĩa 1 endpoint dạng QUERY:
```js
fetchAlbums: builder.query({
  query: (user) => ({
    url: '/albums',
    params: { userId: user.id },
    method: 'GET',
  }),
}),
```
- `query` (tên field, dễ gây nhầm) là 1 function nhận vào ĐÚNG argument mà component sẽ truyền vào khi gọi hook - return về object mô tả chi tiết request (url, params, method, body)
- KEY đặt tên cho endpoint (VD `fetchAlbums`) quyết định tên hook tự động sinh ra theo pattern `use` + TênEndpoint + `Query`/`Mutation`

## Using the Generated Hook
- Sau khi định nghĩa endpoint, cần export hook tự động sinh ra: `export const { useFetchAlbumsQuery } = albumsApi`
- Cần "kết nối" API vào store - PHỨC TẠP HƠN so với kết nối 1 slice thông thường, gồm nhiều bước: (1) thêm combined reducer của API vào `configureStore` (dùng `[albumsApi.reducerPath]: albumsApi.reducer` để tránh gõ tay string dễ sai), (2) thêm `middleware` bắt buộc (`getDefaultMiddleware().concat(albumsApi.middleware)`), (3) gọi `setupListeners(store.dispatch)` 1 LẦN DUY NHẤT (dùng chung cho mọi API, không lặp lại cho từng API)
- Cách dùng hook trong component: gọi TRỰC TIẾP, KHÔNG cần bọc trong `useEffect` hay event handler - hook TỰ ĐỘNG fetch data ngay khi component render lần đầu:
```js
const { data, error, isLoading } = useFetchAlbumsQuery(user);
```
- Argument truyền vào hook chính là argument sẽ được chuyển tới function `query` đã định nghĩa trong endpoint

## Changing Data with Mutations
- Định nghĩa 1 endpoint dạng MUTATION dùng `builder.mutation` thay vì `builder.query` - cấu trúc bên trong (field `query`) giống hệt cách viết của query, chỉ khác method (POST/DELETE...) và có thể có `body`
- Hook mutation tự động sinh có TÊN KHÁC CHÚT ít so với query - không có "s" ở cuối tên endpoint gốc khi tạo tên hook (VD endpoint `addAlbum` → hook `useAddAlbumMutation`)
- **Cách gọi hook mutation KHÁC HẲN hook query** - trả về 1 MẢNG 2 phần tử: (1) 1 FUNCTION dùng để TỰ TAY TRIGGER thực thi mutation khi cần (VD lúc user click nút), (2) object `results` chứa trạng thái (tương tự `data`/`error`/`isLoading` của query)
- Mutation KHÔNG tự động chạy khi component render (khác hẳn query) - phải chủ động gọi function trigger đó trong event handler:
```js
const [addAlbum, results] = useAddAlbumMutation();
// ...
const handleAddAlbum = () => addAlbum(user);
```

## Differences Between Queries and Mutations
- **Query hook**: TỰ ĐỘNG chạy fetch ngay khi component hiển thị lần đầu (hành vi mặc định, có thể tuỳ chỉnh trì hoãn) - trả về TRỰC TIẾP 1 object (`data`, `error`, `isLoading`...)
- **Mutation hook**: KHÔNG tự động chạy - trả về 1 MẢNG gồm (function để trigger, object kết quả) - vì thay đổi dữ liệu thường cần xảy ra ĐÚNG LÚC user thao tác (click save, submit form...), không nên tự động chạy ngay khi component mount
- `results` object của mutation có thêm property `status` (VD `'uninitialized'` khi chưa từng gọi, chuyển trạng thái sau khi gọi) - cung cấp thông tin chi tiết hơn về trạng thái của lần gọi mutation gần nhất

## Refetching with Tags
- Vấn đề cốt lõi: sau khi chạy 1 MUTATION (VD thêm album mới), dữ liệu trên SERVER đã đổi, nhưng dữ liệu đã CACHE SẴN trong Redux store (từ lần QUERY trước đó) vẫn CŨ, không tự động cập nhật - gây ra tình trạng UI không hiển thị dữ liệu mới dù request đã thành công
- **Hệ thống TAG**: giải pháp của RTK Query để TỰ ĐỘNG refetch lại đúng những query đã bị dữ liệu ảnh hưởng, sau khi 1 mutation chạy xong
- Cách hoạt động cơ bản: gắn `providesTags: ['Album']` vào 1 endpoint QUERY (đánh dấu: "query này liên quan tới tag Album") - gắn `invalidatesTags: ['Album']` vào 1 endpoint MUTATION (đánh dấu: "khi mutation này chạy xong, mọi query có tag 'Album' cần bị coi là LỖI THỜI")
- Khi mutation chạy xong và tag bị invalidate, RTK Query TỰ ĐỘNG chạy lại (refetch) TẤT CẢ query có tag khớp - không cần tự tay gọi lại hook hay dispatch gì thêm
- Giá trị chuỗi tag không có ý nghĩa đặc biệt về mặt kỹ thuật, chỉ cần KHỚP CHÍNH XÁC giữa `providesTags` và `invalidatesTags` - convention: đặt tên singular, viết hoa chữ cái đầu (VD `'Album'`)
- **Vấn đề thực tế phát sinh khi dùng tag đơn giản (string cố định):** nếu nhiều component cùng gọi 1 query (VD nhiều user panel cùng mở, mỗi cái tự fetch album riêng), TẤT CẢ đều mang CHUNG 1 tag string → khi chỉ 1 mutation ảnh hưởng tới 1 user cụ thể, TOÀN BỘ các query khác (của user không liên quan) cũng bị coi là stale và bị refetch KHÔNG CẦN THIẾT - gây lãng phí request

## Fine-Grained Tag Validation
- Giải pháp cho vấn đề "refetch thừa": thay vì tag là STRING đơn giản, dùng OBJECT có 2 property `{ type, id }` - cho phép TAG HOÁ CHI TIẾT theo từng bản ghi cụ thể (VD theo ID của user), không chỉ theo "loại" dữ liệu chung chung
- `providesTags` khi cần tag động: viết dưới dạng FUNCTION (không phải mảng cố định), nhận vào `(result, error, arg)` - trong đó `arg` chính là argument đã truyền vào hook lúc gọi (VD user object) - return về mảng tag object động dựa trên dữ liệu thực tế: `[{ type: 'Album', id: user.id }]`
- `invalidatesTags` tương tự cũng viết dưới dạng FUNCTION `(result, error, arg)`, với `arg` là argument đã truyền vào khi gọi MUTATION - return về đúng tag object cần vô hiệu hoá, khớp CHÍNH XÁC với tag của query cụ thể cần refetch
- Kết quả: chỉ ĐÚNG query liên quan tới user cụ thể bị đánh dấu lỗi thời và refetch lại, các query của user khác không bị ảnh hưởng - giải quyết triệt để vấn đề refetch thừa
- Bài học tổng quát quan trọng: việc thiết kế tag KHÔNG có công thức cố định, y hệt hoàn toàn giữa mọi trường hợp - luôn phải TỰ PHÂN TÍCH xem 1 mutation cụ thể sẽ ẢNH HƯỞNG tới NHỮNG QUERY NÀO, rồi thiết kế tag sao cho khớp đúng ý đồ đó, không có 1 pattern universal áp dụng máy móc cho mọi tình huống

## Getting Clever with Cache Tags (Kỹ thuật tag nâng cao khi endpoint không có sẵn dữ liệu cần thiết)
- Vấn đề nâng cao: đôi khi 1 mutation (VD xoá album) CHỈ CÓ trong tay 1 phần dữ liệu hạn chế (VD chỉ có album ID, KHÔNG có sẵn user ID) - nhưng cần invalidate đúng tag đang gắn theo user ID → không thể áp dụng công thức đơn giản như bài trước
- Giải pháp KHÔNG NÊN làm: cố tình sửa cách TRUYỀN PROPS giữa các component (VD truyền thêm user vào nơi không cần) chỉ để phục vụ nhu cầu của Redux - nên GIỮ NGUYÊN cấu trúc component, tìm giải pháp khác ở tầng Redux
- Giải pháp ĐÚNG - "trở nên khôn khéo hơn" với tag: 1 endpoint QUERY có thể trả về NHIỀU TAG CÙNG LÚC (không chỉ 1), bao gồm: (1) 1 tag RIÊNG cho TỪNG bản ghi (VD mỗi album 1 tag `{ type: 'Album', id: album.id }`), VÀ (2) 1 tag TỔNG QUÁT hơn đại diện cho "toàn bộ nhóm" (VD `{ type: 'UsersAlbums', id: user.id }`)
- Nhờ có NHIỀU tag gắn vào CÙNG 1 query, các mutation KHÁC NHAU (có dữ liệu sẵn có khác nhau) đều có thể tìm được ĐÚNG loại tag phù hợp với thông tin chúng đang có để invalidate: mutation XOÁ album (chỉ có album ID) → invalidate theo tag `{ type: 'Album', id: albumId }`; mutation THÊM album (có sẵn user object) → invalidate theo tag `{ type: 'UsersAlbums', id: user.id }` - cả 2 đều trỏ tới ĐÚNG 1 query, dù xuất phát từ 2 loại dữ liệu khác nhau
- Bài học tổng quát: khi thiết kế tag, luôn cân nhắc gắn NHIỀU LOẠI TAG khác nhau cho cùng 1 query nếu cần - để các mutation với dữ liệu đầu vào khác nhau đều có "cửa" phù hợp để invalidate đúng, tránh phải thay đổi cấu trúc data/props chỉ để phục vụ nhu cầu của caching system

## Adding Automatic Data Refetching (Áp dụng thực chiến cho Photos)
- Áp dụng lại đúng chiến lược tag "kết hợp nhiều loại" (từ bài trước) cho feature Photos: mỗi ảnh có 1 tag riêng theo ID của chính nó (`{ type: 'Photo', id: photo.id }`), CỘNG THÊM 1 tag đại diện cho CẢ ALBUM chứa những ảnh đó (`{ type: 'AlbumPhoto', id: album.id }`)
- Mutation XOÁ ảnh (chỉ có sẵn photo object, có photo ID) → invalidate theo tag `{ type: 'Photo', id: photo.id }` - khớp đúng tag riêng của từng ảnh
- Mutation THÊM ảnh (chỉ có sẵn album object, có album ID) → invalidate theo tag `{ type: 'AlbumPhoto', id: album.id }` - khớp đúng tag đại diện cho cả album
- Đây là minh chứng thực tế cho nguyên tắc: khi các mutation khác nhau có QUYỀN TRUY CẬP DỮ LIỆU khác nhau (cái thì chỉ biết ID bản ghi, cái thì chỉ biết ID nhóm cha), thiết kế NHIỀU TẦNG TAG trên cùng 1 query giúp TẤT CẢ các mutation đều tìm được đúng "chìa khoá" để trigger refetch chính xác, mà không cần thay đổi cấu trúc dữ liệu hay props truyền giữa các component
- Hệ thống TAG được xác nhận là "TÍNH NĂNG QUAN TRỌNG NHẤT" (killer feature) của toàn bộ RTK Query - đáng đầu tư thời gian hiểu sâu vì sẽ áp dụng lặp lại liên tục trong bất kỳ dự án thực tế nào dùng RTK Query