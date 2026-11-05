---
id: "mob:mob_tho_phi"
type: "bestiary_entity"
archetype_class: "HUMANOID_BANDIT"
ai_profile: "[[sys_ai_tactical_humanoid]]"
vision_cone_deg: 120
hearing_radius_blocks: 25
target_body_parts: ["[[part_neck]]", "[[part_chest_upper]]"]
inflicts_wounds: ["[[wnd_laceration]]", "[[wnd_puncture]]"]
transmits_disease: "[[dis_lam_chuong_sot_ret]]"
drops_materials: ["[[mat_than_da]]", "[[mat_quang_sat_song_thuong]]"]
habitat_zones: ["[[zone_rung_thieng_lam_chuong]]", "[[zone_mo_quang_song_thuong]]"]
---
# Sinh Vật Dị Chủng: Thổ Phỉ Thượng Du (Lâm Tặc Rừng Sâu)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Những toán tàn binh và kẻ sống ngoài vòng pháp luật, chiếm cứ các điểm nút giao thông hiểm trở để chặn đường cướp bóc đoàn xe [[crv_xe_tho_boc_sat_15_parts]].

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_tactical_humanoid]]
- **Giác quan cảm biến:** Nón nhìn `120°`, thính giác `25 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Sử dụng [[wpn_dao_ngon_mia]] và [[wpn_cau_liem]], phối hợp đặt bẫy chông nhắm vào bàn chân [[part_foot_l]] của phu xe.
