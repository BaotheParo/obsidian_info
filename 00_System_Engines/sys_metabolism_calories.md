---
id: "sys:metabolism_calories"
type: "core_engine_system"
standard_weight_kg: 80.0
calorie_burn_rate_base_hour: 75.0
hydration_burn_rate_base_hour: 120.0
tracked_parts: ["[[part_head]]", "[[part_neck]]", "[[part_chest_upper]]"]
associated_systems: ["[[sys_thermoregulation]]", "[[sys_panic_stress]]"]
---
# Cơ Chế Chuyển Hóa & Năng Lượng Sinh Học (Metabolism & Calories)

## 1. Chỉ Số Cơ Bản
- **Cân nặng tiêu chuẩn:** `80.0 kg`
- **Tốc độ đốt cháy Calo lúc nghỉ ngơi:** `75 kcal/giờ`
- **Tốc độ tiêu hao Nước lúc nghỉ ngơi:** `120 ml/giờ`

## 2. Hao Hụt Khi Lao Động & Chiến Đấu
Vác nặng, đẩy xe thồ [[crv_xe_tho_boc_sat_15_parts]], chặt gỗ rừng thiêng hoặc chiến đấu kéo dài đẩy mức tiêu hao calo lên tới `600 - 900 kcal/giờ`. Khi đói kiệt, nhân vật bị suy nhược cơ bắp, giảm tốc độ vung kiếm và không thể thi triển các đặc tính võ học như [[mst_cleave]] hay [[mst_topple]].
