# Guideline gán nhãn — v0.1 (bản cá nhân) — 101

- Phiên bản: v0.1-101 (bản cá nhân Pha 1, viết từ v0 của giảng viên)
- Ngày: 2026-11-16
- Người viết: 101 · Người duyệt: (chờ họp nhóm Pha 2)
- Loại nhãn: bounding box (hộp chữ nhật, song song trục ảnh), CVAT, xuất COCO 1.0
- Phạm vi: 40 ảnh `data/images/img_001.jpg` … `img_040.jpg`

> Cách đọc: mỗi quy tắc có mã `R-xx`. Khi gán, gặp ca khó thì tra **Mục 4** trước. Nếu không có ca giống,
> áp quy tắc gần nhất và tick `can_review`.

---

## 1. Mục tiêu

**Mô hình dùng nhãn:** bộ phát hiện vật 2D cho **camera giao thông / hỗ trợ lái xe** trên đường phố
đông xe máy kiểu Đông Nam Á (Hà Nội, Sài Gòn, Huế, Bangkok, Pattaya…). Mô hình nhận ảnh, trả về box + lớp
cho **người tham gia giao thông và phương tiện**.

**Nhãn phải giúp mô hình:**
1. Tìm được **mọi người và mọi xe thật** trong ảnh, kể cả xe đỗ, người đứng bên đường.
2. Phân biệt các loại xe đủ để ước lượng kích thước và hành vi (xe máy ≠ ô tô ≠ xe tải ≠ xe buýt).
3. **Không** báo vật giả: tượng, đồ chơi, ma-nơ-canh, người trên biển quảng cáo, hình phản chiếu.

**Sai ở đâu thì nguy hiểm nhất (theo thứ tự):**

| Mức | Lỗi | Hậu quả |
|---|---|---|
| Rất nặng | Bỏ sót `person`, `motorcycle`, `bicycle` | Mô hình học "không có ai" → không phanh cho người đi bộ / xe máy |
| Nặng | Gán vật giả (tượng, poster, phản chiếu) thành vật thật | Mô hình phanh "ma", mất tin cậy |
| Trung bình | Box lệch nhiều, cắt mất một phần vật | Ước lượng sai khoảng cách, vị trí |
| Nhẹ | Nhầm giữa các lớp xe gần nhau (`van` ↔ `truck`, `van` ↔ `bus`) | Sai kích thước dự đoán, ít nguy hiểm hơn bỏ sót |

→ **Khi phân vân giữa "vẽ" và "không vẽ" một người/xe thật: vẽ** và tick `can_review`.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp (mỗi box thuộc đúng một lớp)

v0 chỉ có 5 lớp nên không có chỗ cho xe buýt, tuk-tuk, xe đạp, xích lô, mà các loại này xuất hiện ở hơn 10 ảnh.
Bản này đề xuất **8 lớp vật + 1 vùng bỏ qua**:

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhìn thấy trực tiếp, đứng, đi, ngồi, kể cả người đang lái/ngồi trên xe máy, xe đạp, xích lô, tuk-tuk. | img_014 (2 người qua đường), img_005 (người trên xe máy), img_004 (tài xế tuk-tuk) | Tượng (img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018), người ngồi **sau kính** ô tô/xe buýt (R-13) |
| `car` | Ô tô chở người cỡ nhỏ: sedan, hatchback, SUV, MPV 7 chỗ, taxi, xe đua/xe diễu hành chạy được. | img_015, img_023, img_034, img_019 (xe diễu hành vàng), img_024 (xe trưng bày) | Pickup → `truck`. Xe thùng cao, cửa trượt → `van`. Mô hình/đồ chơi (img_038) → không gán |
| `van` | Xe thân hộp **liền khối** (khoang sau nối liền cabin), mui cao, thường có cửa trượt hông, chở người hoặc hàng; dài dưới ~6 m. | img_005 (xe 51LD bạc), img_016 (xe trắng bên phải) | SUV/MPV mui thấp → `car`. Thân dài, nhiều hàng cửa sổ, ≥ 2 cửa khách → `bus` |
| `truck` | Xe chở hàng có **thùng hàng tách riêng khỏi cabin**: pickup/bán tải, xe tải nhẹ, xe tải thùng, đầu kéo + rơ-moóc, xe chở ô tô. | img_001, img_012, img_028, img_037 (pickup), img_036, img_007, img_002 | Xe thùng liền cabin → `van`. Tượng xe tải bằng đá (img_027) → không gán |
| `bus` | Xe chở khách nhiều chỗ: xe buýt, xe khách, minibus thân dài có từ 2 cửa khách hoặc ≥ 4 ô cửa sổ bên phía sau ghế lái. | img_009 (minibus trắng), img_031 (xe buýt 82), img_037 (xe buýt xanh trái), img_040 (xe buýt vàng) | Van chở khách 1 cửa trượt (img_016) → `van` |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô, xe máy kèm thùng sidecar; có người hay không đều gán. | img_006, img_010, img_030 (bãi đỗ), img_022 (sidecar) | Xe đạp, xe đạp điện dáng xe đạp → `bicycle`. Xe ba bánh có cabin → `three_wheeler` |
| `bicycle` | Xe đạp chạy bằng sức người, 2 hoặc 3 bánh, **kể cả xích lô** và xe đạp chở hàng. | img_006 (xe đạp phải), img_029 (xe đạp cạnh xe máy), img_023 (xích lô), img_003 (xe đạp chở hàng) | Xe máy → `motorcycle` |
| `three_wheeler` | Xe ba bánh **có động cơ** chở người/hàng: tuk-tuk, xe lam, auto-rickshaw, xe ba gác máy. | img_004, img_016 (tuk-tuk xanh lá + xanh dương), img_035 (rickshaw vàng), img_040 (tuk-tuk vàng) | Xích lô đạp → `bicycle`. Xe máy + sidecar → `motorcycle` |
| `ignore_region` | **Vùng** (không phải vật) có nhiều vật thuộc lớp nhưng không tách được từng cái; bỏ sót trong vùng này không tính lỗi. | Đám đông sau rào img_019, cụm xe máy dày phía xa img_021 / img_030 | Không dùng để che vật giả (xem R-09) |

**Vật không thuộc lớp nào** (chó, chim, ngựa tượng, cột, nón giao thông, xe đẩy tay không bánh xe đạp, gánh hàng):
**không gán gì**, cũng **không** vẽ `ignore_region`. Mô hình cần thấy chúng như nền.

### 2.2 Thuộc tính

Mỗi box lớp vật (không áp cho `ignore_region`) có 3 thuộc tính chung: `occluded_level`, `truncated`,
`can_review`. Riêng box `person` có thêm thuộc tính thứ 4 là `on_vehicle`.

#### `occluded_level`: mức bị **vật khác** che (không tính phần bị mép ảnh cắt)

Ước lượng phần trăm **thân vật** (tưởng tượng cả vật) bị vật khác che khuất:

| Mức | Ngưỡng bị che | Ví dụ |
|---|---|---|
| `0` | ≤ 10 %: thấy gần trọn vật | Người áo trắng qua đường img_034 |
| `1` | > 10 % đến 40 % | Xe máy bị chân người lái che một phần; ô tô sau hàng rào lưới thưa img_018 |
| `2` | > 40 % đến 70 % | Xe máy giữa bãi đỗ img_030 chỉ thấy yên và đầu xe |
| `3` | > 70 %: chỉ còn một phần nhỏ nhưng **vẫn nhận ra lớp** | Người phía sau đám đông chỉ thấy đầu + vai img_021 |

- Hàng rào / song sắt / lưới **thưa** nhìn xuyên được (img_018, img_019): tính tối đa mức `1` nếu không có vật đặc nào khác che.
- Người lái che xe máy của chính họ: tính bình thường (thường là `1`).
- Nếu che > 70 % **và** không nhận ra lớp → không vẽ box.

#### `truncated`: vật bị **mép ảnh** cắt

| Giá trị | Khi nào |
|---|---|
| `no` | Toàn bộ vật nằm trong ảnh. |
| `yes` | Box chạm mép ảnh (cách mép ≤ 2 px) **và** rõ ràng vật còn tiếp ra ngoài khung. |

Ví dụ `yes`: ô tô đỏ và ô tô bạc img_015, người áo đỏ img_016, xe tải img_012, người lái xe máy img_040.

#### `on_vehicle`: người có đang ở trên xe không (chỉ cho `person`)

| Giá trị | Khi nào | Ví dụ |
|---|---|---|
| `no` | Người đi bộ, đứng hoặc ngồi trên mặt đất, ghế, vỉa hè; người **dắt xe** hoặc **đứng cạnh xe** | 2 người qua đường img_014, khách quán img_001, người dắt xe đạp img_003 |
| `yes` | Người đang **ngồi hoặc đứng trên** xe máy, xe đạp, xích lô, tuk-tuk, xe hở: người lái, người ngồi sau, trẻ em ngồi trước | img_005, img_026, img_023, tài xế tuk-tuk img_004 |

- Xe **đang đỗ** nhưng người vẫn ngồi trên yên → `yes` (cô gái xem điện thoại img_010).
- Ngồi trên yên, chân chống đất → `yes`.
- Đứng trong thùng xe tải hoặc bám ngoài xe → `yes`.
- Người sau kính xe kín không được vẽ (R-13), nên không có ca `yes` cho loại này.
- Mặc định là `no`. Khi vẽ người trên xe, nhớ đổi sang `yes`.
- Thuộc tính này không áp cho các lớp xe; với chúng, để giá trị mặc định.

Mục đích: mô hình vẫn học một lớp `person` chung (không làm giảm đồng thuận lớp), nhưng dữ liệu vẫn tách được
người đi bộ, nhóm cần ưu tiên cảnh báo, với người đi xe.

#### `can_review`: cờ "tôi chưa chắc"

Tick khi: (a) ca không có trong Mục 4 và hai quy tắc cho hai kết quả khác nhau; (b) ca nằm trong nhóm
**"cần hỏi reviewer"** ở Mục 4. Không tick chỉ vì box khó vẽ. Box khó vẽ thì vẽ cẩn thận hơn.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01 Ôm sát phần nhìn thấy** | Box ôm sát **phần nhìn thấy** của vật, không đoán phần bị che. Mỗi cạnh lệch tối đa ~2 px so với mép vật. Vật bị vật khác cắt làm hai phần nhìn thấy (người sau cột) → **một** box ôm cả hai phần. |
| **R-02 Vật nhỏ** | Box có chiều cao **< 20 px** → không vẽ. Nhiều vật nhỏ như vậy dồn thành cụm → được vẽ một `ignore_region` ôm cụm (không bắt buộc). |
| **R-03 Cắt mép** | Vẽ đến sát mép ảnh, `truncated = yes`. Phần bị cắt **không** tính vào `occluded_level`. |
| **R-04 Che khuất** | Vẫn vẽ nếu nhận ra lớp và phần thấy được cao ≥ 20 px; chọn `occluded_level` theo bảng 2.2. |
| **R-05 Người trên xe** | Người trên `motorcycle` / `bicycle` / `three_wheeler` / xe hở: **2 loại box riêng**. Mỗi người một box `person` (cả người ngồi sau, trẻ em ngồi trước). Box xe ôm bánh, thân, tay lái, gương, kính chắn gió của xe, **không** kéo lên để ôm đầu người lái. Box `person` ôm cả phần thân người nằm trước xe. Mọi box `person` này đặt `on_vehicle = yes`. |
| **R-06 Đồ mang theo của người** | Box `person` ôm cơ thể, quần áo, mũ/nón **đang đội**, ba lô **đeo sát lưng**. Không ôm đồ cầm tay đung đưa, gánh, đồ vác dài nhô ra (túi xách img_014, quang gánh img_003, ống nhựa img_029). |
| **R-07 Hàng trên xe** | Box xe ôm thân xe + kết cấu cố định (thùng, mui, giàn, khung, rơ-moóc đang nối). Hàng hóa nhô ra khỏi thân xe **không** kéo box ra (bao hàng chất cao img_035, ống img_029). Hàng nằm gọn trong thùng thì tự nhiên nằm trong box (img_012). |
| **R-08 Xe chở xe** | Xe thật được chở trên xe khác (img_002, img_024) → mỗi xe được chở **một box riêng** theo lớp của nó; xe chở **không** ôm chúng (theo R-07). Đầu kéo + rơ-moóc nối liền = **1 box** `truck`. |
| **R-09 Vật không thật** | Tượng, tượng đài, mô hình, đồ chơi, ma-nơ-canh, người/xe in trên biển quảng cáo, tranh tường, ảnh dán, **hình phản chiếu** (gương, kính, vũng nước) → **không vẽ gì**, không vẽ `ignore_region`. Lý do: đây là mẫu âm mô hình cần học. |
| **R-10 Xe chủ (ego)** | Phần xe của người chụp (capô, gương, khung cửa ngay trước ống kính, như img_020) → không vẽ. |
| **R-11 Đám đông** | Vẽ box riêng cho từng vật **tách được ranh giới** và cao ≥ 20 px. Phần còn lại chồng lấn không tách được → gom vào **một** `ignore_region` ôm sát cụm đó. `ignore_region` không phủ quá nửa một box đã vẽ (script coi box bị phủ ≥ 50 % là bỏ qua). Mỗi ảnh tối đa ~3 vùng. |
| **R-12 Mờ, tối** | Mờ chuyển động / thiếu sáng: vẫn vẽ nếu nhận ra lớp; box ôm **lõi** vật, bỏ vệt mờ mỏng. Không xác định được mép trong khoảng ±10 px → không vẽ box lớp; nếu đó là cụm nhiều vật → `ignore_region`. |
| **R-13 Người sau kính** | Người ngồi trong ô tô / van / xe buýt / xe tải và chỉ thấy **qua kính** → không vẽ `person`. Người ở xe hở (xe máy, xích lô, tuk-tuk nhìn từ bên hông, xe mui trần img_019) → vẽ. |
| **R-14 Hoá trang, mascot** | Thấy bộ phận người thật (mặt, tay, chân) → `person`. Không thấy (thú bông, hình nộm, tượng) → không vẽ. |
| **R-15 Xe đỗ, không người** | Vẫn gán lớp bình thường (bãi xe máy img_030, xe máy đỗ img_036 / img_039). |
| **R-16 Ảnh không có vật** | Không vẽ gì; ảnh **vẫn phải có trong file COCO** export. |

**Thứ tự làm mỗi ảnh:** (1) lướt ảnh, gạch ra vật giả, xe chủ (R-09, R-10) → (2) vẽ `person` → (3) vẽ xe → (4) vẽ `ignore_region` nếu có đám đông → (5) đặt thuộc tính (nhớ `on_vehicle` cho từng `person`) → (6) tick `can_review` nếu cần.

---

## 4. Ca khó

Cột **Loại**: `QT` = đã có quy tắc, cứ làm theo; `HỎI` = tôi đề xuất quyết định nhưng cần reviewer chốt → tick `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_005 | Nhiều người đội mũ bảo hiểm trên xe máy, sát nhau, bị cắt mép dưới | R-05, R-03, R-04 | Mỗi người 1 `person`; xe máy thấy được (đầu xe, gương) → `motorcycle`; xe 51LD → `van`; xe bạc trái → `car`, `truncated=yes` | Người tham gia giao thông là ưu tiên số 1 | QT |
| img_006, img_021 | Dòng xe máy dày, xa dần | R-05, R-11, R-02 | Cụm gần: vẽ từng `person` + `motorcycle`; cụm xa chồng lấn, cao < 20 px → một `ignore_region` | Tách được thì vẽ, không tách được thì bỏ qua có kiểm soát | QT |
| img_029 | Người đi xe máy vác ống nhựa dài; người bên cạnh đi xe đạp | R-05, R-06, R-07 | 2 `person`, 1 `motorcycle`, 1 `bicycle`; box **không** ôm ống nhựa; người đi bộ/xe đạp nhỏ phía xa: vẽ nếu cao ≥ 20 px | Đồ vác không phải một phần của người/xe | QT |
| img_022 | Mô tô kèm thùng sidecar, ảnh đen trắng | R-05 | Mô tô + sidecar = 1 box `motorcycle`; người ngồi → `person` | Sidecar gắn liền, cùng chuyển động | QT |
| img_023 | Xích lô đạp có người đạp phía sau; ô tô đen | R-05 | Xích lô → `bicycle`; người đạp → `person`; ô tô → `car` | Xích lô chạy bằng sức người | QT |
| img_004, img_016 | Tuk-tuk có tài xế | R-05, R-13 | Tuk-tuk → `three_wheeler`; tài xế nhìn thấy trực tiếp từ hông → `person` | Xe hở, không qua kính | QT |
| img_035 | Auto-rickshaw chất bao hàng cao gấp đôi thân xe | R-07 | Box `three_wheeler` dừng ở mui xe, **không** ôm đống bao; xe máy + người phía trước vẽ riêng | Hàng hóa thay đổi theo chuyến, không phải hình dạng xe | QT |
| img_002 | Xe tải chở 9 ô tô trên giàn 2 tầng | R-08, R-07, R-04 | 1 box `truck` (đầu kéo + giàn + rơ-moóc, ôm đến đỉnh khung giàn); mỗi ô tô thấy được → 1 `car`, `occluded_level` theo song giàn che; người đứng cuối xe → `person` | Các ô tô là xe thật, cần phát hiện | QT |
| img_024 | Ô tô vàng đặt trên giá nóc ô tô đen trong bảo tàng | R-08, R-09 | Cả hai là xe thật → 2 box `car` riêng; xe đen không ôm xe vàng; các xe trưng bày khác cũng `car` | Xe trưng bày vẫn là xe thật (khác đồ chơi) | QT |
| img_001, img_012, img_028, img_037 | Xe bán tải (pickup), có/không nắp thùng | Lớp `truck` | → `truck`. img_037: nắp thùng (truck cap) vẫn là `truck`, không phải `van` | Thùng hàng tách khỏi cabin | QT |
| img_028 | Tượng ngựa đứng trong thùng xe bán tải thật | R-09, R-07 | Xe → `truck`; ngựa tượng → không gán; ô tô đỏ phía sau → `car` | Ngựa không thuộc lớp, lại là tượng | QT |
| img_027 | Tượng đài hình xe bán tải bằng đá ở nghĩa trang | R-09 | Không vẽ gì (ảnh 0 box) | Không phải xe thật | QT |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông | R-09 | Không vẽ gì | Tượng | QT |
| img_038 | Tủ kính đầy ô tô đồ chơi, búp bê | R-09 | Không vẽ gì | Đồ chơi là mẫu âm | QT |
| img_033 | Ma-nơ-canh, poster người mẫu, hình phản chiếu xe trắng trên kính | R-09 | Không vẽ gì | Cả ba đều là vật không thật | QT |
| img_018 | Người trên biển quảng cáo; ô tô thật sau hàng rào lưới | R-09, R-04 | Người trên biển: không vẽ. Ô tô thật cao ≥ 20 px → `car`, `occluded_level=1` (lưới thưa) hoặc cao hơn nếu xe trước che | Biển quảng cáo là ảnh in | QT |
| img_020 | Gương chiếu hậu của xe người chụp, trong gương có xe khác | R-10, R-09 | Không vẽ gương / cửa xe chủ; xe trong gương: không vẽ | Xe chủ và phản chiếu đều loại | QT |
| img_019 | Xe diễu hành vàng có người lái + mascot chuột; đám đông sau rào; đầu người mờ ở góc dưới phải | R-13, R-14, R-11 | Xe → `car`; người lái → `person`; mascot chuột (không thấy bộ phận người) → không vẽ; khán giả hàng đầu tách được → `person`; phía sau → `ignore_region`; đầu mờ góc dưới → `person`, `truncated=yes` | Xe chạy được, có biển số | HỎI (mascot có thể là người hóa trang) |
| img_011 | Hình Predator khổng lồ; dãy mô tô kim loại tự chế đang trưng bày; người ở xa | R-14, R-09, R-15 | Predator → không vẽ (không thấy bộ phận người thật); mô tô tự chế có bánh, khung, động cơ → `motorcycle`; người xa ≥ 20 px → `person`; xe trắng phía sau → `car` | Chưa chắc mô tô là xe chạy được hay tác phẩm điêu khắc | HỎI |
| img_040 | Người đội mũ bảo hiểm sát ống kính; 2 gương xe máy ở mép dưới; taxi, tuk-tuk, xe buýt | R-05, R-10, R-03, R-13 | Người → `person`, `truncated=yes`; 2 gương **là của xe người đó** → vẽ 1 `motorcycle` ôm phần thấy được (gương + tay lái), `truncated=yes`, `occluded_level=3`; taxi → `car`; tuk-tuk → `three_wheeler`; xe buýt → `bus`; người trong tuk-tuk sau kính → không vẽ | Chưa chắc gương thuộc xe người kia hay xe người chụp (ego) | HỎI |
| img_030 | Bãi đỗ xe máy rất dày, xe chồng lên nhau; người đứng phải | R-11, R-15, R-04 | Xe hàng trước tách được → từng `motorcycle`; khối phía sau không tách → `ignore_region`; người → `person` | Đỡ hàng chục box mơ hồ | QT |
| img_032 | Hàng người chờ xe buýt dài ~30 người | R-11, R-02 | Người tách được, cao ≥ 20 px → từng `person`; chỗ 3+ người chồng kín → `ignore_region`; ô tô xanh trái → `car`, `truncated=yes`; xe sau hàng người → `car` nếu ≥ 20 px | Người đi bộ là lớp quan trọng nhất | QT |
| img_014 | Người đi xe máy bị mờ chuyển động mạnh phía sau 2 người qua đường | R-12, R-05 | 2 người rõ → `person`; xe máy mờ: lõi nhận ra được → `person` + `motorcycle`; vệt mờ không xác định mép → không vẽ | Ổn định giữa người gán | QT |
| img_007, img_008, img_013, img_031 | Ban đêm, ánh đèn, mặt đường phản chiếu | R-12, R-09 | Vẽ vật nhìn rõ; hình phản chiếu trên mặt đường ướt: không vẽ; img_031 xe buýt → `bus` | Phản chiếu là vật giả | QT |
| img_009 | Minibus trắng thân dài (~29 chỗ) | Lớp `bus` / `van` | → `bus` (≥ 4 ô cửa sổ sau ghế lái) | Theo ngưỡng định nghĩa `bus` | QT |
| img_003 | Người bán hàng rong gánh hàng, đội nón lá; xe đạp chở hàng | R-06, Lớp `bicycle` | Box `person` ôm nón lá, **không** ôm quang gánh; người gánh hàng và người đứng dắt xe đạp → `on_vehicle=no`; người ngồi trên xe máy → `on_vehicle=yes`; xe đạp → `bicycle`; xe máy → `motorcycle` | Nón đang đội là một phần người | QT |
| img_010 | Người ngồi trên xe máy đỗ đang xem điện thoại; xe bán rau (xe đạp chở hàng) | R-05, R-15 | Người ngồi trên xe máy đỗ → `person`, `on_vehicle=yes`; người bán rau đứng → `person`, `on_vehicle=no`; xe máy đỗ → `motorcycle`; xe đạp chở rau → `bicycle` nếu thấy bánh và khung | Xe đỗ vẫn gán | HỎI (xe rau có thể là xe đẩy, không phải xe đạp) |
| img_025, img_039 | Người đi bộ, xe máy nhỏ ở xa | R-02 | Vẽ khi cao ≥ 20 px; nhỏ hơn → bỏ, cụm nhỏ → `ignore_region` tùy chọn | Ngưỡng `min_box_height_px` = 20 | QT |

**Ca chưa chốt** khi gán thật: tick `can_review` trên box, hoặc ghi vấn đề cấp ảnh vào
`submission/qc/image_issues.csv` (`image,annotator,region,note`). Trong nhật ký QC ghi mã quy tắc `R-xx`.

---

## 5. QC và phiên bản

### Ngưỡng QC (lấy từ `submission/taxonomy.json`, mục `rules`, không đổi)

| Chỉ số | Ngưỡng |
|---|---|
| Chiều cao box tối thiểu | 20 px |
| Ghép box giữa hai người gán | IoU ≥ 0.5 |
| IoU đạt cho box cao ≥ 50 px | ≥ 0.85 |
| IoU đạt cho box cao < 50 px | ≥ 0.70 |
| Đồng thuận lớp (IAA class) | ≥ 0.95 |
| Tỉ lệ ảnh lỗi tối đa để Release | ≤ 2 % |

### Ai quyết ca mơ hồ
1. Người gán: tick `can_review`, không tự sửa quy tắc giữa chừng.
2. Ca `HỎI` ở Mục 4 và mọi box `can_review` → đưa ra họp calibration (Pha 5); **reviewer nhóm**
   (xem `TEAM.md`) chốt và ghi mã quy tắc vào `calibration_decisions.csv`.
3. Quyết định đổi cách chọn vật / lớp / box / thuộc tính → `semantic`, tăng phiên bản và rà lại cả lô.
   Chỉ sửa câu chữ → `editorial`.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên: 5 lớp + `ignore_region`, 3 quy tắc vẽ | — |
| v0.1-101 | Thêm lớp `bus`, `bicycle`, `three_wheeler`; định nghĩa lại `car`/`van`/`truck` (pickup → `truck`) | Xe buýt img_009/031/037/040, tuk-tuk img_004/016/035/040, xe đạp/xích lô img_006/023/029 không có lớp |
| v0.1-101 | Đặt ngưỡng % cho `occluded_level`, định nghĩa `truncated` | v0 chỉ ghi "che ít / che nhiều", hai người đọc ra hai cách |
| v0.1-101 | Thêm thuộc tính `on_vehicle` (`no`/`yes`) cho `person` | Cần tách người đi bộ với người đi xe mà không thêm lớp `rider` (tránh bất đồng ở ca dắt xe, img_003 / img_010) |
| v0.1-101 | Thêm R-01 … R-16 (người trên xe 2 box riêng, hàng trên xe, xe chở xe, vật không thật, xe chủ, đám đông, mờ/tối, người sau kính) | Các ca ở Mục 4 |

---

## Phụ lục: lớp và thuộc tính đề xuất cho `taxonomy.json` (dùng ở Pha 2 nếu nhóm đồng ý)

```json
"classes": [
  {"name": "person",        "definition": "Người thật nhìn thấy trực tiếp, kể cả người lái/ngồi trên xe hở; không tính người sau kính xe kín."},
  {"name": "car",           "definition": "Ô tô chở người cỡ nhỏ: sedan, hatchback, SUV, MPV, taxi."},
  {"name": "van",           "definition": "Xe thân hộp liền khối, mui cao, khoang sau nối liền cabin, dài dưới ~6 m."},
  {"name": "truck",         "definition": "Xe chở hàng có thùng tách riêng khỏi cabin: pickup, xe tải, đầu kéo + rơ-moóc."},
  {"name": "bus",           "definition": "Xe buýt, xe khách, minibus có từ 2 cửa khách hoặc ≥ 4 ô cửa sổ bên sau ghế lái."},
  {"name": "motorcycle",    "definition": "Xe hai bánh có động cơ, kể cả xe kèm sidecar."},
  {"name": "bicycle",       "definition": "Xe đạp 2 hoặc 3 bánh chạy bằng sức người, kể cả xích lô."},
  {"name": "three_wheeler", "definition": "Xe ba bánh có động cơ: tuk-tuk, xe lam, auto-rickshaw."},
  {"name": "ignore_region", "definition": "Vùng bỏ qua (không phải vật): cụm vật không tách được; bỏ sót ở đây không tính lỗi."}
],
"attributes": [
  {"name": "occluded_level", "type": "select",   "values": ["0", "1", "2", "3"], "default": "0"},
  {"name": "truncated",      "type": "select",   "values": ["no", "yes"],         "default": "no"},
  {"name": "on_vehicle",     "type": "select",   "values": ["no", "yes"],         "default": "no"},
  {"name": "can_review",     "type": "checkbox", "values": [],                    "default": "false"}
]
```

CVAT gắn thuộc tính cho mọi lớp, nên box xe cũng sẽ có `on_vehicle`. Với box xe, cứ để mặc định `no` và
bỏ qua thuộc tính này khi QC.
