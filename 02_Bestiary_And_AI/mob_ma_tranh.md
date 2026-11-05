---
id: "mob:mob_ma_tranh"
type: "bestiary_entity"
archetype_class: "SPECTRAL_MINION"
ai_profile: "[[sys_ai_ambush_hit_and_run]]"
vision_cone_deg: 140
hearing_radius_blocks: 30
target_body_parts: ["[[part_head]]"]
inflicts_wounds: ["[[wnd_blunt_trauma]]"]
transmits_disease: "[[dis_hoang_loan_tam_than]]"
drops_materials: ["[[mat_chu_sa]]"]
habitat_zones: ["[[zone_rung_thieng_lam_chuong]]"]
---
# Sinh Vật Dị Chủng: Ma Trành (Trành Quỷ Dẫn Mồi)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Linh hồn kẻ bị cọp vồ chết, bị hổ tinh giam giữ sai khiến làm tay sai, chuyên giả tiếng người khóc than giữa rừng đêm để dẫn dụ lữ khách vào ổ phục kích.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_ambush_hit_and_run]]
- **Giác quan cảm biến:** Nón nhìn `140°`, thính giác `30 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Tạo ra ảo giác âm thanh kích hoạt [[sys_panic_stress]], dẫn dụ người chơi đạp vào bẫy hố chông trước khi [[mob_ho_tinh]] xuất đầu lộ diện.
