---
id: "sys_pipeline:doc01"
type: "system_tick_pipeline_spec"
document_source: "Doc 01: Project Zomboid B42 Architecture and Design Bible (Complete Re-extraction)"
associated_master_moc: "[[00_Master_MOC]]"
state_machines: [
  "IsoZombie FSM (3-Sensory AI)",
  "Pathology FSM (3-Tier Affliction)",
  "Animal Husbandry & Migration FSM",
  "Grid Wave Sound & Blood Scent 60Hz Pipeline"
]
tick_rate_hz: 60
---
# ## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Máy Trạng Thái Cốt Lõi (Core State Machines)

#### 1.1. Máy Trạng Thái AI Xác Sống 3 Giác Quan (`IsoZombie_FSM`)

```mermaid
stateDiagram-v2
    [*] --> WANDER: Khởi tạo thực thể
    WANDER --> INVESTIGATE: Nghe sóng âm (Sound Wave) hoặc ngửi mùi máu (Blood Scent)
    INVESTIGATE --> PURSUIT: Mục tiêu trong nón nhìn 120 độ & LOS Clear (60Hz)
    PURSUIT --> ATTACK: Khoảng cách <= 1.2 blocks
    ATTACK --> PURSUIT: Mục tiêu tháo chạy vượt tầm đánh
    PURSUIT --> INVESTIGATE: Mất dấu LOS > 5 giây
    INVESTIGATE --> WANDER: Không tìm thấy mục tiêu sau 15 giây
```

#### 1.2. Máy Trạng Thái Bệnh Lý 3 Cấp Độ (`Pathology_FSM`)

```mermaid
stateDiagram-v2
    [*] --> HEALTHY
    HEALTHY --> SCRATCH: Nhận đòn cào xước (7% Virus)
    HEALTHY --> LACERATION: Nhận đòn rách sâu (25% Virus)
    HEALTHY --> BITE: Nhận đòn cắn (100% Virus)
    
    SCRATCH --> IS_INFECTED: Trúng độc Knox
    LACERATION --> IS_INFECTED: Trúng độc Knox
    BITE --> IS_INFECTED: Chắc chắn tử vong
    
    SCRATCH --> INFECTED_WOUND: Vết thương bẩn / Không băng bó tiệt trùng
    LACERATION --> INFECTED_WOUND: Vết thương bẩn
    
    INFECTED_WOUND --> HEALTHY: Rửa cồn / Nước sôi / Kháng sinh
    IS_INFECTED --> ZOMBIE_TRANSFORMATION: Sốt cao -> Suy kiệt -> Tử vong
```

#### 1.3. Máy Trạng Thái Sinh Sản & Di Cư Động Vật (`Animal_Husbandry_FSM`)

```mermaid
stateDiagram-v2
    [*] --> JUVENILE: Đẻ con / Ấp nở
    JUVENILE --> ADULT_MATURE: Phát triển sinh học (Growth Rate)
    ADULT_MATURE --> PREGNANT_GESTATION: Thụ tinh có chọn lọc (Selective Breeding)
    PREGNANT_GESTATION --> ADULT_MATURE: Sinh sản thế hệ mới (Alen di truyền)
    ADULT_MATURE --> MIGRATION_FLEE: Bị zombie tấn công / Thiếu thức ăn
    MIGRATION_FLEE --> ADULT_MATURE: Tìm được chunk an toàn mới
```

### 2. Luồng Dữ Liệu Tick Lưới Ô 60 Hz & Mạng UDP

```mermaid
sequenceDiagram
    participant World as IsoWorld Grid (60Hz)
    participant Sound as Audio Wave & FMOD
    participant Scent as Blood Scent Map
    participant Zombie as IsoZombie / PZPopMan64
    participant Player as Player BodyParts 17

    World->>Sound: Đòn chém Fireaxe (SoundRadius 10 blocks)
    Player->>Scent: Chảy máu cổ Neck (bleedingTimeModifier 1.5)
    Sound-->>Zombie: Sóng âm lưới ô truyền tới
    Scent-->>Zombie: Khứu giác máu nhận diện cự ly gần
    Zombie->>Zombie: Kích hoạt chuyển trạng thái PURSUIT
    Zombie->>Player: Tấn công trúng yết hầu (Neck)
    Player->>Player: Kích hoạt ma trận nhiễm trùng & đồng hồ đếm ngược tử vong
```

---
