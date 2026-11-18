# Guideline gán nhãn — v1.1 — Nhóm NhangNhang

- Phiên bản: **v1.1** (sau calibration Pha 5; dùng cho nhãn cuối và duyệt QC)
- Ngày: 2026-11-18 (v1.0: 2026-11-16)
- Thay đổi so với v1.0: xem "Lịch sử thay đổi" ở Mục 5; căn cứ ở `submission/qc/iaa_analysis.md` và `calibration_decisions.csv`
- Nguồn: gộp từ 3 bản cá nhân trong `submission/guidelines/individual/` (gọi là bản **A**, **B**, **C**, xem Phụ lục A)
- Người viết: 101, 102, 103 · Người duyệt: (người review bản cuối trong `TEAM.md`)
- Loại nhãn: bounding box (hộp chữ nhật song song trục ảnh) · Công cụ: CVAT · Xuất: COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy.json` (version `v1.0`)
- Phạm vi: 40 ảnh `data/images/img_001.jpg` … `img_040.jpg`

> **Cách dùng:** mỗi quy tắc có mã `R-xx`. Gặp ca khó thì tra **Mục 4** trước. Không có ca giống thì áp quy tắc
> gần nhất và tick `can_review`. Khi tranh luận hay ghi nhật ký QC, dẫn mã quy tắc, không dẫn "theo mình nghĩ".
> Câu nào hai người đọc ra hai cách là lỗi của guideline: báo để sửa ở v1.2.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện vật 2D** cho **camera giao thông / hỗ trợ lái xe** trên đường phố
đông xe máy kiểu Đông Nam Á. Mô hình nhận ảnh, trả về box + lớp cho **người tham gia giao thông và phương tiện thật**.

Nhãn phải giúp mô hình:
1. Tìm được **mọi người và mọi xe thật** trong ảnh, kể cả xe đỗ, xe trưng bày, người đứng bên đường.
2. Phân biệt loại xe đủ để ước lượng kích thước và hành vi.
3. **Không** báo vật giả: tượng, đồ chơi, mô hình, ma-nơ-canh, ảnh in, hình phản chiếu.

**Sai ở đâu thì nguy hiểm nhất:**

| Mức | Lỗi | Hậu quả |
|---|---|---|
| **Cao nhất** | Bỏ sót `person`, `motorcycle`, `bicycle` (người đi bộ, người đi xe hai bánh) | Mô hình không thấy → nguy cơ va chạm |
| Cao | Gán vật giả (tượng, poster, phản chiếu, đồ chơi) thành vật thật | Báo động giả, phanh "ma", học sai khái niệm |
| Trung bình | Sai lớp giữa các loại xe (`car` / `van` / `truck` / `bus`) | Ước lượng sai kích thước, tốc độ |
| Thấp | Box lệch vài pixel, sai mức che khuất | Giảm độ chính xác định vị |

→ **Phân vân giữa "vẽ" và "không vẽ" một người/xe thật: vẽ**, rồi tick `can_review`.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp: mỗi box thuộc **đúng một** lớp

8 lớp vật + 1 vùng bỏ qua:

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhìn thấy **trực tiếp** (không qua kính xe): đứng, đi, ngồi, hoặc đang ở trên xe hở. | img_014 (2 người qua đường), img_021 (người đi xe máy), img_032 (người chờ xe buýt), img_019 (tài xế xe mui trần) | Tượng (img_017), ma-nơ-canh và ảnh in (img_033), người trên biển quảng cáo (img_018), linh vật kín (img_019), người ngồi sau kính ô tô/xe buýt (R-05) |
| `car` | Ô tô chở người cỡ nhỏ: sedan, hatchback, SUV, crossover, wagon, MPV, taxi, xe mui trần, xe diễu hành chạy được. | img_015, img_023, img_034, img_002 (xe trên giàn chở), img_024 (xe trưng bày), img_019 (xe diễu hành có biển số) | Pickup → `truck`. Thân hộp cao, cửa trượt → `van`. Mô hình/đồ chơi (img_038, ô tô vàng trên nóc img_024) → không gán |
| `van` | Xe **thân hộp liền một khối** (khoang sau nối liền cabin), mui cao, thường có cửa trượt hông, chở người hoặc hàng, đến ~16 chỗ. | img_005 (xe 51LD bạc), img_016 (xe trắng bên phải), img_011 (xe trắng sau tượng) | SUV/MPV mui thấp → `car`. Có cửa khách kiểu xe buýt → `bus` |
| `truck` | Xe chở hàng có **cabin tách rời thùng hàng**: pickup/bán tải (kể cả có nắp thùng), xe tải nhỏ, xe tải thùng, đầu kéo + rơ-moóc, xe chở ô tô. | img_001, img_012, img_028, img_037 (pickup), img_036 (tải nhỏ có bạt), img_007, img_002 | Thân liền cabin → `van`. Tượng xe bằng đá (img_027) → không gán |
| `bus` | Xe chở khách có **cửa khách kiểu xe buýt** (cửa gập/cửa đôi trên thân, không phải cửa trượt) hoặc rõ ràng > 16 chỗ: xe buýt, xe khách, minibus. | img_009 (minibus trắng), img_031 (xe buýt 82), img_037 (xe buýt xanh trái), img_040 (xe buýt vàng) | Van 16 chỗ cửa trượt (img_016) → `van` |
| `motorcycle` | Xe **hai bánh** có động cơ: xe máy, xe tay ga, mô tô, xe độ; **kể cả xe gắn sidecar**; có người hay không đều gán. | img_006, img_010, img_021, img_030 (bãi đỗ), img_022 (sidecar), img_011 (xe độ) | Xe đạp → `bicycle`. Xe 3 bánh có khoang chở → `three_wheeler` |
| `bicycle` | Xe **hai bánh** chạy bằng sức người (có bàn đạp), kể cả xe đạp chở hàng. | img_006 (xe đạp bên phải), img_029 (xe đạp cạnh xe máy), img_003 (xe đạp bán hàng) | Xích lô (3 bánh) → `three_wheeler`. Xe máy → `motorcycle` |
| `three_wheeler` | Xe **ba bánh có khoang hoặc thùng chở** người/hàng, có động cơ hay đạp chân: tuk-tuk, xe lam, auto-rickshaw, **xích lô**, ba gác. | img_004, img_016 (tuk-tuk), img_035 (rickshaw vàng), img_040 (tuk-tuk vàng), img_023, img_039 (xích lô) | Mô tô + sidecar → `motorcycle` |
| `ignore_region` | **Vùng** (không phải vật) chứa cụm người/xe dày đặc không tách được từng cái; bỏ sót trong vùng này không tính lỗi. | Đám đông sau rào img_019, hậu cảnh xe máy img_021, khối xe đỗ phía sau img_030 | Không dùng để "né" vật tách được (R-13), không dùng để che vật giả (R-01) |

**Vật không thuộc lớp nào** (chó, chim, ngựa, cọc tiêu, cột, xe đẩy tay, đòn gánh, biển báo, đèn giao thông):
**không gán, không vẽ `ignore_region`**. Mô hình cần thấy chúng như nền.
Thấy một loại xe thật xuất hiện nhiều mà không có lớp → ghi vấn đề cấp ảnh để nhóm cân nhắc ở v1.1.

**Cây quyết định cho xe** (chỉ áp cho xe thật, sau khi qua R-01 … R-03):

```
Mấy bánh?
├─ 2 bánh (kể cả gắn sidecar)
│     ├─ có động cơ ─────────────────────────────> motorcycle
│     └─ đạp chân ───────────────────────────────> bicycle
├─ 3 bánh, có khoang/thùng chở (tuk-tuk, xích lô) ─> three_wheeler
└─ ≥ 4 bánh
      ├─ cabin tách thùng hàng (pickup, tải) ─────> truck
      ├─ cửa khách kiểu xe buýt / > 16 chỗ ───────> bus
      ├─ thân hộp cao liền khối, cửa trượt ───────> van
      └─ còn lại ─────────────────────────────────> car
```

### 2.2 Thuộc tính

Mỗi box lớp vật (không áp cho `ignore_region`) có 4 thuộc tính. CVAT hiện cả 4 cho mọi lớp; `on_vehicle` chỉ có nghĩa với `person`.

#### `occluded_level`: mức bị **vật khác** che

Ước lượng % **thân vật** (tưởng tượng hình dạng đầy đủ) bị vật khác che (người, xe, cột, khung giàn, hàng rào…):

| Mức | Ngưỡng bị che | Ví dụ |
|---|---|---|
| `0` | < 10 % | Người áo trắng qua đường img_034 |
| `1` | 10 % đến < 40 % | Người đi xe máy bị xe che chân (~30 %); ô tô sau hàng rào lưới thưa img_018 |
| `2` | 40 % đến < 70 % | Xe máy giữa bãi đỗ img_030 chỉ thấy yên và đầu xe; ô tô tầng dưới giàn img_002 |
| `3` | ≥ 70 %, chỉ còn phần nhỏ nhưng **vẫn chắc chắn nhận ra lớp** | Người hàng sau chỉ lộ đầu + vai img_032 |

- Phần bị **mép ảnh** cắt → tính vào `truncated`, **không** tính vào `occluded_level`.
- Hàng rào / song sắt / lưới **thưa** nhìn xuyên được: tối đa mức `1` nếu không có vật đặc nào khác che.
- Tối, mờ **không phải** che khuất → xử lý theo R-15, không nâng `occluded_level`.
- Che ≥ 70 % **và** không nhận ra lớp → không vẽ box.

#### `truncated`: vật bị **mép ảnh** cắt

| Giá trị | Khi nào |
|---|---|
| `no` | Vật nằm trọn trong khung ảnh. |
| `yes` | Box chạm mép ảnh (cách mép ≤ 2 px) **và** rõ ràng vật còn tiếp ra ngoài khung. |

Ví dụ `yes`: ô tô đỏ và ô tô bạc img_015, người áo đỏ img_016, xe tải img_012, người đội mũ bảo hiểm img_040.

#### `on_vehicle`: người có đang ở trên xe không (chỉ cho `person`)

| Giá trị | Khi nào | Ví dụ |
|---|---|---|
| `no` | Đi bộ, đứng, ngồi trên mặt đất / ghế / vỉa hè; **dắt xe** hoặc **đứng cạnh xe** | img_014, khách quán img_001, người gánh hàng img_003 |
| `yes` | Đang **ngồi hoặc đứng trên** xe: người lái, người ngồi sau, trẻ em ngồi trước, người đạp xích lô, tài xế tuk-tuk, người đứng trong thùng pickup | img_005, img_026, img_023, img_004 |

- Xe đang đỗ nhưng người vẫn ngồi trên yên → `yes` (img_010).
- Ngồi trên yên, chân chống đất → `yes`.
- Mặc định là `no`, **nhớ đổi** khi vẽ người trên xe. Box các lớp xe: để mặc định, QC bỏ qua thuộc tính này.

> **Bổ sung v1.1: kiểm thuộc tính trước khi chuyển ảnh.** Ở Pha 4, hai trên ba người gán để **toàn bộ** thuộc tính
> ở giá trị mặc định (0/143 và 0/63 người đi xe có `on_vehicle=yes`). IAA vẫn báo hai người này "đồng thuận 100 %"
> vì cùng mặc định. Từ v1.1, trước khi bấm sang ảnh khác phải soát 3 điểm:
> (1) mỗi `person` đang ở trên xe → `on_vehicle=yes`; (2) mỗi box chạm mép ảnh → xét `truncated`;
> (3) mỗi box có vật khác đè lên → xét `occluded_level`. Một ảnh mà mọi box đều giữ mặc định là dấu hiệu chưa soát.

#### `can_review`: cờ "tôi chưa chắc"

Tick khi (a) không chắc **lớp**, (b) không chắc **có nên gán** hay không, hoặc (c) ca nằm trong nhóm **HỎI** ở Mục 4.
**Bắt buộc** ghi kèm một dòng vấn đề cấp ảnh (ảnh nào, vùng nào, phân vân giữa gì với gì).
Không tick chỉ vì box khó vẽ; box khó thì vẽ cẩn thận hơn.

---

## 3. Quy tắc

### 3.1 Chọn vật nào để gán

| Mã | Quy tắc |
|---|---|
| **R-01 Chỉ vật thật** | Chỉ gán vật **thật, có mặt vật lý** trong cảnh. Không gán: tượng, tượng đài, tác phẩm điêu khắc, ma-nơ-canh, đồ chơi, mô hình xe (kể cả mô hình cỡ lớn), ảnh in trên poster / biển quảng cáo / tranh tường, hình trên màn hình. Chất liệu không đổi quyết định. Vật giả **không** được che bằng `ignore_region`: chúng là mẫu âm mô hình cần học. |
| **R-02 Phản chiếu** | Không gán hình phản chiếu (gương chiếu hậu, kính cửa hàng, mặt đường ướt, vũng nước). Vật thật đứng **sau** kính cửa hàng, nhìn xuyên qua kính, vẫn gán (trừ người trong xe, R-05). |
| **R-03 Xe chủ (ego)** | Không gán xe mà người chụp đang ngồi/lái: capô, gương, cửa, táp-lô ngay trước ống kính. |
| **R-04 Vật nhỏ** | Box cao **< 20 px** → không gán riêng. Nhiều vật nhỏ dồn cụm → xem R-13. |
| **R-05 Người trong xe** | Người trong **khoang kín** (ô tô, van, xe buýt, cabin tải) hoặc chỉ thấy **qua kính chắn gió** (kể cả kính tuk-tuk nhìn thẳng) → không gán. Người thấy **trực tiếp** trên xe hở (xe máy, xe đạp, xích lô, xe mui trần, thùng pickup, tuk-tuk nhìn từ hông) → gán `person`, `on_vehicle=yes`. **v1.1:** người thấy **một phần qua kính, một phần trực tiếp** (tài xế tuk-tuk: đầu sau kính chắn gió, thân/tay/chân lộ ra bên hông, img_004) → **gán** `person`, `on_vehicle=yes`, box ôm cả phần thấy trực tiếp lẫn phần sau kính. Chỉ khi **toàn bộ** người nằm sau kính thì mới không gán. |
| **R-06 Xe lạ, xe đỗ, xe trưng bày** | Xe thật nhận ra được loại theo cây quyết định → gán lớp đó, dù đang đỗ, trưng bày, độ, hay hình dáng lạ. Không chắc là xe thật hay loại gì → tick `can_review`, **không đoán**. Không tạo lớp mới khi đang gán. |
| **R-07 Hóa trang, linh vật** | Thấy bộ phận người thật (mặt, tay, chân) → `person`. Linh vật kín, thú bông, hình nộm → không gán. |

### 3.2 Cách vẽ box

| Mã | Quy tắc |
|---|---|
| **R-08 Ôm sát phần nhìn thấy** | Box ôm sát **phần nhìn thấy**, không đoán phần bị che. Sai số ~2 px mỗi cạnh. |
| **R-09 Một vật một box** | Vật bị che thành hai mảnh rời (người sau cột) → **một** box bao cả hai mảnh. |
| **R-10 Người trên xe** | **2 loại box riêng:** mỗi người 1 box `person` (gồm mũ bảo hiểm, chân vắt trên xe, cả người ngồi sau, trẻ em ngồi trước), xe 1 box riêng (bánh, thân, tay lái, gương, kính chắn gió). Box xe **không** kéo lên ôm đầu người lái. Hai box chồng nhau là **đúng**. Áp cho xe máy, xe đạp, xích lô, tuk-tuk. |
| **R-11 Đồ của người** | Box `person` gồm quần áo, mũ/nón **đang đội**, ba lô / túi **đeo sát người**. **Không** gồm đồ cầm tay vươn xa, ô, đòn gánh, thúng, đồ vác dài (túi xách xách tay img_014, quang gánh img_003, ống nhựa img_029). |
| **R-12 Đồ của xe** | Box xe gồm phần **cố định**: gương, giá nóc, bạt thùng, khung giàn, sidecar. **Không** gồm hàng hóa chất vượt ra ngoài thân xe (bao tải trên nóc img_035, ống nhựa img_029). Hàng nằm gọn trong thùng thì tự nhiên nằm trong box (img_012). |
| **R-13 Đám đông** | Gán **từng vật** khi (a) cao ≥ 20 px **và** (b) thấy đường viền tách khỏi vật bên cạnh (người: ít nhất đầu + vai; xe: ít nhất một bánh hoặc đầu/đuôi xe). Phần còn lại dồn cụm → **một `ignore_region`** ôm sát cụm. `ignore_region` **không** đè quá ~20 % diện tích một box đã gán riêng. Tối đa ~3 vùng mỗi ảnh. **v1.1:** "không tách được" chỉ áp cho vật mà **không** thấy được đầu + vai (người) hoặc một bánh / đầu / đuôi (xe). Vật cao ≥ 30 px gần như luôn tách được → phải gán riêng, **không** được bỏ qua và cũng không được trùm `ignore_region` lên. Ở Pha 4, 122/179 ca bỏ sót là box cao ≥ 50 px, tức vật rõ ràng bị bỏ sót chứ không phải cụm khó tách. Ảnh không vẽ `ignore_region` thì phải vẽ **đủ** mọi vật ≥ 20 px. |
| **R-14 Rơ-moóc, xe chở xe** | Đầu kéo + rơ-moóc nối liền = **1 box `truck`**. Mỗi xe **thật** được chở trên đó → box riêng theo lớp của nó, `occluded_level` theo phần khung che. Các box chồng nhau là đúng, không cắt box cho khỏi chồng. |
| **R-15 Mờ, tối** | Mờ chuyển động / thiếu sáng: vẫn gán nếu nhận ra lớp; box ôm **lõi** vật, bỏ vệt mờ mỏng. Không xác định được mép trong ±10 px → không gán; nếu là cụm nhiều vật → `ignore_region`. |
| **R-16 Ảnh không có vật** | Không vẽ gì; ảnh **vẫn phải có trong file COCO** export. |

**Thứ tự làm mỗi ảnh:**
(1) lướt ảnh, gạch vật giả / phản chiếu / xe chủ (R-01 … R-03) → (2) vẽ `person` → (3) vẽ xe → (4) vẽ `ignore_region` nếu có cụm →
(5) đặt thuộc tính (nhớ `on_vehicle` cho từng `person`) → **(5b) soát 3 điểm thuộc tính ở mục 2.2 (v1.1)** →
(6) tick `can_review` + ghi vấn đề cấp ảnh nếu cần → (7) **soát lại mép ảnh và nền xa** xem còn xe/người ≥ 20 px chưa gán
(v1.1: ở img_005 cả hai người cùng bỏ sót ô tô bạc và xe van trắng ở mép trái).

---

## 4. Ca khó

Cột **Loại**: `QT` = đã có quy tắc, làm theo, không tick. `HỎI` = làm theo quyết định tạm, **tick `can_review`**, reviewer chốt ở Pha 5.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_002 | Xe tải chở ~9 ô tô trên giàn 2 tầng; 1 người ở đuôi rơ-moóc | R-14, R-12, R-09 | **1 `truck`** từ cản trước đến đuôi rơ-moóc, cao đến đỉnh khung giàn. **Mỗi ô tô 1 `car`**, tầng dưới `occluded_level` 1–2, xe sau bị xe trước che 2–3. Người ≥ 20 px → `person` | Ô tô được chở là xe thật, mô hình cần nhận ra | QT |
| img_024 | Ô tô vàng không kính đặt trên nóc wagon đen trong bảo tàng | R-01, R-06 | Wagon đen và các xe trưng bày khác → `car`. **Ô tô vàng là mô hình (vỏ không kính) → không gán**; box wagon không ôm nó | Mô hình không phải xe thật; xe trưng bày thật vẫn gán | QT |
| img_001, img_012, img_028, img_037 | Xe bán tải (pickup), có/không nắp thùng | Lớp `truck` | → `truck`. img_037: nắp thùng vẫn `truck` | Cabin tách thùng hàng | QT |
| img_036 | Xe tải nhỏ cabin riêng, thùng có bạt | R-12 | `truck`, box gồm cabin + thùng bạt | Cabin tách thùng, dù cỡ nhỏ | QT |
| img_009 | Minibus ~29 chỗ, cửa khách trên thân | Lớp `bus` | `bus`; người bên trong không gán | Cửa khách kiểu xe buýt | QT |
| img_005 | Nhiều người đội mũ bảo hiểm trên xe máy, sát nhau, bị cắt mép dưới | R-10, R-05 | Mỗi người 1 `person` (`on_vehicle=yes`, box chạm mép dưới → `truncated=yes`); xe máy thấy được → `motorcycle`; xe 51LD → `van`, `truncated=yes`; **ô tô bạc mép trái** → `car`, `truncated=yes`, `occluded_level` theo phần xe 51LD che; **xe van trắng phía sau ô tô bạc** → `van`, `truncated=yes` | Người tham gia giao thông là ưu tiên số 1. **v1.1:** ca "cùng sai" ở Pha 4 (cả hai người bỏ sót 2 xe ở mép trái) | QT |
| img_006, img_021 | Dòng xe máy dày, xa dần; xe đạp; ô tô trắng | R-10, R-13, R-04 | Tiền cảnh và trung cảnh: từng `person` + `motorcycle`; xe đạp → `bicycle`; ô tô → `car` (người trong xe không gán). Hậu cảnh chen chúc / < 20 px → `ignore_region`. Xe buýt xa → `bus` nếu ≥ 20 px | Không bỏ sót vật tách được, không ép vẽ hàng trăm box tí hon | QT |
| img_029 | Người đi xe máy vác ống nhựa dài; người đi xe đạp | R-10, R-11, R-12 | 2 `person` (`on_vehicle=yes`), 1 `motorcycle`, 1 `bicycle`; **không** box nào ôm ống nhựa | Đồ vác không phải hình dạng người/xe | QT |
| img_022 | Mô tô gắn sidecar, có người | R-12, R-10 | **1 `motorcycle`** bao cả xe và sidecar; người → `person` | Sidecar là phần cố định của mô tô | QT |
| img_023, img_039 | Xích lô có người đạp | Lớp `three_wheeler`, R-10 | Xích lô → `three_wheeler`; người đạp → `person`, `on_vehicle=yes`; ô tô đen img_023 → `car` | 3 bánh có khoang chở | QT |
| img_004, img_016 | Tuk-tuk, tài xế nhìn từ hông (img_004: đầu sau kính chắn gió, thân/chân lộ bên hông) | R-05 (v1.1), R-10 | Tuk-tuk → `three_wheeler`; tài xế thấy trực tiếp ít nhất một phần → `person`, `on_vehicle=yes` | Chốt ở Pha 5 (ca C048) | QT |
| img_016 | Xe bán tải phía sau tuk-tuk bên trái (thấy thùng sau, chữ MIGHTY-X) | Lớp `truck`, R-04 | `truck`, `occluded_level` theo phần tuk-tuk che; không gán `car` | Chốt ở Pha 5 (ca C160) | QT |
| img_007 | Xe bán tải trắng có khung/thùng cao phía sau trong nhà kho | Lớp `truck`, R-12 | `truck`; box ôm cả cabin và khung thùng phía sau | Chốt ở Pha 5 (ca C092) | QT |
| img_035 | Auto-rickshaw chất bao tải cao trên nóc; xe máy; ô tô | R-12, R-05, R-10 | `three_wheeler`, box chỉ đến mui xe, **không** bao chồng bao tải. Tài xế sau kính chắn gió → không gán. Người đi xe máy → `person` + `motorcycle` | Hàng hóa đổi theo chuyến, không phải hình dạng xe | QT |
| img_003 | Người bán hàng rong gánh hàng, đội nón lá; xe đạp chở hàng | R-11 | `person` gồm nón lá, **không** gồm quang gánh, `on_vehicle=no`; xe đạp → `bicycle`; người ngồi trên xe máy → `on_vehicle=yes` | Nón đang đội là một phần người | QT |
| img_010 | Người ngồi trên xe máy đỗ; người bán rau; xe chở rau | R-10, R-06 | Người trên xe máy đỗ → `person`, `on_vehicle=yes`; người bán rau đứng → `on_vehicle=no`; xe máy → `motorcycle`. Xe chở rau: thấy 2 bánh + bàn đạp/khung xe đạp → `bicycle`; không thấy → không gán (xe đẩy) | Xe đỗ vẫn gán; xe đẩy không thuộc lớp | QT |
| img_011 | Tượng Predator khổng lồ; dãy mô tô kim loại độ đang trưng bày; xe trắng; người và đám đông xa | R-01, R-06, R-13 | Tượng → không gán. Xe độ đủ bánh, khung, tay lái → `motorcycle`. Xe trắng sau tượng → `van` (`occluded_level` theo phần tượng che). 2 người bên trái → `person`. Đám đông xa dưới lều → 1–2 `ignore_region` | Xe độ vẫn là phương tiện thật; tượng là vật giả | QT |
| img_019 | Xe diễu hành vàng có tài xế, linh vật chuột; đám đông sau rào; đầu người mờ góc dưới phải | R-06, R-07, R-05, R-13 | Xe (có bánh, biển số) → `car`. Tài xế mui trần → `person`, `on_vehicle=yes`. Linh vật chuột → không gán. Khán giả hàng đầu tách được → `person`; phía sau → `ignore_region`. Đầu mờ góc dưới → `person`, `truncated=yes` | Xe thật dù hình dáng lạ; linh vật là đồ vật | QT |
| img_017 | Tượng người cưỡi ngựa đội cọc tiêu | R-01 | **Không gán gì** (0 box hợp lệ) | Tượng | QT |
| img_027 | Tượng đài xe bán tải bằng đá ở nghĩa trang | R-01 | **Không gán gì** | Không phải xe thật | QT |
| img_028 | Tượng ngựa trong thùng pickup thật; ô tô đỏ phía sau | R-01, R-12 | Pickup → `truck`; ngựa → không gán; ô tô đỏ → `car` | Ngựa là tượng và không thuộc lớp | QT |
| img_033 | Ảnh in người mẫu, ma-nơ-canh, phản chiếu xe trắng + gương trên kính | R-01, R-02 | **Không gán gì** | Ba loại vật giả trong một ảnh | QT |
| img_038 | Tủ kính ô tô đồ chơi, vịt cao su, búp bê | R-01 | **Không gán gì** | Đồ chơi là mẫu âm | QT |
| img_018 | Người trên biển quảng cáo; ô tô thật sau hàng rào lưới | R-01, R-04 | Người trên biển → không gán. Ô tô ≥ 20 px → `car`, `occluded_level=1` (lưới thưa) hoặc cao hơn nếu xe trước che | Biển quảng cáo là ảnh in | QT |
| img_020 | Gương chiếu hậu xe người chụp, trong gương có ô tô | R-02, R-03 | **Không gán gì** | Phản chiếu sai vị trí không gian; gương là xe chủ | QT |
| img_030 | Bãi đỗ xe máy rất dày; người đứng phải | R-13, R-06 | Xe hàng trước tách được → từng `motorcycle`; khối phía sau → `ignore_region`; người → `person` | Đỡ hàng chục box mơ hồ | QT |
| img_032 | ~30 người đứng chờ xe buýt; ô tô xanh trái | R-13 | Người cao ~60–100 px, tách được → **gán từng người**, người hàng sau `occluded_level` 2–3. Chỉ dùng `ignore_region` cho cụm thực sự không tách được. Ô tô xanh → `car`, `truncated=yes` | Người đi bộ là lớp quan trọng nhất | QT |
| img_014 | Người đi xe máy mờ chuyển động sau 2 người qua đường | R-15, R-10, R-11 | 2 người rõ → `person` (box không gồm túi xách cầm tay); xe máy mờ nhận ra lõi → `person` + `motorcycle`; vệt mờ không rõ mép → không gán | Ổn định giữa người gán | QT |
| img_007, img_008, img_013, img_031 | Ban đêm, đèn, mặt đường ướt phản chiếu | R-15, R-02 | Gán vật nhìn rõ; phản chiếu trên mặt đường → không gán; img_031 xe buýt → `bus` | Phản chiếu là vật giả | QT |
| img_025, img_039 | Người đi bộ, xe máy nhỏ ở xa | R-04, R-13 | Gán khi cao ≥ 20 px; cụm nhỏ hơn → `ignore_region` tùy chọn | Ngưỡng `min_box_height_px` = 20 | QT |
| img_040 | Người đội mũ bảo hiểm sát ống kính; 2 gương xe máy ở mép dưới; taxi, tuk-tuk, xe buýt; xe ở mép trái | R-03, R-10, R-05, R-08 | Người → `person`, `truncated=yes`, `on_vehicle=yes`. **2 gương là xe của người đó** (không phải xe chủ) → 1 `motorcycle` ôm phần thấy được, `truncated=yes`, `occluded_level=3`. Taxi → `car`; tuk-tuk vàng → **1** `three_wheeler` ôm cả mui; xe buýt vàng phía sau → `bus`; **xe mép trái là đầu xe buýt** → `bus`, `truncated=yes`; người trong tuk-tuk sau kính → không gán | Chốt ở Pha 5 (ca C368–C376) | QT |
| (chung) | Xe 16–24 chỗ không rõ cửa khách kiểu gì | Lớp `van` / `bus` | Tạm gán `van` | Ranh giới van/bus chưa có ảnh mẫu | **HỎI** |

**Ca 0 box theo quy tắc** (không cần hỏi): img_017, img_020, img_027, img_033, img_038.

**Ca chưa chốt khi gán thật:** tick `can_review` trên box **và** ghi một dòng vào `submission/qc/image_issues.csv`
(`image,annotator,region,note`) sau khi khóa nhãn. Trong nhật ký QC luôn ghi mã `R-xx`.

---

## 5. QC và phiên bản

### Ngưỡng QC (từ `submission/taxonomy.json`, mục `rules`, không đổi)

| Chỉ số | Ngưỡng |
|---|---|
| Chiều cao box tối thiểu | 20 px |
| Ghép box giữa 2 người gán | IoU ≥ 0.5 |
| IoU đạt, box cao ≥ 50 px | ≥ 0.85 |
| IoU đạt, box cao < 50 px | ≥ 0.70 |
| Đồng thuận lớp trên các cặp ghép được | ≥ 0.95 |
| Tỉ lệ ảnh lỗi để Release | ≤ 2 % |

### Loại lỗi khi review (ghi kèm mã quy tắc)

| Loại lỗi | Ví dụ |
|---|---|
| Bỏ sót | Thiếu người ngồi sau xe máy (R-10) |
| Gán thừa | Gán tượng / ma-nơ-canh / phản chiếu / mô hình (R-01, R-02) |
| Sai lớp | Pickup gán `car` thay vì `truck`; xích lô gán `bicycle` thay vì `three_wheeler` |
| Sai box | Box tuk-tuk bao cả bao tải trên nóc (R-12); box xe máy kéo lên ôm đầu người (R-10) |
| Sai thuộc tính | Bị mép ảnh cắt mà `truncated=no`; người trên xe mà `on_vehicle=no` |

### Ai quyết ca mơ hồ

1. Người gán: tick `can_review` + ghi vấn đề cấp ảnh. **Không** tự sửa quy tắc giữa lượt gán.
2. **Người review bản cuối** (`TEAM.md`) quyết định ở Pha 5, ghi mã quy tắc vào `calibration_decisions.csv`.
3. Quyết định làm đổi cách chọn vật / lớp / box / thuộc tính → `semantic`: tăng phiên bản, ghi bảng dưới, kiểm lại **cả lô**
   các ảnh cùng tình huống. Chỉ sửa câu chữ → `editorial`.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên: 5 lớp + `ignore_region`, 3 quy tắc vẽ | — |
| v1.0 | Gộp 3 bản cá nhân. 8 lớp vật (thêm `bus`, `bicycle`, `three_wheeler`; pickup → `truck`; xích lô → `three_wheeler`). Ngưỡng % cho `occluded_level`, định nghĩa `truncated`, thêm `on_vehicle`. Quy tắc R-01 … R-16, bảng ca khó 30 dòng | Xem Phụ lục A |
| v1.1 (`semantic`) | **R-05:** người thấy một phần qua kính, một phần trực tiếp → gán (đổi nghĩa, rà cả lô). **R-13:** định nghĩa rõ "không tách được"; vật ≥ 30 px phải gán riêng. **2.2:** thêm bước soát 3 điểm thuộc tính; **thứ tự làm ảnh** thêm bước soát mép ảnh và nền xa. **Mục 4:** img_005 bổ sung 2 xe mép trái; đóng ca HỎI img_040 (gương thuộc xe người đội mũ; xe mép trái là `bus`); thêm img_007, img_016 (pickup → `truck`). Taxonomy **không đổi** (vẫn `v1.0`) | Pha 4: IoU TB 0.847 < 0.85 trên 251 cặp; 380 ca / 35 ảnh; ca C048 (R-05), C092 / C160 (pickup), C368–C376 (img_040), ca cùng sai img_005; xem `submission/qc/iaa_analysis.md` |

---

## Phụ lục A: các điểm khác nhau giữa 3 bản và quyết định

Nguồn: **A** = `guideline_101.md`, **B** = `guideline_103.md`, **C** = `guideline_102.md`.

| # | Điểm khác | A | B | C | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|---|
| 1 | Bộ lớp | 8 lớp + ignore | 7 lớp + ignore (không có xe đạp) | 19 lớp, gồm cả vùng nền (`sky`, `road`, `building`…) | **8 lớp + ignore** | Bài là bbox phát hiện vật; vùng nền cần phân đoạn (segmentation), không vẽ box được |
| 2 | Xe đạp | `bicycle` | Không gán | `bicycle` | **Có `bicycle`** | 2/3 bản; người đi xe đạp là người tham gia giao thông dễ tổn thương |
| 3 | Xích lô | `bicycle` | `three_wheeler` | — | **`three_wheeler`** | Mô hình học theo hình dạng: 3 bánh + khoang chở giống tuk-tuk hơn xe đạp; người gán không cần đoán có động cơ hay không |
| 4 | Tên lớp | `three_wheeler`, `bicycle` | `three_wheeler` | `tuktuk`, `bycycle` (sai chính tả) | **`three_wheeler`, `bicycle`** | Tên rộng hơn (gồm xe lam, xích lô), đúng chính tả |
| 5 | Ranh giới `van` / `bus` | Đếm ô cửa sổ | Cửa khách kiểu xe buýt, > 16 chỗ | — | **Theo cửa khách** (B) | Dễ nhìn hơn đếm cửa sổ; xe Hiace cũng có ~4 ô cửa sổ |
| 6 | `occluded_level` mức 2/3 | 40–70 / > 70 % | 40–80 / > 80 % hoặc quá tối | Như v0 (không có ngưỡng) | **40–<70 / ≥ 70 %**, tối/mờ **không** tính là che | Tách bạch che khuất với chất lượng ảnh (R-15) |
| 7 | `ignore_region` đè box đã gán | ≤ 50 % | ≤ ~20 % | — | **≤ ~20 %** | Script bỏ qua box bị phủ ≥ 50 %; giữ 20 % cho an toàn |
| 8 | Thuộc tính `on_vehicle` | Có | Không | Không | **Có** | Tách người đi bộ với người đi xe mà không thêm lớp `rider` |
| 9 | Túi đeo của người | Không gồm túi xách | Gồm túi đeo | — | **Gồm túi đeo sát người, không gồm túi xách tay** | Đồ đeo sát đi theo hình dạng người, đồ cầm vươn xa thì không |
| 10 | Ô tô vàng trên nóc img_024 | Xe thật → `car` | (không nêu) | Mô hình → không gán | **Mô hình, không gán** | Kiểm lại ảnh: vỏ xe không có kính, cỡ nhỏ hơn xe thật |
| 11 | Xe độ img_011 | `motorcycle`, HỎI | `motorcycle` | Gán nếu nhận ra, không thì `can_review` | **`motorcycle`, không cần hỏi** | Đủ bánh, khung, tay lái → R-06 |
| 12 | Xe trắng img_011 | `car` | `van` | — | **`van`** | Thân hộp cao, liền khối |
| 13 | Linh vật chuột img_019 | Không gán, HỎI | Không gán | `can_review` | **Không gán, không cần hỏi** | Không thấy bộ phận người thật (R-07) |
| 14 | Người đi xe máy | 2 box | 2 box | Chờ nhóm | **2 box riêng** | Thống nhất A + B |
