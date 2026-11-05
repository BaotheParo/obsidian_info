---
id: "sys_pipeline:doc14"
type: "system_tick_pipeline_spec"
document_source: "Doc 14: Project Zomboid B42 Meta Secrets, Emergent Storytelling & World Mysteries Bible (80kg Metabolism, 7 Weight Tiers, StashSystem & 164 Stories)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[sys_metabolism_calories]]",
  "[[doc_prp_01_goals_milestones]]",
  "[[sys_math_doc14]]"
]
state_machines: [
  "Nutrition & 80.0kg Metabolic Energy Expenditure State Machine",
  "Dynamic Weight Trait Lifecycle & Body Transformation FSM",
  "Meta Director Ambient Event & Helicopter Pursuit Pipeline"
]
tick_rate_hz: 60
---

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

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Động lực học & Chuyển hóa calo cốt lõi: [[00_System_Engines]], [[sys_metabolism_calories]]
- Cột mốc & Kể chuyện nổi sinh RPG: [[doc_prp_01_goals_milestones]]
- Phương trình toán học đi kèm: [[sys_math_doc14]]
