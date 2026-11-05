---
id: "sys_math:doc03"
type: "system_mathematics_spec"
document_source: "Doc 03: Project Zomboid B42 Crafting & Tech Tree Guide (1,026 craftRecipes & 20 Workstations)"
associated_master_moc: "[[00_Master_MOC]]"
key_equations: [
  "craftRecipe Execution Engine & Tool Degradation (MayDegradeLight/Heavy)",
  "Workstation Thermal Threshold & Reaction Kinetics (Dome Kiln > 1200°C)",
  "Skill XP Award Multiplication & Book Multiplier",
  "Food Preservation Shelf-life Multiplier (Jerky & Salted Curing 6-12 Months)"
]
---
# ## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Chuyển hóa Công thức, Tiêu hao Công cụ & Trao thưởng XP (`craftRecipe` Execution Engine)



Một công thức chế tác $R \in \text{Recipes}_{1026}$ tại trạm làm việc $W_{\text{station}}$ yêu cầu thời gian thực thi $T_{\text{craft}}$ (tính bằng ticks), tập nguyên liệu tiêu hao $\mathcal{I}_{\text{consume}}$ và tập công cụ $\mathcal{I}_{\text{keep}}$ (có gắn cờ `mode:keep` cùng cờ hao mòn $f \in \{\text{None}, \text{MayDegradeLight}, \text{MayDegradeHeavy}\}$):

$$\Delta \text{Condition}(i_{\text{tool}}) = \begin{cases} 0 & \text{nếu } f = \text{None} \\ -\delta_{\text{light}} & \text{với xác suất } P_{\text{deg}}(\text{MayDegradeLight}) \\ -\delta_{\text{heavy}} & \text{với xác suất } P_{\text{deg}}(\text{MayDegradeHeavy}) \end{cases} \quad \forall i_{\text{tool}} \in \mathcal{I}_{\text{keep}}$$

$$\Delta \text{XP}(\text{Skill}_k) = \text{xpAward}(R, \text{Skill}_k) \times M_{\text{book}}(\text{Skill}_k)$$

```text
FUNCTION Execute_Workstation_CraftRecipe(player, workstation_entity, recipe):
    // 1. Xác thực Trạm làm việc & Nhiệt độ (ví dụ: Dome Kiln > 1200°C cho Glassmaking)
    IF recipe.RequiresWorkstation == TRUE AND workstation_entity.Type != recipe.WorkstationType THEN
        RETURN CRAFT_ABORTED_INVALID_STATION
    END IF
    IF recipe.RequiresThermalThreshold == TRUE AND workstation_entity.TemperatureC <= recipe.MinTempC THEN
        RETURN CRAFT_ABORTED_INSUFFICIENT_HEAT
    END IF

    // 2. Chạy bộ đếm thời gian chế tác (Time Ticks: 20..600 ticks)
    WaitTicks(recipe.TimeTicks)

    // 3. Tiêu hao nguyên liệu đầu vào & Xử lý hao mòn công cụ mode:keep
    FOR EACH input_item IN recipe.Inputs DO
        IF input_item.Mode == "keep" THEN
            IF input_item.HasFlag("MayDegradeLight") THEN
                ApplyToolDegradation(input_item, DEGRADE_TIER_LIGHT)
            ELSE IF input_item.HasFlag("MayDegradeHeavy") THEN
                ApplyToolDegradation(input_item, DEGRADE_TIER_HEAVY)
            END IF
        ELSE
            ConsumeInventoryItem(player, input_item)
        END IF
    END FOR

    // 4. Xuất thành phẩm & Thưởng điểm kinh nghiệm trực tiếp (xpAward = Skill:XP)
    SpawnOutputItems(player, recipe.Outputs)
    FOR EACH (skill_id, base_xp) IN recipe.xpAward DO
        player.AddSkillXP(skill_id, base_xp)
    END FOR

    RETURN CRAFT_COMPLETED
END FUNCTION

```

#### 1.2. Phương trình Kéo dài Vòng đời Thực phẩm qua Chế biến Bảo quản (Food Spoilage Extension Factor)



Thịt tươi nguyên tảng sau khi mổ trên móc treo (`entity_butcher_hook`) có thời gian phân hủy tự nhiên ngoài trời $T_{\text{spoil\_raw}} \in [3, 5]\text{ ngày}$. Khi được xử lý qua giàn phơi khô (`entity_Drying_Rack`) thành thịt sấy khô (`Jerky`), thời gian bảo quản $T_{\text{spoil\_jerky}}$ được kéo dài lên $[6, 12]\text{ tháng}$:

$$\frac{T_{\text{spoil\_jerky}}}{T_{\text{spoil\_raw}}} = \frac{[180, 365]\text{ ngày}}{[3, 5]\text{ ngày}} \in [36.0, 121.67]\text{ lần}$$

```text
FUNCTION Process_Meat_Drying_Preservation(raw_meat_item, drying_rack_entity):
    // Thịt tươi ngoài trời hỏng sau 3 - 5 ngày
    ASSERT raw_meat_item.BaseSpoilDays >= 3.0 AND raw_meat_item.BaseSpoilDays <= 5.0
    
    jerky_output = TransformToJerky(raw_meat_item)
    // Thịt khô (Jerky) bảo quản được từ 6 đến 12 tháng
    jerky_output.PreservedShelfLifeMonths = RandomRange(6.0, 12.0)
    RETURN jerky_output
END FUNCTION

```

---

### 2. Bảng Hằng Số Vật Lý, Thời Gian Chế Tác & Kinh Nghiệm (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Công Thức Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Database Index** | `Total_Craft_Recipes` | `1026` | `recipes` | Tổng số công thức chế tác (`craftRecipe`) trong Build 42.21.0.

 |
| **Database Index** | `Total_Workstation_Archetypes` | `20` | `workstations` | Số lượng trạm máy móc/thực thể chế tác chuyên dụng độc lập.

 |
| **Database Index** | `Tech_Tree_Progression_Ages` | `5` | `ages (0..4)` | 5 Thời kỳ tiến hóa công nghệ (từ Đồ đá Age 0 đến Hậu kỳ Age 4).

 |
| **Blacksmithing** | `Forge_Crude_Blade` | `Time: 200 | XP: +25` | `ticks | XP` | Rèn lưỡi dao găm thô sơ tại Lò rèn + Đe sắt.

 |
| **Blacksmithing** | `Forge_Long_Crude_Blade` | `Time: 300 | XP: +45` | `ticks | XP` | Rèn lưỡi kiếm dài thô để lắp cán thành trường đao/Katana tự chế.

 |
| **Blacksmithing** | `Forge_Scrap_Shortsword` | `Time: 500 | XP: +50` | `ticks | XP` | Rèn đoản kiếm từ sắt phế liệu có sát thương chém cực mạnh.

 |
| **Blacksmithing** | `Forge_Gorget_And_Armored_Gloves` | `Time: 300 - 600` | `ticks` | Rèn giáp che cổ (Gorget) và găng tay sắt bảo vệ **100%** trước vết cắn.

 |
| **Knapping (Age 0)** | `MakeSharpFlintFlake` | `Time: 230 | XP: +10` | `ticks | XP` | Ghè đá lửa tách mảnh đá sắc làm dao cạo dã chiến.

 |
| **Knapping (Age 0)** | `MakeStoneBlade` | `Time: 230 | XP: +20` | `ticks | XP` | Ghè lưỡi dao đá dài dùng cắt thịt hoặc buộc làm giáo đá.

 |
| **Knapping (Age 0)** | `MakeStoneAwl` | `Time: 230 | XP: +10` | `ticks | XP` | Ghè dùi đá nhọn dùng đục lỗ trên da thú để khâu quần áo.

 |
| **Pottery (Age 1)** | `Water_Shutoff_Survival_Milestone` | `30` | `days (ngày)` | Mốc thời gian hệ thống cấp nước ngừng hoạt động, buộc phải có chum gốm hứng nước mưa.

 |
| **Pottery (Age 1)** | `PressClayBrick` | `Time: 50 | XP: +5` | `ticks | XP` | Ép đất sét thành gạch xây tường kiên cố hơn tường gỗ.

 |
| **Pottery (Age 1)** | `PressClayShingle` | `Time: 50 | XP: +5` | `ticks | XP` | Ép đất sét thành ngói lợp mái chống dột cho căn cứ.

 |
| **Pottery (Age 1)** | `PressBarMold` | `Time: 100 | XP: +5` | `ticks | XP` | Nặn khuôn đúc thỏi sắt/thép chịu nhiệt.

 |
| **Glassmaking** | `Dome_Kiln_Glass_Melting_Temp` | `> 1200` | `°C (Celsius)` | Ngưỡng nhiệt độ trong lò nung vòm (`Dome Kiln`) để nấu chảy cát và tro Soda thành thủy tinh.

 |
| **Glassmaking** | `MeltSand_And_MeltGlass` | `Time: 20` | `ticks` | Thời gian nấu chảy cát thạch anh/mảnh vỡ thành thủy tinh lỏng.

 |
| **Glassmaking** | `MakeGlassJar` | `Time: 100 | XP: +75` | `ticks | XP` | Thổi lọ thủy tinh có nắp kín dùng để muối chua (`Jarring`) rau củ qua mùa đông.

 |
| **Glassmaking** | `MakeGlassBottle` | `Time: 120 | XP: +60` | `ticks | XP` | Thổi chai thủy tinh đựng cồn y tế, nhiên liệu hoặc bom cháy.

 |
| **Leatherworking** | `Cured_Leather_Armor_Bite_Def` | `80 - 100` | `% (percentage)` | Tỷ lệ cản vết cắn của áo khoác da thuộc, găng tay da và giáp chân chế từ bàn thuộc da.

 |
| **Preservation** | `Raw_Meat_Outdoor_Spoil_Time` | `3 - 5` | `days (ngày)` | Thời gian thịt thú săn tươi bị ôi thiu nếu để ngoài trời không xử lý.

 |
| **Preservation** | `Dried_Jerky_Preservation_Time` | `6 - 12` | `months (tháng)` | Thời gian bảo quản thịt sấy khô (`Jerky`) sau khi phơi trên `entity_Drying_Rack`.

 |
| **Progression** | `Skill_Book_Volume_Tiers` | `1 - 5` | `volumes (Tập)` | 5 cấp độ sách kỹ năng cho các nghề Rèn, Gốm, Nông nghiệp, May mặc.

 |

---
