---
id: "sys_math:doc14"
type: "system_mathematics_spec"
document_source: "Doc 14: Project Zomboid B42 Meta Secrets, Emergent Storytelling & World Mysteries Bible (80kg Metabolism, 7 Weight Tiers, StashSystem & 164 Stories)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[sys_metabolism_calories]]",
  "[[doc_prp_01_goals_milestones]]",
  "[[sys_pipeline_doc14]]"
]
key_equations: [
  "Metabolic Calorie Derivative & 80.0kg Weight Equilibrium (dW/dt)",
  "7 Dynamic Weight Thresholds & Trait Eradication/Acquisition Triggers",
  "StashSystem Floorboard Cache Probability & Unlocking Mechanics",
  "Meta Director Ambient Virtual Sound Distribution & Helicopter AI"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Trao đổi chất 80.0 kg, Đạo hàm Calo & Biến thiên Cân nặng (`Nutrition.java`)



Mọi phép tính đốt cháy năng lượng cơ bản trong engine đều quy chiếu về mốc cân nặng chuẩn $W_{\text{standard}} = 80.0\text{ kg}$. Hệ số tỷ lệ tiêu hao Calo ($\text{caloriesDelta}$) của một nhân vật có cân nặng thực tế $W_{\text{body}}$ (kg) và tốc độ đốt cháy Calo mỗi tick ($\frac{d(\text{Calories})}{dt}$) theo trạng thái vận động $A \in \{\text{Sleeping}, \text{Normal}, \text{Exercise}\}$ được xác định bởi:

$$\text{caloriesDelta} = \frac{W_{\text{body}}}{80.0}, \quad \text{Calories} \in [-2200.0, \, +3700.0]\text{ kcal}$$



$$\frac{d(\text{Calories})}{dt} = -\text{BurnRate}(A) \times \text{caloriesDelta} \times M_{\text{thermo}}, \quad \text{với } \text{BurnRate}(A) = \begin{cases} 0.003\text{ kcal/tick} & \text{nếu } A = \text{Sleeping} \\ 0.016\text{ kcal/tick} \ (5.3\times) & \text{nếu } A = \text{Normal} \\ 0.130\text{ kcal/tick} \ (43.3\times) & \text{nếu } A = \text{Exercise} \end{cases}$$



Tốc độ biến thiên cân nặng cơ thể mỗi tick ($\frac{dW_{\text{body}}}{dt}$) và hệ số thưởng kinh nghiệm Sức mạnh từ Protein (`Proteins`) tuân theo hệ phương trình:

$$\frac{dW_{\text{body}}}{dt} = \begin{cases} -8.5 \times 10^{-6}\text{ kg/tick} \ (\text{Đốt mỡ tối đa}) & \text{nếu } \text{Calories} = -2200.0\text{ kcal} \\ +1.3 \times 10^{-5}\text{ kg/tick} \ (\text{Mũi tên đôi } \text{\textasciicircum\textasciicircum: } \text{incWeightLot} = \text{true}) & \text{nếu } \text{Calories} > 0 \land (\text{Carbohydrates} > 400.0\text{g} \lor \text{Lipids} > 400.0\text{g}) \end{cases}$$



$$M_{\text{Strength\_XP}}(\text{Proteins}) = \begin{cases} 1.30 \ (+30\% \text{ Strength XP Gain}) & \text{nếu } 50.0\text{g} \le \text{Proteins} \le 300.0\text{g} \\ 1.00 & \text{trường hợp khác} \end{cases}$$



```text
FUNCTION Update_Player_Metabolism_And_Weight_Tick(player, activity_state):
    // 1. Tính hệ số tiêu hao Calo quanh mốc chuẩn 80.0 kg
    calories_delta = player.BodyWeightKg / 80.0
    
    IF activity_state == SLEEPING THEN
        base_burn = 0.003
    ELSE IF activity_state == NORMAL_IDLE_WALK THEN
        base_burn = 0.016 // Gấp 5.3 lần khi ngủ
    ELSE IF activity_state == EXERCISE_SPRINT_COMBAT THEN
        base_burn = 0.130 // Gấp 43.3 lần khi ngủ (8.1 lần đi bộ)
    END IF

    // Cập nhật Calories trong dải giới hạn cứng [-2200.0, +3700.0]
    player.Calories = Clamp(player.Calories - (base_burn * calories_delta), -2200.0, 3700.0)

    // 2. Cập nhật biến thiên cân nặng (Đốt mỡ vs Tăng cân mũi tên đôi ^^)
    IF player.Calories <= -2200.0 THEN
        player.BodyWeightKg = player.BodyWeightKg - 0.0000085 // -8.5E-6 / tick
    ELSE IF player.Calories > 0.0 AND (player.CarbohydratesGrams > 400.0 OR player.LipidsGrams > 400.0) THEN
        player.incWeightLot = TRUE
        player.BodyWeightKg = player.BodyWeightKg + 0.000013  // +1.3E-5 / tick
    END IF

    // 3. Kiểm tra ngưỡng tử vong suy dinh dưỡng (<= 35.0 kg) và cập nhật Dynamic Weight Traits
    IF player.BodyWeightKg <= 35.0 THEN
        player.KillPlayer(STARVATION_EMACIATION_DEATH)
    END IF
    Evaluate_Dynamic_Weight_Trait_Bands(player, player.BodyWeightKg)
END FUNCTION

```

#### 1.2. Hàm Phân bậc Đặc điểm Thể trạng Động theo 7 Ngưỡng Cân nặng (`Dynamic Weight Trait Triggers`)



Trạng thái thể chất, hình phạt sát thương cận chiến, điểm `Fitness` và rủi ro gãy xương khi tiếp đất từ trên cao tự động gắn hoặc gỡ theo 7 miền giá trị của $W_{\text{body}}$ (kg):

$$\text{WeightBand}(W_{\text{body}}) = \begin{cases} \text{DEATH\_BY\_STARVATION} & \text{nếu } W_{\text{body}} \le 35.0\text{ kg} \\ \text{Trait: Emaciated} \ (\text{MeleeDmg } -60\%, \text{ Fitness } -4, \Delta\text{HP} < 0) & \text{nếu } 35.1 \le W_{\text{body}} \le 50.0\text{ kg} \\ \text{Trait: Very Underweight} \ (\text{MeleeDmg } -40\%, \text{ Fitness } -2, \text{FenceTrip}\uparrow) & \text{nếu } 50.1 \le W_{\text{body}} \le 65.0\text{ kg} \\ \text{Trait: Underweight} \ (\text{MeleeDmg } -20\%, \text{ Fitness } -1) & \text{nếu } 65.1 \le W_{\text{body}} \le 75.0\text{ kg} \\ \text{GOLDEN\_WEIGHT\_OPTIMAL} \ (\text{0\% Penalty, 100\% Efficiency}) & \text{nếu } 75.1 \le W_{\text{body}} \le 84.9\text{ kg} \\ \text{Trait: Overweight} \ (\text{Fitness } -1, \text{ EnduranceDrain } +50\%) & \text{nếu } 85.0 \le W_{\text{body}} \le 99.9\text{ kg} \\ \text{Trait: Obese} \ (\text{Fitness } -2, \text{ FallFractureRisk } +200\%, \text{ FastExhaustion}) & \text{nếu } W_{\text{body}} \ge 100.0\text{ kg} \end{cases}$$



#### 1.3. Thuật toán Mở khóa Kho báu Dưới Ván sàn & Bẫy Nhà Cố thủ (`StashSystem` & `StashUtil.lua`)



Khi người chơi tiêu diệt một thực thể đặc biệt có cờ `stashMap.spawnOnlyOnZed = true` và đọc bản đồ ghi chú (`Annotated Map`), hệ thống khởi tạo hòm kho báu ẩn dưới ô ván sàn `floors_interior_tilesandwood_01_60` (yêu cầu `Crowbar` để cạy mở `GunBox`) hoặc kích hoạt nhà bẫy (`trapped = true`, `barricades = true`) nhốt $N_{\text{trap\_zeds}} \sim \mathcal{U}_{\text{int}}(10, 30)$ thực thể xác sống bên trong.

#### 1.4. Thuật toán Đạo diễn Âm thanh Ảo & Hành vi Bám đuổi của Siêu sự kiện Trên không (`SLSoundManager` & Helicopter AI)



Trong khung ngày $d \in [6, 9]$, khi Siêu sự kiện Trên không (Helicopter Meta-Entity) kích hoạt với bán kính sóng âm $R_{\text{heli}} > 500\text{ tiles}$, thời gian lơ lửng trên đầu người chơi $T_{\text{hover}}$ phụ thuộc vào trạng thái ẩn nấp trong nhà kín ($\mathbb{I}_{\text{indoors\_hidden}}$):

$$T_{\text{hover}} = \begin{cases} [3.0, 6.0]\text{ giờ game} \ (\text{Phát sóng âm } > 500\text{ ô liên tục}) & \text{nếu người chơi lộ diện ngoài trời} \\ 1.0\text{ giờ game} \ (\text{Mất dấu và bay đi}) & \text{nếu trốn trong nhà kín, đóng rèm, tắt đèn} \end{cases}$$



---

### 2. Bảng Hằng Số Trao Đổi Chất 80kg, Kho Báu Ngầm & 164 Kịch Bản Thế Giới (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **World Stories** | `Total_Randomized_World_Stories` | `164` (`RDS` + `RB` + `RZS` + `RVS`) | `scenarios` | Tổng số kịch bản kể chuyện môi trường ngẫu nhiên trong `zombie/randomizedWorld`.

 |
| **World Stories** | `RDS_Poker_And_RPG_Zed_Counts` | `RDSRPGNight: 4 zeds (d20) | RDSPokerNight: 4 zeds | PrisonEscape: 2+2 zeds` | `entities` | Số lượng thực thể xác sống cố định trong các kịch bản RDS đặc biệt.

 |
| **World Stories** | `RB_ToiletPaper_Hoarder_Rate` | `100` | `% Toilet Paper` | Tỷ lệ toàn bộ tủ/ngăn kéo trong kịch bản `RBToiletPaper` chứa giấy vệ sinh.

 |
| **Stash System** | `StashMap_SpawnOnlyOnZed` | `true` | Tile: `floors_interior_tilesandwood_01_60` | `boolean / tile_id` | Bản đồ kho báu chỉ rơi từ xác zombie đặc biệt; cạy ván sàn bằng `Crowbar` lấy `GunBox`.

 |
| **Stash System** | `Trapped_Stash_House_Zeds` | `10 - 30` (khi `trapped = true & barricades = true`) | `entities` | Số lượng xác sống bị nhốt kín bên trong căn nhà bẫy từ bản đồ chú thích.

 |
| **Nutrition Math** | `Golden_Standard_Body_Weight` | `80.0` (`100kg: +25% burn | 60kg: -25% burn`) | `kg / %` | Mốc cân nặng chuẩn để tính tỷ lệ đốt cháy Calo ($\text{caloriesDelta} = W / 80.0$).

 |
| **Nutrition Math** | `Calorie_Storage_Bounds` | `-2200.0 to +3700.0` | `kcal` | Giới hạn sàn âm và trần dương tuyệt đối của thanh dự trữ Calorie.

 |
| **Nutrition Math** | `Calorie_Burn_Sleeping` | `0.003` | `kcal / tick` | Tốc độ tiêu hao Calo tối thiểu khi ngủ trên giường.

 |
| **Nutrition Math** | `Calorie_Burn_Normal` | `0.016` (`5.3x` vs Sleeping) | `kcal / tick` | Tốc độ tiêu hao Calo khi đứng yên hoặc đi bộ chậm.

 |
| **Nutrition Math** | `Calorie_Burn_Exercise` | `0.130` (`43.3x` vs Sleeping, `8.1x` vs Normal) | `kcal / tick` | Tốc độ tiêu hao Calo khi chạy nước rút hoặc vung vũ khí chiến đấu.

 |
| **Nutrition Math** | `Max_Fat_Loss_Rate_At_Floor` | `-8.5E-6` (khi `Calories == -2200 kcal`) | `kg / tick` | Tốc độ giảm cân đốt mỡ tối đa khi chạm sàn $-2200\text{ kcal}$.

 |
| **Nutrition Math** | `Rapid_Weight_Gain_DoubleArrow` | `+1.3E-5` (khi `Carbs > 400g` OR `Lipids > 400g`) | `kg / tick` | Tốc độ tăng cân nhanh mũi tên đôi (`^^` / `incWeightLot = true`).

 |
| **Nutrition Math** | `Protein_Strength_XP_Window` | `50.0g - 300.0g -> +30% Strength XP Gain` | `grams / %` | Dải nồng độ Protein trong máu kích hoạt bùa lợi $+30\%$ tốc độ cày Sức mạnh.

 |
| **Weight Bands** | `Lethal_And_Emaciated_Bands` | `<= 35.0kg: Death | 35.1-50.0kg: Emaciated (-60% Dmg, -4 Fit, HP drain)` | `kg / penalties` | Ngưỡng chết đói ($\le 35.0\text{ kg}$) và ngưỡng Kiệt quệ (`Emaciated`).

 |
| **Weight Bands** | `Underweight_2_Tiers_Bands` | `50.1-65.0kg: Very Under (-40% Dmg, -2 Fit) | 65.1-75.0kg: Under (-20% Dmg, -1 Fit)` | `kg / penalties` | Hai ngưỡng Thiếu cân nặng và Thiếu cân nhẹ.

 |
| **Weight Bands** | `Golden_Weight_Optimal_Band` | `75.1 - 84.9` | `kg (0% penalty)` | Vùng cân nặng hoàn hảo đạt $100\%$ hiệu suất thể lực và chiến đấu.

 |
| **Weight Bands** | `Overweight_And_Obese_Bands` | `85.0-99.9kg: Over (-1 Fit, +50% EndLoss) | >=100.0kg: Obese (-2 Fit, +200% FallFracture)` | `kg / penalties` | Ngưỡng Thừa cân và Béo phì (tăng $+200\%$ nguy cơ gãy chân khi nhảy lầu).

 |
| **Meta Director** | `Meta_Gunfire_Sound_Radius` | `200 - 400` | `tiles (blocks)` | Bán kính vòng tròn kích động di cư bầy đàn của tiếng súng ảo (`SLSoundManager`).

 |
| **Meta Director** | `Helicopter_Event_Parameters` | `Days: 6 - 9 | Radius: > 500 tiles | Hover: 3-6h (Outdoor) vs 1h (Indoor)` | `days / tiles / hours` | Thông số đầy đủ của Sự kiện Trực thăng săn lùng.

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Động lực học & Chuyển hóa calo cốt lõi: [[00_System_Engines]], [[sys_metabolism_calories]]
- Cột mốc & Kể chuyện nổi sinh RPG: [[doc_prp_01_goals_milestones]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc14]]
