## Into the World of Redux
- Redux: thư viện JS quản lý state cho toàn bộ ứng dụng, dùng CHUNG kỹ thuật với `useReducer` (dispatch, action, reducer) - về bản chất là cùng 1 tư duy, chỉ khác quy mô
- Khác biệt 1 - vị trí state: `useReducer` tạo state SỐNG BÊN TRONG 1 component (và children của nó); Redux tạo 1 object riêng biệt gọi là **Redux Store**, nằm HOÀN TOÀN BÊN NGOÀI cây component - mọi component cần state phải tự "kết nối" tới store
- Thư viện **React Redux**: giúp kết nối Redux Store với React app dễ dàng hơn - 100% optional về kỹ thuật nhưng gần như MỌI dự án thực tế đều dùng. Bản chất bên dưới nó cũng chỉ dùng Context, không có gì "phép màu"
- Khác biệt 2 - số lượng reducer: `useReducer` thường chỉ có 1 reducer duy nhất quản lý hết; Redux thường có NHIỀU reducer khác nhau, mỗi cái phụ trách 1 phần riêng của state (VD reducer cho users, reducer cho videos, reducer cho messages) - nếu dự án chỉ cần đúng 1 reducer, đó là dấu hiệu có thể chưa cần dùng tới Redux
- Khác biệt 3 - độ phức tạp của state object: state trong Redux thường có NHIỀU property con (VD `{ users, videos, messages }`), mỗi property được tạo ra bởi 1 reducer riêng - giúp tách trách nhiệm quản lý state, tránh 1 reducer khổng lồ ôm hết logic

## Redux vs Redux Toolkit
- Lý do pattern dispatch/action/reducer phổ biến: khi app có RẤT NHIỀU component, dispatch function đóng vai trò là "điểm thay đổi trung tâm" duy nhất - giúp engineer dễ hiểu VÌ SAO state đang đổi, DỄ DEBUG (VD chỉ cần console.log mọi action được dispatch là thấy hết lịch sử thay đổi state)
- Nhược điểm của pattern này (Redux thuần/cổ điển): phải viết RẤT NHIỀU boilerplate - định nghĩa constant action types, export/import chúng giữa nhiều file, viết switch statement dài trong reducer - toàn bộ chỉ để "báo" cho reducer biết nó đang được gọi vì lý do gì
- **Redux Toolkit (RTK)**: là 1 THƯ VIỆN BỌC (wrapper) quanh Redux gốc - dùng RTK vẫn là đang dùng Redux, chỉ đơn giản hoá quá trình tạo action type + giảm code boilerplate + tự động sinh switch statement bên dưới
- RTK là cách làm ĐƯỢC KHUYẾN NGHỊ hiện nay cho mọi dự án Redux mới - khoá học từ đây trở đi khi nói "Redux" thường ngầm hiểu là "Redux Toolkit"

## Understanding the Store
- Store: 1 object DUY NHẤT (hầu như mọi dự án chỉ có 1 store) chứa TOÀN BỘ state của ứng dụng
- Thông thường KHÔNG tương tác trực tiếp với store (việc đó do React Redux lo) - chỉ thao tác thủ công khi debug: `store.dispatch(actionObject)` để đổi state, `store.getState()` để xem toàn bộ state hiện tại
- Cấu trúc state object trong store: các KEY ở tầng ngoài cùng được định nghĩa NGAY KHI TẠO STORE (`configureStore({ reducer: { songs: songsSlice.reducer } })`) - key nào xuất hiện trong object `reducer` truyền vào `configureStore`, key đó sẽ xuất hiện trên state object; GIÁ TRỊ của từng key do đúng REDUCER tương ứng sinh ra và cập nhật theo thời gian
- Muốn đổi tên/cấu trúc của state object tầng ngoài cùng → sửa ở chính nơi gọi `configureStore`, không sửa ở đâu khác

## Understanding Slices
- `createSlice`: hàm của Redux Toolkit, nhận vào 1 object cấu hình gồm `name`, `initialState`, và `reducers` (object chứa các "mini-reducer" function) - trả về 1 object gọi là "slice"
- Mỗi function trong `reducers` object có thể hình dung như 1 CASE riêng lẻ trong 1 switch statement lớn - `createSlice` tự động GOM tất cả các mini-reducer này lại thành 1 reducer TỔNG (combined reducer), nằm ở property `slice.reducer` (số ít, khác với `reducers` truyền vào lúc đầu)
- Combined reducer chính là thứ được đưa vào `configureStore` - nó tự biết cách "định tuyến" 1 action object tới đúng mini-reducer tương ứng
- Cách combined reducer xác định NÊN CHẠY mini-reducer nào: dựa vào TYPE của action, theo pattern cố định `tênSlice/tênMiniReducer` (VD slice tên `song`, mini-reducer tên `addSong` → type tương ứng là `song/addSong`) - không cần tự nhớ pattern này vì có công cụ tự động tạo (action creators, học ở bài sau)
- Bên trong mỗi mini-reducer: TỰ ĐỘNG được dùng kèm thư viện Immer (được phép mutate state trực tiếp), KHÔNG cần return state mới - chỉ cần áp dụng đúng thay đổi lên state là đủ
- Điểm CỰC KỲ QUAN TRỌNG (nhắc lại nhiều lần xuyên suốt khoá): trong mỗi mini-reducer, biến `state` KHÔNG PHẢI toàn bộ state object của store - nó CHỈ là phần state riêng do đúng slice đó quản lý (VD với songsSlice, `state` chỉ là mảng songs, không phải `{ songs, movies, ... }`)

## Understanding Action Creators
- Action creator: 1 FUNCTION được `createSlice` TỰ ĐỘNG tạo ra cho MỖI mini-reducer đã định nghĩa - gọi function này sẽ trả về 1 action object sẵn sàng để dispatch, KHÔNG cần tự viết tay `{ type: '...', payload: ... }`
- Vị trí truy cập: nằm trên property `slice.actions` (số nhiều - LƯU Ý dễ nhầm với `slice.reducer` số ít ở bài trước) - property này bị đặt tên hơi gây nhầm lẫn (giảng viên nhận xét lẽ ra nên gọi là "actionCreators" mới đúng bản chất)
- Cách dùng: `slice.actions.addSong('tên bài hát')` → trả về `{ type: 'song/addSong', payload: 'tên bài hát' }` - đối số ĐẦU TIÊN (và duy nhất) truyền vào action creator chính là `payload`
- Mục đích DUY NHẤT của action creator: tránh phải tự tay gõ đúng string type (dễ gõ sai chính tả) và tự tạo object thủ công - hoàn toàn không có gì phức tạp hơn thế, không cần "nghĩ quá nhiều" về khái niệm này

## Updating State from a Component (6 bước cập nhật state)
1. Vào slice, thêm 1 mini-reducer mới xử lý đúng thay đổi state mong muốn
2. Ở cuối file slice, EXPORT action creator tương ứng (`export const addSong = songsSlice.actions.addSong`)
3. Xác định component cần dispatch (nơi có event handler liên quan)
4. Import action creator vừa export + import hook `useDispatch` từ `react-redux`
5. Gọi hook: `const dispatch = useDispatch()` - hook này dùng Context bên dưới để lấy đúng hàm `dispatch` của store
6. Trong event handler: gọi action creator để tạo action object, rồi gọi `dispatch(actionObject)` - thường viết gọn thành 1 dòng: `dispatch(addSong(song))`
- Không cần tạo mini-reducer MỚI mỗi lần muốn update state theo cùng 1 cách - có thể tái dùng lại action creator đã có ở bất kỳ đâu trong app

## Accessing State in a Component (4 bước đọc state)
1. Xác định component cần đọc state
2. Import hook `useSelector` từ `react-redux`
3. Gọi hook, truyền vào 1 "selector function": `useSelector(state => state.songs)` - selector nhận vào TOÀN BỘ state object của store, CHỈ return đúng phần component thực sự cần (tránh component phải nhận cả state không liên quan)
4. Dùng giá trị trả về từ hook để render UI
- **Điểm gây nhầm lẫn quan trọng (nhắc đi nhắc lại):** chữ "state" có 2 NGHĨA KHÁC NHAU tuỳ ngữ cảnh - BÊN TRONG 1 slice (trong mini-reducer), "state" chỉ là phần state riêng của slice đó; BÊN NGOÀI slice (VD trong `useSelector`), "state" luôn là TOÀN BỘ state object của store - nhầm lẫn giữa 2 nghĩa này là nguồn gốc bug rất phổ biến khi làm việc với Redux

## Resetting State (Vấn đề reset về mảng rỗng với Immer)
- Với hầu hết update, có thể mutate trực tiếp `state` (VD `state.push(...)`) nhờ Immer tự xử lý phía sau
- Riêng trường hợp muốn RESET state về giá trị hoàn toàn mới (VD mảng rỗng `[]`): viết `state = []` KHÔNG hoạt động - đây chỉ là GÁN LẠI biến cục bộ `state`, không phải MUTATE nó, nên Immer không nhận diện được đây là thay đổi cần áp dụng
- Cách đúng trong trường hợp đặc biệt này: RETURN giá trị mới mong muốn (`return []`) - Immer/Redux Toolkit hiểu rằng nếu reducer return 1 giá trị, đó chính là state mới, bất kể có mutate gì trước đó hay không

## Watching for Other Actions (extraReducers)
- Sự thật quan trọng về cơ chế dispatch: khi gọi `dispatch(action)`, action đó được gửi tới **MỌI combined reducer** trong store (không chỉ 1 slice cụ thể) - hành vi này không thể tắt, và cũng không cần tắt
- Mỗi combined reducer mặc định CHỈ PHẢN ỨNG với đúng những action type khớp với các mini-reducer nó tự định nghĩa (VD `song/addSong`, `song/removeSong`) - nếu nhận action type lạ, nó im lặng bỏ qua, không đổi gì
- `extraReducers`: 1 property BỔ SUNG có thể thêm vào cấu hình `createSlice`, dùng để dạy 1 slice "để ý" tới các action type KHÔNG PHẢI do chính nó tạo ra (VD dạy songsSlice phản ứng với action `movie/reset` vốn thuộc về movieSlice)
- Cú pháp: `extraReducers: (builder) => { builder.addCase(actionTypeOrCreator, (state, action) => {...}) }` - hàm xử lý bên trong `addCase` có cấu trúc/hành vi giống hệt 1 mini-reducer bình thường (cùng quy tắc Immer, cùng cách xử lý `state`)
- Nhược điểm cách làm này: tạo ra sự PHỤ THUỘC ngầm giữa 2 slice - nếu sau này đổi tên/xoá reducer gốc bên slice kia, slice đang "nghe ké" sẽ ngừng hoạt động đúng mà không có cảnh báo rõ ràng gì

## Manual Action Creation (createAction - giải pháp tách phụ thuộc)
- Giải pháp khắc phục nhược điểm ở trên: tạo ra 1 ACTION TYPE ĐỘC LẬP, KHÔNG gắn với bất kỳ slice cụ thể nào, rồi cho NHIỀU slice khác nhau cùng lắng nghe type đó qua `extraReducers`
- Dùng `createAction` (import từ Redux Toolkit) để tự tay tạo 1 action creator, không thông qua `createSlice`: `const reset = createAction('app/reset')`
- `reset` tạo ra vẫn là 1 action creator bình thường - gọi `reset()` sẽ trả về action object `{ type: 'app/reset' }`, dùng được y hệt các action creator tự động khác
- String truyền vào `createAction` không có ý nghĩa đặc biệt về mặt kỹ thuật, chỉ cần là 1 string DUY NHẤT, có ý nghĩa dễ hiểu (convention: `domain/hànhĐộng`)
- Cả 2 (hoặc nhiều) slice liên quan đều dùng `extraReducers` + `builder.addCase(reset, handler)` để cùng lắng nghe ĐÚNG 1 action type độc lập này - loại bỏ hoàn toàn sự phụ thuộc trực tiếp giữa các slice với nhau, mỗi slice chỉ phụ thuộc vào 1 action type dùng chung, không phụ thuộc vào cấu trúc nội bộ của slice khác