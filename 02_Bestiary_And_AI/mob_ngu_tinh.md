---
id: "mob:mob_ngu_tinh"
type: "bestiary_entity"
archetype_class: "AQUATIC_BEHEMOTH"
ai_profile: "[[sys_ai_colossal_boss]]"
vision_cone_deg: 150
hearing_radius_blocks: 45
target_body_parts: ["[[part_thigh_l]]", "[[part_foot_l]]", "[[part_foot_r]]"]
inflicts_wounds: ["[[wnd_laceration]]", "[[wnd_puncture]]"]
transmits_disease: "[[dis_trung_han]]"
drops_materials: ["[[mat_vay_ngu_tinh]]"]
habitat_zones: ["[[zone_tran_doanh_song_da]]"]
---
# Sinh Vật Dị Chủng: Ngư Tinh Khổng Lồ (Thủy Quái Sông Cả)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Con cá thần thành tinh dài hàng chục trượng, thân mình vảy sắt lấp lánh như giáp trụ, chuyên tạo sóng dữ lật úp thuyền bè và xe thồ qua sông.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_colossal_boss]]
- **Giác quan cảm biến:** Nón nhìn `150°`, thính giác `45 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Thân bọc vảy sắt tương đương [[arm_giap_tam_ho_tam]]. Muốn hạ sát phải ném khối sắt nung đỏ từ [[bst_lo_ren_than_da]] vào miệng nó.
