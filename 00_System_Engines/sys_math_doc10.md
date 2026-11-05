---
id: "sys_math:doc10"
type: "system_mathematics_spec"
document_source: "Doc 10: Project Zomboid B42 Sandbox Customization & Rules Bible (5 Presets, Population Director, 21 Loot Rarity & 35-Skill MultiplierConfig)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[doc_dmg_01_survival_box]]",
  "[[sys_pipeline_doc10]]"
]
key_equations: [
  "Population Peak & Unseen Respawn Curve (Start 1.0x -> Peak 1.5x Day 28)",
  "Granular 21-Category Loot Scaling Multipliers (0.05 to 3.0)",
  "Per-Skill 35-Channel MultiplierConfig XP Pipeline"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Tiến hóa Mật độ Quần thể Thực thể theo Ngày & Hồi sinh Ô Bản đồ (`PopulationMultiplier` & Cell Respawn Curve)



Tại ngày sinh tồn thứ $d \ge 0$, mật độ quần thể mục tiêu $P_{\text{target}}(d)$ trong một ô bản đồ (Cell) có mật độ gốc $P_{\text{base}}(\text{Cell}, \text{VoronoiNoise})$ được nội suy tuyến tính từ Hệ số Ngày đầu ($M_{\text{start}} = 1.0$) đến Hệ số Đỉnh điểm ($M_{\text{peak}} = 1.5$) tại Ngày Đỉnh điểm ($D_{\text{peak}} = 28$), nhân với Hệ số Quần thể Toàn cục ($M_{\text{pop}} = 1.0$):

$$M_{\text{day}}(d) = \begin{cases} M_{\text{start}} + \left(\frac{d}{D_{\text{peak}}}\right) \times (M_{\text{peak}} - M_{\text{start}}) & \text{nếu } 0 \le d < D_{\text{peak}} \\ M_{\text{peak}} & \text{nếu } d \ge D_{\text{peak}} \end{cases}$$

$$P_{\text{target}}(\text{Cell}, d) = P_{\text{base}}(\text{Cell}) \times M_{\text{pop}} \times M_{\text{day}}(d)$$

Khi số lượng thực thể hiện tại $P_{\text{current}}(\text{Cell}) < P_{\text{target}}(\text{Cell}, d)$, lượng thực thể hồi sinh $\Delta P_{\text{respawn}}$ sau mỗi chu kỳ $T_{\text{respawn}} = 72.0\text{ giờ}$ chỉ được kích hoạt nếu thời gian ô gạch không bị người chơi nhìn thấy hoặc đi qua ($T_{\text{unseen}}$) thỏa mãn $T_{\text{unseen}} \ge 16.0\text{ giờ}$:

$$\Delta P_{\text{respawn}}(\text{Cell}) = \begin{cases} \min\Big( P_{\text{target}} - P_{\text{current}}, \, P_{\text{target}} \times M_{\text{respawn}} \Big) & \text{nếu } \Delta t \ge T_{\text{respawn}} \land T_{\text{unseen}} \ge 16.0\text{h} \\ 0 & \text{trường hợp khác} \end{cases}$$

(Trong đó mặc định $M_{\text{respawn}} = 0.10$ tương đương $10\%$ quần thể mục tiêu mỗi $72.0\text{ giờ}$, và cứ mỗi $T_{\text{redist}} = 12.0\text{ giờ}$ các thực thể sẽ tự động di cư lấp vào khoảng trống; tụ thành bầy `RallyGroupSize` $= 20$ con với bán kính bám đàn `RallyTravelDistance` $= 20$ ô và truy vết âm thanh `FollowSoundDistance` $= 100$ ô [tối đa $1000$ ô]).

```text
FUNCTION Update_Cell_Population_And_Respawn(cell, current_day, elapsed_hours):
    // 1. Tính hệ số tiến hóa quần thể theo ngày (Peak Day = 28)
    IF current_day < SandboxVars.PopulationPeakDay THEN
        progress = current_day / SandboxVars.PopulationPeakDay
        day_mult = SandboxVars.PopulationStartMultiplier + progress * (SandboxVars.PopulationPeakMultiplier - SandboxVars.PopulationStartMultiplier)
    ELSE
        day_mult = SandboxVars.PopulationPeakMultiplier
    END IF

    target_pop = cell.BaseVoronoiPop * SandboxVars.PopulationMultiplier * day_mult

    // 2. Kiểm tra điều kiện Hồi sinh (RespawnHours = 72.0, RespawnUnseenHours = 16.0)
    IF SandboxVars.RespawnHours > 0.0 AND cell.HoursSinceLastRespawn >= SandboxVars.RespawnHours THEN
        IF cell.HoursUnseenByPlayers >= SandboxVars.RespawnUnseenHours THEN
            spawn_quota = target_pop * SandboxVars.RespawnMultiplier // Mặc định 0.1 (10%)
            actual_spawn = Min(target_pop - cell.CurrentPop, spawn_quota)
            IF actual_spawn > 0 THEN
                SpawnVirtualEntitiesInCell(cell, actual_spawn)
            END IF
            cell.HoursSinceLastRespawn = 0.0
        END IF
    END IF

    // 3. Tái phân bổ lấp khoảng trống mỗi 12.0 giờ (RedistributeHours)
    IF cell.HoursSinceLastRedistribute >= SandboxVars.RedistributeHours THEN
        RedistributeEntitiesToEmptyChunks(cell, SandboxVars.RallyGroupSize, SandboxVars.RallyTravelDistance)
        cell.HoursSinceLastRedistribute = 0.0
    END IF
END FUNCTION

```

#### 1.2. Phương trình Điều biến Xác suất Rơi Đồ 21 Danh mục (`21-Category Granular Loot Factor`)



Kết hợp với phương trình `ProceduralDistributions` (từ Doc 05), xác suất rơi thực tế của một vật phẩm $i_k$ thuộc danh mục chiến lợi phẩm $C_m \in \text{LootCategories}_{21}$ được nhân trực tiếp với Hệ số Độ hiếm Sandbox $F_{\text{loot}}(C_m) \in \{0.05, 0.2, 0.6, 1.0, 2.0, 3.0\}$:

$$P_{\text{final}}(i_k \mid T_{\text{table}}, C_m) = \text{rolls}(T_{\text{table}}) \times \left( \frac{W(i_k)}{\sum_{j} W(i_j)} \right) \times F_{\text{loot}}(C_m)$$

#### 1.3. Phương trình Tổng hợp Kinh nghiệm Đa kênh 35 Kỹ năng (`MultiplierConfig` & `XPMultiplier`)



Mở rộng phương trình kinh nghiệm từ Doc 04 và Doc 07, lượng XP cuối cùng $\Delta \text{XP}_{\text{total}}(k)$ của kỹ năng $k \in \text{Skills}_{35}$ là tích của 5 thành phần độc lập:

$$\Delta \text{XP}_{\text{total}}(k) = \Delta \text{XP}_{\text{raw}}(k) \times M_{\text{init}}(L_0(k)) \times M_{\text{book}}(L(k)) \times \left(1 + 0.30 \cdot \mathbb{I}_{\text{FastLearner}}\right) \times \text{XPMultiplier}_{\text{global}} \times \text{MultiplierConfig}(k)$$

---

### 2. Bảng Hằng Số Tham Số Máy Chủ Toàn Cục & Preset Sandbox (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Sandbox Schema** | `SandboxVars_Version` | `6` | `version_id` | Phiên bản cấu trúc tệp `SandboxVars.lua` của Build 42.

 |
| **Sandbox Schema** | `Granular_Loot_Categories` | `21` | `categories` | 21 biến số độ hiếm chiến lợi phẩm tách biệt độc lập trong Build 42.

 |
| **Sandbox Schema** | `Individual_XP_Config_Skills` | `35` | `skills` | Tổng số kỹ năng có thể cấu hình hệ số nhân XP riêng trong `MultiplierConfig`.

 |
| **Loot Factors** | `Insane_To_Abundant_Factors` | `0.05 | 0.2 | 0.6 | 1.0 | 2.0 | 3.0` | `float_mult` | 6 bậc hệ số rơi đồ: Insane ($5\%$), Extreme ($20\%$), Rare ($60\%$), Normal ($100\%$), Common ($200\%$), Abundant ($300\%$).

 |
| **Preset Matrix** | `Apocalypse_Preset_Core` | `Zombies:4 | MultiHit:OFF | SkillBook:0.6 | Ranged/Ammo: 1.2/0.6` | `preset_config` | Cấu hình chuẩn Hardcore: không đánh lan, sách kỹ năng hiếm ($0.6$), cắt điện nước $0\text{--}30\text{ ngày}$ (TB $14\text{ ngày}$).

 |
| **Preset Matrix** | `Extinction_Preset_Core` | `Zombies:3 | Respawn:2 | SkillBook:0.4 | Ranged/Ammo: 0.6/0.2` | `preset_config` | Cấu hình Tuyệt chủng: mật độ rất cao, hồi sinh rất nhanh, đạn dược gần như bằng không ($0.2$).

 |
| **Preset Matrix** | `SixMonthsLater_Preset_Core` | `Zombies:1(Insane, 4x) | Month:12 | Shutoff:-1 | Erosion:MAX` | `preset_config` | Khởi đầu tháng 12 mùa đông, điện nước đã cắt từ trước (`-1`), mật độ gấp $4\times$, bào mòn đạt cực đại.

 |
| **Population Curve** | `Pop_Multipliers_Default` | `Base: 1.0 | Start: 1.0 | Peak: 1.5 | PeakDay: 28` | `mult / day` | Đường cong tăng trưởng quần thể từ $1.0\times$ (Ngày 1) lên $1.5\times$ tại Ngày thứ 28.

 |
| **Respawn Cycle** | `Respawn_Timers_And_Quota` | `Hours: 72.0 | Unseen: 16.0 | Mult: 0.1 (10%)` | `hours / ratio` | Hồi sinh $10\%$ lượng thiếu hụt sau mỗi $72\text{ giờ}$ nếu ô không có người nhìn thấy trong $16\text{ giờ}$.

 |
| **Horde Dynamics** | `Redistribute_And_Rally` | `Redist: 12.0h | FollowSound: 100 (Max 1000) | Group: 20 | Travel: 20` | `hours / tiles / count` | Chu kỳ dàn đều $12\text{h}$, tầm truy âm $100\text{ ô}$ (tối đa $1000$), cỡ bầy $20\text{ con}$, cự ly bám đàn $20\text{ ô}$.

 |
| **Zombie Lore** | `Speed_And_Transmission_Enums` | `Speed: 1..4 (Def 2) | Trans: 1..4 (Def 1)` | `enum_index` | Tốc độ (`1`=Sprinters, `2`=Fast Shamblers, `3`=Shamblers, `4`=Random) và Lây nhiễm (`1`=Máu+Nước bọt, `2`=Chỉ Nước bọt).

 |
| **Zombie Lore** | `InfectionMortality_8_Tiers` | `1:Instant | 2:0-30s | 3:1-2m | 4:1-2h | 5:12-24h | 6:2-3d(Def) | 7:1-2w | 8:Never` | `duration` | 8 bậc thời gian ủ bệnh tử vong từ lúc nhiễm virus đến khi hóa xác sống.

 |
| **Time & Utility** | `DayLength_And_Shutoff_Mod` | `DayLength: 3(1h Def), 5(2h Opt) | ShutMod: 14d (Range 0-30d)` | `enum / days` | Độ dài 1 ngày trong game (`1`=15m, `2`=30m, `3`=1h, `4`=1.5h, `5`=2h... `14`=24h) và mốc cắt điện nước trung bình $14\text{ ngày}$.

 |
| **Generator Math** | `GeneratorFuelConsumption_Def` | `0.1` (Range `0.0 - 10.0`, Opt `0.05` = `10-14d`) | `Liters/hour` | Tốc độ tiêu thụ nhiên liệu máy phát điện mặc định $0.1\text{ L/giờ}$.

 |
| **Vehicle Rules** | `ZombieAttractionMultiplier` | `1.0` (Range `0.0 - 10.0`, at `2.0` $\rightarrow$ `100m` radius) | `multiplier` | Hệ số nhân độ ồn động cơ và ống xả phương tiện thu hút thực thể.

 |
| **Stat Grind Time** | `Vanilla_Strength_5_To_10_Time` | `~1` (`MultiplierConfig` Range `1x - 100x`, Opt `5.0x`) | `in-game year` | Thời gian tập thể dục liên tục để nâng Strength từ Cấp 5 lên Cấp 10 ở hệ số $1.0\times$ gốc.

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Động lực học & Quy tắc hộp công cụ D&D / Sandbox: [[00_System_Engines]], [[doc_dmg_01_survival_box]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc10]]
