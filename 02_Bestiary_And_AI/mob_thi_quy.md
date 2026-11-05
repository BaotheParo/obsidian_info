---
id: "mob:mob_thi_quy"
type: "bestiary_entity"
archetype_class: "UNDEAD_HORDE"
ai_profile: "[[sys_ai_aggressive_pack]]"
vision_cone_deg: 90
hearing_radius_blocks: 20
target_body_parts: ["[[part_abdomen_lower]]", "[[part_groin]]"]
inflicts_wounds: ["[[wnd_laceration]]", "[[wnd_infected_bite]]"]
transmits_disease: "[[dis_trung_doc_o_huyet]]"
drops_materials: ["[[mat_xua_co_thi]]"]
habitat_zones: ["[[zone_lang_mo_tang_1]]", "[[zone_dong_dia_dao_quan_co]]"]
---
# Sinh Vật Dị Chủng: Thi Quỷ Đáy Mộ (Ancient Ghoul)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Những phu phen, tù binh bị chôn sống theo các vị tướng thời cổ đại, sau hàng trăm năm hấp thụ tử khí dưới lòng đất hóa thành lũ quỷ đói ăn thịt người.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_aggressive_pack]]
- **Giác quan cảm biến:** Nón nhìn `90°`, thính giác `20 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Ưa bóng tối đặc quánh, rất sợ ánh sáng từ đuốc lửa và sát thương từ [[wpn_ong_hoa_ho]].
