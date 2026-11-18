# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 251 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Cùng on_vehicle | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 251 | 0.847 | 99.2% | 77.3% | 90.4% | 85.7% | 80/99 | 0 | CHƯA ĐẠT |
| 101|102 | 13 | 74 | 0.838 | 100.0% | 100.0% | 100.0% | 100.0% | 53/4 | 0 | CHƯA ĐẠT |
| 101|103 | 13 | 121 | 0.846 | 99.2% | 66.1% | 86.0% | 81.0% | 26/52 | 0 | CHƯA ĐẠT |
| 102|103 | 14 | 56 | 0.861 | 98.2% | 71.4% | 87.5% | 76.8% | 1/43 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 380 ({'bo_sot_thua': 179, 'iou_thap': 95, 'sai_thuoc_tinh': 94, 'can_review_anh': 5, 'co_can_review': 5, 'sai_lop': 2}), trên 35 ảnh cần calibration.
