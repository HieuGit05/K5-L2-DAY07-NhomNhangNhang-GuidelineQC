# Phân tích IAA — Pha 4 (taxonomy v1.0)

- Ngày: 2026-11-18 · Người chạy: 101 · Người review bản cuối: 103
- Đầu vào: `submission/labels/101.json` (26 ảnh), `102.json` (27), `103.json` (27); `validate_labels.py --all` sạch
- Số liệu gốc: `iaa_summary.md` / `.json`, `calibration_candidates.csv`, `work/viz/` (do script sinh, không sửa tay).
  File này chỉ là phần đọc và phân loại của nhóm.

## 1. Số liệu và mẫu số

| Chỉ số | Giá trị | Mẫu số | Ngưỡng | Kết quả |
|---|---|---|---|---|
| Lượt gán | 80/80 | 40 ảnh × 2 người | 100 % | Đủ |
| IoU trung bình | 0.847 | 251 cặp box ghép 1–1 (IoU ≥ 0.5) | ≥ 0.85 | **Chưa đạt** |
| Cùng lớp | 99.2 % | 251 cặp box ghép | ≥ 95 % | Đạt |
| Cùng `occluded_level` / `truncated` / `on_vehicle` | 77.3 % / 90.4 % / 85.7 % | 251 cặp box ghép | — | — |
| Box chưa ghép | 80 (bên A) / 99 (bên B) | — | — | Không tính vào IoU |
| Ca bất thường | 380 trên 35/40 ảnh | — | — | `bo_sot_thua` 179, `iou_thap` 95, `sai_thuoc_tinh` 94, `can_review_anh` 5, `co_can_review` 5, `sai_lop` 2 |

**Điều IAA không cho biết:** IoU và % cùng lớp chỉ tính trên box đã ghép được. Box mà **cả hai** người cùng bỏ sót
không xuất hiện ở đâu cả, và hai người cùng gán sai giống nhau vẫn được tính là đồng thuận (xem mục 3).

## 2. Phân loại nguyên nhân

| # | Nhóm ca | Dấu hiệu | Loại | Hướng xử lý ở Pha 5 |
|---|---|---|---|---|
| N1 | Thuộc tính để mặc định | 101: `on_vehicle=yes` 0/143 người, `truncated=yes` 2, `occluded_level≠0` 4. 102: 0/63, 0, 0. 103: 59/131, 46, 84 | **Hai người (101, 102) cùng sai**, 103 đúng quy tắc | Sửa thuộc tính theo quy tắc đã có (2.2); `pham_vi=lo` vì ảnh hưởng mọi ảnh của 101 và 102 |
| N2 | 102 vẽ ít box | 102: 135 box / 27 ảnh, so với 101: 274 / 26 và 103: 272 / 27. Cặp 101–102 chưa ghép 53/4, cặp 102–103 chưa ghép 1/43 | **Một người lệch** (chưa nắm R-04, R-10, R-13) | Bổ sung box bị sót trên ảnh của 102 |
| N3 | Không ai dùng `ignore_region` | 0 vùng ở cả 3 file; ca dồn ở ảnh đông: img_006 (33), img_003 (30), img_021 (29), img_032 (26), img_030 (22) | **Cả ba cùng lệch** → R-13 chưa đủ rõ (khi nào "không tách được") | Làm rõ R-13 ở v1.1 (`semantic`, `pham_vi=lo`) |
| N4 | Pickup gán `car` | C160 img_016: 101 `car` / 103 `truck`. C092 img_007: 102 `car` (box chỉ ôm đầu xe) / 103 `truck` (ôm cả thùng) | **Một người lệch** trên quy tắc đã chốt (pickup → `truck`, R-12) | Sửa ảnh (`pham_vi=anh`) |
| N5 | Ca 103 gắn `can_review` | img_004 tài xế tuk-tuk nửa qua kính nửa trực tiếp; img_014 người mờ mép phải; img_016 đầu mờ góc dưới; img_040 hai gương xe máy; img_040 xe xanh mép trái | **Guideline còn hở**: R-05 chưa nói ca "vừa qua kính vừa trực tiếp"; ca HỎI img_040 chưa chốt | Reviewer chốt từng ca, ghi quy tắc vào v1.1 |
| N6 | IoU thấp (95 ca) | IoU TB 0.847, hụt ngưỡng 0.003 | Cần xem `work/viz/` từng ca: lệch mép box hay lệch phạm vi (vd. img_007 có/không ôm thùng) | Sửa theo R-08, R-12 |

## 3. Đối chiếu ca tham chiếu (ảnh coach gợi ý trong `docs/COACH_GUIDE.md`)

Đối chiếu với quyết định đã chốt trong guideline v1.0 (Mục 4). **Cần coach xác nhận** bộ tham chiếu chính thức.

| Ảnh | Người gán | So với v1.0 | Kết luận |
|---|---|---|---|
| img_017 (tượng) | 102, 103 | Cả hai 0 box | **Đúng**, không có lỗi đồng thuận sai |
| img_002 (xe chở xe) | 102, 103 | Cả hai 1 `truck` + 8 `car`. 102 thiếu `person` ở đuôi rơ-moóc; box `truck` của 102 kéo lên ôm cả xe tầng trên (y = 52), trái R-12/R-14; 103 dừng ở khung giàn (y = 125) | Một người lệch (102) |
| img_026 (2 người trên xe máy) | 101, 103 | 103: `on_vehicle=yes`, xe máy `truncated=yes`, `occluded_level=1`. 101: tất cả mặc định | Một người lệch (101) |
| **img_005** (nhiều người trên xe máy) | 101, 102 | **Cả hai cùng bỏ sót** ô tô bạc bên trái (cao ~150 px) và xe van trắng phía sau nó (v1.0 ghi rõ: "xe bạc trái → `car`, `truncated=yes`"). **Cả hai cùng** để `on_vehicle=no` cho 5 người đi xe máy và `truncated=no` cho các box chạm mép dưới | **Giống nhau nhưng cùng sai** |

**Ca "cùng sai" chính: img_005 (101 & 102).**
- IAA không phát hiện: hai chiếc xe bị cả hai bỏ sót không có box nào để ghép, nên không có dòng `bo_sot_thua`;
  thuộc tính cùng mặc định nên được tính là đồng thuận (cặp 101–102 đạt 100 % ở cả 3 thuộc tính).
- Cách phát hiện: mở ảnh gốc, đối chiếu từng vật với bảng ca khó Mục 4 của guideline v1.0.
- Hệ quả: chỉ sửa ca bị cờ là chưa đủ; phải rà **cả lô** ảnh của 101 và 102 cho nhóm N1 và các xe ở mép ảnh.
