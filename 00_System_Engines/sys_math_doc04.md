---
id: "sys_math:doc04"
type: "system_mathematics_spec"
document_source: "Doc 04: Project Zomboid B42 Professions & Traits Bible (25 Occupations & 95 Traits)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[doc_phb_01_origins]]",
  "[[doc_adnd_03_proficiencies]]",
  "[[sys_pipeline_doc04]]"
]
key_equations: [
  "Starting Skill Level XP Multiplier (L0: 0.25x, L1: 0.75x, L2: 1.00x, L3+: 1.25x) with Fast Learner (+30%)",
  "Point-Buy Character Creation Zero-Sum Constraint",
  "Dynamic Trait Eradication (Underweight 80.0kg Threshold)"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Hệ số Nhân Kinh nghiệm Vĩnh viễn theo Cấp Kỹ năng Khởi đầu (Starting Skill Level XP Multiplier)



Với mỗi kỹ năng $k$ (trừ `Strength` và `Fitness`), tốc độ tích lũy kinh nghiệm thực tế $\Delta \text{XP}_{\text{final}}(k)$ trong suốt vòng đời nhân vật không phụ thuộc tuyến tính vào hành động mà bị chi phối bởi bậc thang hệ số nhân khởi đầu $M_{\text{init}}(L_0(k))$ dựa trên cấp độ ban đầu $L_0(k) \in \{0, 1, 2, \ge 3\}$ (tổng hợp từ Nghề nghiệp và Traits), kết hợp với hệ số `Fast Learner` ($+30\%$):

$$M_{\text{init}}(L_0(k)) = \begin{cases} 0.25 & \text{nếu } L_0(k) = 0 \quad (\text{Phạt } -75\% \text{ XP gốc}) \\ 0.75 & \text{nếu } L_0(k) = 1 \quad (\text{Nhanh gấp } 3\times \text{ so với } L_0 = 0) \\ 1.00 & \text{nếu } L_0(k) = 2 \quad (\text{Nhanh gấp } 4\times \text{ so với } L_0 = 0) \\ 1.25 & \text{nếu } L_0(k) \ge 3 \quad (\text{Nhanh gấp } 5\times \text{ so với } L_0 = 0) \end{cases}$$

$$\Delta \text{XP}_{\text{final}}(k) = \Delta \text{XP}_{\text{raw}}(k) \times M_{\text{init}}(L_0(k)) \times \left(1 + 0.30 \cdot \mathbb{I}_{\text{FastLearner}}\right) \times M_{\text{book}}(k)$$

```text
FUNCTION Calculate_Skill_XP_Gain(player, skill_id, raw_xp_amount):
    // 1. Xác định hệ số nhân vĩnh viễn từ cấp độ khởi đầu (L_0)
    L0 = player.GetStartingSkillLevel(skill_id)
    IF L0 == 0 THEN
        init_multiplier = 0.25
    ELSE IF L0 == 1 THEN
        init_multiplier = 0.75
    ELSE IF L0 == 2 THEN
        init_multiplier = 1.00
    ELSE // L0 >= 3
        init_multiplier = 1.25
    END IF

    final_xp = raw_xp_amount * init_multiplier

    // 2. Áp dụng Trait Fast Learner (+30% cho mọi kỹ năng trừ Strength & Fitness)
    IF player.HasTrait("base:fastlearner") AND skill_id NOT IN ["Strength", "Fitness"] THEN
        final_xp = final_xp * 1.30
    END IF

    // 3. Áp dụng Trait Crafty / Artisan cho nhóm kỹ năng chế tác B42
    IF player.HasTrait("base:crafty") AND IsB42CraftingSkill(skill_id) THEN
        final_xp = ApplyCraftyBonus(final_xp)
    END IF

    RETURN final_xp
END FUNCTION

```

#### 1.2. Phương trình Cân bằng Ngân sách Điểm Khởi tạo Nhân vật (Point-Buy Character Creation Constraint)



Một cấu hình nhân vật hợp lệ bao gồm 1 Nghề nghiệp $P \in \text{Professions}_{25}$ có chi phí điểm $\text{Cost}(P) \in [-8, +8]$ và một tập hợp các Đặc điểm tính cách $\mathcal{T} \subset \text{Traits}_{95}$ (gồm các Trait tích cực có $\text{Cost}(t^+) < 0$ và các Trait tiêu cực có $\text{Cost}(t^-) > 0$) sao cho tổng ngân sách điểm ròng không âm:

$$\text{Balance}_{\text{net}} = \text{Cost}(P) + \sum_{t^+ \in \mathcal{T}^+} \text{Cost}(t^+) + \sum_{t^- \in \mathcal{T}^-} \text{Cost}(t^-) \ge 0$$

```text
FUNCTION Validate_Character_Point_Buy(profession, selected_traits):
    net_points = profession.Cost // Từ -8 (Veteran) đến +8 (Unemployed)
    
    FOR EACH trait IN selected_traits DO
        IF HasMutualExclusionConflict(trait, selected_traits) THEN
            RETURN INVALID_BUILD_CONFLICT
        END IF
        net_points = net_points + trait.Cost
    END FOR

    IF net_points < 0 THEN
        RETURN INVALID_BUILD_NEGATIVE_POINTS
    END IF
    RETURN VALID_CHARACTER_BUILD
END FUNCTION

```

#### 1.3. Phương trình Xóa bỏ Động Đặc điểm Thể chất & Triệt tiêu Khiếm khuyết Giác quan (Dynamic Trait Eradication & Sensory Modifiers)



* **Cơ chế tự xóa Trait `Underweight` (`base:underweight`, $+6\text{ điểm}$)**: Nhân vật khởi đầu bị phạt $-1\text{ Fitness}$ và giảm sát thương cận chiến, nhưng khi nạp calo chất béo cao (bơ, mỡ động vật) trong $3\text{--}5\text{ ngày}$ đầu để đưa cân nặng $W_{\text{body}}$ chạm mốc chuẩn $80.0\text{ kg}$, trait này bị xóa bỏ vĩnh viễn:



$$\text{HasTrait}(\text{Underweight}, t) = \text{FALSE} \quad \forall t \ge t^* \text{ nếu } W_{\text{body}}(t^*) \ge 80.0\text{ kg}$$

* **Cơ chế Bán kính Nhận thức Hậu phương (`Keen Hearing` vs `Deaf`)** và **Tốc độ Thao tác Khẩn cấp (`Dextrous`)**:



$$R_{\text{rear\_perception}} = R_{\text{rear\_base}} \times \begin{cases} 2.0 & \text{nếu có } \text{Keen Hearing } (-6\text{ pts}) \\ 0.0 \text{ (Mất } 100\% \text{ âm thanh)} & \text{nếu có } \text{Deaf } (+12\text{ pts}) \end{cases}$$

$$T_{\text{inventory\_transfer}} = \frac{T_{\text{transfer\_base}}}{2.0} \quad (\text{khi có } \text{Dextrous } -2\text{ pts})$$

---

### 2. Bảng Hằng Số Vật Lý, Chi Phí Điểm & Hệ Số Nhân Kỹ Năng (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Database Index** | `Total_Professions_Count` | `25` | `professions` | Tổng số nghề nghiệp trong `character_professions.txt` (Build 42.21.0).

 |
| **Database Index** | `Total_Character_Traits_Count` | `95` | `traits` | Tổng số đặc điểm tính cách trong `character_traits.txt` (Build 42.21.0).

 |
| **XP Multiplier** | `XP_Rate_Level_0` | `25` | `% (0.25x)` | Tỷ lệ nhận XP khi kỹ năng khởi đầu ở Cấp 0 (bị phạt $75\%$ XP).

 |
| **XP Multiplier** | `XP_Rate_Level_1` | `75` | `% (0.75x)` | Tỷ lệ nhận XP khi có $+1$ điểm khởi đầu (nhanh gấp $3\times$ so với Cấp 0).

 |
| **XP Multiplier** | `XP_Rate_Level_2` | `100` | `% (1.00x)` | Tỷ lệ nhận XP khi có $+2$ điểm khởi đầu (nhanh gấp $4\times$ so với Cấp 0).

 |
| **XP Multiplier** | `XP_Rate_Level_3_Plus` | `125` | `% (1.25x)` | Tỷ lệ nhận XP khi có $\ge +3$ điểm khởi đầu (nhanh gấp $5\times$ so với Cấp 0).

 |
| **Innate Perks** | `Ax_pert_Swing_And_Chop_Boost` | `+25` | `% (percentage)` | Tăng $25\%$ tốc độ vung rìu chiến đấu và chặt cây của nghề Tiều phu (`base:lumberjack`).

 |
| **Innate Perks** | `Desensitized_Panic_Immunity` | `100` | `% (percentage)` | Miễn nhiễm $100\%$ Hoảng loạn (Panic), giữ độ chính xác tầm xa $100\%$ của `base:veteran`.

 |
| **Positive Traits** | `KeenHearing_Rear_Radius_Mult` | `2.0` (`-6 pts`) | `multiplier` | Tăng gấp đôi bán kính vòng nhận diện phía sau lưng (`base:keenhearing`).

 |
| **Positive Traits** | `Strong_Combat_And_Carry_Mod` | `Lv 9-10 | +40% Dmg | 18-20 kg` | `level / % / kg` | Tăng Strength lên 9–10, $+40\%$ sát thương cận chiến, sức mang vác $18\text{--}20\text{ kg}$ (`base:strong`, $-10\text{ pts}$).

 |
| **Positive Traits** | `Dextrous_Transfer_Speed_Mult` | `2.0` (`-2 pts`) | `multiplier` | Tăng gấp đôi tốc độ chuyển đồ giữa túi và mặt đất (`base:dextrous`).

 |
| **Positive Traits** | `FastLearner_Global_XP_Boost` | `+30` (`-6 pts`) | `% (percentage)` | Tăng $+30\%$ XP cho toàn bộ kỹ năng trừ Thể lực và Sức mạnh (`base:fastlearner`).

 |
| **Positive Traits** | `Inconspicuous_Stealth_Mod` | `-50` (`-4 pts`) | `% (percentage)` | Giảm $50\%$ xác suất bị thực thể thù địch phát hiện khi đi lén.

 |
| **Negative Traits** | `HighThirst_Water_Consumption` | `2.0` (`+6 pts`) | `multiplier` | Tăng gấp đôi tần suất khát nước (`base:highthirst`).

 |
| **Negative Traits** | `Underweight_Cure_Threshold` | `80.0` (`+6 pts`) | `kg (3-5 days)` | Phạt $-1\text{ Fitness}$ & giảm sát thương nhẹ; tự biến mất vĩnh viễn khi ăn đạt mốc $80.0\text{ kg}$.

 |
| **Negative Traits** | `Deaf_Audio_Loss` | `100` (`+12 pts`) | `% (percentage)` | Mất $100\%$ âm thanh toàn cục.

 |
| **Negative Traits** | `SundayDriver_Speed_Cap` | `30` (`+1 pt`) | `mph` | Giới hạn tốc độ phương tiện tối đa ở $30\text{ mph}$ và giảm cực mạnh lực kéo.

 |
| **Negative Traits** | `Asthmatic_Endurance_Loss_Mult` | `2.0` (`+5 pts`) | `multiplier` | Mất thể lực nhanh gấp đôi khi vận động/vung vũ khí (chỉ 3–4 nhát đã kiệt sức).

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực & Y học Nam Dược: [[04_Character_And_Med]]
- Quy tắc gốc AD&D & 5E: [[doc_phb_01_origins]], [[doc_adnd_03_proficiencies]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc04]]
