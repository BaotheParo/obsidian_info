---
id: "mob:mob_moc_tinh"
type: "bestiary_entity"
archetype_class: "ANCIENT_COLOSSAL"
ai_profile: "[[sys_ai_colossal_boss]]"
vision_cone_deg: 180
hearing_radius_blocks: 50
target_body_parts: ["[[part_head]]", "[[part_chest_upper]]"]
inflicts_wounds: ["[[wnd_blunt_trauma]]", "[[wnd_fracture]]"]
transmits_disease: "[[dis_lam_chuong_sot_ret]]"
drops_materials: ["[[mat_loi_moc_tinh]]", "[[mat_go_nghien]]"]
habitat_zones: ["[[zone_rung_thieng_lam_chuong]]"]
---
# Sinh Vật Dị Chủng: Mộc Tinh Cổ Thụ (Thần Cây Sanh Ngàn Năm)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Gốc cây cổ thụ ngàn năm từ thời Hồng Bàng, hút tinh huyết của người qua đường hóa thành yêu ma khổng lồ cành lá rậm rạp.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_colossal_boss]]
- **Giác quan cảm biến:** Nón nhìn `180°`, thính giác `50 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Miễn nhiễm đòn đâm và đập thông thường, da gỗ cực dày. Điểm yếu duy nhất là Hỏa công từ [[wpn_ong_hoa_ho]] và tiếng nổ vang động của [[wpn_sung_hoa_mai]].
