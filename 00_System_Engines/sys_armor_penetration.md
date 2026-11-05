---
id: "sys:armor_penetration"
type: "core_engine_system"
damage_types: ["SLASHING", "BLUDGEONING", "PIERCING"]
armor_classes: ["[[arm_ao_vai_bong]]", "[[arm_giap_vay_sat]]", "[[arm_giap_tam_ho_tam]]"]
---
# Ma Trận Khắc Chế Binh Khí vs. Giáp Trụ (Armor Penetration Matrix)

## 1. Nguyên Tắc Vật Lý
- **Chém (`SLASHING`):** Cực mạnh trước mục tiêu không giáp hoặc [[arm_ao_vai_bong]], nhưng bị giảm 70% sát thương và gây mòn lưỡi khi đánh vào [[arm_giap_tam_ho_tam]].
- **Đập (`BLUDGEONING`):** Bỏ qua giáp kim loại, truyền lực chấn động gây gãy xương [[wnd_fracture]] và đụng dập [[wnd_blunt_trauma]].
- **Đâm (`PIERCING`):** Lách qua khe hở của giáp vảy [[arm_giap_vay_sat]], tạo vết đâm sâu [[wnd_puncture]].
