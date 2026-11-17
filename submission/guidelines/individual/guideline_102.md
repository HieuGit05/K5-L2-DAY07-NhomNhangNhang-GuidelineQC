# Guideline gán nhãn — v0.1 — 102

- Phiên bản: v0.1 (bản nháp cá nhân)
- Ngày: 2026-11-16
- Người viết: 102
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Ghi nhận các lớp xuất hiện trong bộ 40 ảnh đường phố để xây dựng nhãn phục vụ phân tích
cảnh giao thông. Danh sách dưới đây là kết quả rà soát ban đầu; cần thống nhất với nhóm
trước khi dùng làm taxonomy chính thức.

## 2. Lớp và thuộc tính

### Lớp (tên lớp giữ nguyên như đã tìm thấy)

| Lớp | Định nghĩa / cách hiểu ban đầu | Ghi chú |
|---|---|---|
| `person` | Người xuất hiện trong cảnh. | Phân biệt người với xe họ đang đi. |
| `truck` | Xe tải chở hàng. | Phân biệt với `van` và `bus`. |
| `sky` | Vùng bầu trời nhìn thấy. | Là vùng nền, không phải vật thể rời. |
| `vegetation` | Cây cối, bụi cây hoặc thảm thực vật. | Là vùng nền/vùng phủ. |
| `terain` | Địa hình hoặc bề mặt đất tự nhiên. | Tên có thể là lỗi chính tả của `terrain`; cần xác nhận. |
| `car` | Ô tô con. | Phân biệt với xe tải, xe van và xe buýt. |
| `sidewalk` | Vỉa hè dành cho người đi bộ. | Là vùng bề mặt. |
| `motorcycle` | Xe máy hoặc mô tô. | Người lái là lớp `person` riêng nếu quy tắc yêu cầu. |
| `bycycle` | Xe đạp. | Tên có thể là lỗi chính tả của `bicycle`; cần xác nhận. |
| `tuktuk` | Xe tuk-tuk/xe ba bánh có động cơ. | Cần thống nhất ranh giới với `motorcycle` và `car`. |
| `van` | Xe van hoặc xe chở hàng cỡ nhỏ. | Cần thống nhất ranh giới với `truck`. |
| `road` | Phần mặt đường dành cho phương tiện. | Là vùng bề mặt. |
| `crosswalk` | Vạch hoặc khu vực sang đường dành cho người đi bộ. | Cần chốt gán vùng vạch hay toàn bộ khu vực. |
| `bus` | Xe buýt chở khách. | Phân biệt với `van` và `truck`. |
| `building` | Tòa nhà hoặc công trình xây dựng trong cảnh. | Cần thống nhất có tính cả công trình không phải tòa nhà hay không. |
| `traficsign` | Biển báo giao thông. | Tên có thể là lỗi chính tả của `traffic_sign`; cần xác nhận. |
| `fence` | Hàng rào hoặc vật ngăn cách dạng rào. | Cần thống nhất với dải phân cách nếu xuất hiện. |
| `traficlight` | Đèn tín hiệu giao thông. | Tên có thể là lỗi chính tả của `traffic_light`; cần xác nhận. |
| `singlewhite` | Chưa xác định chắc; có thể là vạch kẻ đường màu trắng đơn. | Cần kiểm tra ảnh/mẫu nhãn trước khi chốt định nghĩa. |

Các tên `terain`, `bycycle`, `tuktuk`, `traficsign`, `traficlight` và `singlewhite`
được giữ nguyên như dữ liệu đã ghi nhận, không tự đổi tên label. Nhóm cần xác nhận tên chuẩn
và ý nghĩa trước khi cập nhật taxonomy/CVAT.

Danh sách có cả vật thể có thể vẽ box (ví dụ `person`, `car`) và vùng nền/bề mặt
(ví dụ `sky`, `road`, `sidewalk`). Cần xác nhận loại hình annotation cho nhóm vùng này;
không mặc định vẽ bounding box quanh vùng nền.

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

Các thuộc tính che khuất/cắt mép áp dụng cho vật thể; cần xác nhận chúng có áp dụng cho
nhãn vùng hay không.

## 3. Quy tắc vẽ box

1. Với các vật thể được gán bằng bounding box, box ôm sát phần nhìn thấy của vật.
2. Không dùng box cho `sky`, `vegetation`, `terain`, `sidewalk`, `road` hoặc các lớp
   vùng khác cho tới khi nhóm xác nhận cách gán vùng.
3. Với người đi xe, chờ nhóm thống nhất số box và cách gán lớp cho người và phương tiện.
4. Ca chưa rõ lớp hoặc cách gán: đánh dấu `can_review` và ghi lại ảnh/vùng cần xem.
5. **R-01 — Vật mô phỏng:** không gán `person` hoặc lớp phương tiện cho tượng, tác phẩm
   điêu khắc, ma-nơ-canh, mô hình/đồ chơi hoặc hình ảnh in. Chất liệu tượng không làm thay
   đổi quyết định.
6. **R-02 — Phương tiện tự chế/khó nhận dạng:** nếu xác định đó là phương tiện thật và nhận
   ra được loại xe theo taxonomy hiện có, gán lớp tương ứng; không tạo lớp mới. Nếu chưa xác
   định được loại xe, đánh dấu `can_review`, không đoán lớp.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_024` | Mô hình ô tô đặt trên nóc ô tô thật. | R-01 | Gán `car` cho ô tô thật; bỏ qua mô hình trên nóc. | Mô hình không phải phương tiện thật trong cảnh. |
| `img_019` | Ô tô tự chế/khác thường. | R-02 | Nếu là ô tô thật, gán `car`; nếu không đủ căn cứ xác định loại xe, đánh dấu `can_review`. | Không tạo lớp riêng chỉ vì xe có thiết kế khác thường. |
| `img_017` | Tượng ngựa và người cưỡi bằng kim loại/đá (không rõ chất liệu). | R-01 | Không gán `person` hoặc lớp phương tiện. | Đây là tượng; chất liệu không ảnh hưởng quyết định. |
| `img_011` | Tượng quái vật và một phương tiện có kiểu dáng lạ, chưa rõ loại. | R-01, R-02 | Bỏ qua tượng; phương tiện chỉ gán lớp nếu xác định được loại, nếu không đánh dấu `can_review`. | Phân biệt vật mô phỏng với phương tiện thật; không đoán lớp xe. |
| `img_027` | Tượng ô tô bằng đá/gạch trên bia mộ. | R-01 | Không gán `car`. | Hình ô tô là tác phẩm/tượng, không phải phương tiện thật. |
| `img_033` | Ma-nơ-canh trong cửa hàng. | R-01 | Không gán `person`. | Ma-nơ-canh không phải người thật. |
| `img_038` | Các ô tô đồ chơi/mô hình trong tủ trưng bày. | R-01 | Không gán `car`. | Đồ chơi/mô hình không phải phương tiện thật. |

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
- Trước khi phát hành bản thống nhất, kiểm tra tên lớp và loại annotation với taxonomy/CVAT.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Ghi lại 19 lớp tìm thấy khi rà soát 40 ảnh và đánh dấu các lớp cần xác minh. | Bản nháp cá nhân; cần thống nhất với nhóm. |
| v0.1 | Thêm R-01/R-02 và quyết định sơ bộ cho 7 ca khó. | `img_011`, `img_017`, `img_019`, `img_024`, `img_027`, `img_033`, `img_038`. |
