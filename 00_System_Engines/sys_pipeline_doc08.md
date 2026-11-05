---
id: "sys_pipeline:doc08"
type: "system_tick_pipeline_spec"
document_source: "Doc 08: Project Zomboid B42 Farming, Trapping, Fishing & Wilderness Survival Handbook (43 Crops/Herbs, 6 Traps, 21 Fish, 18 Genes & 27 Forage Categories)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[02_Bestiary_And_AI]]",
  "[[doc_srd_05_animals_mounts_swarms]]",
  "[[sys_math_doc08]]"
]
state_machines: [
  "5-Pillar Closed Loop Wilderness Survival FSM",
  "Crop Growth, Water Decay & Pest Infestation Pipeline",
  "Livestock Breeding, 18-Gene Inheritance & Butchering Chain"
]
tick_rate_hz: 60
---

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

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực, Thân phận & Y học Nam Dược: [[04_Character_And_Med]]
- Quái thú & Hệ sinh thái hoang dã: [[02_Bestiary_And_AI]], [[doc_srd_05_animals_mounts_swarms]]
- Phương trình toán học đi kèm: [[sys_math_doc08]]
