---
id: "mob:mob_quy_nhap_trang"
type: "bestiary_entity"
archetype_class: "UNDEAD_SPRINTER"
ai_profile: "[[sys_ai_aggressive_pack]]"
vision_cone_deg: 120
hearing_radius_blocks: 25
target_body_parts: ["[[part_neck]]", "[[part_forearm_l]]", "[[part_forearm_r]]"]
inflicts_wounds: ["[[wnd_laceration]]", "[[wnd_infected_bite]]"]
transmits_disease: "[[dis_trung_doc_o_huyet]]"
drops_materials: ["[[mat_mat_quy_nhap_trang]]"]
habitat_zones: ["[[zone_nghia_dia_co]]", "[[zone_lang_mo_tang_1]]"]
---
# Sinh Vật Dị Chủng: Quỷ Nhập Tràng (Thi Quỷ Biến Dị)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Xác người chết oan chưa kịp khâm liệm bị mèo mun nhảy qua thi thể kích hoạt oán niệm bật dậy. Thân thể cứng đờ nhưng bật nhảy cực xa, lao vào cắn xé xé toạc yết hầu nạn nhân.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_aggressive_pack]]
- **Giác quan cảm biến:** Nón nhìn `120°`, thính giác `25 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Khi phát hiện người chơi bị thương chảy máu thông qua [[sys_ai_sensory]], Quỷ Nhập Tràng sẽ hú vang báo hiệu và lao trực diện không màng thương tích.
