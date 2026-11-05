---
id: "sys_math:doc08"
type: "system_mathematics_spec"
document_source: "Doc 08: Project Zomboid B42 Farming, Trapping, Fishing & Wilderness Survival Handbook (43 Crops/Herbs, 6 Traps, 21 Fish, 18 Genes & 27 Forage Categories)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[02_Bestiary_And_AI]]",
  "[[doc_srd_05_animals_mounts_swarms]]",
  "[[sys_pipeline_doc08]]"
]
key_equations: [
  "Foraging Optical Search Radius Equation (R_eff = 3.0 * (1 + 0.25 * Level))",
  "Trapping 75-Tile Distance Exclusion & Nocturnal Catch Matrix",
  "Fishing Peak Hours (04:00-07:00 & 18:00-21:00) & Line Tension Gauge",
  "Agricultural Chessboard Spacing & Animal Husbandry 18-Gene Inheritance"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Bán kính Tầm nhìn Thu lượm Quang học (`forageSystem.lua` Search Mode Radius)

Bán kính vòng tròn tìm kiếm hiệu dụng $R_{\text{eff}}$ (đơn vị: tiles/blocks) xung quanh nhân vật biến thiên từ $3.0\text{ ô}$ (ở Cấp 0) lên tới $10.5\text{ ô}$ (ở Cấp 10), chịu sự điều phối nhân tính của cấp độ kỹ năng $\text{Level} \in [0, 10]$ và 4 nhóm hệ số môi trường/trạng thái:

$$R_{\text{eff}} = R_{\text{base}} \times (1 + \text{Level} \times 0.25) \times M_{\text{light}} \times M_{\text{weather}} \times M_{\text{moodle}} \times M_{\text{gear}}$$

Trong đó $R_{\text{base}} = 3.0\text{ tiles}$ (khi $\text{Level} = 10 \implies 3.0 \times (1 + 10 \times 0.25) = 3.0 \times 3.5 = 10.5\text{ tiles}$) và các hệ số phạt cụ thể:

* $M_{\text{light}} = 0.40$ khi đêm tối không trăng (Phạt $-60\%$).


* $M_{\text{weather}} = 0.50$ khi sương mù dày đặc (Phạt $-50\%$), hoặc $0.70$ khi mưa bão lớn (Phạt $-30\%$).


* $M_{\text{moodle}} \in [0.70, 0.90]$ khi mệt mỏi (Phạt $-10\%$ đến $-30\%$), và $\in [0.60, 0.85]$ khi buồn ngủ (Phạt $-15\%$ đến $-40\%$).


* $M_{\text{gear}} = 0.70$ khi đội mũ bảo hiểm trùm đầu hoặc mặt nạ hàn (Phạt $-30\%$); đeo kính mắt (`Glasses`) khử hoàn toàn phạt của đặc điểm `Short Sighted`.



```text
FUNCTION Calculate_Foraging_Search_Radius(player, environment):
    base_radius = 3.0
    skill_level = player.GetSkillLevel("Foraging") // 0..10
    radius = base_radius * (1.0 + (skill_level * 0.25)) // Max 10.5 tiles at Lv 10

    // 1. Ánh sáng & Thời tiết
    IF environment.IsMoonlessNight AND NOT player.HasActiveFlashlightOrLantern() THEN
        radius = radius * (1.0 - 0.60) // Phạt -60%
    END IF
    IF environment.HasHeavyFog THEN
        radius = radius * (1.0 - 0.50) // Phạt -50%
    ELSE IF environment.HasHeavyRain THEN
        radius = radius * (1.0 - 0.30) // Phạt -30%
    END IF

    // 2. Moodles (Mệt mỏi & Buồn ngủ)
    IF player.IsExhausted THEN
        radius = radius * (1.0 - player.GetExhaustionPenalty(0.10, 0.30))
    END IF
    IF player.IsDrowsy THEN
        radius = radius * (1.0 - player.GetDrowsyPenalty(0.15, 0.40))
    END IF

    // 3. Trang bị đầu & Kính mắt
    IF player.IsWearingFullHelmetOrWeldingMask() THEN
        radius = radius * (1.0 - 0.30) // Phạt -30%
    END IF
    IF player.HasTrait("base:shortsighted") AND NOT player.IsWearingGlasses() THEN
        radius = ApplyShortSightedPenalty(radius)
    END IF

    RETURN radius
END FUNCTION

```

#### 1.2. Phương trình Ràng buộc Khoảng cách Bẫy 75-Tile & Ma trận Bắt Thú Sống (`STrapGlobalObject.lua`)

Một bẫy thú đặt tại tọa độ $\vec{P}_{\text{trap}}$ chỉ được phép kích hoạt kiểm tra bắt mồi nếu khoảng cách Euclid đến mọi người chơi $\vec{P}_{\text{player}}$ thỏa mãn bất đẳng thức tuyệt đối $\ge 75.0\text{ tiles}$ và nằm trong khung giờ hoạt động của loài thú $a$:

$$D_{\text{trap}} = \Vert{}\vec{P}_{\text{player}} - \vec{P}_{\text{trap}}\Vert{}_2 \ge 75.0\text{ tiles}$$

$$\mathbb{I}_{\text{active}}(a, t_{\text{hour}}) = \begin{cases} 1 & \text{nếu } a \in \{\text{Rabbit}, \text{Squirrel}, \text{Raccoon}\} \land t_{\text{hour}} \in \ (\text{Ban đêm}) \\ 1 & \text{nếu } a \in \{\text{Bird}, \text{Mouse}, \text{Rat}\} \land t_{\text{hour}} \in \ (24/24\text{h}) \\ 0 & \text{trường hợp khác} \end{cases}$$

```text
FUNCTION Evaluate_Trap_Tick(trap, nearest_player, current_hour, zone_type):
    distance = EuclideanDistance(trap.position, nearest_player.position)
    
    // Quy tắc 75-Tile bất di bất dịch
    IF distance < 75.0 THEN
        // Nếu người chơi ở gần dưới 75 ô: bẫy không kích hoạt, thú rừng phá bẫy và ăn mất bả
        IF RollBaitStolenOrTrapDamaged(trap.trapStrength) == TRUE THEN
            trap.RemoveBait()
            trap.DestroyIntoBrokenComponents()
        END IF
        RETURN TRAP_BLOCKED_PLAYER_TOO_CLOSE
    END IF

    FOR EACH prey IN GetEligiblePreyForTime(current_hour) DO
        catch_chance = ComputeTrapCatchChance(trap.Type, trap.Bait, zone_type, prey)
        IF RandomFloat(0.0, 100.0) < catch_chance THEN
            IF trap.SupportsLiveCapture == TRUE AND prey.canBeAlive == TRUE THEN
                SpawnLiveCapturedAnimal(trap, prey) // Thỏ, Gấu mèo, Chuột còn sống để thuần hóa
            ELSE
                SpawnDeadPreyItem(trap, prey)
            END IF
            RETURN TRAP_TRIGGERED_SUCCESS
        END IF
    END FOR
END FUNCTION

```

#### 1.3. Phương trình Sinh trưởng Nông nghiệp, Cách ly Bàn cờ & Thủy sinh Câu cá (`SFarmingSystem` & `TensionUI`)

* **Điều kiện Nước Tối thiểu/Tối ưu & Ngưỡng Úng rễ Cà rốt**: Mỗi cây trồng $c$ yêu cầu độ ẩm đất $W_{\text{soil}} \ge W_{\min}(c)$ để sinh trưởng trong thời gian $T_{\text{grow}}(c) \in [144, 480]\text{ giờ}$. Riêng Cà rốt (`Carrot`) có ngưỡng trần chết úng rễ khi $W_{\text{soil}} > 85$. Khi nhiễm Nấm mốc (`Mildew`), năng suất giảm $-50\%$; khi nhiễm Rầy mềm (`Aphids`), cây khô héo trong $48\text{ giờ}$. Khoảng cách cách ly bàn cờ $\Delta_{\text{grid}} \ge 1\text{ ô trống}$ chặn hoàn toàn sự lây lan dịch bệnh giữa các luống.


* **Hệ số Khung Giờ Vàng & Mùa Đông Câu Cá**:

$$P_{\text{bite}}(t_{\text{hour}}, T_{\text{ambient}}) = P_{\text{base}}(\text{Lure}) \times \begin{cases} 2.00 \ (+100\%) & \text{nếu } t_{\text{hour}} \in \cup \\ 1.00 & \text{khung giờ thường} \end{cases} \times \begin{cases} [0.30, 0.50] \ (\text{Giảm } 50\text{--}70\%) & \text{nếu Mùa đông } (T_{\text{ambient}} < 0^\circ\text{C}) \\ 1.00 & \text{mùa ấm} \end{cases}$$

---

### 2. Bảng Hằng Số Nông Nghiệp, Bẫy Thú, Câu Cá, Di Truyền & Thu Lượm (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **System Scope** | `Water_Power_Shutoff_Window` | `14 - 30` | `days (ngày)` | Khoảng thời gian cắt điện và nước máy vĩnh viễn.

 |
| **Farming (43 Crops)** | `Total_Crops_And_Herbs` | `43` (`20` Crops + `23` Herbs) | `plant_types` | Tổng số giống cây trồng và thảo mộc trong Build 42.

 |
| **Farming (Crops)** | `Cabbage_Fastest_Growth` | `Seed: 9 | W: 40-85 | 144h (6.0d) | Y: 4-8` | `seeds/water/h/yield` | Bắp cải — cây lớn nhanh nhất game ($144\text{h}$), gieo Tháng 3–8.

 |
| **Farming (Crops)** | `Potato_Storage_Growth` | `Seed: 4 | W: 45-70 | 288h (12.0d) | Shelf: 30d` | `seeds/water/h/days` | Khoai tây — chịu lạnh (`coldHardy`), tươi lâu tới $30\text{ ngày}$.

 |
| **Farming (Crops)** | `Carrot_Overwater_Death_Cap` | `Seed: 12 | W: 35-80 (Max 85) | 168h (7.0d)` | `seeds/water/h` | Cà rốt — nếu tưới nước $> 85$ sẽ bị thối úng rễ chết.

 |
| **Farming (Crops)** | `Corn_Longest_Growth` | `Seed: 6 | W: 60-90 | 480h (20.0d) | Y: 3-8` | `seeds/water/h/yield` | Ngô — cây có thời gian sinh trưởng dài nhất ($480\text{h} \approx 20\text{ ngày}$).

 |
| **Farming (Herbs)** | `Herbs_Growth_And_Water` | `Time: 160 - 240h (7-10d) | Water: 30 - 60` | `hours / water` | Thông số sinh trưởng của 23 loại thảo mộc vườn nhà.

 |
| **Farming (Herbs)** | `Herbs_Culinary_Mood_Bonus` | `Boredom: -15 | Unhappiness: -10` | `stat_delta` | Hiệu ứng giảm chán nản và u sầu khi nêm thảo mộc vào món hầm.

 |
| **Crop Diseases** | `Mildew_Yield_Penalty_And_Cure` | `-50% Yield | Cure: 50ml Milk + Water` | `% / ml` | Nấm mốc giảm $50\%$ sản lượng; chữa bằng $50\text{ml}$ sữa tươi + nước.

 |
| **Crop Diseases** | `Aphids_Wither_Time_And_Cure` | `48h Wither | Cure: 5 Cigarettes + Water` | `hours / items` | Rầy mềm làm khô héo cây trong $48\text{h}$; chữa bằng $5$ điếu thuốc lá + nước.

 |
| **Crop Diseases** | `Flies_And_Slugs_Cures` | `Flies: 1 Garlic + Water | Slugs: Can + 1 Beer` | `recipe_inputs` | Công thức trị Ruồi đục thân ($1$ củ tỏi) và Bẫy ốc sên ($1$ ngụm bia).

 |
| **Trapping Rules** | `Trap_Minimum_Player_Distance` | `75` | `tiles (blocks)` | Khoảng cách tối thiểu từ người chơi tới bẫy để bẫy hoạt động.

 |
| **Trapping Rules** | `Trap_Strength_6_Types` | `Cage:20 | Crate:15 | Box:15 | Stick:10 | Snare:10 | Mouse:5` | `trapStrength` | Chỉ số độ bền cơ học của 6 loại bẫy thú.

 |
| **Trapping Rules** | `Nocturnal_Trapping_Window` | `19:00 - 05:00` | `24h clock` | Khung giờ săn đêm bắt buộc của Thỏ, Sóc, Gấu mèo (Chim & Chuột: `24/24h`).

 |
| **Trapping Rules** | `Top_Bait_Catch_Rates` | `Carrot(Rabbit): 45% | Worm(Bird): 50% | Cheese(Mouse): 60%` | `% catch_rate` | Tỷ lệ bắt trúng cao nhất của từng loại bả nhử chủ lực.

 |
| **Fishing (21 Fish)** | `Total_Freshwater_Fish_Species` | `21` | `species` | Tổng số loài cá nước ngọt (từ `BaitFish` $10\text{cm}/0.05\text{kg}$ đến `AligatorGar`).

 |
| **Fishing (Trophy)** | `AligatorGar_Max_And_Trophy` | `Normal: 180cm / 45.0kg | Trophy: 243cm / 150kg` | `cm / kg` | Kích thước và cân nặng kỷ lục của thủy quái Cá sấu hỏa tiễn.

 |
| **Fishing (Trophy)** | `Paddlefish_And_BlueCatfish_Trophy` | `Paddlefish: 220cm/90kg | BlueCatfish: 165cm/68kg` | `cm / kg` | Kỷ lục Trophy của Cá thìa sông ($90\text{kg}$) và Cá trê xanh ($68\text{kg}$).

 |
| **Fishing Hours** | `Peak_Dawn_And_Dusk_Bonus` | `04:00-07:00 & 18:00-21:00 -> +100%` | `24h clock / %` | Hai khung giờ vàng tăng $+100\%$ tỷ lệ cá cắn câu.

 |
| **Fishing Winter** | `Winter_SubZero_Bite_Penalty` | `-50 to -70` (khi `< 0°C`) | `% (percentage)` | Mức sụt giảm tỷ lệ cắn câu vào mùa đông đóng băng.

 |
| **Husbandry (Genes)** | `Total_Animal_Genetic_Loci` | `18` | `genes (alleles)` | 18 mã gen di truyền trội/lặn trong `AnimalGenomeDefinitions.lua`.

 |
| **Husbandry (Genes)** | `Default_Gene_MaxSize_Range` | `0.85 - 0.95` | `scale_factor` | Kích thước khung xương 3D mặc định của gen `maxSize`.

 |
| **Husbandry (Genes)** | `High_Gene_Milk_ And_Eggs` | `Milk: 20 - 30 L/day | EggClutch: 1-2 to 6-8` | `liters / eggs` | Sản lượng sữa bò gen cao ($20\text{--}30\text{ lít/ngày}$) và số trứng mỗi ổ.

 |
| **Foraging (27 Cats)** | `Foraging_Radius_Lv0_To_Lv10` | `3.0 -> 10.5` (`+0.25` per level) | `tiles (blocks)` | Bán kính tìm kiếm tăng từ $3.0$ ô (Cấp 0) lên $10.5$ ô (Cấp 10).

 |
| **Foraging (27 Cats)** | `Total_Forage_Categories` | `27` | `categories` | 27 danh mục tài nguyên thu lượm trong `forageCategories.lua`.

 |
| **Foraging (Herbs)** | `Comfrey_And_Poison_Timers` | `Comfrey: 2x Bone Heal | Poison Death: 24h` | `multiplier / hours` | Cỏ liên mộc giúp liền xương nhanh gấp đôi; nấm/quả độc gây chết trong $24\text{h}$.

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực, Thân phận & Y học Nam Dược: [[04_Character_And_Med]]
- Quái thú & Hệ sinh thái hoang dã: [[02_Bestiary_And_AI]], [[doc_srd_05_animals_mounts_swarms]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc08]]
