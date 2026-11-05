---
id: "sys_math:doc06"
type: "system_mathematics_spec"
document_source: "Doc 06: Project Zomboid B42 Vehicles & Mechanics Handbook (52 Models, 15 Mechanical Parts & Hotwiring Dynamics)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[crv_xe_tho_boc_sat_15_parts]]",
  "[[sys_pipeline_doc06]]"
]
key_equations: [
  "Engine Ignition Quality vs Condition Stall Matrix",
  "Muffler Degradation Acoustic Amplification Radius (20m -> 150m)",
  "Tire PSI Drag & Brake Distance Multiplier",
  "Hotwiring Access & 24h Mechanics XP Cooldown"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Khởi động Động cơ & Xác suất Chết máy (Engine Ignition & Stall Probability Matrix)

Xác suất khởi động thành công của cỗ máy kéo/phương tiện ($P_{\text{start}}$) phụ thuộc vào 3 biến số độc lập: Điện lượng Ắc quy ($E_{\text{bat}} \in [0, 100]\%$), Chất lượng Động cơ bẩm sinh không thể sửa chữa ($Q_{\text{eng}} \in [0, 100]\%$), và Tình trạng Độ bền Động cơ ($C_{\text{eng}} \in [0, 100]\%$):

$$P_{\text{start}} = \begin{cases} 0 & \text{nếu } E_{\text{bat}} < 10\% \text{ hoặc } C_{\text{eng}} = 0\% \\ 1.00 \ (\text{Nổ ngay lần 1, bất chấp âm độ}) & \text{nếu } E_{\text{bat}} \ge 10\% \land Q_{\text{eng}} > 85\% \\ p_{\text{low}} \in \left[\frac{1}{5}, \frac{1}{3}\right] \ (\text{Đề 3--5 lần mới nổ}) & \text{nếu } E_{\text{bat}} \ge 10\% \land Q_{\text{eng}} < 50\% \end{cases}$$

Khi phương tiện đang di chuyển, xác suất chết máy ngẫu nhiên giữa đường ($P_{\text{stall}}$) và cơ chế truyền sát thương va chạm trực diện qua Nắp chắn trước (`Hood`) tuân theo phương trình:

$$P_{\text{stall}}(C_{\text{eng}}) > 0 \iff C_{\text{eng}} < 20\%$$

$$\Delta D_{\text{engine\_impact}} = \begin{cases} 0 & \text{nếu va chạm bằng Đuôi xe / Hông xe (Reverse Spin)} \\ 1.00 \times D_{\text{collision}} \ (100\%) & \text{nếu đâm trực diện đầu xe VÀ } C_{\text{hood}} = 0\% \ (\text{Nát sau 3--5 cú đâm}) \end{cases}$$

```text
FUNCTION Process_Caravan_Ignition_And_Collision(vehicle, is_frontal_impact, impact_damage):
    // 1. Kiểm tra điều kiện đánh lửa khởi động (Ignition Check)
    IF vehicle.BatteryChargePct < 10.0 OR vehicle.EngineConditionPct <= 0.0 THEN
        RETURN IGNITION_FAILED
    END IF

    IF vehicle.HasKey == FALSE AND vehicle.IsHotwired == FALSE THEN
        RETURN ACCESS_DENIED_LOCKED
    END IF

    // Engine Quality (>85% nổ ngay lần 1 kể cả mùa đông âm độ; <50% cần 3-5 lần đề)
    IF vehicle.EngineQualityPct > 85.0 THEN
        vehicle.IsRunning = TRUE
    ELSE IF vehicle.EngineQualityPct < 50.0 THEN
        attempts_needed = RandomInt(3, 5)
        IF vehicle.CurrentAttemptCount >= attempts_needed THEN
            vehicle.IsRunning = TRUE
        END IF
    END IF

    // 2. Kiểm tra chết máy ngẫu nhiên khi Engine Condition < 20%
    IF vehicle.IsRunning == TRUE AND vehicle.EngineConditionPct < 20.0 THEN
        IF RollRandomStall() == TRUE THEN
            vehicle.IsRunning = FALSE
        END IF
    END IF

    // 3. Phân bổ sát thương va chạm (Frontal Hood vs Reverse Spin)
    IF is_frontal_impact == TRUE THEN
        IF vehicle.HoodConditionPct > 0.0 THEN
            // Sau 3-5 cú đâm trực diện, nắp chắn (Hood) tụt về 0%
            vehicle.HoodConditionPct = Max(0.0, vehicle.HoodConditionPct - impact_damage)
        ELSE
            // Hood = 0% -> Động cơ nhận 100% sát thương trực tiếp
            vehicle.EngineConditionPct = Max(0.0, vehicle.EngineConditionPct - (1.00 * impact_damage))
        END IF
    ELSE
        // Đánh lái đuôi xe (Reverse Spin): chỉ móp cốp (giảm dung tích Trunk), động cơ an toàn 100%
        vehicle.TrunkConditionPct = Max(0.0, vehicle.TrunkConditionPct - impact_damage)
        RecalculateTrunkEffectiveCapacity(vehicle)
    END IF
END FUNCTION

```

#### 1.2. Phương trình Khuếch đại Bán kính Âm thanh theo Độ bền Bộ Giảm Thanh (`Muffler` Acoustic Radius)



Bán kính sóng âm ($R_{\text{sound}}$, tính bằng mét/blocks) phát ra từ động cơ đang hoạt động là hàm bậc thang phụ thuộc vào độ bền của Bộ giảm thanh/Ống xả ($C_{\text{muffler}} \in [0, 100]\%$):

$$R_{\text{sound}}(C_{\text{muffler}}) = \begin{cases} [20, 30]\text{ m} & \text{nếu } C_{\text{muffler}} > 80\% \ (\text{Nguyên vẹn}) \\ [20, 120]\text{ m} & \text{dải thu hút tiêu chuẩn của động cơ} \\ [100, 150]\text{ m} & \text{nếu } C_{\text{muffler}} < 20\% \ (\text{Hỏng/Rách nát}) \end{cases}$$

#### 1.3. Phương trình Động lực học Bánh xe, Quãng đường Phanh & Tương thích Phụ tùng (`PZBullet64` Physics)



* **Áp suất bánh/lốp ($P_{\text{tire}}$)** và **Tốc độ di chuyển ($V_{\text{current}}$)**: Áp suất chuẩn đạt $30\text{--}35\text{ psi}$; khi nổ hoặc non hơi, tốc độ bị phạt giảm $70\%$:



$$V_{\text{effective}} = V_{\text{base}} \times \begin{cases} 1.00 & \text{nếu } P_{\text{tire}} \in [30, 35]\text{ psi} \\ 0.30 \ (\text{Giảm } 70\%) & \text{nếu } P_{\text{tire}} \ll 30\text{ psi hoặc Nổ bánh} \end{cases}$$

* **Hệ số Quãng đường Phanh ($D_{\text{brake}}$)** và **Độ bền Phụ tùng Tải nặng (`Heavy-Duty`)**:



$$D_{\text{brake\_broken}} = 4.0 \times D_{\text{brake\_normal}}, \quad \text{Durability}_{\text{HeavyDuty}} = 1.50 \times \text{Durability}_{\text{Standard}} \ (+50\%)$$

#### 1.4. Phương trình Chiếm quyền Điều khiển (`Hotwiring`) & Giới hạn Nhận XP Cơ khí 24 Giờ



* **Điều kiện Bẻ khóa/Kích hoạt cưỡng bức (`Hotwire`)** trong thời gian $T_{\text{hotwire}} \in [5, 10]\text{ giây}$:



$$\text{CanHotwire}(p) = \text{HasTrait}(p, \text{Burglar}) \lor \left( L(\text{Electricity}) \ge 1 \land L(\text{Mechanics}) \ge 2 \land \text{HasItem}(\text{Screwdriver}) \right)$$

* **Quy tắc Chống cày XP trùng lặp trong 24 giờ (`24-Hour Part XP Cooldown`)**: Với mỗi bộ phận $part_j$ trên phương tiện $veh_i$, điểm XP tháo/lắp chỉ được cấp tối đa 1 lần mỗi $24\text{ giờ}$:



$$\Delta \text{XP}_{\text{mech}}(veh_i, part_j, t) = \begin{cases} \text{XP}_{\text{base}}(part_j) \times M_{\text{book}} & \text{nếu } t - t_{\text{last}}(veh_i, part_j) \ge 24.0\text{ hours} \\ 0 & \text{nếu } t - t_{\text{last}}(veh_i, part_j) < 24.0\text{ hours} \end{cases}$$

---

### 2. Bảng Hằng Số Vật Lý, Cơ Khí & Tải Trọng Phương Tiện (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Vehicle Database** | `Total_Vehicle_Archetypes` | `52` | `vehicles` | Tổng số mẫu phương tiện trong `media/scripts/generated/vehicles/`.

 |
| **Vehicle Database** | `Vehicle_Class_Categories` | `3` (+ `Trailers` B42) | `classes` | 3 phân hạng cốt lõi (`Standard`, `Heavy-Duty`, `Sport`) + Rơ-moóc (`Trailers`).

 |
| **Parts Durability** | `HeavyDuty_Parts_Durability_Bonus` | `+50` | `% (1.5x)` | Phụ tùng phân hạng Tải nặng bền hơn $50\%$ so với tiêu chuẩn.

 |
| **Trunk & Seats** | `HeavyDuty_Trunk_And_Seats` | `Trunk: 100 - 160+ | Seats: 2 - 6` | `kg | seats` | Sức chứa cốp và số ghế của nhóm Tải nặng (`PickUpTruck`, `PickUpVan`, `VanAmbulance`, `StepVan`).

 |
| **Trunk & Seats** | `Standard_Trunk_And_Seats` | `Trunk: 40 - 80 | Seats: 4 - 6` | `kg | seats` | Sức chứa cốp và số ghế của nhóm Tiêu chuẩn (`CarNormal`, `CarLuxury`, `CarStationWagon`, `CarTaxi`, `SUV`).

 |
| **Trunk & Seats** | `Sport_Trunk_Seats_And_Speed` | `Trunk: 25 - 40 | Seats: 2 | >100 mph` | `kg | seats | mph` | Sức chứa, số ghế và tốc độ $>100\text{ mph}$ của nhóm Thể thao (`SportsCar`, `RaceCar`).

 |
| **Trunk & Seats** | `Trailers_B42_Trunk_And_Seats` | `Trunk: 100 - 250 | Seats: 0` | `kg | seats` | Sức chứa rơ-moóc và thùng kéo gia súc/ngựa sống (`Trailer`, `TrailerCover`, `Trailer_Horsebox`, `Trailer_Livestock`).

 |
| **Engine & Battery** | `Engine_Quality_Range` | `0 - 100` (Good `>85`, Bad `<50`) | `% (Fixed)` | Chất lượng động cơ bẩm sinh cố định vĩnh viễn; $<50\%$ phải đề $3\text{--}5$ lần.

 |
| **Engine & Battery** | `Engine_Condition_Stall_Threshold` | `< 20` (Range `0 - 100`) | `% (percentage)` | Ngưỡng độ bền động cơ bắt đầu gây chết máy ngẫu nhiên khi đang chạy.

 |
| **Engine & Battery** | `Battery_Min_Ignition_Charge` | `10` | `% (percentage)` | Điện lượng ắc quy tối thiểu để đề nổ động cơ.

 |
| **Acoustic Stealth** | `Muffler_Good_Sound_Radius` | `20 - 30` (khi Condition `> 80%`) | `meters (blocks)` | Bán kính âm thanh động cơ khi ống xả còn tốt ($>80\%$).

 |
| **Acoustic Stealth** | `Muffler_Broken_Sound_Radius` | `100 - 150` (khi Condition `< 20%`) | `meters (blocks)` | Bán kính âm thanh gầm rú khi ống xả hỏng/rách ($<20\%$).

 |
| **Acoustic Stealth** | `General_Engine_Attract_Range` | `20 - 120` | `meters (blocks)` | Dải cự ly thu hút thực thể thù địch tổng quát từ sơ đồ luồng âm thanh động cơ.

 |
| **Hotwiring** | `Hotwire_Skill_Requirements` | `Electricity 1 + Mechanics 2` | `skill_levels` | Yêu cầu kỹ năng + Tua vít (`Screwdriver`) để câu dây nếu không có trait `Burglar`.

 |
| **Hotwiring** | `Hotwire_Action_Duration` | `5 - 10` | `seconds (giây)` | Thời gian thực hiện thao tác đấu dây nổ máy.

 |
| **15 Vehicle Parts** | `Hood_Zero_Condition_Transfer` | `100` (Nát sau `3 - 5` cú đâm đầu) | `% damage` | Khi nắp chắn trước (`Hood`) hỏng $0\%$, động cơ chịu $100\%$ sát thương va chạm trực diện.

 |
| **15 Vehicle Parts** | `Tires_Standard_Pressure` | `30 - 35` | `psi` | Áp suất hơi chuẩn của lốp khi bơm bằng `Tire Pump`.

 |
| **15 Vehicle Parts** | `Tires_Flat_Speed_Reduction` | `-70` | `% (percentage)` | Mức suy giảm tốc độ và trượt bánh khi nổ lốp hoặc non hơi.

 |
| **15 Vehicle Parts** | `Brakes_Broken_Distance_Mult` | `4.0` | `multiplier (x4)` | Quãng đường phanh kéo dài gấp 4 lần khi hỏng phanh.

 |
| **Combat & Escape** | `Surrounded_Exit_Death_Time` | `2` (khi bị `10 - 20` zombie vây) | `seconds (giây)` | Thời gian bị dìm chết ngay lập tức nếu mở cửa nhảy ra khi bị $10\text{--}20$ xác sống vây kín.

 |
| **Mechanics XP** | `Part_XP_Cooldown_Window` | `24` | `hours (giờ)` | Thời gian hồi (cooldown) để nhận lại XP khi tháo/lắp cùng 1 bộ phận trên cùng 1 xe.

 |
| **Mechanics XP** | `Daily_Scraping_Progression` | `5 - 10` xe/ngày $\rightarrow$ `Lv 4 - 6` sau `3 - 5` ngày | `vehicles / days` | Lộ trình tháo 4 đèn, đài, ắc quy, 4 bánh trên $5\text{--}10$ xe mỗi ngày (kèm Sách Vol 1 $\times 3\text{--}\times 5$ XP).

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Quân giới, Khí tài & Công trình: [[03_Arsenal_And_Bastions]]
- Đoàn xe thồ bọc sắt Đại Việt: [[crv_xe_tho_boc_sat_15_parts]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc06]]
