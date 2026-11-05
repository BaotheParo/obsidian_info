---
id: "sys_pipeline:doc02"
type: "system_tick_pipeline_spec"
document_source: "Doc 02: Project Zomboid B42 Complete Item Encyclopedia (5,105 Items Database)"
associated_master_moc: "[[00_Master_MOC]]"
state_machines: [
  "Encumbrance & Container FSM (Overburdened & Spinal Injury)",
  "Armor Degradation & Bite Absorption FSM (100% Protection Holing)"
]
tick_rate_hz: 60
---
# ## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Máy Trạng Thái Cốt Lõi (Core State Machines)

#### 2.1. Máy Trạng Thái Tải Trọng & Quản Lý Thùng Chứa Balo (`Encumbrance_Container_FSM`)

```mermaid
stateDiagram-v2
    [*] --> UNENCUMBERED: Tải trọng <= 50% MaxCapacity
    UNENCUMBERED --> HEAVY_LOAD: Tải trọng > 50%
    HEAVY_LOAD --> OVERBURDENED: Tải trọng > 80% (Giảm 20% tốc độ chạy)
    OVERBURDENED --> EXTREME_ENCUMBRANCE: Tải trọng > 100% (Cấm chạy nước rút, mất thể lực liên tục)
    EXTREME_ENCUMBRANCE --> SPINAL_INJURY: Duy trì quá tải > 2 phút (Gây chấn thương đụng dập cột sống)
    
    EXTREME_ENCUMBRANCE --> OVERBURDENED: Vứt bỏ bớt vật phẩm / Chuyển vào xe thồ Caravan
    OVERBURDENED --> HEAVY_LOAD: Trang bị balo giảm tải Large Framepack
    HEAVY_LOAD --> UNENCUMBERED: Trút hàng vào kho lương bst_kho_luong
```

#### 2.2. Máy Trạng Thái Suy Hao Giáp Trụ Khi Hấp Thụ Đòn Cắn (`Armor_Degradation_FSM`)

```mermaid
stateDiagram-v2
    [*] --> PRISTINE_CONDITION: Giáp mới 100%
    PRISTINE_CONDITION --> WORN_ABSORBING: Hấp thụ đòn cắn 100% (Kháng hoàn toàn Virus Knox)
    WORN_ABSORBING --> DAMAGED_HOLING: Độ bền Condition chạm mốc 0
    DAMAGED_HOLING --> COMPROMISED_ZERO_DEFENSE: Rách toạc tạo lỗ hổng (Hole)
    
    COMPROMISED_ZERO_DEFENSE --> PRISTINE_CONDITION: Vá da thú tại trạm entity_leather_prep
    COMPROMISED_ZERO_DEFENSE --> INFECTED_PENETRATION: Bị cắn vào lỗ rách (100% dính Virus)
```

### 2. Luồng Xử Lý Trao Đổi Nhiệt & Dinh Dưỡng Tick Pipeline

```mermaid
sequenceDiagram
    participant Player as Nhân vật (Body Parts 17)
    participant Inventory as Balo ALICE (WeightReduction 85%)
    participant Nutrition as Hệ Thống Chuyển Hóa (Digestion)
    participant World as Môi trường (Nhiệt độ & Độ ẩm)

    Player->>Inventory: Mang vác 35kg vật tư qua rừng rậm
    Inventory-->>Player: Áp dụng hệ số giảm tải (Chỉ còn gánh 9.25kg)
    Player->>Player: Đốt cháy 3.500 kcal năng lượng lao động nặng
    Player->>Nutrition: Tiêu hóa món xào B42 (Cung cấp 516 kcal & mỡ)
    Nutrition-->>Player: Cân bằng thể trọng 80kg, phục hồi thể lực cơ bắp
    World->>Player: Nắng gắt 38°C kích hoạt Thermoregulator tiết mồ hôi làm mát
```

---
