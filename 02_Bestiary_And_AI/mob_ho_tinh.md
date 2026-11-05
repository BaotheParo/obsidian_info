---
id: "mob:mob_ho_tinh"
type: "bestiary_entity"
archetype_class: "BEAST_APEX"
ai_profile: "[[sys_ai_ambush_hit_and_run]]"
vision_cone_deg: 160
hearing_radius_blocks: 35
target_body_parts: ["[[part_neck]]", "[[part_thigh_l]]", "[[part_thigh_r]]"]
inflicts_wounds: ["[[wnd_laceration]]", "[[wnd_bleeding_arterial]]", "[[wnd_fracture]]"]
transmits_disease: "[[dis_lam_chuong_sot_ret]]"
drops_materials: ["[[mat_da_cop_ho_tinh]]"]
habitat_zones: ["[[zone_rung_thieng_lam_chuong]]"]
---
# Sinh Vật Dị Chủng: Bạch Hổ Tinh (Chúa Sơn Lâm Thành Tinh)

## 1. Điển Tích & Nguồn Gốc Dã Sử
Loài hổ trắng sống ngàn năm trên núi sâu, ăn thịt vô số người, hiểu tiếng người và biết biến hóa mây mù che khuất mặt trời.

## 2. Hành Vi Chiến Thuật & Trí Tuệ Nhân Tạo (Tactical Behavior)
- **Kiểu hình AI vận hành:** [[sys_ai_ambush_hit_and_run]]
- **Giác quan cảm biến:** Nón nhìn `160°`, thính giác `35 blocks` theo quy chuẩn [[sys_ai_sensory]].
- **Chiến thuật cụ thể:**
Đòn vồ nặng nề làm sập nẹp giáp [[arm_giap_vay_sat]], xé rách đùi nạn nhân gây gãy xương hở [[wnd_fracture]]. Khi bị thương nặng sẽ lui về bóng tối chờ ma trành quấy rối.
