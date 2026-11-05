---
id: "sys_pipeline:doc04"
type: "system_tick_pipeline_spec"
document_source: "Doc 04: Project Zomboid B42 Professions & Traits Bible (25 Occupations & 95 Traits)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[doc_phb_01_origins]]",
  "[[sys_math_doc04]]"
]
state_machines: [
  "Character Creation Point-Buy & Validation FSM",
  "Dynamic Trait Lifecycle & Physical Eradication Pipeline"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khởi Tạo Point-Buy, Nhân Kinh Nghiệm Vĩnh Viễn & Xóa Bỏ Động Khiếm Khuyết (Character Creation & Trait Lifecycle Pipeline)

```mermaid
stateDiagram-v2
    [*] --> POINT_BUY_ALLOCATION : Khởi tạo Nhân vật (25 Nghề nghiệp & 95 Traits)

    state "1. Ma trận Point-Buy & Khóa Hệ số XP Vĩnh viễn (Initialization)" as InitPhase {
        POINT_BUY_ALLOCATION --> SELECT_OCCUPATION : Chọn 1 Nghề (Cost từ -8 đến +8)\nVí dụ: Lumberjack (0), Burglar (-6), Smither (-6), Veteran (-8)
        SELECT_OCCUPATION --> BALANCE_TRAITS : Cộng Trait Tiêu cực (+26..+30 pts) & Trừ Trait Tích cực\nSao cho Net_Points >= 0
        BALANCE_TRAITS --> LOCK_XP_MULTIPLIERS : Tính cấp khởi đầu L0 cho từng Skill:\nL0 = 0 -> 25% XP (Phạt -75%)\nL0 = 1 -> 75% XP (x3 tốc độ)\nL0 = 2 -> 100% XP (x4 tốc độ)\nL0 >= 3 -> 125% XP (x5 tốc độ)
    }

    state "2. Vòng lặp Chiến đấu, Giác quan & Thể lực (Runtime Sensory & Combat Modifiers)" as RuntimePhase {
        LOCK_XP_MULTIPLIERS --> SENSORY_AND_STEALTH_TICK : Mỗi Tick Giác quan AI
        SENSORY_AND_STEALTH_TICK --> REAR_DETECTION : Keen Hearing (-6): x2 bán kính nhìn sau lưng\nDeaf (+12): Mất 100% âm thanh\nInconspicuous (-4): -50% bị phát hiện khi lén lút
        SENSORY_AND_STEALTH_TICK --> COMBAT_AND_STAMINA : Strong (-10): +40% sát thương cận chiến, mang vác 18-20 kg\nAx-pert: +25% tốc độ vung rìu & chặt cây\nDesensitized: 100% miễn nhiễm Panic\nAsthmatic (+5): Mất thể lực x2 (3-4 nhát vung là kiệt sức)\nDextrous (-2): x2 tốc độ chuyển đồ/băng bó khẩn cấp
    }

    state "3. Cơ chế Tự chữa lành & Triệt tiêu Khiếm khuyết Động (Dynamic Trait Mitigation)" as DynamicTraitPhase {
        LOCK_XP_MULTIPLIERS --> CHECK_EQUIPMENT_AND_METABOLISM : Kiểm tra Trang bị & Cân nặng mỗi ngày
        CHECK_EQUIPMENT_AND_METABOLISM --> NULLIFY_SHORTSIGHTED : Đeo kính (Glasses) -> Vô hiệu hóa phạt Foraging của Short Sighted (+2)
        CHECK_EQUIPMENT_AND_METABOLISM --> CURE_UNDERWEIGHT : Ăn bơ/mỡ đạt chuẩn 80.0 kg sau 3-5 ngày\n-> Xóa vĩnh viễn Trait Underweight (+6) & Phục hồi +1 Fitness
        CHECK_EQUIPMENT_AND_METABOLISM -->XP_ACCUMULATION : Mỗi hành động cộng XP = Raw_XP * M_init(L0) * (1 + 0.30 * FastLearner)
    }

```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực & Y học Nam Dược: [[04_Character_And_Med]]
- Quy tắc gốc AD&D & 5E: [[doc_phb_01_origins]]
- Phương trình toán học đi kèm: [[sys_math_doc04]]
