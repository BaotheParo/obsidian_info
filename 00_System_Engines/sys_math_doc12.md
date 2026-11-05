---
id: "sys_math:doc12"
type: "system_mathematics_spec"
document_source: "Doc 12: Project Zomboid B42 Base Building, Fortifications & Architecture Bible (32-Floor Depth, Dual Barricades, 3x3 Plumbing & Generator Grids)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[doc_dmg_04_bastions]]",
  "[[sys_pipeline_doc12]]"
]
key_equations: [
  "IsoThumpable Structural HP & Handy Trait Bonus (HP = BaseHP + 50*Carpentry + 100*Handy)",
  "Dual-Sided Barricade HP Matrix (Planks 8000 HP, Metal Sheet 10000 HP, Iron Bars 6000 HP)",
  "Plumbing 3x3 Z+1 Auto-Purification Constraint Logic",
  "Generator 20-Tile 3-Floor Grid, Explosion (<20%) & CO Poisoning Hazard"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Máu Công trình Kiến trúc (`ISBuildUtil.lua` & `IsoThumpable` Structural HP)



Máu tối đa ($\text{HP}_{\text{max}}$) của một cấu trúc gỗ hoặc kim loại khi hoàn tất xây dựng là tổng của Máu cơ bản theo loại công trình ($\text{BaseHP}$), điểm thưởng theo cấp độ Kỹ năng xây dựng ($\text{CarpentryLevel}$ hoặc $\text{MetalworkingLevel} \in [0, 10]$) và điểm thưởng từ đặc điểm `Handy` ($+100\text{ HP}$):

$$\text{WoodHealth} = (\text{CarpentryLevel} \times 50) + \left(100 \cdot \mathbb{I}_{\text{Handy}}\right)$$

$$\text{HP}_{\text{structure}}(S) = \text{BaseHP}(S) + \text{SkillBonus}(S) + \left(100 \cdot \mathbb{I}_{\text{Handy}}\right)$$

Cụ thể với các cấu trúc tiêu biểu:

* **Tường Gỗ Thường (Cấp 1 / Cấp 2 / Cấp 3)**: $\text{BaseHP} \in \{200, 250, 300\} \implies \text{HP}_{\text{WoodenWall\_Lv1}} = 200 + (\text{Carpentry} \times 50) + [100]$ (đạt $700\text{--}800\text{ HP}$ ở Cấp 10).


* **Tường Gỗ Tròn Nguyên Khối (`Log Wall`)**: $\text{BaseHP} = 400 \implies \text{HP}_{\text{LogWall}} = 400 + (\text{Carpentry} \times 50) + [100]$ (đạt $900\text{--}1000\text{ HP}$ ở Cấp 10, không tốn đinh).


* **Hàng Rào Chông Sắt (`Metal Pole Fence`)**: $\text{BaseHP} = 500$, đạt $1200\text{ HP}$ ở `Metalworking` Cấp 10 ($\text{Bonus} = 70\text{ HP/cấp}$).



#### 1.2. Phương trình Gia cố Cửa 2 Mặt (`ISBarricadeAction.lua` Dual-Sided Barricade Matrix)



Một khung cửa sổ hoặc cửa chính có 2 mặt độc lập $f \in \{\text{Interior}, \text{Exterior}\}$. Tổng lượng máu lá chắn gia cố $\text{HP}_{\text{barricade\_total}}$ và trạng thái truyền sáng/xuyên đạn đạo được xác định bởi loại vật liệu $m \in \{\text{Plank}, \text{SheetMetal}, \text{MetalBars}\}$ và số lớp $n_f$:

$$\text{HP}_{\text{barricade\_total}} = \sum_{f \in \{\text{Int}, \text{Ext}\}} n_f \times \text{HP}_{\text{unit}}(m) = \begin{cases} (n_{\text{Int}} + n_{\text{Ext}}) \times 1000 \le 8000\text{ HP} & \text{nếu } m = \text{Wooden Plank } (n_f \in [1, 4]) \\ (n_{\text{Int}} + n_{\text{Ext}}) \times 5000 \le 10000\text{ HP} & \text{nếu } m = \text{Sheet Metal } (n_f = 1) \\ (n_{\text{Int}} + n_{\text{Ext}}) \times 3000 \le 6000\text{ HP} & \text{nếu } m = \text{Metal Bars } (n_f = 1 \text{ bộ 3 thanh}) \end{cases}$$

$$\text{CanShootAndSpearThrough}(m, n_f) = \begin{cases} \text{TRUE} & \text{nếu } m = \text{Metal Bars} \lor (m = \text{Wooden Plank} \land n_f \in \{1, 2\}) \\ \text{FALSE} \ (\text{Chặn } 100\% \text{ tầm nhìn \& ánh sáng}) & \text{nếu } m = \text{Sheet Metal} \lor (m = \text{Wooden Plank} \land n_f \in \{3, 4\}) \end{cases}$$

#### 1.3. Thuật toán Đấu nối Đường ống Tự lọc Nước mưa (`ISPlumbItem.lua` 3x3 Z+1 Plumbing Logic)



Một thiết bị đầu cuối $S_{\text{sink}}$ tại tọa độ lưới 3D $(X_s, Y_s, Z_s)$ chuyển hóa nước mưa nhiễm khuẩn (`Tainted Water`) thành nước tinh khiết $100\%$ khi và chỉ khi thỏa mãn đồng thời 4 ràng buộc hình học và trạng thái:

$$\text{IsPlumbable}(S_{\text{sink}}) \iff \begin{cases} 1.\ \text{HasItem}(\text{Player}, \text{PipeWrench}) = \text{TRUE} \\ 2.\ \text{GetRoom}(X_s, Y_s, Z_s) \neq \text{nil} \quad (\text{Phòng kín } 100\% \text{ tường, sàn, trần}) \\ 3.\ \exists B_{\text{barrel}} \text{ tại } (X_b, Y_b, Z_b) \text{ sao cho } Z_b = Z_s + 1 \land \vert{}X_b - X_s\vert{} \le 1 \land \vert{}Y_b - Y_s\vert{} \le 1 \\ 4.\ \text{WaterAmount}(B_{\text{barrel}}) \ge 1\text{ đơn vị} \end{cases}$$

```text
FUNCTION Validate_And_Execute_Plumbing(player, sink_object):
    // 1. Kiểm tra công cụ Pipe Wrench
    IF NOT player.HasItem("PipeWrench") THEN RETURN PLUMB_FAILED_NO_TOOL END IF

    // 2. Kiểm tra phòng kín hoàn chỉnh (square:getRoom() ~= nil)
    sink_sq = sink_object.GetSquare()
    IF sink_sq.GetRoom() == NIL OR sink_sq.HasMissingRoofOrWall() THEN
        RETURN PLUMB_FAILED_NOT_ENCLOSED_ROOM
    END IF

    // 3. Quét ma trận 3x3 ở tầng ngay phía trên (Z + 1) tìm Rain Collector Barrel có >= 1 đơn vị nước
    valid_barrel = NIL
    FOR dx = -1 TO 1 DO
        FOR dy = -1 TO 1 DO
            upper_sq = GetWorldSquare(sink_sq.X + dx, sink_sq.Y + dy, sink_sq.Z + 1)
            barrel = upper_sq.GetRainCollectorBarrel()
            IF barrel != NIL AND barrel.WaterUnits >= 1 THEN
                valid_barrel = barrel
                BREAK
            END IF
        END FOR
    END FOR

    IF valid_barrel == NIL THEN RETURN PLUMB_FAILED_NO_BARREL_IN_3X3_ZPLUS1 END IF

    // 4. Kích hoạt tự động lọc sạch 100% vi khuẩn (Tainted -> Pure Water)
    sink_object.ConnectPlumbingTo(valid_barrel)
    sink_object.SetAutoFilterTaintedWater100Pct(TRUE)
    RETURN PLUMB_SUCCESS
END FUNCTION

```

#### 1.4. Phương trình Điện trường Máy phát, Cháy nổ $<20\%$ & Ngộ độc Khí CO (`MOGenerator.lua`)



Một máy phát năng lượng tại tọa độ $(X_g, Y_g, Z_g)$ phủ sóng một hình trụ không gian có bán kính ngang $R_{\text{horiz}} = 20\text{ tiles}$ và chiều cao $3\text{ tầng}$ ($Z \in \{Z_g - 1, Z_g, Z_g + 1\}$):

$$\text{InPowerGrid}(X, Y, Z) \iff \sqrt{(X - X_g)^2 + (Y - Y_g)^2} \le 20.0 \land \vert{}Z - Z_g\vert{} \le 1$$

* **Ngưỡng Phát nổ Tức thì**: Khi độ bền máy phát $C_{\text{gen}} < 20\%$, xác suất kích hoạt vụ nổ hỏa hoạn thiêu rụi căn cứ $P_{\text{explode}}(C_{\text{gen}}) > 0$ (khuyến nghị bảo trì bằng `Electronics Scrap` khi $C_{\text{gen}} < 50\%$).


* **Cơ chế Ngạt khí CO Tử vong (`Carbon Monoxide Poisoning`)**: Nếu đặt máy phát hoạt động trong phòng kín hoặc tầng hầm ($\text{GetRoom}(X_g, Y_g, Z_g) \neq \text{nil}$), toàn bộ sinh vật bên trong bị rút máu liên tục ($\Delta \text{HP}_{\text{CO}} < 0$) và tử vong không tiếng động nếu đang ngủ.



---

### 2. Bảng Hằng Số Kiến Trúc, Máu Công Trình, Gia Cố & Hạ Tầng (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **32-Floor Engine** | `Max_Vertical_Building_Floors` | `32` (tăng từ `7` tầng cũ) | `floors (tầng)` | Giới hạn chiều cao xây dựng tối đa của engine kiến trúc Build 42.

 |
| **Structure HP** | `Carpentry_HP_Per_Level` | `+50` (`+100` nếu có `Handy`) | `HP / level` | Lượng máu cộng thêm cho mỗi cấp kỹ năng Mộc và từ đặc điểm `Handy`.

 |
| **Structure HP** | `WoodenFrame_HP_And_Req` | `Lv2 | 2 Planks, 2 Nails | HP: 50 -> 550` | `lv / items / HP` | Thông số Khung tường gỗ cơ bản.

 |
| **Structure HP** | `WoodenWall_Lv1_2_3_HP` | `Lv1(C2):200-700 | Lv2(C4):250-750 | Lv3(C7):300-800` | `HP (2 Planks, 4 Nails)` | Máu từ Cấp 0 đến Cấp 10 của Tường gỗ Cấp 1, 2, 3 (Cấp 3 trát vữa & sơn được; cộng thêm $+100$ nếu có `Handy`).

 |
| **Structure HP** | `LogWall_HP_And_Inputs` | `Lv0 | 4 Logs, 2 Twine/Rope | HP: 400 -> 900-1000` | `lv / items / HP` | Tường gỗ tròn nguyên khối — trâu nhất hệ gỗ, không cần đinh và búa.

 |
| **Structure HP** | `Doors_Single_And_Double_HP` | `Single(C3): 200-700 | Double(C6, 4 tiles): 400-900` | `HP` | Máu của Cửa đơn ($4$ ván, $4$ đinh, $2$ bản lề, $1$ nắm đấm) và Cổng đôi xe tải rộng $4\text{ ô}$ ($12$ ván, $12$ đinh, $4$ bản lề, $2$ nắm đấm).

 |
| **Structure HP** | `WoodenFence_Stairs_Composter` | `Fence(C2): 100-600 HP | Stairs(C6) & Composter(C2): Invulnerable` | `HP / status` | Hàng rào thấp gây `Fence Slump`; Cầu thang gỗ ($15$ ván, $15$ đinh) và Thùng ủ phân ($5$ ván, $4$ đinh) bất tử.

 |
| **Structure HP** | `Metal_Structures_HP_Matrix` | `Frame(M3):150-850 | WireFence(M4):300-1000 | PoleFence(M6):500-1200` | `HP` | Máu từ Cấp 0 đến Cấp 10 của Khung thép, Hàng rào dây thép gai và Hàng rào chông sắt.

 |
| **Barricading** | `WoodenPlank_Barricade_Stats` | `1000 HP/plank | Max 4/side (8 total = 8000 HP)` | `HP / planks` | Chốt ván gỗ: $1\text{--}2$ tấm vẫn nhìn/bắn xuyên qua được; $3\text{--}4$ tấm bịt kín tầm nhìn và ánh sáng.

 |
| **Barricading** | `SheetMetal_Barricade_Stats` | `5000 HP/sheet | Max 1/side (2 total = 10000 HP)` | `HP / sheets` | Chốt tấm thép kín: chặn $100\%$ ánh sáng và tầm nhìn.

 |
| **Barricading** | `MetalBars_Barricade_Stats` | `3000 HP/set (3 bars) | Max 2 sides = 6000 HP` | `HP / bar_sets` | Chốt song sắt thưa: giữ nguyên tầm nhìn, cho phép đâm giáo và bắn súng xuyên qua từ trong nhà.

 |
| **Water & Plumb** | `RainBarrel_Capacities` | `Small (Carp 4): 160 L | Large (Carp 7): 400 L` | `Liters (lít)` | Dung tích 2 cấp Thùng hứng nước mưa ($4$ ván, $4$ đinh, $4$ túi rác).

 |
| **Water & Plumb** | `Plumbing_3x3_ZPlus1_Matrix` | `Layer: Z + 1 | Grid: 3x3 (9 tiles) | MinWater: >= 1` | `grid / units` | Điều kiện hình học đặt thùng nước mưa phía trên bồn rửa phòng kín để tự lọc sạch $100\%$ vi khuẩn.

 |
| **Generator Grid** | `Generator_Coverage_Cylinder` | `Horizontal Radius: 20 tiles | Vertical: 3 floors` | `tiles / floors` | Vùng phủ sóng điện trường của máy phát điện (tầng hiện tại $\pm 1$ tầng).

 |
| **Generator Grid** | `Generator_Tank_And_BurnRate` | `100% = 10-12 L (2.5 Gas Cans) | Burn: 0.1 L/h (10-14d)` | `Liters / days` | Dung tích bình nhiên liệu máy phát và thời gian vận hành liên tục.

 |
| **Generator Grid** | `Generator_Explosion_Threshold` | `< 20% Condition` (Repair at `< 50%` w/ `Electronics Scrap`) | `% durability` | Ngưỡng độ bền tử thần gây xì khói đen và phát nổ cháy rụi căn cứ.

 |
| **B42 Features** | `Basement_Cooling_And_Airlock` | `Basement Freshness: +50% | Double Airlock Gap: 6m` | `% / meters` | Tầng hầm giữ rau củ tươi lâu hơn $+50\%$; khoảng cách chuẩn giữa 2 lớp cổng an ninh Airlock là $6\text{ m}$.

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Căn cứ, Pháo đài & Công trình Bastion: [[03_Arsenal_And_Bastions]], [[doc_dmg_04_bastions]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc12]]
