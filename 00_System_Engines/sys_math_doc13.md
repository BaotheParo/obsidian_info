---
id: "sys_math:doc13"
type: "system_mathematics_spec"
document_source: "Doc 13: Project Zomboid B42 Medical, Surgery & Health Bible (17 Body Parts, 6 Wound Kinetics, Dual Infection & Surgery Equations)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[04_Character_And_Med]]",
  "[[doc_vn_nam_duoc_than_hieu_y_hoc]]",
  "[[sys_pipeline_doc13]]"
]
key_equations: [
  "Anatomy 17-Part Modifiers Matrix (skinSurface, painMod, dmgMod, bleedMod)",
  "Wound Healing Kinetics & Poultice Acceleration (Plantain, Comfrey)",
  "Bacterial Infection 40,000 Denominator Reduction & Sterilization",
  "Surgical Procedures, Stitch Maturity (stitchTime > 30-40) & Glass/Bullet Extraction"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Động lực học Hồi phục Tổn thương & Tiêu hao Băng gạc (`BodyPart.java#DamageUpdate()`)



Với mỗi bộ phận giải phẫu $bp \in \text{BODY\_PARTS\_17}$, tốc độ suy giảm thời gian tổn thương mỗi tick ($\frac{dW_{\text{time}}}{dt}$) phụ thuộc vào trạng thái băng bó ($\mathbb{I}_{\text{bandaged}}$), loại thuốc đắp thảo dược ($\mathbb{I}_{\text{herb}}$) và hệ số nẹp xương ($\text{SplintFactor}$):

$$\frac{d(\text{scratchTime})}{dt} = \begin{cases} -1.0 \times 10^{-5} & \text{nếu để trần} \\ -1.5 \times 10^{-4} \ (15\times) - 1.0 \times 10^{-4}\cdot\mathbb{I}_{\text{Plantain}} \ (\text{tổng } 25\times) & \text{nếu có băng bó} \end{cases}$$



$$\frac{d(\text{cutTime})}{dt} = \begin{cases} -1.0 \times 10^{-6} & \text{nếu để trần} \\ -5.0 \times 10^{-5} \ (50\times) - 5.0 \times 10^{-5}\cdot\mathbb{I}_{\text{Plantain}} \ (\text{tổng } 100\times) & \text{nếu có băng bó} \end{cases}$$



$$\frac{d(\text{deepWoundTime})}{dt} = \begin{cases} -2.0 \times 10^{-6} & \text{nếu để trần} \\ -2.0 \times 10^{-5} \ (10\times) - 7.0 \times 10^{-6}\cdot\mathbb{I}_{\text{Plantain}} & \text{nếu có băng bó} \end{cases}$$



$$\frac{d(\text{fractureTime})}{dt} = \begin{cases} -5.0 \times 10^{-6} & \text{nếu không nẹp} \\ -5.0 \times 10^{-5} \times \text{SplintFactor} - 5.0 \times 10^{-6}\cdot\mathbb{I}_{\text{Comfrey}} & \text{với } \text{SplintFactor} = \frac{\text{DoctorLevel} + 1}{2} \in [0.5, 5.5] \ (10\text{--}55\times) \end{cases}$$



(Lưu ý: Nếu còn mảnh kính găm trong thịt `haveGlass == true`, biến `bleedingTime` bị khóa cứng ở mức sàn $\text{bleedingTime} \ge 3.0$, không bao giờ tự cầm máu cho đến khi gắp kính ra).

#### 1.2. Phương trình Xác suất Nhiễm trùng Vi khuẩn Mỗi Nhịp (`isInfectedWound` Denominator Reduction)



Xác suất phát sinh nhiễm trùng vi khuẩn thông thường trong mỗi nhịp kiểm tra là $P(\text{BacterialInfection}) = \frac{1}{\text{Chance}}$, trong đó mẫu số $\text{Chance}$ khởi đầu từ $\text{BaseChance} = 40000$, bị trừ lũy kế bởi các yếu tố phơi nhiễm bẩn và bị chặn dưới ở sàn $5000$:

$$\text{Chance} = \max\left( 5000, \; 40000 - \sum_{k \in \text{RiskFactors}} \Delta C_k \right)$$



Với tập trọng số trừ mẫu số $\Delta C_k$:

* Không băng bó (`!bandaged`): $-10000$ | Băng gạc hết hạn/bẩn (`bandageLife == 0`): $-35000$ (tăng xác suất nhiễm trùng gấp $7\times$).


* Loại vết thương: `Scratch` ($-20000$), `Cut` ($-25000$), `Deep Wound` ($-30000$).


* Còn dị vật kính (`haveGlass`): $-24000$ | Vết bỏng chưa rửa (`burn` $-23000$ + `isNeedBurnWash` $-7000 = -30000$).


* Quần áo bẩn (`hasDirtyClothing`): $-20000$ | Quần áo dính máu (`hasBloodyClothing`): $-24000$.



```text
FUNCTION Update_Wound_Infection_And_Treatment_Tick(player, body_part):
    // 1. Khóa chảy máu nếu còn mảnh kính vỡ găm trong thịt
    IF body_part.haveGlass == TRUE THEN
        body_part.bleedingTime = Max(3.0, body_part.bleedingTime)
    END IF

    // 2. Tính xác suất nhiễm trùng vi khuẩn thông thường (BaseChance = 40,000, Floor = 5,000)
    IF body_part.HasOpenWound() AND body_part.isInfectedWound == FALSE THEN
        chance_denom = 40000
        IF NOT body_part.bandaged THEN chance_denom = chance_denom - 10000 END IF
        IF body_part.bandaged AND body_part.bandageLife <= 0.0 THEN chance_denom = chance_denom - 35000 END IF
        
        IF body_part.HasScratch() THEN chance_denom = chance_denom - 20000 END IF
        IF body_part.HasCut() THEN chance_denom = chance_denom - 25000 END IF
        IF body_part.HasDeepWound() THEN chance_denom = chance_denom - 30000 END IF
        IF body_part.haveGlass THEN chance_denom = chance_denom - 24000 END IF
        IF body_part.HasBurn() THEN
            chance_denom = chance_denom - 23000
            IF body_part.isNeedBurnWash THEN chance_denom = chance_denom - 7000 END IF
        END IF
        IF body_part.hasDirtyClothing THEN chance_denom = chance_denom - 20000 END IF
        IF body_part.hasBloodyClothing THEN chance_denom = chance_denom - 24000 END IF

        chance_denom = Max(5000, chance_denom)
        IF RandomInt(1, chance_denom) == 1 THEN
            body_part.isInfectedWound = TRUE
        END IF
    END IF

    // 3. Tốc độ tiêu diệt vi khuẩn khi điều trị
    IF body_part.isInfectedWound == TRUE THEN
        IF body_part.IsAlcoholBandaged() THEN
            body_part.WoundInfectionLevel = body_part.WoundInfectionLevel - 0.0006 // -6.0E-4 / tick
        END IF
        IF player.reduceInfectionPower > 0.0 THEN // Từ Antibiotics (50.0 điểm)
            body_part.WoundInfectionLevel = body_part.WoundInfectionLevel - 0.0002 // -2.0E-4 / tick
        END IF
        IF body_part.HasWildGarlicCataplasm() THEN
            body_part.WoundInfectionLevel = body_part.WoundInfectionLevel - 0.0002 // -2.0E-4 / tick
        END IF
    END IF
END FUNCTION

```

#### 1.3. Phương trình Thời gian Ủ bệnh Tử vong & Phẫu thuật Ngoại khoa (`pickMortalityDuration` & Surgery Equations)



* **Biến thiên Thời gian Ủ bệnh (`isInfected`) theo Trait (`del` modifier)**: Với thời gian ủ bệnh gốc $T_{\text{mort\_base}} \in [48.0, 72.0]\text{ giờ}$ ($2\text{--}3\text{ ngày}$), thời gian thực tế đến khi tử vong và biến đổi thành xác sống (sau $30\text{--}120\text{ giây}$ kể từ lúc tắt thở) là:



$$T_{\text{mortality}} = T_{\text{mort\_base}} \times \text{del}, \quad \text{với } \text{del} = \begin{cases} 1.25 \ (+25\%) & \text{nếu có Trait } \text{Resilient} \\ 0.75 \ (-25\%) & \text{nếu có Trait } \text{Prone to Illness} \\ 1.00 & \text{tiêu chuẩn} \end{cases}$$



* **Phương trình Thời gian (`Ticks`), Cơn đau (`Pain`) và Rủi ro Nhiễm trùng Phẫu thuật theo Cấp Bác sĩ ($L_{\text{doc}} = \text{DoctorLevel} \in [0, 10]$)**:


* **Băng bó (`Bandaging`)**: $T_{\text{bandage}} = 120 - (4 \times L_{\text{doc}})\text{ ticks}$.


* **Khâu vết thương sâu (`ISStitch.lua`)**:
* Dùng `Needle` + `Thread`: $T = 200 - 4 L_{\text{doc}}\text{ ticks}$, $\text{Pain} = 20 - L_{\text{doc}}$.


* Dùng `Suture Needle`: $T = 150 - 4 L_{\text{doc}}\text{ ticks}$, $\text{Pain} = 10 - L_{\text{doc}}$.


* Dùng `Suture Needle Holder`: $T = 150 - 4 L_{\text{doc}}\text{ ticks}$, $\text{Pain} = 5 - L_{\text{doc}}$.


* Xác suất gây nhiễm trùng khi khâu: $P_{\text{stitch\_inf}} = \frac{1}{5 + (L_{\text{doc}} \times 2.5)}$ ($20\%$ ở Cấp 0 $\rightarrow 3.3\%$ ở Cấp 10).


* Trưởng thành mũi khâu: $\text{stitchTime} \in [0, 50.0]$; nếu cắt chỉ khi $\text{stitchTime} < 30.0 \implies$ rách toác lại (`deepWounded = true`) và chảy máu tức thì (an toàn khi $> 30.0\text{--}40.0$, tương đương $4\text{--}5\text{ ngày}$).




* **Gắp mảnh kính (`ISRemoveGlass.lua`)**:
* Dùng `Tweezers` / `Forceps`: $T = 150 - 4 L_{\text{doc}}\text{ ticks}$, $\text{Pain} = 30 - L_{\text{doc}}$.


* Dùng Tay không: $T = 300 - 4 L_{\text{doc}}\text{ ticks}$ (chậm gấp đôi, gây xước bàn tay thao tác).




* **Phẫu thuật Gắp đầu đạn (`ISRemoveBullet.lua`)**:
* Bắt buộc dùng `Suture Needle Holder` / `Forceps` / `Forceps_Forged`: $T = 250 - 6 L_{\text{doc}}\text{ ticks}$, $\text{Pain} = 80 - L_{\text{doc}}$, gắp xong chuyển trạng thái thành `deepWounded = true`.







---

### 2. Bảng Hằng Số Giải Phẫu 17 Bộ Phận, Phẫu Thuật & Độc Học (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Anatomy (17)** | `Head_Modifiers` | `Skin:0.08 | Pain:0.80 | Dmg:0.60 | Bleed:1.00 | Umb:0.05 | ActPen:40% | MovPen:25%` | `ratios / %` | Thông số giải phẫu vùng Đầu (`HEAD`).

 |
| **Anatomy (17)** | `Neck_Lethal_Modifiers` | `Skin:0.02 | Pain:0.80 | Dmg:0.70 | Bleed:1.50 | Umb:0.10 | ActPen:5% | MovPen:5%` | `ratios / %` | Tử huyệt Cổ (`NECK`): hệ số xuất huyết cực đại $1.50$, tụt $100\% \to 0\%$ máu trong $12\text{--}18\text{ giây}$.

 |
| **Anatomy (17)** | `Torso_Upper_And_Lower` | `Upper: 0.18/0.70/0.35/0.50/0.20/20%/5% | Lower: 0.12/0.78/0.40/0.90/0.25/10%/5%` | `Skin/Pain/Dmg/Bleed/Umb/Act/Mov` | Thông số giải phẫu Ngực (`TORSO_UPPER`) và Bụng (`TORSO_LOWER`).

 |
| **Anatomy (17)** | `Groin_Modifiers` | `Skin:0.06 | Pain:0.70 | Dmg:0.40 | Bleed:0.50 | Umb:0.30 | ActPen:5% | MovPen:15%` | `ratios / %` | Thông số giải phẫu vùng Háng / Hạ bộ (`GROIN`).

 |
| **Anatomy (17)** | `Arms_Upper_And_Forearm_LR` | `UpperArm: 0.045/0.60/0.30/0.40/0.25/40%/10% | ForeArm: 0.035/0.60/0.20/0.30/0.30/60%/10%` | `Skin/Pain/Dmg/Bleed/Umb/Act/Mov` | Thông số Bắp tay Trái/Phải và Cẳng tay Trái/Phải.

 |
| **Anatomy (17)** | `Hands_LR_Max_Action_Penalty` | `Skin:0.01 | Pain:0.50 | Dmg:0.10 | Bleed:0.20 | Umb:0.35 | ActPen:100% | MovPen:5%` | `ratios / %` | Bàn tay Trái/Phải (`HAND_L/R`): phạt tối đa $100\%$ tốc độ thao tác/vung vũ khí khi chấn thương nặng.

 |
| **Anatomy (17)** | `Legs_Upper_And_Lower_LR` | `UpperLeg: 0.09/0.70/0.30/0.40/0.55/10%/40% | LowerLeg: 0.07/0.60/0.20/0.30/0.75/10%/60%` | `Skin/Pain/Dmg/Bleed/Umb/Act/Mov` | Thông số Đùi Trái/Phải và Cẳng chân Trái/Phải.

 |
| **Anatomy (17)** | `Feet_LR_Max_Move_Penalty` | `Skin:0.02 | Pain:0.50 | Dmg:0.20 | Bleed:0.20 | Umb:1.00 | ActPen:10% | MovPen:100%` | `ratios / %` | Bàn chân Trái/Phải (`FOOT_L/R`): kháng ô $1.00$, phạt tối đa $100\%$ tốc độ di chuyển (đi khập khiễng).

 |
| **Wound Healing** | `Scratch_Heal_And_BandageLife` | `Bandaged: -1.5E-4 | Raw: -1.0E-5 (15x) | Life: -8.0E-5 | Plantain: +(-1.0E-4) (25x)` | `per tick` | Động lực học lành vết cào (`Scratch`) và tốc độ bẩn băng.

 |
| **Wound Healing** | `Cut_Heal_And_BandageLife` | `Bandaged: -5.0E-5 | Raw: -1.0E-6 (50x) | Life: -1.0E-5 | Plantain: +(-5.0E-5) (100x)` | `per tick` | Động lực học lành vết rách sâu (`Cut` / `Laceration`), đắp Mã đề nhanh gấp $100\times$.

 |
| **Wound Healing** | `DeepWound_And_Bite_Burn` | `Deep: -2.0E-5 vs -2.0E-6 (10x, Life -0.005, Plantain -7.0E-6) | Bite/Burn: -1.0E-4 vs -5.0E-6 (20x, Life -1.0E-4)` | `per tick` | Động lực học lành vết toác sâu (hút bẩn băng cực nhanh `-0.005/tick`), vết cắn và vết bỏng.

 |
| **Wound Healing** | `Fracture_And_SplintFactor` | `Splinted: -5.0E-5 * ((DocLv+1)/2) vs Raw: -5.0E-6 (10-55x) | Comfrey: +(-5.0E-6)` | `per tick` | Động lực học liền xương gãy; Bác sĩ Cấp 10 (`SplintFactor = 5.5`) liền nhanh gấp $11\times$ Cấp 0 ($0.5$).

 |
| **Pathogen & Sepsis** | `Knox_Mortality_And_Reanim` | `Default: 48-72h | Resilient: del=1.25 (+25%) | Prone: del=0.75 (-25%) | Reanim: 30-120s` | `hours / seconds` | Thời gian ủ bệnh tử vong và thời gian xác chết đứng dậy thành zombie ($30\text{--}120\text{s}$).

 |
| **Pathogen & Sepsis** | `Bacterial_Cure_Rates` | `AlcoholBandage: -6.0E-4/tick | Antibiotics(Power 50.0): -2.0E-4/tick | Garlic: -2.0E-4/tick` | `per tick / power` | Tốc độ diệt khuẩn của Băng tẩm cồn, Thuốc kháng sinh và Cao tỏi rừng.

 |
| **Surgery Math** | `Stitch_Maturity_Threshold` | `Max: 50.0 | Safe Removal: > 30.0 (Good > 40.0, 4-5 days) | Glass Bleed Lock: >= 3.0` | `stitchTime / bleedTime` | Ngưỡng cắt chỉ an toàn (cắt dưới $30.0$ bị toác lại) và mức khóa chảy máu khi còn kính găm ($\ge 3.0$).

 |
| **Pharmacology** | `Vitamins_And_Disinfectant` | `Vitamins: fatigueChange = -4.0 (0.2kg) | Disinfectant: RubbingAlcohol = 1.0 (0.5L, 0.2kg)` | `stat / liters / kg` | Thông số dược lý của Vitamin/Caffeine, Cồn sát trùng và các thuốc Tây y ($0.1\text{--}0.3\text{ kg}$).

 |
| **Toxicology** | `CorpseSickness_ And_CO_Time` | `Corpse Trigger: > 20 corpses | Indoor Generator CO Death: 10 - 15 minutes` | `corpses / minutes` | Ngưỡng số lượng xác thối gây ngộ độc mùi xác và thời gian chết ngạt khí CO máy phát trong nhà.

 |
| **Doctor Perks** | `Diagnostic_Thresholds` | `Lv3-4: Scratch/Cut >17(Sev), >14(Mod) | Lv7-8: Fracture >50(Sev), >20(Mod), Stitch >40 | Lv9-10: Inf Lv*10 >= 2.5-DocLv` | `time_thresholds` | Các ngưỡng hiển thị chẩn đoán y khoa trên `ISHealthPanel.lua` theo cấp kỹ năng Sơ cứu.

 |

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Nhân lực & Y học Nam Dược: [[04_Character_And_Med]], [[doc_vn_nam_duoc_than_hieu_y_hoc]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc13]]
