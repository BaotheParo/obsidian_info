---
id: "sys_pipeline:doc13"
type: "system_tick_pipeline_spec"
document_source: "Doc 13: Project Zomboid B42 Medical, Surgery & Health Bible (17 Body Parts, 6 Wound Kinetics, Dual Infection & Surgery Equations)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[doc_vn_nam_duoc_than_hieu_y_hoc]]",
  "[[sys_math_doc13]]"
]
state_machines: [
  "17-Part Anatomical Damage Update & Bleed Decay Pipeline",
  "Dual Infection (Knox Virus vs Bacterial Wound) State Machine",
  "4-Stage Field Surgical Procedure & Stitch Lifecycle FSM"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Giải Phẫu 17 Bộ Phận, Phẫu Thuật Ngoại Khoa, Nhiễm Khuẩn Kép & Độc Học (`Medical, Surgery & Toxicology Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> ANATOMICAL_TRAUMA_EVENT : Nhận sát thương vào 1 trong 17 Bộ phận (BodyPartType)

    state "1. Phân bổ Sát thương & Chảy máu Tử huyệt (17-Part Trauma & Exsanguination)" as TraumaPhase {
        ANATOMICAL_TRAUMA_EVENT --> APPLY_PART_MODIFIERS : Tính PainMod (0.50..0.80), DmgMod (0.10..0.70), BleedMod (0.20..1.50)\n- NECK (Cổ): BleedMod = 1.50, DmgMod = 0.70 -> Chết sau 12 - 18 giây!\n- HAND_L/R: MaxActionPenalty = 100% | FOOT_L/R: MaxMovementPenalty = 100%
        APPLY_PART_MODIFIERS --> CHECK_FOREIGN_OBJECT : Kiểm tra Dị vật găm trong thịt:
        CHECK_FOREIGN_OBJECT --> GLASS_LODGED : haveGlass == true -> Khóa cứng bleedingTime >= 3.0 (Máu chảy không ngừng!)\nGắp bằng Tweezers/Forceps (150 - 4*Lv ticks, Pain 30 - Lv) hoặc Tay không (300 ticks, xước tay)
        CHECK_FOREIGN_OBJECT --> BULLET_LODGED : haveBullet == true -> Mổ bằng SutureNeedleHolder/Forceps (250 - 6*Lv ticks, Pain 80 - Lv)\nMổ xong chuyển thành Deep Wound (deepWounded = true)
    }

    state "2. Phẫu thuật Khâu, Nẹp xương & Băng bó Thảo dược (Surgery & Poultice Engine)" as SurgeryPhase {
        BULLET_LODGED --> STITCH_PROCEDURE : Khâu Deep Wound (SutureNeedleHolder: 150-4*Lv ticks, Pain 5-Lv)\nRủi ro nhiễm trùng khi khâu = 1 / (5 + Lv*2.5) (20% ở Lv0 -> 3.3% ở Lv10)
        STITCH_PROCEDURE --> STITCH_MATURATION : Tích lũy stitchTime (0 -> 50.0)\n- Cắt chỉ khi < 30.0 -> Toác lại vết thương sâu & Chảy máu!\n- Cắt chỉ khi > 30.0 (Doctor Lv7+ báo 'Good' ở > 40.0, ~4-5 ngày) -> Lành hoàn toàn
        
        ANATOMICAL_TRAUMA_EVENT --> SPLINT_FRACTURE : Gãy xương (Fracture) -> Bó nẹp SplintFactor = (DocLv + 1) / 2 (0.5 -> 5.5 = x11 tốc độ)\n+ Đắp Cao Liên Mộc (ComfreyCataplasm: -5.0E-6/tick) -> Tăng tốc liền xương 10 - 55 lần
        
        GLASS_LODGED --> BANDAGE_AND_POULTICE : Băng bó (120 - 4*Lv ticks) + Đắp Cao Mã Đề (PlantainCataplasm):\n- Scratch: Băng bó 15x -> Đắp Mã Đề 25x (-2.5E-4/tick)\n- Cut (Rách sâu): Băng bó 50x -> Đắp Mã Đề 100x (-1.0E-4/tick)
    }

    state "3. Kiểm tra Nhiễm trùng Kép & Độc học Môi trường (Sepsis & Toxicology Tick)" as SepsisPhase {
        BANDAGE_AND_POULTICE --> ROLL_BACTERIAL_SEPSIS : Mỗi nhịp tính Chance = Max(5000, 40000 - Sum(Penalties))\n(Băng bẩn BandageLife==0 trừ -35.000 -> Tăng nguy cơ gấp 7x! DeepWound hút bẩn -0.005/tick)\nChữa isInfectedWound: AlcoholBandage (-6.0E-4/tick) | Antibiotics (50.0, -2.0E-4/tick) | Tỏi rừng (-2.0E-4/tick)
        
        ANATOMICAL_TRAUMA_EVENT --> KNOX_MORTALITY_TIMER : Nếu trúng isInfected (7% Scratch / 25% Cut / 100% Bite):\nỦ bệnh 48-72h * del (Resilient: 1.25 | ProneToIllness: 0.75) -> Tử vong -> Hóa Zombie sau 30-120s
        
        [*] --> ENVIRONMENTAL_TOXICOLOGY : Kiểm tra Môi trường & Dược phẩm mỗi Tick:\n- Đứng gần > 20 xác thối -> CorpseSickness (Queasy -> Fever -> Chết)\n- Máy phát trong nhà kín -> Ngạt khí CO chết sau 10 - 15 phút\n- Ăn nấm/đồ thiu -> Chữa bằng Sả Chanh (Lemongrass) | Thuốc ngủ/Giảm đau + Rượu -> Quá liều tử vong!
    }

```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực & Y học Nam Dược: [[04_Character_And_Med]], [[doc_vn_nam_duoc_than_hieu_y_hoc]]
- Phương trình toán học đi kèm: [[sys_math_doc13]]
