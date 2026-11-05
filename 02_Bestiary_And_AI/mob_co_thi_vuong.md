---
id: "mob:mob_co_thi_vuong"
type: "bestiary_entity"
archetype_class: "UNDEAD_OVERLORD"
ai_profile: "[[sys_ai_colossal_boss]]"
vision_cone_deg: 140
hearing_radius_blocks: 40
target_body_parts: ["[[part_chest_upper]]", "[[part_head]]"]
inflicts_wounds: ["[[wnd_blunt_trauma]]", "[[wnd_puncture]]"]
transmits_disease: "[[dis_trung_doc_o_huyet]]"
drops_materials: ["[[mat_dong_den]]", "[[mat_xua_co_thi]]"]
habitat_zones: ["[[zone_lang_mo_tang_1]]", "[[zone_dong_dia_dao_quan_co]]"]
---
# Sinh Vật Dị Chủng: Cổ Thi Vương (Chúa Ngục Lăng Tẩm)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Vị tướng thời cổ mang giáp đồng đen chôn sâu đáy huyệt mộ, hấp thụ tinh hoa âm mạch ngàn năm, thức tỉnh thống lĩnh bầy thi quỷ tàn sát kẻ xâm nhập.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_colossal_boss]]
- **Giác quan cảm biến:** Nón nhìn `140°`, thính giác `40 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Cầm đại đao thi triển các thế võ [[mst_cleave]] và [[mst_graze]]. Đòn chém sượt vẫn bẻ gãy giáp sắt người chơi.
