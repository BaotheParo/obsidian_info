---
id: "sys_math:doc09"
type: "system_mathematics_spec"
document_source: "Doc 09: Project Zomboid B42 Climate, Seasons & World Erosion Bible (12 Months, Severe Weather, 37.0°C Core Equilibrium & 100-Day Erosion Curve)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[sys_metabolism_calories]]",
  "[[dis_trung_han]]",
  "[[dis_trung_thu]]",
  "[[sys_pipeline_doc09]]"
]
key_equations: [
  "Thermoregulation 37.0°C 17-Part Metabolic Balance & Speed Penalty Matrix",
  "Catch A Cold 100% Accumulation & Sneeze Acoustic Wave (15-20 tiles)",
  "Severe Weather Visibility Attenuation (Fog -70%, Blizzard -15°C to -25°C)",
  "World Erosion 4-Stage 100-Day Dynamic Reclamation Curve"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Điều hòa Thân nhiệt Sinh học 17 Bộ phận (`Thermoregulation System`)

Hệ thống theo dõi nhiệt độ lõi cơ thể ($T_{\text{core}}$, chuẩn $37.0^\circ\text{C} / 98.6^\circ\text{F}$) và nhiệt độ bề mặt da ($T_{\text{skin}}$) trên $17$ bộ phận giải phẫu độc lập. Sự biến thiên nhiệt độ lõi $\frac{dT_{\text{core}}}{dt}$ là tổng đại số của nhiệt sinh ra từ trao đổi chất ($Q_{\text{metabolic}}$) trừ đi nhiệt thất thoát ra môi trường qua đối lưu gió ($Q_{\text{wind\_chill}}$) và dẫn nhiệt do ẩm ướt ($Q_{\text{wetness}}$), được bảo vệ bởi 3 chỉ số vật lý của các lớp trang phục trên từng bộ phận $bp \in \text{BODY\_PARTS\_17}$ ($\text{Insulation}$, $\text{WindResistance}$, $\text{WaterResistance}$):

$$T_{\text{core\_equilibrium}} = 37.0^\circ\text{C} \quad (\text{Vùng an toàn: } [36.5^\circ\text{C}, 37.5^\circ\text{C}])$$

Các trạng thái bệnh lý nhiệt độ và hệ số phạt cơ học theo ngưỡng $T_{\text{core}}$:

$$\text{State}(T_{\text{core}}) = \begin{cases} \text{SEVERE\_HYPOTHERMIA} \ (\Delta \text{HP} < 0 \to \text{Death}) & \text{nếu } T_{\text{core}} < 30.0^\circ\text{C} \\ \text{MILD\_HYPOTHERMIA / FROSTBITE} \ (V_{\text{move}} -50\%, V_{\text{swing}} -50\%, V_{\text{action}} -40\%) & \text{nếu } 30.0^\circ\text{C} \le T_{\text{core}} < 35.0^\circ\text{C} \\ \text{SHIVERING} \ (\text{EnergyDrainMultiplier} \in [3.0, 5.0]\times) & \text{nếu } 35.0^\circ\text{C} \le T_{\text{core}} < 36.5^\circ\text{C} \\ \text{HOMEOSTASIS\_SAFE\_ZONE} & \text{nếu } 36.5^\circ\text{C} \le T_{\text{core}} \le 37.5^\circ\text{C} \\ \text{SWEATING} \ (\text{ThirstRate} \times 4.0, \text{InnerClothingWetness}\uparrow) & \text{nếu } 37.5^\circ\text{C} < T_{\text{core}} \le 39.0^\circ\text{C} \\ \text{HYPERTHERMIA} \ (\text{Dizziness, Exhaustion, } \Delta \text{HP} < 0) & \text{nếu } T_{\text{core}} > 39.0^\circ\text{C} \end{cases}$$

```text
FUNCTION Update_Body_Thermoregulation_Tick(player, climate):
    // 1. Tính toán trao đổi nhiệt trên 17 bộ phận giải phẫu
    FOR EACH part IN player.BodyParts17 DO
        insulation = part.GetTotalClothingInsulation()
        wind_res   = part.GetTotalClothingWindResistance()
        water_res  = part.GetTotalClothingWaterResistance() // Poncho/Raincoat = 100%

        effective_wind_chill = climate.WindChillFactor * (1.0 - wind_res)
        IF climate.IsRaining AND water_res < 1.0 THEN
            part.Wetness = Min(1.0, part.Wetness + (climate.RainIntensity * (1.0 - water_res)))
        END IF

        part.UpdateSkinTemperature(climate.AmbientTempC, effective_wind_chill, part.Wetness, insulation)
    END FOR

    // 2. Cập nhật Nhiệt độ lõi (Core Temperature - Chuẩn 37.0°C)
    T_core = player.CalculateCoreTemperature()

    IF T_core < 30.0 THEN
        // Giai đoạn 3: Chết cóng - rút máu liên tục về 0
        player.ApplyContinuousHealthDrain(HYPOTHERMIA_LETHAL_DRAIN)
    ELSE IF T_core >= 30.0 AND T_core < 35.0 THEN
        // Giai đoạn 2: Hạ thân nhiệt nhẹ / Tê cóng (Frostbite)
        player.ApplyMoveSpeedPenalty(0.50)   // -50% tốc độ chạy
        player.ApplyWeaponSwingPenalty(0.50) // -50% tốc độ vung vũ khí
        player.ApplyActionSpeedPenalty(0.40) // -40% tốc độ nhặt đồ/thay đạn
    ELSE IF T_core >= 35.0 AND T_core < 36.5 THEN
        // Giai đoạn 1: Rét run (Shivering) - Đốt Calo và năng lượng gấp 300% - 500%
        player.SetEnergyDrainMultiplier(RandomRange(3.0, 5.0))
    ELSE IF T_core > 37.5 AND T_core <= 39.0 THEN
        // Đổ mồ hôi (Sweating) - Khát nước tăng gấp 4 lần, làm ướt sũng lớp áo bên trong
        player.SetThirstAccumulationMultiplier(4.0)
        player.SoakInnerClothingLayersFromSweat()
    ELSE IF T_core > 39.0 THEN
        // Sốc nhiệt (Hyperthermia) - Choáng váng, mệt lả, tụt máu
        player.ApplyHyperthermiaDizzinessAndHealthDrain()
    END IF
END FUNCTION

```

#### 1.2. Phương trình Tích lũy Cảm lạnh & Phát tán Âm thanh Ho/Hắt hơi (`CatchAColdDelta` & Acoustic Exposure)

Khi cơ thể bị ẩm ướt ($\text{Wetness} > 0$, do dầm mưa hoặc do mồ hôi làm ướt lớp áo trong rồi gặp gió lạnh khi ngừng vận động) kết hợp nhiệt độ không khí thấp, biến tích lũy `CatchAColdDelta` tăng dần đến ngưỡng $100\%$:

$$\int_{t_0}^{t} \text{CatchAColdDelta}(\text{Wetness}, T_{\text{ambient}}) \, dt \ge 100\% \implies \text{Affliction: COLD\_ACTIVE}$$

$$R_{\text{sound}}(\text{Cough / Sneeze}) = \begin{cases} [15, 20]\text{ tiles} & \text{nếu không có vật giảm âm} \\ R_{\text{muffled}} \ll 15\text{ tiles} & \text{nếu trang bị } \text{Handkerchief / Tissue} \text{ ở ô phụ} \end{cases}$$

#### 1.3. Phương trình Tác động Vật lý Khí tượng Cực đoan & Bào mòn Thế giới (`WeatherPeriod` & `ErosionMain`)

* **Hàm suy giảm Tầm nhìn & Tốc độ di chuyển theo Thời tiết**:


* **Dông bão nhiệt đới (`Tropical Storm`)**: Thu hẹp tầm nhìn xa còn $3\text{--}5\text{ tiles}$; mỗi tia sét (`Thunder Meta-Sound`) phát xung động âm thanh bán kính $100\text{--}200\text{ tiles}$ khuấy động bầy đàn.


* **Bão tuyết Bắc Cực (`Arctic Blizzard`, Tháng 12 – Tháng 2)**: Nhiệt độ tụt xuống $[-25^\circ\text{C}, -15^\circ\text{C}]$, tuyết dày làm giảm $-50\%$ tốc độ chạy bộ của nhân vật và gây trượt bánh phương tiện.


* **Sương mù dày đặc (`Heavy Pea-Soup Fog`, Tháng 9 – Tháng 11)**: Giảm $-70\%$ tầm nhìn quang học, thực thể thù địch chỉ hiện hình ở cự ly $2\text{--}3\text{ bước chân}$.




* **Hàm Tiến trình Bào mòn Thế giới (`ErosionMain`)**:
Gọi $D_{\text{max}} \in \{20, 50, 100, 200, 500\}$ (hoặc tùy chỉnh $0\text{--}365\text{ ngày}$, $-1$ để tắt) là số ngày đạt trạng thái hoang dã hóa cực đại (mặc định `ErosionSpeed = 3` $\implies D_{\text{max}} = 100\text{ ngày}$):



$$E_{\text{progress}}(d) = \min\left(1.0, \frac{d}{D_{\text{max}}}\right) \times 100\%$$

---

### 2. Bảng Hằng Số Khí Hậu 12 Tháng, Thân Nhiệt & Bào Mòn (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **World Geography** | `Simulated_Latitude_And_Start` | `38° Bắc | Start: July 9, 1993` | `latitude / date` | Vĩ độ khí hậu mô phỏng và ngày khởi đầu mặc định (giữa mùa hè).

 |
| **Season Summary** | `Summer_Temp_Range` (Jun-Aug) | `22°C to 38°C` (Min T6: `16°C`) | `°C (Celsius)` | Dải nhiệt độ mùa hè, nắng gắt, dông nhiệt đới, nguy cơ sốc nhiệt.

 |
| **Season Summary** | `Autumn_Temp_Range` (Sep-Nov) | `4°C to 28°C` (Đêm T11: `-2°C`) | `°C (Celsius)` | Dải nhiệt độ mùa thu, sương mù dày, sương muối tháng 11 diệt cây không chịu lạnh.

 |
| **Season Summary** | `Winter_Temp_Range` (Dec-Feb) | `-15°C to 17°C` (Bão tuyết: `-25°C`) | `°C (Celsius)` | Dải nhiệt độ mùa đông, mặt hồ đóng băng, đêm tháng 12 dài tới $14\text{ tiếng}$.

 |
| **Season Summary** | `Spring_Temp_Range` (Mar-May) | `1°C to 28°C` | `°C (Celsius)` | Dải nhiệt độ mùa xuân, tuyết tan gây bùn lầy tháng 3, nấm mốc phát triển.

 |
| **Monthly Matrix** | `Jan_Feb_Dec_Dawn_Dusk` | `Jan: 07:45-17:15 | Feb: 07:30-17:30 | Dec: 07:45-17:00` | `24h clock` | Giờ mặt trời mọc/lặn và biên độ nhiệt 3 tháng mùa đông (T1 hạ thân nhiệt chết trong $1\text{ giờ}$).

 |
| **Monthly Matrix** | `Mar_Apr_May_Dawn_Dusk` | `Mar: 07:00-18:00 | Apr: 06:30-18:30 | May: 06:00-19:30` | `24h clock` | Giờ mặt trời mọc/lặn 3 tháng mùa xuân.

 |
| **Monthly Matrix** | `Jun_Jul_Aug_Dawn_Dusk` | `Jun: 05:30-20:30 | Jul: 05:15-21:00 | Aug: 05:45-20:30` | `24h clock` | Giờ mặt trời mọc/lặn 3 tháng mùa hè (Tháng 7 nóng cực đại $26^\circ\text{C}\text{--}38^\circ\text{C}$).

 |
| **Monthly Matrix** | `Sep_Oct_Nov_Dawn_Dusk` | `Sep: 06:15-19:30 | Oct: 06:45-18:30 | Nov: 07:15-17:45` | `24h clock` | Giờ mặt trời mọc/lặn 3 tháng mùa thu.

 |
| **Severe Weather** | `TropicalStorm_Vis_And_Thunder` | `Vis: 3 - 5 | Thunder: 100 - 200` | `tiles (blocks)` | Tầm nhìn trong bão nhiệt đới ($3\text{--}5$ ô) và bán kính âm thanh sấm sét ($100\text{--}200$ ô).

 |
| **Severe Weather** | `ArcticBlizzard_Temp_And_Slow` | `Temp: -15°C to -25°C | RunSpeed: -50%` | `°C / %` | Nhiệt độ bão tuyết Bắc Cực và mức phạt giảm $50\%$ tốc độ chạy bộ do tuyết dày.

 |
| **Severe Weather** | `HeavyFog_Vis_Penalty` | `-70% Visibility | Reveal: 2 - 3 steps` | `% / steps` | Sương mù dày đặc giảm $70\%$ tầm nhìn, kẻ địch chỉ lộ diện khi cách $2\text{--}3$ bước chân.

 |
| **Thermoregulation** | `Standard_Human_Core_Temp` | `37.0` (`98.6`) | Safe: `36.5 - 37.5` | `°C` (`°F`) | Thân nhiệt lõi chuẩn và vùng cân bằng sinh học an toàn trên 17 bộ phận.

 |
| **Thermoregulation** | `Shivering_Energy_Drain_Mult` | `300 - 500` (`3.0x - 5.0x` at `35-36.5°C`) | `% (multiplier)` | Tốc độ đốt Calo và năng lượng khi cơ thể rét run sinh nhiệt.

 |
| **Thermoregulation** | `Frostbite_Hypothermia_Penalties` | `Move: -50% | Swing: -50% | Action: -40%` | `% (at 30-35°C)` | Hình phạt tốc độ chạy, vung vũ khí và thao tác khi hạ thân nhiệt/tê cóng ($<30^\circ\text{C}$ rút máu chết).

 |
| **Thermoregulation** | `Sweating_Thirst_Multiplier` | `4.0` (at `37.5°C - 39.0°C`) | `multiplier (x4)` | Tốc độ khát nước tăng gấp 4 lần khi đổ mồ hôi ($>39^\circ\text{C}$ gây sốc nhiệt tụt máu).

 |
| **Cold Affliction** | `CatchACold_Trigger_And_Noise` | `Threshold: 100% | Cough/Sneeze: 15 - 20` | `% / tiles` | Ngưỡng mắc bệnh cảm lạnh và bán kính âm thanh mỗi tiếng ho/hắt hơi ($15\text{--}20$ ô).

 |
| **World Erosion** | `Erosion_4_Timeline_Stages` | `0-20d | 20-50d | 50-80d | 80-100+d` | `days (ngày)` | 4 giai đoạn bào mòn: Mầm mống $\rightarrow$ Xâm lấn $\rightarrow$ Nứt vỡ $\rightarrow$ Hoang dã hóa.

 |
| **World Erosion** | `ErosionSpeed_5_Presets` | `1:20d | 2:50d | 3:100d(Def) | 4:200d | 5:500d` | `preset:days` | 5 mức tốc độ bào mòn trong Sandbox (`ErosionDays`: $0\text{--}365$, `-1` để tắt).

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Động lực học & Chuyển hóa calo: [[00_System_Engines]], [[sys_metabolism_calories]]
- Bệnh dịch thân nhiệt Đại Việt: [[dis_trung_han]] (Hạ thân nhiệt), [[dis_trung_thu]] (Say nắng say khí)
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc09]]
