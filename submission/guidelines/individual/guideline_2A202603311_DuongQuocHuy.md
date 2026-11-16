# Guideline gán nhãn — v1.0 (bản nháp) — Nhóm Nhang Nhang

- Phiên bản: v1.0-draft (chưa khóa — cần cả nhóm duyệt ở Pha 2)
- Ngày: 2026-11-16
- Người viết / duyệt: … / …
- Loại nhãn: bounding box trên ảnh 2D · Công cụ: CVAT · Xuất: COCO 1.0
- Phạm vi: 40 ảnh trong `data/images/`

> Cách đọc: mỗi quy tắc có mã `R-xx`. Khi ghi nhật ký QC hay tranh luận, dẫn mã quy tắc,
> không dẫn "theo mình nghĩ". Câu nào hai người đọc ra hai cách là lỗi của guideline — báo để sửa.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện giao thông thật** trên ảnh đường phố
(camera giao thông, hỗ trợ lái xe), đặc biệt là đường phố Đông Nam Á với nhiều xe máy, xe ba bánh.

**Sai ở đâu thì nguy hiểm nhất:**

| Loại sai | Hậu quả | Ưu tiên |
|---|---|---|
| Bỏ sót người / xe thật (nhất là người đi xe máy, người đi bộ) | Mô hình không thấy → nguy cơ va chạm | **Cao nhất** |
| Gán cho vật không thật (tượng, ma-nơ-canh, đồ chơi, ảnh in, phản chiếu) | Mô hình báo động giả, học sai khái niệm "người" | Cao |
| Sai lớp giữa các loại xe (car / van / truck…) | Ảnh hưởng ước lượng kích thước, tốc độ | Trung bình |
| Box lệch vài pixel, sai thuộc tính che khuất | Giảm độ chính xác định vị | Thấp hơn |

---

## 2. Lớp và thuộc tính

### 2.1. Lớp — mỗi vật thuộc **đúng một** lớp

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật, đang đứng, đi, ngồi, hoặc ngồi trên xe hở (xe máy, xe đạp, xích lô). | Người đi xe máy img_021, người đứng chờ xe img_032, tài xế xe diễu hành img_019 | Tượng (img_017), ma-nơ-canh và ảnh in (img_033), người hóa trang thành tượng/linh vật lớn (img_011, img_019) → **không** gán |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, xe tay ga, mô tô), **kể cả xe gắn thùng bên (sidecar)** và xe độ/trang trí. | img_021, img_030, img_022 (sidecar), img_011 (xe độ) | Xe đạp → không gán (R-14). Xe ba bánh có khoang chở khách/hàng → `three_wheeler` |
| `three_wheeler` | Xe ba bánh có **khoang chở khách hoặc hàng** gắn liền: tuk-tuk, xe lam, auto-rickshaw, xích lô, xe lôi. | img_004, img_035 (tuk-tuk), img_023, img_039 (xích lô) | Mô tô có sidecar → `motorcycle`. Xe máy kéo rơ-moóc rời → `motorcycle` (rơ-moóc không gán) |
| `car` | Ô tô chở người cỡ nhỏ: sedan, hatchback, SUV, crossover, wagon, xe mui trần, xe diễu hành chạy được. | img_002 (xe trên giá chở), img_024 (wagon), img_019 (xe diễu hành có biển số) | Pickup → `truck`. Minivan thùng liền 7–16 chỗ → `van`. Xe đồ chơi/mô hình → không gán (img_038) |
| `van` | Xe có **thân hộp liền một khối** (không tách cabin và thùng), chở người hoặc hàng, cỡ đến ~16 chỗ. | Xe van trắng phía sau img_011, Ford Transit, Toyota Hiace | Minibus có cửa khách kiểu xe buýt (County, ~29 chỗ) → `bus`. SUV → `car` |
| `truck` | Xe có **cabin tách rời với thùng/khoang hàng**: xe tải, pickup, xe tải nhỏ, xe đầu kéo, xe chở xe. | img_002 (xe chở xe), img_012/img_028/img_037 (pickup), img_036 (xe tải nhỏ có bạt) | Van thân liền → `van` |
| `bus` | Xe chở khách cỡ lớn có cửa khách kiểu xe buýt: xe buýt, minibus ~25–30 chỗ, xe khách. | img_009 (minibus trắng) | Xe 16 chỗ thân van → `van` |
| `ignore_region` | **Vùng** (không phải vật) chứa đám đông/xe dày đặc không thể tách từng cái; bỏ sót trong vùng này không tính là lỗi. | Đám đông xa img_021, img_019, img_011 | Không dùng để "né" vật khó nhưng tách được (R-12) |

**Vật không thuộc lớp nào** (xe đạp, ngựa, chó, xe đẩy hàng rong, cọc tiêu, đồ chơi…): **không gán, không vẽ ignore**.
Nếu thấy một loại xe thật xuất hiện nhiều mà không có lớp → ghi vấn đề cấp ảnh để nhóm cân nhắc thêm lớp ở v1.1.

**Cây quyết định nhanh cho xe:**

```
Có động cơ / dùng chở người-hàng trên đường?  ── không ──> không gán (xe đạp, xe đẩy)
        │ có
   Mấy bánh? ── 2 bánh (kể cả có sidecar) ──> motorcycle
        │ 3 bánh, có khoang chở ──> three_wheeler   (xích lô dù đạp chân cũng vào đây)
        │ ≥ 4 bánh
   Cabin tách thùng hàng? ── có ──> truck
        │ không
   Cửa khách kiểu xe buýt, dài, nhiều hàng ghế (> 16 chỗ)? ── có ──> bus
        │ không
   Thân hộp cao, liền khối (van, minivan chở hàng)? ── có ──> van
        │ không
        └──> car
```

### 2.2. Thuộc tính

| Thuộc tính | Giá trị | Ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0` | Thấy rõ, bị che **< 10%** diện tích ước lượng của vật |
| | `1` | Bị che **10% – < 40%** |
| | `2` | Bị che **40% – 80%** |
| | `3` | Bị che **> 80%** hoặc quá tối/mờ không ước lượng được (vẫn chắc chắn là vật thật) |
| `truncated` | `yes` | Vật **bị mép ảnh cắt** (box chạm mép ảnh và vật còn tiếp ra ngoài khung) |
| | `no` | Vật nằm trọn trong khung ảnh |
| `can_review` | tick | Không chắc lớp **hoặc** không chắc có nên gán. Bắt buộc ghi kèm lý do trong vấn đề cấp ảnh |

- Che bởi **mép ảnh** tính vào `truncated`, **không** tính vào `occluded_level`.
- Che bởi **vật khác** (người, xe, cột, khung giá…) tính vào `occluded_level`.
- Ước lượng theo hình dạng đầy đủ mà vật lẽ ra có (ví dụ người chỉ thấy nửa trên vì xe máy che chân → khoảng 30–40%).

---

## 3. Quy tắc vẽ box

### Chọn vật nào để gán

| Mã | Quy tắc |
|---|---|
| **R-01** | **Chỉ gán vật thật, có mặt vật lý trong cảnh.** Không gán: tượng, ma-nơ-canh, người hóa trang thành tượng/linh vật kín mặt, đồ chơi, mô hình xe, ảnh in trên poster/billboard/bảng quảng cáo, hình trên màn hình. |
| **R-02** | **Không gán ảnh phản chiếu** (trong gương chiếu hậu, kính cửa hàng, vũng nước). Vật nhìn **xuyên qua** kính (đứng thật phía sau kính) thì vẫn gán. |
| **R-03** | **Không gán xe chủ (ego vehicle)** — xe mà người chụp đang ngồi trong: nắp capo, gương, cửa, bảng táp-lô. |
| **R-04** | **Vật quá nhỏ:** box có chiều cao **< 20 px** thì không gán riêng. Nếu nhiều vật nhỏ dồn cụm → xem R-12. |
| **R-05** | **Người trong khoang kín** (ô tô, van, bus, cabin tải, sau kính chắn gió tuk-tuk) → **không gán**. Người ngồi trên xe **hở** (xe máy, xích lô, xe mui trần, thùng pickup) → **gán** `person`. |

### Cách vẽ box

| Mã | Quy tắc |
|---|---|
| **R-06** | Box **ôm sát phần nhìn thấy** của vật (không đoán phần bị che). Sai số cho phép ~2 px mỗi cạnh. |
| **R-07** | **Một vật = một box**, kể cả khi bị che thành hai mảnh rời: một box bao cả hai mảnh nhìn thấy. |
| **R-08** | **Người đi xe máy / xích lô:** vẽ **2 box riêng** — 1 `person` (bao cả người, gồm phần chân vắt trên xe, mũ bảo hiểm) và 1 `motorcycle`/`three_wheeler` (bao cả xe). Hai box được phép chồng nhau. Có 2–3 người trên một xe → mỗi người một box `person`, xe vẫn một box. |
| **R-09** | **Người:** bao cả đồ mặc/đội/đeo sát người (mũ, ba lô, túi đeo). **Không** bao đồ cầm/gánh vươn xa (ô, đòn gánh, thúng, hàng hóa). |
| **R-10** | **Xe:** bao cả phần cố định của xe (gương, giá nóc, bạt thùng). **Không** bao hàng hóa chất vượt ra ngoài thân xe (bao tải chất trên nóc tuk-tuk img_035, hàng trên xe máy img_029). Sidecar là phần của xe máy → nằm trong box `motorcycle`. |
| **R-11** | **Xe kéo rơ-moóc / sơ-mi rơ-moóc nối liền:** một box `truck` bao cả đầu kéo và rơ-moóc khi chúng nối với nhau. Xe chở trên rơ-moóc là vật riêng (R-13). |
| **R-12** | **Đám đông / xe dày đặc:** gán **từng vật** khi (a) cao ≥ 20 px **và** (b) thấy được đường viền để tách khỏi vật bên cạnh (thấy ít nhất đầu + vai với người, ít nhất một bánh hoặc đuôi/đầu xe với xe). Phần còn lại dồn cụm thì vẽ **một `ignore_region`** bao cụm đó. `ignore_region` **không được** đè lên vật đã gán riêng quá ~20% box của vật đó. |
| **R-13** | **Xe chở xe / xe trên xe:** mỗi xe được chở là vật thật → gán riêng theo lớp của nó, `occluded_level` theo phần bị khung giá che. Xe tải chở chúng vẫn là một box `truck` riêng. |
| **R-14** | **Xe đạp không có lớp** → không gán xe đạp; người đạp xe vẫn gán `person`. |

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe chở ô tô: 1 xe tải (đầu xe + rơ-moóc) chở ~9 ô tô xếp 2 tầng chồng lên nhau, 1 người đứng cạnh đuôi rơ-moóc | R-11, R-13, R-07, R-06 | **1 box `truck`** bao từ cản trước đến đuôi rơ-moóc. **Mỗi ô tô 1 box `car`** riêng, ôm sát phần thấy được; xe tầng dưới bị khung cam che → `occluded_level` 1–2; xe phía sau bị xe trước che → 2–3. Các box xe chồng nhau là **đúng**, không cắt box cho khỏi chồng. 1 box `person` ở đuôi (nếu ≥ 20 px). | Ô tô được chở vẫn là ô tô thật, mô hình cần nhận ra. Box chồng nhau phản ánh đúng cảnh. |
| img_011 | Tượng Predator khổng lồ bên phải; hàng xe máy độ trang trí kim loại; xe van trắng; đám đông ở xa | R-01, R-12, R-08 | Tượng → **không gán**. Xe độ có đủ hình dạng xe máy (bánh, khung, tay lái) → **`motorcycle`**, kể cả đang trưng bày; xe độ 3 bánh không có khoang → `motorcycle`. Xe van trắng → `van` (`occluded_level` theo phần tượng che, `truncated=no`). 2 người đứng bên trái (cầm điện thoại) → `person`. Đám đông xa dưới lều, cao < 20 px hoặc không tách được → **1–2 `ignore_region`**. | Tượng không phải người thật. Xe độ vẫn là phương tiện thật. |
| img_017 | Tượng người cưỡi ngựa (đội cọc tiêu) | R-01 | **Không gán gì.** Ảnh có thể có 0 box — hợp lệ. | Tượng đồng, không phải người; ngựa không thuộc lớp nào. |
| img_020 | Chụp gương chiếu hậu: trong gương có 1 ô tô đang chạy phía sau và thân xe mình | R-02, R-03 | **Không gán gì.** Ô tô trong gương là ảnh phản chiếu; gương + cửa xe là xe chủ. | Mô hình phát hiện vật thật trong khung hình; vật phản chiếu sai vị trí không gian. |
| img_021 | Ngã tư Sài Gòn: hàng trăm người đi xe máy, ô tô, xe buýt xa | R-08, R-12, R-05, R-04 | **Tiền cảnh và trung cảnh**: mỗi người 1 `person`, mỗi xe máy 1 `motorcycle` (R-08), xe chỉ thấy đuôi/biển số vẫn gán nếu tách được. Ô tô trắng bên trái → `car` (người trong xe không gán). **Hậu cảnh** từ khoảng giữa ảnh trở lên, đầu người chen nhau không tách được → **`ignore_region`** bao cụm. Xe buýt xa bên phải: gán `bus` nếu ≥ 20 px và tách được. | Cân bằng: không bỏ sót vật tách được, không bắt người gán ép hàng trăm box dưới 20 px. |
| img_032 | Đám đông đứng chờ xe buýt (≈ 30 người, đứng thành hàng) | R-12, R-06 | Người đứng hàng đầu thấy rõ → **gán từng người**. Người phía sau chỉ lộ đầu + vai → vẫn gán, `occluded_level` 2–3. Chỉ dùng `ignore_region` nếu có cụm thực sự không tách được. | Người cao ~60–100 px, đủ lớn và tách được. |
| img_033 | Kính cửa hàng: ảnh in người mẫu, ma-nơ-canh, phản chiếu xe van trắng và gương xe | R-01, R-02 | **Không gán gì.** Ảnh in và ma-nơ-canh không phải người; xe van là ảnh phản chiếu trên kính. | Ba loại "vật giả" trong cùng một ảnh — dễ bị gán nhầm thành person/van. |
| img_038 | Tủ kính trưng bày ô tô đồ chơi, vịt cao su, búp bê | R-01 | **Không gán gì.** | Mô hình xe đồ chơi không phải phương tiện thật dù hình dạng giống hệt. |
| img_019 | Xe diễu hành hình hoạt hình, tài xế ngồi trong, linh vật chuột lớn, đám đông sau rào | R-01, R-05, R-12 | Xe có bánh, biển số, chạy được → **`car`**. Tài xế ở xe mui trần → `person`. Linh vật chuột → không gán. Người hàng đầu sau rào thấy rõ → gán từng người; phía sau dồn cụm → `ignore_region`. | Xe thật dù hình dáng lạ; linh vật là đồ vật. |
| img_022 | Mô tô gắn sidecar, có người ngồi | R-10, R-08 | **1 box `motorcycle`** bao cả xe và sidecar; người → `person`. | Sidecar là phần của mô tô, không phải xe 3 bánh có khoang chở kiểu tuk-tuk. |
| img_035 | Tuk-tuk chở bao tải chất cao trên nóc; xe máy; ô tô | R-10, R-05 | Tuk-tuk → **`three_wheeler`**, box chỉ bao thân xe, **không** bao chồng bao tải. Tài xế sau kính chắn gió → không gán. Người đi xe máy → R-08. | Hàng hóa không phải hình dạng của xe; giữ box xe nhất quán giữa ảnh có/không chở hàng. |
| img_036 | Xe tải nhỏ cabin riêng, thùng có bạt | R-10 | **`truck`**, box bao cả cabin và thùng bạt. | Cabin tách thùng → truck, dù cỡ nhỏ. |
| img_009 | Minibus ~29 chỗ, cửa khách giữa thân | R-05 | **`bus`**. Người bên trong không gán. | Cửa khách kiểu xe buýt, thân dài hơn van. |

**Ca loại theo quy tắc** (không cần hỏi): img_017, img_020, img_033, img_038 → 0 box theo R-01/R-02/R-03.
**Ca cần hỏi reviewer**: ranh giới `van`/`bus` với xe 16–24 chỗ; cụm đám đông khi không chắc tách được → tick `can_review`.

---

## 5. QC và phiên bản

### Ngưỡng QC (lấy từ `submission/taxonomy.json`, không thay đổi)

| Chỉ số | Ngưỡng |
|---|---|
| Ghép box giữa 2 người gán | IoU ≥ 0.5 |
| IoU trung bình — vật lớn (cạnh ≥ 50 px) | ≥ 0.85 |
| IoU trung bình — vật nhỏ (< 50 px) | ≥ 0.70 |
| Đồng thuận lớp trên các cặp ghép được | ≥ 0.95 |
| Tỉ lệ lỗi theo ảnh để Release | ≤ 2% |
| Chiều cao box tối thiểu | 20 px |

### Loại lỗi khi review (ghi kèm mã quy tắc)

| Loại lỗi | Ví dụ |
|---|---|
| Bỏ sót | Thiếu người ngồi sau xe máy (R-08) |
| Gán thừa | Gán tượng/ma-nơ-canh/phản chiếu (R-01, R-02) |
| Sai lớp | Pickup gán `car` thay vì `truck` |
| Sai box | Box bao cả hàng trên nóc tuk-tuk (R-10) |
| Sai thuộc tính | Bị mép ảnh cắt nhưng `truncated=no` |

### Ai quyết ca mơ hồ

1. Người gán tick `can_review` + ghi vấn đề cấp ảnh (ảnh nào, phân vân giữa gì với gì).
2. **Người review bản cuối** (ghi trong `TEAM.md`) quyết định theo guideline hiện hành.
3. Nếu quyết định **làm đổi nhãn** → tăng phiên bản guideline, ghi vào bảng dưới, kiểm lại cả lô với các ảnh cùng tình huống.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0-draft | Thêm lớp `three_wheeler`, `bus`; định nghĩa lại `truck` (gồm pickup), `van`; ngưỡng `occluded_level` theo %; quy tắc R-01 → R-14; bảng ca khó | img_002, 004, 009, 011, 017, 019, 020, 021, 022, 032, 033, 035, 036, 038 |
