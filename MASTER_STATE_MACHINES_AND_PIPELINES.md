# TỔNG TẬP MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU TICK PIPELINE (MASTER STATE MACHINES & PIPELINES)
> Dự án: Grimdark Historical Survival RPG (Đại Việt XIV-XV) & Project Zomboid B42 Architecture
> Tích lũy liên tục từ Doc 01 đến Doc 14

================================================================================
# DOCUMENT 01: ARCHITECTURE AND DESIGN BIBLE (100% HOÀN CHỈNH)
================================================================================

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

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


================================================================================
# DOCUMENT 02: COMPLETE ITEM ENCYCLOPEDIA & DATABASE GUIDE
================================================================================

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

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


================================================================================
# DOCUMENT 03: CRAFTING & TECH TREE GUIDE (1,026 RECIPES & 20 WORKSTATIONS)
================================================================================

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Thực Thi Công Thức Chế Tác & Tiến Hóa 5 Thời Kỳ (Workstation CraftRecipe & 5-Age Progression Pipeline)

```mermaid
stateDiagram-v2
    [*] --> AGE_0_PRIMITIVE : Khởi đầu sinh tồn (Chưa có công cụ kim loại)

    state "1. Chuỗi Tiến hóa 5 Thời kỳ (5-Age Tech Tree Dependency Graph)" as TechProgression {
        AGE_0_PRIMITIVE --> AGE_1_POTTERY : Ghè đá lửa (230 ticks) + Thu hái Đất sét bờ sông\nXây Bàn xoay gốm + Máy ép gạch (50 ticks) + Hố đốt than
        AGE_1_POTTERY --> AGE_2_METALLURGY : Xây Lò nung vòm (Dome Kiln) -> Nung Gạch chịu nhiệt (Firebricks)\n& Khuôn đúc (100 ticks) + Giải quyết cắt nước ngày 30
        AGE_2_METALLURGY --> AGE_3_GLASS_TEXTILE : Xây Lò luyện quặng (Furnace I) + Lò rèn (Forge I) + Đe sắt\nNấu chảy cát + tro Soda (>1200°C, 20 ticks) -> Thổi lọ thủy tinh (100 ticks)\nĐập/Chải/Quay/Dệt vải lanh + Thuộc da chống cắn 80-100%
        AGE_3_GLASS_TEXTILE --> AGE_4_RECONSTRUCTION : Nâng cấp Forge II/III + Bàn cưa vòng + Máy khoan đứng\nRèn Giáp tấm, Gorget che cổ (100% Bite Def), Kiếm ngắn (500 ticks)\nĐúc thuốc đạn (Lưu huỳnh + Than + Chì) + Phơi thịt khô Jerky (6-12 tháng)
    }

    state "2. Máy Trạng Thái Thực Thi craftRecipe (Per-Recipe Tick Pipeline)" as RecipeEngine {
        [*] --> VALIDATE_CONTEXT : Người chơi gửi yêu cầu chế tác (1 trong 1.026 công thức)
        VALIDATE_CONTEXT --> CHECK_THERMAL_AND_TOOLS : Kiểm tra khoảng cách tới Workstation (20 trạm)\nKiểm tra nguyên liệu & công cụ mode:keep
        CHECK_THERMAL_AND_TOOLS --> RUNNING_CRAFT_TICKS : Hợp lệ -> Khóa hành động trong T_craft (20..600 ticks)
        RUNNING_CRAFT_TICKS --> TOOL_DEGRADATION_ROLL : Hoàn tất T_craft -> Tiêu hao nguyên liệu thường\nGiữ lại công cụ mode:keep
        TOOL_DEGRADATION_ROLL --> APPLY_TOOL_WEAR : Kiểm tra cờ flags[MayDegradeLight] / flags[MayDegradeHeavy]\nTrừ độ bền công cụ (Phục hồi vũ khí cùn tại Grindstone)
        APPLY_TOOL_WEAR --> GRANT_OUTPUT_AND_XP : Sinh vật phẩm Output + Cộng trực tiếp xpAward = Skill:XP\n(Nhân với hệ số Sách kỹ năng Tập 1..5 nếu đã đọc)
    }

```

---


---

# DOCUMENT 04: PROFESSIONS & TRAITS BIBLE (25 OCCUPATIONS & 95 TRAITS)

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

# DOCUMENT 05: WORLD MAP & LOOT DISTRIBUTION GUIDE (11 REGIONS, 362 BASEMENTS & PROCEDURAL DISTRIBUTIONS)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khởi Tạo Thùng Chứa Thế Giới, Tính Trọng Số Loot & Sinh Tầng Hầm Ngầm (`World Map & Procedural Basement Tick Pipeline`)

```mermaid
flowchart TD
    StartChunk["Chunk Thế Giới Được Nạp (10x10x8 Tiles)"] --> CheckBasement{"Kiểm Tra Xác Suất Sinh Hầm Ngầm<br/>(P = 8% * ZoneMod * TierMod)"}
    
    CheckBasement -- "Xúc Xắc Thất Bại" --> SpawnSurfaceBuilding["Chỉ Khởi Tạo Công Trình Mặt Đất (z = 0)"]
    CheckBasement -- "Xúc Xắc Thành Công" --> QueryPzby["Truy Vấn 1 Trong 362 Layout .pzby Phù Hợp"]
    
    QueryPzby --> StitchVertical["Ghép Nối Lô Ngầm (z = -1)<br/>Tạo Cầu Thang Ngầm / Cửa Nắp Sàn"]
    StitchVertical --> ApplyDarkness["Gán Mức Sáng Ambient = 0.0 Lux<br/>Triệt Tiêu Hoàn Toàn Tầm Nhìn Tự Nhiên"]
    
    SpawnSurfaceBuilding --> ScanContainers["Quét Toàn Bộ Đối Tượng IsoObject (Container)"]
    ApplyDarkness --> ScanContainers
    
    ScanContainers --> ContainerLoop["Vòng Lặp Qua Từng Thùng Chứa"]
    ContainerLoop --> CheckSpecialEvent{"Kiểm Tra Sự Kiện Tòa Nhà<br/>(Survivor House 5%? Party House?)"}
    
    CheckSpecialEvent -- "Survivor House" --> ForceBarricades["Gia Cố Cửa/Cửa Sổ 3 Ván Gỗ (3000 HP)<br/>100% Đồ Hộp Tủ Bếp, 60% Súng Tủ Đồ"]
    CheckSpecialEvent -- "Bình Thường" --> LoadProceduralTable["Đọc Bảng Loot Từ ProceduralDistributions.lua"]
    
    ForceBarricades --> RollLoot["Thực Thi R Lần Tung (Rolls) Theo Trọng Số W_i / Tổng W"]
    LoadProceduralTable --> RollLoot
    
    RollLoot --> PopulateInventory["Nạp Vật Phẩm Vào Thùng Chứa Thế Giới"]
    PopulateInventory --> EndChunk["Hoàn Tất Khởi Tạo Chunk"]
```

---

### 2. Sơ Đồ Pipeline Lộ Trình 7 Ngày Đầu Tiên Tối Ưu (`Day 1 - Day 7 Survival Route Execution`)

```mermaid
graph TD
    Day1["Ngày 1: Sinh Tồn Tại Chỗ<br/>- Dọn dẹp nhà xuất phát<br/>- Thu thập: 2 chai nước, dao bếp, chảo xào, balo học sinh<br/>- Đọc sách kỹ năng Tập 1 ngay trong ngày"]
    
    Day2["Ngày 2: Đột Kích Trạm Cứu Hỏa / Cảnh Sát<br/>- Tiếp cận Rosewood hoặc Riverside<br/>- Thu thập: Rìu cứu hỏa (Vũ khí chính), Áo chống cắn 100%, Balo ALICE"]
    
    Day3["Ngày 3: Kiếm Xe Hơi & Xăng Dầu<br/>- Kiểm tra bãi đỗ xe tìm chìa khóa / Nghề Burglar bẻ khóa nổ máy<br/>- Dùng bình xăng rỗng hút xăng (Siphon) sang xe tốt"]
    
    Day4["Ngày 4: Cửa Hàng Kim Khí & Hiệu Sách<br/>- Thu thập: Búa, Cưa gỗ, Đinh sắt, Xẻng, Sách kỹ năng Vol 1-3<br/>- Tối thượng: Sách 'Cách dùng máy phát điện' (ElectronicsMag4)"]
    
    Day5["Ngày 5: Truy Lùng Búa Tạ & Máy Phát Điện<br/>- Đột kích nhà kho công nghiệp (Muldraugh / Riverside Warehouses)<br/>- Tìm Búa tạ (Sledgehammer) phá cửa tiệm súng<br/>- Nhặt ít nhất 2 Máy phát điện (Căn cứ & Cây xăng)"]
    
    Day6_7["Ngày 6 - 7: Rút Về Căn Cứ Ven Sông & Thiết Lập Cây Công Nghệ B42<br/>- Thiết lập biệt thự ven sông có rào sắt an toàn<br/>- Khai thác Đất sét bờ suối, nặn bàn gốm, đốt than, xây lò nung<br/>- Hoàn tất nông nghiệp & trữ nước trước Ngày 30 cúp điện cúp nước"]

    Day1 --> Day2
    Day2 --> Day3
    Day3 --> Day4
    Day4 --> Day5
    Day5 --> Day6_7
```

---


---

# DOCUMENT 06: VEHICLES & MECHANICS HANDBOOK (52 VEHICLE MODELS, 15 MECHANICAL PARTS & HOTWIRING DYNAMICS)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khởi Động, Vật Lý Va Chạm, Phát Tán Âm Thanh Ống Xả & Bảo Trì 15 Bộ Phận (`Vehicle Physics & Mechanics Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> VEHICLE_ENTERED : Người chơi leo vào ghế (Ưu tiên đập kính ghế phụ nếu cửa khóa)

    state "1. Chiếm quyền & Đánh lửa Khởi động (Hotwire & Ignition Pipeline)" as IgnitionPhase {
        VEHICLE_ENTERED --> CHECK_AUTH : Kiểm tra Chìa khóa / Trạng thái Hotwired
        CHECK_AUTH --> HOTWIRE_PROCESS : Chưa mở khóa -> Yêu cầu: Nghề Burglar\nHOẶC (Electricity >= 1 & Mechanics >= 2 & Screwdriver)\nThao tác đấu dây: 5 - 10 giây
        HOTWIRE_PROCESS --> IGNITION_ATTEMPT : Đấu dây vĩnh viễn thành công
        CHECK_AUTH --> IGNITION_ATTEMPT : Đã có chìa / Đã Hotwire
        
        IGNITION_ATTEMPT --> BATTERY_AND_QUALITY_CHECK : Yêu cầu Battery >= 10% & EngineCondition > 0%\n- Nếu EngineQuality > 85%: Nổ máy ngay lần 1 (kể cả âm độ)\n- Nếu EngineQuality < 50%: Phải đề 3 - 5 lần mới nổ
    }

    state "2. Vòng lặp Vận hành, Sóng âm & Va chạm (PZBullet64 Runtime Tick)" as DrivingPhase {
        BATTERY_AND_QUALITY_CHECK --> ENGINE_RUNNING_TICK : Động cơ nổ máy -> Tiêu hao xăng (Rò rỉ nếu Gas Tank thủng)
        
        ENGINE_RUNNING_TICK --> MUFFLER_SOUND_WAVE : Tính SoundRadius theo Muffler Condition:\n- Muffler > 80%: Bán kính 20 - 30m\n- Muffler < 20%: Bán kính 100 - 150m (Gom zombie cả thị trấn)
        
        ENGINE_RUNNING_TICK --> CHASSIS_DYNAMICS : - Lốp chuẩn 30-35 psi (Nổ/non hơi -> Giảm 70% tốc độ)\n- Phanh hỏng -> Quãng đường phanh x4\n- Giảm xóc hỏng -> Lật nghiêng khi cua gấp\n- EngineCondition < 20% -> Chết máy ngẫu nhiên
        
        ENGINE_RUNNING_TICK --> COLLISION_RESOLUTION : Va chạm với bầy Zombie
        COLLISION_RESOLUTION --> FRONTAL_IMPACT : Đâm đầu xe -> Sau 3-5 lần Hood = 0%\n-> Động cơ nhận 100% sát thương trực tiếp -> Chết máy!
        COLLISION_RESOLUTION --> REVERSE_SPIN_SWEEP : Lùi đuôi xe (Reverse Spin) -> Bánh xe nghiền nát đầu zombie\nĐộng cơ an toàn 100% (Chỉ móp cốp giảm dung tích Trunk)
    }

    state "3. Thoát hiểm Bao vây & Bảo trì Cơ khí 24h (Escape & 24h Mechanics Cooldown)" as MaintenancePhase {
        FRONTAL_IMPACT --> SURROUNDED_STALLED : Xe chết máy giữa 10 - 20 con zombie
        SURROUNDED_STALLED --> INSTANT_DEATH_2S : Mở cửa nhảy ra ngoài ngay -> Bị dìm chết trong 2 giây!\n(Hoặc kính chắn gió/cửa sổ vỡ -> Bị cắn xuyên cửa vào mặt)
        SURROUNDED_STALLED --> SEAT_SWITCH_ESCAPE : Kính còn nguyên 100% -> Bấm 'Z' chuyển sang ghế phụ đối diện\nMở cửa phía ít zombie để thoát thân
        
        VEHICLE_ENTERED --> MECHANICS_SERVICE : Mở giao diện Cơ khí (15 Bộ phận / 3 Phân hạng)
        MECHANICS_SERVICE --> CHECK_24H_COOLDOWN : Tháo/Lắp Đèn, Đài, Ắc quy, 4 Bánh trên 5-10 xe/ngày\nNếu (T_now - T_last_part_id) >= 24 giờ -> Cộng XP * M_book(x3..x5)\nSau 3-5 ngày đạt Mechanics Cấp 4-6
    }

```

---


---



# DOCUMENT 07: MEDIA BROADCASTS, KNOX LORE & 64 SKILL VHS BIBLE (INSTRUCTIONAL MEDIA, XP MULTIPLIERS & AEBS ORACLE)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Phát Sóng 14 Ngày, Dự Báo AEBS Mỗi Giờ & Xử Lý Chồng XP Bản Ghi (`Media, AEBS Oracle & Skill Leveling Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> WORLD_INIT_DAY_0 : Khởi tạo Thế giới (Ngày 0 = 09/07)

    state "1. Bộ Lập Lịch Phát Sóng & Điềm Báo AEBS (Hourly & 14-Day Cron Pipeline)" as BroadcastCron {
        WORLD_INIT_DAY_0 --> LOCK_AEBS_FREQ : Chạy getRandomFrequency(88000, 108000) bước nhảy 200 (0.2 MHz)\nKhóa 1 trong 101 tần số AEBS cho toàn bộ Save Game
        LOCK_AEBS_FREQ --> HOURLY_AEBS_TICK : Đầu mỗi giờ (XX:00) phát bản tin AEBS:\n- Dự báo Gió / Sương mù / Bão (trước 24-48h)\n- Cảnh báo Lưới điện 3 giai đoạn (Dao động 48-72h -> Sụp đổ -> Blackout)\n- Nếu >= 09:00 có "Air Activity detected." -> Kích hoạt Siêu sự kiện Trên không (12:00-19:00)
        
        WORLD_INIT_DAY_0 --> LIFE_AND_LIVING_9DAYS : Kênh 203 phát sóng Ngày 0..Ngày 8 (06:00, 12:00, 18:00)\nNgười chơi vặn Volume = 1..2 vạch để tránh lộ SoundRadius
        LIFE_AND_LIVING_9DAYS --> SIGNAL_COLLAPSE_DAY_8 : Sau 06:00 Ngày 8 (Ngày 7 bùng phát chủng không khí)\nToàn bộ kênh TV tắt thành Nhiễu trắng (Static White Noise)
    }

    state "2. Máy Trạng Thái Tiêu Thụ Bản Ghi & Nhân XP 5 Bậc (Media Consumption Engine)" as MediaXPEngine {
        [*] --> READ_SKILL_BOOK_FIRST : Đọc Sách Kỹ năng Tập 1..5 trước khi xem/nghe\nKích hoạt Multiplier: Vol1(x3), Vol2(x5), Vol3(x8), Vol4(x12), Vol5(x16)
        READ_SKILL_BOOK_FIRST --> CONSUME_MEDIA_LINE : Nhân vật nghe từng dòng thoại (13.166 lines / 352 media items)
        
        CONSUME_MEDIA_LINE --> CHECK_STACKING_RULE : Kiểm tra Nguồn Phát (TV vs 155 Retail VHS vs 128 Home VHS)
        CHECK_STACKING_RULE --> BLOCK_DUPLICATE_XP : Nếu là Retail VHS VÀ đã xem tập đó trên TV\n-> Bỏ qua XP (0% XP)
        CHECK_STACKING_RULE --> GRANT_BOOSTED_XP : Nếu là TV lần đầu HOẶC Retail VHS chưa xem TV\nHOẶC là Home VHS (Độc lập 100% với TV)
        
        GRANT_BOOSTED_XP --> APPLY_FORMULA_AND_RECIPES : XP_final = Base_XP * M_book(x3..x16) * (1 + 0.30 * FastLearner) * M_init\nMở khóa Recipe (VD: MakeMildewCure, MetalWallLvl1, CraftMakeshiftRadio)\nHoặc biến đổi Tâm lý (The Dog Goblin: Stress +0.2, +Panic)
    }

```

---

# DOCUMENT 08: FARMING, TRAPPING, FISHING & WILDERNESS SURVIVAL HANDBOOK (43 CROPS/HERBS, 6 TRAPS, 21 FISH, 18 GENES & 27 FORAGE CATEGORIES)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Chu Trình Sinh Tồn Hoang Dã 5 Trụ Cột Khép Kín (`5-Pillar Wilderness Survival & Genetics Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> INFRASTRUCTURE_COLLAPSE_DAY_14_30 : Cắt điện & nước vĩnh viễn (Ngày 14-30)\nKích hoạt Chu trình Sinh tồn Tự cung Tự cấp 365 Ngày

    state "1. Trụ cột Thu lượm Quang học (Foraging Search Mode Tick)" as ForagePhase {
        INFRASTRUCTURE_COLLAPSE_DAY_14_30 --> CALCULATE_SEARCH_RADIUS : R_eff = 3.0 * (1 + Lv*0.25) * M_light * M_weather * M_moodle * M_gear\n(Từ 3.0 tiles ở Lv0 -> 10.5 tiles ở Lv10; Đêm -60%, Sương -50%, Mũ kín -30%)
        CALCULATE_SEARCH_RADIUS --> HARVEST_27_CATEGORIES : Thu thập Đá lửa (Flint), Mồi câu (Trùn/Đỉa), Dấu vết thú (Tracks)\n& 5 Thảo dược (Comfrey liền xương x2; Nấm độc chết sau 24h nếu thiếu Herbalist)
    }

    state "2. Trụ cột Nông nghiệp 43 Cây & Cách ly Bàn cờ (Farming Tick)" as FarmPhase {
        INFRASTRUCTURE_COLLAPSE_DAY_14_30 --> SOW_CHECKERBOARD_PLOTS : Gieo 20 Nông sản + 23 Thảo mộc (Cách nhau 1 ô trống chống lây bệnh)\nBắp cải (144h), Cà rốt (168h, chết úng nếu Nước > 85), Khoai tây (288h, tươi 30 ngày), Ngô (480h)
        SOW_CHECKERBOARD_PLOTS --> PEST_AND_DISEASE_CHECK : Kiểm tra 4 bệnh cây trồng mỗi Tick:\n- Mildew (-50% sản lượng) -> Xịt 50ml Sữa + Nước\n- Aphids (Khô héo sau 48h) -> Xịt 5 Thuốc lá + Nước\n- Flies -> Xịt 1 Tỏi + Nước | Slugs -> Lon + 1 Ngụm Bia
    }

    state "3. Trụ cột Bẫy thú 75-Tile & Câu cá Lực căng (Trapping & Fishing Tick)" as TrapFishPhase {
        PEST_AND_DISEASE_CHECK --> TRAP_75TILE_EVALUATION : Dùng Cà rốt (45%) / Bắp cải (40%) làm bả bẫy Thỏ (19:00-05:00)\nBắt buộc khoảng cách Người chơi >= 75 Tiles (Nếu < 75 tiles: thú ăn mất bả & phá bẫy)\nBắt sống (canBeAlive = true): Thỏ, Gấu mèo, Chuột
        HARVEST_27_CATEGORIES --> FISHING_TENSION_LOOP : Câu 21 loài cá (Giờ vàng 04:00-07:00 & 18:00-21:00 -> +100% cắn câu; Đông <0°C -> -50..70%)\nNhấp nhả Tension UI (Xanh -> Vàng -> Đỏ) săn Thủy quái Aligator Gar (Trophy 243cm / 150kg)
    }

    state "4. Trụ cột Chăn nuôi 18 Gen & Móc mổ thịt 5 Bước (Husbandry & Butchering)" as HusbandryPhase {
        TRAP_75TILE_EVALUATION --> BREED_18_GENES : Lai tạo cặp Allele trội/lặn (maxSize 0.85-0.95, maxMilk 20-30L/ngày, eggClutch 6-8 trứng)\nKiểm soát Stress (ngừng đẻ/tắt sữa) & Aggressiveness (Bò/Lợn đực húc người)
        BREED_18_GENES --> BUTCHER_HOOK_5_STEPS : Treo xác lên Butcher Hook:\n1. Hứng tiết (Blood) -> 2. Lột da (Leather) -> 3. Cắt đầu (Head)\n-> 4. Xẻ thịt (Meat) -> 5. Lọc xương (Bones làm Keo, Kim khâu, Bột xương bón ruộng)
    }

```

---


---

# DOCUMENT 09: CLIMATE, SEASONS & WORLD EROSION BIBLE (12 MONTHS, SEVERE WEATHER, 37.0°C CORE EQUILIBRIUM & 100-DAY EROSION CURVE)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khí Hậu 4 Mùa, Cân Bằng Thân Nhiệt $37.0^\circ\text{C}$ & Bào Mòn Thế Giới (`Climate, Thermoregulation & Erosion Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> CLIMATE_MANAGER_TICK : Khởi chạy từ Ngày 09/07/1993 (Vĩ độ 38° Bắc)

    state "1. Vòng lặp Khí quyển 12 Tháng & Thời tiết Cực đoan (Climate & WeatherPeriod)" as ClimatePhase {
        CLIMATE_MANAGER_TICK --> SEASONAL_TRANSITION : Hè (22..38°C) -> Thu (4..28°C, Sương muối T11 -2°C diệt cây)\n-> Đông (-15..17°C, Đêm T12 dài 14h) -> Xuân (1..28°C, Bùn lầy T3)
        SEASONAL_TRANSITION --> SEVERE_WEATHER_EVENT : Kích hoạt Khí tượng Cực đoan:
        SEVERE_WEATHER_EVENT --> TROPICAL_STORM : Bão nhiệt đới -> Tầm nhìn 3-5 ô | Tiếng mưa át bước chân\nSấm sét (Thunder) nổ 100-200 ô khuấy động bầy đàn
        SEVERE_WEATHER_EVENT --> ARCTIC_BLIZZARD : Bão tuyết (T12-T2) -> -15°C đến -25°C + Max Wind Chill\nTuyết dày phạt -50% tốc độ chạy & làm trượt bánh xe
        SEVERE_WEATHER_EVENT --> HEAVY_FOG : Sương mù (T9-T11) -> Giảm -70% tầm nhìn (Zombie cách 2-3 bước mới thấy)
    }

    state "2. Máy Trạng Thái Thân Nhiệt 17 Bộ phận & Cảm Lạnh (Thermoregulator Tick)" as ThermoPhase {
        CLIMATE_MANAGER_TICK --> EVALUATE_17_BODYPARTS : Tính Nhiệt độ Da & Lõi (Chuẩn 37.0°C / An toàn 36.5-37.5°C)\nQua 3 lớp chỉ số quần áo: Insulation, WindResistance, WaterResistance (Poncho = 100%)
        
        EVALUATE_17_BODYPARTS --> COLD_CASCADE : T_core < 36.5°C
        COLD_CASCADE --> SHIVERING_35_365 : 35.0 - 36.5°C: Rét run -> Đốt Calo & Năng lượng x3.0 - x5.0
        COLD_CASCADE --> FROSTBITE_30_35 : 30.0 - 35.0°C: Tê cóng -> Tốc độ chạy -50%, Vung vũ khí -50%, Thao tác -40%
        COLD_CASCADE --> LETHAL_HYPOTHERMIA : < 30.0°C: Rút máu liên tục đến chết (T1 ngoài trời chết sau 1 giờ)

        EVALUATE_17_BODYPARTS --> HEAT_CASCADE : T_core > 37.5°C
        HEAT_CASCADE --> SWEATING_375_39 : 37.5 - 39.0°C: Đổ mồ hôi -> Khát nước x4.0 & Ướt sũng áo lót trong\n(Ngừng vận động gặp gió -> Đảo chiều hạ nhiệt -> Cảm lạnh giữa mùa hè!)
        HEAT_CASCADE --> HYPERTHERMIA_39PLUS : > 39.0°C: Sốc nhiệt -> Choáng váng, Kiệt sức, Tụt máu

        EVALUATE_17_BODYPARTS --> CATCH_COLD_CHECK : Ướt (Wetness) + Lạnh -> CatchAColdDelta >= 100%\n-> Ho & Hắt hơi phát sóng âm 15-20 ô (Dùng Handkerchief/Tissue ở ô phụ để bịt miệng)
    }

    state "3. Tiến trình Bào mòn Tự nhiên 100 Ngày (ErosionMain Engine)" as ErosionPhase {
        CLIMATE_MANAGER_TICK --> EROSION_PROGRESSION : Tiến độ theo ErosionSpeed (20 / 50 / 100 / 200 / 500 ngày)\n- Ngày 0-20: Cỏ non & Rêu\n- Ngày 20-50: Cỏ cao ngang hông che khuất Crawlers (Cắt bằng Machete/Scythe)\n- Ngày 50-80: Dây leo phủ kín tường (Leo được lên tầng 2) & Nứt đường\n- Ngày 80-100+: Cây mọc giữa lòng đường (Xe đâm tốc độ cao -> Vỡ lốc máy, Nát két nước, Nổ lốp)
    }

```

---


---

# DOCUMENT 10: SANDBOX CUSTOMIZATION & RULES BIBLE (5 PRESETS, POPULATION DIRECTOR, 21 LOOT RARITY & 35-SKILL MULTIPLIERCONFIG)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Điều Phối Máy Chủ Sandbox, Tiến Hóa Quần Thể Voronoi & Nhân Hệ Số Toàn Cục (`Sandbox Director & Runtime Hot-Reload Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> LOAD_SANDBOX_VARS : Khởi tạo Server / Đọc <ServerName>_SandboxVars.lua (Version = 6)

    state "1. Khởi tạo Tham số Thế giới & 5 Preset (World Rules Initialization)" as InitRules {
        LOAD_SANDBOX_VARS --> APPLY_TIME_AND_UTILITIES : Thiết lập DayLength (Def: 1h | Opt: 2h)\nKhóa mốc WaterShut & ElecShut (Def: 0-30 ngày, TB 14 ngày | SixMonthsLater: -1)\nGeneratorFuelConsumption = 0.1 L/h (Opt: 0.05 L/h chạy 10-14 ngày)
        APPLY_TIME_AND_UTILITIES --> APPLY_ZOMBIE_LORE : Thiết lập Speed (1:Sprinters, 2:FastShamblers, 3:Shamblers)\nTransmission (1:Blood+Saliva 7/25/100% | 2:Saliva Only 0/0/100%)\nCognition (1:Biết mở cửa), ActiveTime (2:Night Only), Mortality (6: 2-3 ngày)
    }

    state "2. Động cơ Tiến hóa Quần thể Voronoi & Hồi sinh 72h (Population Director Tick)" as PopDirector {
        APPLY_ZOMBIE_LORE --> DAILY_POP_CURVE : Mỗi ngày cập nhật M_day(d) từ Start(1.0x) -> Peak(1.5x) tại PeakDay(28)\nPhân bổ cụm theo ZombieVoronoiNoise = true & UrbanFocused = 1
        DAILY_POP_CURVE --> REDISTRIBUTE_12H : Mỗi 12.0 giờ (RedistributeHours): Dàn đều các bầy (RallyGroupSize = 20,\nTravel = 20 ô) lấp khoảng trống & Truy âm thanh FollowSoundDistance (100..1000 ô)
        REDISTRIBUTE_12H --> RESPAWN_72H_CHECK : Mỗi 72.0 giờ (RespawnHours): Kiểm tra ô bản đồ\nNếu HoursUnseenByPlayers >= 16.0 giờ -> Hồi sinh 10% (RespawnMultiplier = 0.1)\n(Nếu RespawnHours = 0 -> Khu vực được dọn sạch vĩnh viễn)
    }

    state "3. Bộ Nhân 21 Kênh Loot, 35 Kỹ Năng & Lệnh Admin Nóng (Economy & Hot-Reload)" as EconomyAdmin {
        LOAD_SANDBOX_VARS --> LOOT_21_MODIFIER : Khi mở thùng đồ (Doc 05) -> Nhân xác suất với 1 trong 21 biến Loot\n(Insane 0.05 | Extreme 0.2 | Rare 0.6 | Normal 1.0 | Common 2.0 | Abundant 3.0)
        LOAD_SANDBOX_VARS --> XP_35_MODIFIER : Khi nhận XP (Doc 03/04/07) -> Nhân với XPMultiplier * MultiplierConfig[Skill]\n(Strength/Fitness x5.0 để giảm thời gian cày 1 năm; Blacksmith/Pottery/FlintKnapping x3.0)
        XP_35_MODIFIER --> ADMIN_HOT_RELOAD : Admin gõ lệnh /reloadoptions hoặc /createhorde, /startrain\n-> Cập nhật nóng SandboxVars ngay trong Runtime không cần tắt Server
    }

```

---


---

# DOCUMENT 11: COMBAT, BALLISTICS & STEALTH BIBLE (6 MELEE CLASSES, 22 FIREARMS, BERNOULLI DECAY & JAW STAB DYNAMICS)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Máy Trạng Thái Tác Chiến Cận Chiến, Suy Giảm Độ Bền & Đòn Kết Liễu Dẫm Đầu (`Melee Combat & Durability Tick Pipeline`)

```mermaid
flowchart TD
    AttackInput["Nhấp Chuột Trái Tấn Công (Melee Swing)"] --> CheckRange{"Mục Tiêu Nằm Trong Tầm Đánh (Range)?"}
    
    CheckRange -- "Trượt (Out of Range)" --> MissAction["Vung Hụt: Tiêu Hao Thể Lực Chuẩn<br/>Không Mất Độ Bền Vũ Khí"]
    CheckRange -- "Trúng Đích" --> CheckState{"Trạng Thái Zombie?"}
    
    CheckState -- "Đang Đứng" --> CalcDamage["Tính Sát Thương = Base * M_Str * M_Skill * M_Moodle"]
    CheckState -- "Ngã Trên Đất" --> ApplyProneBonus["Nhân Hệ Số Vị Thế x 1.5 Sát Thương"]
    CheckState -- "Dẫm Đầu (Head Stomp)" --> InstantLethal["Nhân Hệ Số x 2.75 Sát Thương<br/>Tiêu Diệt Tức Thì / Không Tốn Độ Bền Vũ Khí"]
    
    ApplyProneBonus --> CalcDamage
    CalcDamage --> CheckCrit{"Tung Xúc Xắc Bạo Kích (P_crit - PanicPenalty)?"}
    
    CheckCrit -- "Trúng Bạo Kích" --> ApplyCritMultiplier["Nhân Hệ Số Crit (x 2.0 - x 6.0)"]
    CheckCrit -- "Không Bạo Kích" --> ApplyNormalDamage["Gây Sát Thương Lên Máu Zombie"]
    
    ApplyCritMultiplier --> ApplyNormalDamage
    ApplyNormalDamage --> RollDecay{"Tung Xúc Xắc Bernoulli Mất Độ Bền<br/>P = 1 / (BaseChance + Maint*2 + Skill/2)"}
    
    RollDecay -- "Không Mất" --> MaintainDurability["Bảo Toàn Độ Bền Vũ Khí"]
    RollDecay -- "Mất 1 Độ Bền" --> LowerCondition["Condition = Condition - 1"]
    
    LowerCondition --> CheckBreak{"Condition <= 0?"}
    CheckBreak -- "Vũ Khí Gãy" --> WeaponDestroyed["Phát Âm Thanh Vũ Khí Gãy<br/>Rơi Mảnh Vụn / Chuyển Vũ Khí Phụ"]
    CheckBreak -- "Còn Dùng Được" --> EndMelee["Hoàn Tất Đòn Đánh"]
    MaintainDurability --> EndMelee
    InstantLethal --> EndMelee
```

---

### 2. Luồng Xử Lý Đạn Đạo Súng Đạn, Kẹt Đạn & Phát Tán Sóng Âm 200 Ô (`Firearms Ballistics, Jamming & Acoustic Event Pipeline`)

```mermaid
flowchart TD
    FireInput["Bóp Cò Bắn Súng (Fire Trigger)"] --> CheckChamber{"Đạn Có Sẵn Trong Buồng Đạn?"}
    
    CheckChamber -- "Hết Đạn" --> DryFire["Tiếng Click Cơ Học Khô Khốc<br/>Yêu Cầu Nạp Băng Đạn Mới (Phím R)"]
    CheckChamber -- "Có Đạn" --> CheckJam{"Kiểm Tra Tỷ Lệ Kẹt Đạn (JamRate 1-2%)?"}
    
    CheckJam -- "Bị Kẹt Đạn (Jammed)" --> GunJammed["Súng Bị Kẹt Đạn!<br/>Khóa Cò / Cần Bấm Phím X (Rack Slide) Xử Lý"]
    CheckJam -- "Bắn Thành Công" --> EmitAcousticWave["Phát Tán Sóng Âm Tiếng Nổ Lưới Ô<br/>(Revolver .38: 60 ô | JS-2000 Shotgun: 200 ô)"]
    
    EmitAcousticWave --> AlertZombies["Đánh Thức Toàn Bộ Zombie Trong Bán Kính<br/>Chuyển AI Sang Trạng Thái Truy Đuổi (State: Hunt)"]
    EmitAcousticWave --> CalcAccuracy["Tính Tỷ Lệ Trúng = BaseHit + Aiming*7% - Dist - Move"]
    
    CalcAccuracy --> ApplyPanicPenalty["Giảm Tỷ Lệ Trúng Theo Mức Độ Panic"]
    ApplyPanicPenalty --> ResolveShot{"Xúc Xắc Bắn Trúng?"}
    
    ResolveShot -- "Bắn Trượt" --> BulletWhiz["Đạn Bay Xuyên Không Gian Gây Tiếng Rít"]
    ResolveShot -- "Bắn Trúng" --> ApplyBallisticDamage["Gây Sát Thương Đạn Đạo (1.0 - 2.2)"]
    
    ApplyBallisticDamage --> CheckShotgunSpread{"Súng Shotgun Bắn Chùm?"}
    CheckShotgunSpread -- "Shotgun" --> SpreadTo3Zombies["Gây Sát Thương Đồng Thời 3-4 Zombie<br/>Mỗi Con Trúng Đạn Nhận Điểm XP Aiming Độc Lập"]
    CheckShotgunSpread -- "Súng Đơn" --> CheckPiercing{"Aiming >= 8 & Đạn .308 Win?"}
    
    CheckPiercing -- "Đạn Xuyên" --> PierceSecondTarget["Viên Đạn Xuyên Qua Đầu Zombie Thứ 2"]
    CheckPiercing -- "Không Xuyên" --> EndShot["Hoàn Tất Phát Bắn"]
    SpreadTo3Zombies --> EndShot
    PierceSecondTarget --> EndShot
```

---


---

# DOCUMENT 12: BASE BUILDING, FORTIFICATIONS & ARCHITECTURE BIBLE (32-FLOOR DEPTH, DUAL BARRICADES, 3X3 PLUMBING & GENERATOR GRIDS)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Xây Dựng 32 Tầng, Gia Cố 2 Mặt, Lọc Nước $3\times 3$ ($Z+1$) & Giám Sát An Toàn Máy Phát (`Architecture, Plumbing & Generator Grid Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> FORTIFICATION_PLANNING : Quy hoạch Pháo đài (Giới hạn độ cao 32 Tầng + 362 Hầm ngầm giữ tươi rau củ +50%)

    state "1. Pipeline Thi công Kiến trúc & Gia cố 2 Mặt (ISBuildUtil & ISBarricadeAction)" as BuildPhase {
        FORTIFICATION_PLANNING --> CONSTRUCT_PERIMETER : Tính HP = BaseHP + (Carpentry * 50) + [100 nếu có Handy]\n- Vòng ngoài: Log Wall (400 -> 1.000 HP, 4 gỗ + 2 dây, không tốn đinh) hoặc Metal Pole Fence (500 -> 1.200 HP)\n- Cổng xe: Double Airlock Gate (2 lớp cổng đôi rộng 4 ô, 400-900 HP, cách nhau 6m)\n- Tường nhà: Frame -> Wall Lv3 (300-900 HP) -> Trát vữa Plaster -> Lăn sơn Paint
        
        CONSTRUCT_PERIMETER --> DUAL_SIDE_BARRICADE : Gia cố Cửa sổ / Cửa chính (Mặt trong + Mặt ngoài):\n- Metal Bars (3.000 HP/mặt = 6.000 HP): Nhìn & Đâm giáo/Bắn súng xuyên qua\n- Sheet Metal (5.000 HP/mặt = 10.000 HP): Chặn 100% ánh sáng & tầm nhìn\n- Wooden Planks (1.000 HP/tấm, 4 tấm/mặt = 8.000 HP): 1-2 tấm bắn qua được, 3-4 tấm kín mít
        
        DUAL_SIDE_BARRICADE --> VERTICAL_ISOLATION_META : Đắp dốc (ISBuildRampCursor) lái xe lên sân thượng + Rải đất trồng rau (ISNaturalFloor)\nDùng Sledgehammer phá cầu thang tầng 1 -> Leo lên tầng 2 bằng Dây vải (Sheet Rope)
    }

    state "2. Pipeline Nối ống Tự lọc Nước mưa (ISPlumbItem 3x3 Z+1 Verification)" as PlumbPhase {
        FORTIFICATION_PLANNING --> CHECK_PLUMB_CONDITIONS : Người chơi cầm Pipe Wrench chuột phải vào Bồn rửa (Sink)
        CHECK_PLUMB_CONDITIONS --> VERIFY_ENCLOSED_AND_ZPLUS1 : Kiểm tra 4 điều kiện bắt buộc:\n1. Có Pipe Wrench\n2. Phòng khép kín hoàn chỉnh (square:getRoom() ~= nil, kín tường & trần)\n3. Có Rain Barrel (160L ở Carp 4 / 400L ở Carp 7) tại tầng Z+1 trong lưới 3x3 ô quanh bồn rửa\n4. Trong thùng đang có >= 1 đơn vị nước
        VERIFY_ENCLOSED_AND_ZPLUS1 --> AUTO_PURIFY_ACTIVE : Hợp lệ -> Kích hoạt [Plumb Sink]\nNước mưa bẩn (Tainted Water) chảy qua vòi tự động lọc sạch 100% vi khuẩn không cần đun sôi!
    }

    state "3. Pipeline Điện trường Máy phát 20-Tile & Cảnh báo Tử thần (MOGenerator Tick)" as GenPhase {
        FORTIFICATION_PLANNING --> GENERATOR_RUNNING_TICK : Bật Máy phát (Bình 100% = 10-12L = 2.5 can xăng | Tiêu hao 0.1 L/h = 10-14 ngày)\nPhủ sóng điện trường: Bán kính ngang 20 ô gạch x Chiều cao 3 tầng (Z-1, Z, Z+1)
        
        GENERATOR_RUNNING_TICK --> CHECK_INDOOR_CO_POISON : Kiểm tra vị trí đặt máy phát
        CHECK_INDOOR_CO_POISON --> LETHAL_CO_ASPHYXIATION : Nếu đặt TRONG NHÀ KÍN hoặc TẦNG HẦM (getRoom() ~= nil)\n-> Khí độc CO rút máu liên tục -> Ngủ say là chết ngạt không có âm thanh cảnh báo!
        
        GENERATOR_RUNNING_TICK --> CHECK_CONDITION_20PCT : Hao mòn độ bền mỗi giờ (Bảo trì bằng Electronics Scrap khi < 50%)
        CHECK_CONDITION_20PCT --> CATASTROPHIC_FIRE_EXPLOSION : Nếu Condition < 20% -> Xì khói đen & PHÁT NỔ TỨC THÌ\nThiêu rụi toàn bộ căn cứ!
    }

```

---


---

# DOCUMENT 13: MEDICAL, SURGERY & HEALTH BIBLE (17 BODY PARTS, 6 WOUND KINETICS, DUAL INFECTION & SURGERY EQUATIONS)

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

# DOCUMENT 14: META SECRETS, EMERGENT STORYTELLING & WORLD MYSTERIES BIBLE (80KG METABOLISM, 7 WEIGHT TIERS, STASHSYSTEM & 164 STORIES)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Trao Đổi Chất 80.0 kg, Cạy Ván Sàn Kho Báu & Đạo Diễn Vô Hình 164 Kịch Bản (`Metabolism 80kg, StashSystem & Meta Director Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> WORLD_AND_PLAYER_RUNTIME : Khởi chạy Động cơ Sinh học & Đạo diễn Thế giới B42

    state "1. Động cơ Trao đổi chất 80.0 kg & 7 Ngưỡng Cân nặng (Nutrition.java Tick)" as NutritionPhase {
        WORLD_AND_PLAYER_RUNTIME --> COMPUTE_CALORIC_BURN : caloriesDelta = BodyWeightKg / 80.0 (100kg: +25% | 60kg: -25%)\nTrừ Calories [-2200..+3700 kcal] theo hành động:\n- Sleeping: 0.003/tick | Normal: 0.016/tick (5.3x) | Exercise/Combat: 0.130/tick (43.3x!)
        
        COMPUTE_CALORIC_BURN --> WEIGHT_DELTA_KINETICS : Kiểm tra ngưỡng Đốt mỡ & Tăng cân nhanh:\n- Nếu Calories == -2200 kcal -> Đốt mỡ tối đa -8.5E-6 kg/tick\n- Nếu Calories > 0 & (Carbs > 400g HOẶC Lipids > 400g) -> Tăng cân mũi tên đôi (^^) +1.3E-5 kg/tick\n- Nếu Proteins nằm trong [50.0g, 300.0g] -> Thưởng +30% Strength XP Gain
        
        WEIGHT_DELTA_KINETICS --> UPDATE_7_WEIGHT_BANDS : Áp dụng Trait động theo Cân nặng:\n- <= 35.0 kg: TỬ VONG\n- 35.1 - 50.0 kg: Emaciated (-60% Melee Dmg, -4 Fit, Tụt máu)\n- 50.1 - 65.0 kg: Very Underweight (-40% Melee Dmg, -2 Fit, Dễ ngã rào)\n- 65.1 - 75.0 kg: Underweight (-20% Melee Dmg, -1 Fit)\n- 75.1 - 84.9 kg: VÙNG HOÀN HẢO (0% Penalty, 100% Hiệu suất)\n- 85.0 - 99.9 kg: Overweight (-1 Fit, +50% Mất thể lực)\n- >= 100.0 kg: Obese (-2 Fit, +200% Nguy cơ gãy chân khi nhảy lầu)
    }

    state "2. Động cơ 164 Kịch bản Ngẫu nhiên & Kho báu Ván sàn (RandomizedWorld & StashSystem)" as StashPhase {
        WORLD_AND_PLAYER_RUNTIME --> ROLL_164_STORIES : Khởi tạo Chunk -> Gieo 1 trong 164 kịch bản (RDS / RB / RZS / RVS):\nRDSHockeyPsycho (Machete), RDSRatKing (B42), RBToiletPaper (100%),\nRZSOccultActivity (Rừng sâu), RVSAnimalTrailerOnRoad (Lật xe gia súc sống B42)
        
        ROLL_164_STORIES --> LOOT_ANNOTATED_STASH_MAP : Tiêu diệt Zombie đặc biệt (spawnOnlyOnZed = true) -> Nhặt Annotated Map
        LOOT_ANNOTATED_STASH_MAP --> RESOLVE_STASH_TARGET : Đi tới tọa độ đánh dấu 'X':\n- Nếu trapped == true & barricades == true -> Chạm trán ổ phục kích 10 - 30 Zombie!\n- Dùng Xà beng (Crowbar) cạy tấm ván sàn (floors_interior_tilesandwood_01_60) -> Mở GunBox
    }

    state "3. Đạo diễn Âm thanh Ảo & Trực thăng Săn lùng (SLSoundManager & Helicopter AI)" as MetaDirectorPhase {
        WORLD_AND_PLAYER_RUNTIME --> EMIT_AMBIENT_META_SOUNDS : SLSoundManager kích hoạt ngẫu nhiên:\nMeta Gunfire (Bán kính 200 - 400 ô) | Meta Scream | Barking Dog -> Điều hướng bầy đàn di cư
        
        EMIT_AMBIENT_META_SOUNDS --> HELICOPTER_HUNT_DAY_6_9 : Ngày thứ 6 - 9 (Được báo trước bởi AEBS "Air Activity detected."):\nPhát sóng âm cực đại > 500 ô gạch!\n- Nếu người chơi ở NGOÀI TRỜI: Lơ lửng bám trên đầu 3 - 6 giờ liên tục gom hàng ngàn Zombie\n- Nếu trốn TRONG NHÀ KÍN (Đóng rèm, tắt đèn): Mất dấu và bay đi sau 1 giờ
    }

```

---


# DOCUMENT 15: AI PATHFINDING, DUAL-LAYER NAVMESH & HORDE SIMULATION BIBLE (64-LEVEL Z-AXIS, SSFA FUNNEL & 50,000 VIRTUAL HORDE)

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Điều Hướng Đa Giác 64 Tầng, Phễu Tam Giác SSFA & Di Cư Đàn Ảo 50.000 Thực Thể (`Dual-Layer Pathfinding, Animal NavMesh & Virtual Horde Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> ENTITY_NAVIGATION_REQUEST : Thực thể (Zombie / Animal) yêu cầu đường đi tới Target (X, Y, Z_game)

    state "1. Ánh xạ 64 Tầng & Tìm đường 2 Lớp (PolygonalMap2: HLAStar + VGAStar)" as ZombieNavPhase {
        ENTITY_NAVIGATION_REQUEST --> Z_AXIS_64_MAPPING : Chuyển đổi trục đứng: TargetZ_Internal = TargetZ_Game + 32.0f\n(LEVELS_PER_CHUNK = 64 | GROUND_LEVEL = 32 | Hầm -32 -> 0 | Tháp +31 -> 63)
        
        Z_AXIS_64_MAPPING --> MACRO_HLASTAR_CHECK : Nếu khoảng cách > 1 chunk (> 10..50 tiles) -> Chạy HLAStar.java\nTìm chuỗi HLChunkRegion & Điểm chuyển tầng HLLevelTransition (6 nút Stairs: NODE_FLAG_STAIR = 16)
        
        MACRO_HLASTAR_CHECK --> MICRO_VGASTAR_NATIVE : Đẩy xuống VisibilityGraph & PathfindNative (JNI C++ Đa luồng)\nCắt tia Liang-Barsky + Bám tường (CLOSE_TO_WALLS = true)\nĐệm viên nang RADIUS = 0.3f (Đường chéo RADIUS_DIAGONAL = 0.42426f)
        
        MICRO_VGASTAR_NATIVE --> OBSTACLE_AND_STUCK_FSM : Tương tác vật cản trên đường đi (PathFindBehavior2):\n- Gầm xe ô tô (PATHS_UNDER_VEHICLES = true): Bật NODE_FLAG_CRAWL = 1 bò qua gầm xe\n- Cửa khóa (!couldBeOpen): Bật CXN_FLAG_THUMP = 2 -> Vây đập sập cửa\n- Cửa sổ / Rào thấp: Trèo qua (Nếu còn kính vỡ !isGlassRemoved -> Cứa rách thịt; Nếu đông -> Fence Lunge)\n- Cảm biến chống kẹt (WalkingOnTheSpot): Đứng yên nhiều nhịp -> Hủy đường (isCancel = true) & Đập vật cản
    }

    state "2. Lưới Tam giác Động vật & Thuật toán Phễu Kéo Căng Dây (AnimalPathfind SSFA)" as AnimalNavPhase {
        ENTITY_NAVIGATION_REQUEST --> TRIANGULATED_MESH_A_STAR : Động vật trên đồng cỏ (Zone) -> Chia lưới tam giác (Mesh.java)\nChạy LowLevelAStar tìm chuỗi Cạnh Cổng (Portal Edges: portalLeft, portalRight)
        TRIANGULATED_MESH_A_STAR --> FUNNEL_STRING_PULLING : Chạy thuật toán Phễu (SSFA) với tích có hướng triarea2(Apex, Left, Right)\nKẹp góc phễu & Chốt điểm uốn -> Xuất quỹ đạo đường thẳng kéo căng mượt mà\nKết hợp OffMeshConnection (Hươu nhảy rào thấp, Thỏ nhảy mương, Bò qua cổng chuồng)
    }

    state "3. Ảo hóa Quần thể 50.000 Zombie & Ngưỡng Sóng âm 50-Tile (ZombiePopulationManager)" as PopManPhase {
        [*] --> CHUNK_DISTANCE_MONITOR : Giám sát khoảng cách giữa Người chơi và toàn bộ 50.000+ Zombie
        CHUNK_DISTANCE_MONITOR --> VIRTUALIZE_TO_CPP_32BIT : Nếu khoảng cách > 3 Chunks -> Xóa IsoZombie khỏi RAM Java (removeFromWorld)\nNén thành Struct nhị phân 32-bit trong C++ (Chỉ giữ 200 - 500 Real Zombies quanh người chơi)
        
        VIRTUALIZE_TO_CPP_32BIT --> WORLD_SOUND_FILTER_50 : Phát sinh âm thanh thế giới (addWorldSound):\n- Nếu sound.radius < 50 HOẶC sourceIsZombie HOẶC !stressZombies -> BỎ QUA (Đàn ảo đứng yên)\n- Nếu sound.radius >= 50 (VD: Súng .38 = 60 ô, Shotgun = 200 ô, Trực thăng > 500 ô)\n-> Gọi Native n_worldSound() -> Hàng ngàn Virtual Zombies di cư ngầm về nguồn âm!
    }

```

---
