# TỔNG TẬP PHƯƠNG TRÌNH TOÁN HỌC & HẰNG SỐ CỐT LÕI (MASTER MATHEMATICS & CONSTANTS)
> Dự án: Grimdark Historical Survival RPG (Đại Việt XIV-XV) & Project Zomboid B42 Architecture
> Tích lũy liên tục từ Doc 01 đến Doc 14

================================================================================
# DOCUMENT 01: ARCHITECTURE AND DESIGN BIBLE (100% HOÀN CHỈNH)
================================================================================

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Hình nón Thị giác & Lan truyền Sóng Âm (AI Sensory Perception)

Một thực thể AI tại tọa độ $\vec{P}_{mob}$ với vector hướng nhìn đơn vị $\vec{F}_{mob}$ phát hiện mục tiêu tại $\vec{P}_{target}$ thông qua kết hợp kiểm tra góc nhìn (FOV Cone $\approx 120^\circ$), Raycast ánh sáng lưới ô (60 Hz từ `Lighting64.dll`) và khoảng cách lan truyền sóng âm ($R_{sound}$):

$$\theta = \arccos\left( \frac{(\vec{P}_{target} - \vec{P}_{mob}) \cdot \vec{F}_{mob}}{\Vert{}\vec{P}_{target} - \vec{P}_{mob}\Vert{}} \right) \le \frac{\text{FOV}}{2} = 60^\circ$$

$$D_{grid} = \Vert{}\vec{P}_{target} - \vec{P}_{mob}\Vert{}_2 \le R_{sound}(\text{Action})$$

```text
FUNCTION Evaluate_AI_Sensory_Trigger(mob, target, action_sound_radius, has_open_bleeding):
    distance = EuclideanDistance(mob.position, target.position)
    
    // 1. Kiểm tra Sóng âm (Hearing & Grid Wave Propagation qua fmod & Grid)
    IF distance <= action_sound_radius THEN
        mob.SetState(INVESTIGATE, target.last_sound_position)
        RETURN TRUE
    END IF

    // 2. Kiểm tra Khứu giác Máu (Blood Sense - Build 42)
    IF has_open_bleeding == TRUE AND target.is_bandaged == FALSE THEN
        IF distance <= mob.blood_scent_detection_range THEN
            mob.SetState(INVESTIGATE, target.position)
            RETURN TRUE
        END IF
    END IF

    // 3. Kiểm tra Hình nón Thị giác (Vision Cone ~120 deg) & Raycast LOS 60Hz
    direction_to_target = Normalize(target.position - mob.position)
    angle_deg = RadiansToDegrees(Acos(DotProduct(mob.facing_vector, direction_to_target)))
    
    IF angle_deg <= (120.0 / 2.0) THEN
        visibility_score = CalculateVisibility(DayNightCycle, WeatherFogRain, target.Camouflage, target.IsCrouching, target.IsSneaking)
        IF Raycast_Grid_LOS(mob.position, target.position) == CLEAR AND distance <= visibility_score THEN
            mob.SetState(PURSUE_TARGET, target)
            RETURN TRUE
        END IF
    END IF
    
    RETURN FALSE
END FUNCTION
```

#### 1.2. Phương trình Tốc độ Chảy máu Tử huyệt & Xác suất Truyền Nhiễm Tuyệt đối (Pathogen Transmission Matrix)

Khi một bộ phận cơ thể $bp \in \text{BODY\_PARTS\_17}$ nhận vết thương loại $w \in \{\text{SCRATCH}, \text{LACERATION}, \text{INFECTED\_BITE}\}$, hệ số thời gian chảy máu và xác suất nhiễm mầm bệnh tử vong (`isInfected`) được xác định bởi:

$$P(\text{Lethal\_Pathogen} \mid w) = \begin{cases} 0.07 & \text{nếu } w = \text{SCRATCH} \\ 0.25 & \text{nếu } w = \text{LACERATION} \\ 1.00 & \text{nếu } w = \text{INFECTED\_BITE} \end{cases}$$

$$T_{\text{bleed\_death}}(\text{NECK}) \in [10.0, 20.0]\text{ giây} \quad (\text{với } M_{\text{bleed\_time}}(\text{NECK}) = 1.5)$$

```text
FUNCTION Process_Wound_And_Infection(player, body_part, attack_type, is_sterile_bandaged):
    // 1. Xác định nguy cơ tử vong do xuất huyết tại tử huyệt
    IF body_part == BODY_PART_NECK THEN
        bleed_rate = BASE_NECK_BLEED_RATE * 1.5
        player.BleedingTimeRemaining = RandomRange(10.0, 20.0) // giây tử vong nếu không cầm máu
    ELSE
        bleed_rate = GetPartBleedRate(body_part)
    END IF

    // 2. Phân định 3 cấp độ lây nhiễm
    roll = RandomFloat(0.0, 1.0)
    IF attack_type == BITE THEN
        player.isInfected = TRUE // 100% Chắc chắn chuyển hóa thành zombie
    ELSE IF attack_type == LACERATION AND roll <= 0.25 THEN
        player.isInfected = TRUE
    ELSE IF attack_type == SCRATCH AND roll <= 0.07 THEN
        player.isInfected = TRUE
    END IF

    // 3. Phân định nhiễm trùng thông thường do vết thương bẩn
    IF is_sterile_bandaged == FALSE AND RandomFloat(0.0, 1.0) <= DIRTY_WOUND_CHANCE THEN
        player.infectedWound = TRUE // Có thể chữa bằng cồn/kháng sinh
    END IF
END FUNCTION
```

#### 1.3. Phương trình Suy giảm Độ bền Vũ khí theo Phân phối Bernoulli (Weapon Durability Decay)

Mỗi đòn đánh trúng mục tiêu kích hoạt phép thử Bernoulli với xác suất hao mòn $P = \frac{1}{\text{ConditionLowerChanceOneIn}}$:

$$P(\Delta C = -1) = \frac{1}{\text{ConditionLowerChanceOneIn}}$$

$$E[\text{Strikes Until Break}] = C_{\max} \times \text{ConditionLowerChanceOneIn}$$

```text
FUNCTION Evaluate_Weapon_Wear(weapon):
    chance_one_in = weapon.ConditionLowerChanceOneIn
    roll = RandomInteger(1, chance_one_in)
    
    IF roll == 1 THEN
        weapon.Condition = Max(0, weapon.Condition - 1)
        IF weapon.Condition == 0 THEN
            weapon.IsBroken = TRUE
            TriggerSoundEvent("Weapon_Snap_Break", weapon.UserPosition)
        END IF
    END IF
END FUNCTION
```

#### 1.4. Phương trình Cân bằng Kinh tế Nội tại 4 Thành phần (Internal Economic Balance)

Một chu trình tài nguyên hoàn chỉnh tuân theo định luật bảo toàn dòng chảy $S \rightarrow P \rightarrow C \rightarrow D$:

$$\frac{d(\text{Pool})}{dt} = \sum \text{Source\_Rate} - \sum \text{Drain\_Rate} \pm \text{Converter\_Rate}$$

```text
FUNCTION Update_Economy_Cycle(delta_time):
    // Sources: Tái tạo tài nguyên tự nhiên (thu hoạch mùa vụ, quặng tái sinh)
    inflow = Agriculture_Source.Generate(delta_time) + Ore_Deposit.Mine(delta_time)
    
    // Drains: Tiêu hao sinh học và vận hành căn cứ
    outflow = Settlement_Hunger.Consume(delta_time) + Bastion_Maintenance.Burn(delta_time)
    
    // Converter: Biến đổi nguyên liệu thô thành thành phẩm
    converted = Smithy_Station.Convert(Iron_Ore, Refined_Steel, delta_time)
    
    Storage_Pool.Amount = Clamp(Storage_Pool.Amount + inflow - outflow, 0, Storage_Pool.MaxCapacity)
END FUNCTION
```

#### 1.5. Phương trình Điều hòa Thân nhiệt Đa tầng (`Thermoregulator`) & Cân bằng Nhiệt Sinh học

Thân nhiệt vùng lõi ($T_{\text{core}}$, chuẩn $37.0^\circ\text{C}$) được cân bằng thông qua trao đổi nhiệt đa tầng giữa 17 vùng cơ thể, các lớp vải/giáp bảo hộ ($L_i$) và môi trường:

$$\frac{dT_{\text{core}}}{dt} = \frac{1}{C_{\text{body}}} \left[ Q_{\text{metabolic}}(\text{Activity}) - Q_{\text{evaporative}}(\text{Sweat}) - \sum_{bp=1}^{17} \frac{A_{bp} (T_{\text{skin}, bp} - T_{\text{ambient}})}{R_{\text{insulation}}(bp) + R_{\text{wetness}}(bp)} \right]$$

- Khi $T_{\text{core}} < 35.0^\circ\text{C}$: Kích hoạt **Trúng hàn (Hypothermia)**, rung cơ giật, ho tiếng vang thu hút AI trong bán kính 15 blocks.
- Khi $T_{\text{core}} > 39.0^\circ\text{C}$: Kích hoạt **Trúng thử (Hyperthermia)**, tụt thể lực, hoa mắt, mất nước tăng gấp $2.5\times$.

```text
FUNCTION Update_Thermoregulation(player, ambient_temp, is_raining, wind_speed, delta_time):
    total_heat_loss = 0.0
    FOR EACH part IN player.BodyParts17:
        insulation = Sum(part.ClothingLayers.Insulation)
        wetness_penalty = part.ClothingWetness * 0.75 // Quần áo ướt giảm 75% độ cách nhiệt
        effective_r = Max(0.1, insulation - wetness_penalty)
        
        part_flux = (part.SkinTemp - ambient_temp) / effective_r
        total_heat_loss += part_flux * part.SurfaceAreaRatio
    ENDFOR

    metabolic_heat = player.CurrentMetabolicRate // Nghỉ ngơi 75W, lao động/chiến đấu 600-900W
    net_heat = (metabolic_heat - total_heat_loss) * delta_time
    player.CoreTemperature += net_heat / player.ThermalMass

    IF player.CoreTemperature < 35.0 THEN
        player.ApplyAffliction("HYPOTHERMIA")
    ELSE IF player.CoreTemperature > 39.0 THEN
        player.ApplyAffliction("HYPERTHERMIA")
    END IF
END FUNCTION
```

#### 1.6. Phương trình Động học Chuyển hóa 4 Thành phần Dinh dưỡng (`Nutrition`) & Thể trọng

Cân nặng nhân vật ($W$, mốc chuẩn $80.0\text{ kg}$) biến thiên theo tích lũy thặng dư hoặc thâm hụt năng lượng từ 4 nhóm dinh dưỡng (`Calories`, `Carbohydrates`, `Proteins`, `Lipids`):

$$\Delta E_{\text{net}} = E_{\text{ingested}}(\text{Calories}) - \left[ BMR + E_{\text{activity}}(\text{Work}) \right]$$

$$\frac{dW}{dt} = \frac{\Delta E_{\text{net}}}{7700.0\text{ kcal/kg}}$$

- Khi $W < 65.0\text{ kg}$ (Thiếu cân nặng): Giảm $35\%$ sức mang vác, giảm $25\%$ sát thương cận chiến.
- Khi $W > 95.0\text{ kg}$ (Thừa cân): Tăng tiêu hao thể lực gấp $1.4\times$, giảm $15\%$ tốc độ chạy nước rút.

```text
FUNCTION Update_Nutrition_And_Weight(player, delta_time_hours):
    bmr = 1800.0 // Nhu cầu năng lượng nền hàng ngày (kcal/ngày)
    activity_burn = player.CalorieBurnRatePerHour * delta_time_hours
    total_burn = (bmr / 24.0) * delta_time_hours + activity_burn
    
    net_calories = player.DigestedCalories - total_burn
    player.BodyWeightKg += net_calories / 7700.0 // 7700 kcal thặng dư = 1 kg thể trọng
    
    // Ảnh hưởng của Protein lên phục hồi cơ bắp & mệt mỏi
    IF player.Nutrients.Proteins < MIN_PROTEIN_THRESHOLD THEN
        player.MuscleFatigueRecoveryMultiplier = 0.5
    ELSE
        player.MuscleFatigueRecoveryMultiplier = 1.0
    END IF
END FUNCTION
```

#### 1.7. Phương trình Suy giảm Xác suất Sửa chữa Lũy tiến (`fixing.txt`)

Mỗi lần sửa chữa một trang bị hoặc vũ khí sử dụng phụ liệu (keo dán, dây buộc, đinh chốt), xác suất thành công giảm dần theo số lần sửa trước đó ($k = \text{RepairsCount}$):

$$P_{\text{success}}(k) = \max\left(0.10, P_{\text{base}} \times (1.0 - k \times 0.20)\right)$$

$$P_{\text{break\_fail}}(k) = 1.0 - P_{\text{success}}(k)$$

```text
FUNCTION Attempt_Item_Fix(item, repair_material, player_skill):
    repairs_count = item.RepairsCount
    base_chance = 0.80 + (player_skill * 0.05)
    success_chance = Max(0.10, base_chance - (repairs_count * 0.20))
    
    roll = RandomFloat(0.0, 1.0)
    IF roll <= success_chance THEN
        item.Condition = Min(item.ConditionMax, item.Condition + CalculateRepairGain(item, repair_material))
        item.RepairsCount += 1
        RETURN SUCCESS
    ELSE
        // Nguy cơ gãy hoàn toàn khi sửa thất bại
        IF RandomFloat(0.0, 1.0) <= 0.30 THEN
            item.IsPermanentlyBroken = TRUE
            RETURN PERMANENT_BREAK
        END IF
        RETURN FAIL
    END IF
END FUNCTION
```

#### 1.8. Phương trình Di truyền Men-đen Động vật (`AnimalGene` & `AnimalAllele`)

Bộ gene của thế hệ con cái ($G_{\text{offspring}}$) được kế thừa từ Alen cha mẹ ($A_{\text{sire}}, A_{\text{dam}}$) trên 5 tính trạng then chốt: Kích thước (`SIZE`), Sản lượng sữa (`MILK`), Sản lượng trứng/con (`FECUNDITY`), Tốc độ sinh trưởng (`GROWTH`), và Tính khí thuần dưỡng (`TAMENESS`):

$$G_{\text{trait}} = \frac{1}{2}(A_{\text{sire}} + A_{\text{dam}}) + \epsilon_{\text{mutation}}, \quad \epsilon_{\text{mutation}} \sim \mathcal{N}(0, \sigma^2)$$

```text
FUNCTION Breed_Animals(sire, dam):
    offspring_genome = New Genome()
    FOR EACH trait IN [SIZE, MILK, FECUNDITY, GROWTH, TAMENESS]:
        inherited_value = (sire.GetAllele(trait) + dam.GetAllele(trait)) / 2.0
        // Xác suất đột biến tự nhiên 5%
        mutation = 0.0
        IF RandomFloat(0.0, 1.0) <= 0.05 THEN
            mutation = RandomGaussian(0.0, 0.15)
        END IF
        offspring_genome.SetTrait(trait, Clamp(inherited_value + mutation, 0.1, 2.5))
    ENDFOR
    RETURN offspring_genome
END FUNCTION
```

#### 1.9. Mô hình Động học Mạng Native UDP & Dự đoán Phía Client (RakNet & Client Prediction)

Để đồng bộ hàng chục ngàn zombie và động vật mà không nghẽn băng thông, server duy trì chu kỳ tick cố định 20 Hz (50ms) trong khi client nội suy ở 60 Hz:

$$\vec{P}_{\text{client}}(t) = \vec{P}_{\text{server}}(t_0) + \int_{t_0}^{t} \vec{V}_{\text{predicted}}(\tau) d\tau + \mathbf{K}_{\text{reconciliation}} (\vec{P}_{\text{server}} - \vec{P}_{\text{client}})$$

---


================================================================================
# DOCUMENT 02: COMPLETE ITEM ENCYCLOPEDIA & DATABASE GUIDE
================================================================================

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 2.1. Phương trình Sát thương Bạo kích & Phân bố Động năng Vũ khí Cận chiến (Crit Multiplier & Kinetic Energy)

Sát thương đầu ra thực tế ($D_{\text{actual}}$) của một đòn đánh cận chiến từ 409 loại vũ khí trong `weapon.txt` được xác định bởi biên độ sát thương gốc $[D_{\min}, D_{\max}]$, kỹ năng võ thuật của nhân vật ($M_{\text{skill}}$) và cơ chế bạo kích biến thiên:

$$D_{\text{actual}} = \left[ D_{\min} + \text{rand}() \times (D_{\max} - D_{\min}) \right] \times M_{\text{skill}} \times \left( 1 + [M_{\text{crit}} - 1] \times \mathbb{I}_{\text{crit}} \right)$$

$$\mathbb{I}_{\text{crit}} \sim \text{Bernoulli}(P_{\text{crit}})$$

*Bảng hằng số sát thương & tỷ lệ Crit của Top vũ khí sát thương đỉnh cao:*
- **Katana (`Base.Katana`):** $D = [8.0, 8.0]$, $W = 2.0\text{ kg}$, $P_{\text{crit}} = 0.35$, $M_{\text{crit}} = 6.0\times$ (Sát thương tối đa khi bạo kích: $48.0$).
- **Jawbone Morningstar (`Base.JawboneBovide_Morningstar`):** $D = [1.3, 3.0]$, $W = 4.0\text{ kg}$, $P_{\text{crit}} = 0.50$, $M_{\text{crit}} = 5.0\times$.
- **Bucket Maul Metal (`Base.BucketMace_Metal`):** $D = [1.5, 3.0]$, $W = 10.0\text{ kg}$, $P_{\text{crit}} = 0.70$, $M_{\text{crit}} = 2.0\times$.
- **Block Maul (`Base.BlockMaul`):** $D = [2.0, 3.0]$, $W = 4.0\text{ kg}$, $P_{\text{crit}} = 0.40$, $M_{\text{crit}} = 2.0\times$.

```text
FUNCTION Calculate_Melee_Damage(weapon, attacker_skill, target_armor):
    base_dmg = RandomFloat(weapon.MinDamage, weapon.MaxDamage)
    skill_mod = 1.0 + (attacker_skill * 0.10)
    
    // Kiểm tra đòn đánh chí mạng (Critical Hit)
    is_crit = (RandomFloat(0.0, 1.0) <= weapon.CritChance)
    crit_mod = is_crit ? weapon.CritMultiplier : 1.0
    
    raw_damage = base_dmg * skill_mod * crit_mod
    // Giảm trừ qua lớp giáp mục tiêu
    net_damage = Max(0.1, raw_damage * (1.0 - target_armor.DamageReductionRatio))
    
    RETURN net_damage, is_crit
END FUNCTION
```

#### 2.2. Phương trình Giảm Tải Trọng Thùng Chứa & Balo (`WeightReduction` Dynamics)

Khối lượng hiệu dụng ($W_{\text{effective}}$) mà nhân vật phải gánh khi mang một trong 308 loại balo (`container.txt`) chứa $N$ vật phẩm có khối lượng thành phần $w_i$ được tối ưu hóa theo hệ số giảm tải $\eta \in [0.65, 0.85]$:

$$W_{\text{effective}} = W_{\text{bag\_empty}} + \sum_{i=1}^{N} w_i \times (1.0 - \eta)$$

$$\text{Điều kiện sức chứa:} \quad \sum_{i=1}^{N} w_i \le C_{\max}$$

*Hằng số các lớp Balo hàng đầu Build 42:*
- **Large Framepack (`Base.Bag_CraftedFramepack_Large3`):** $C_{\max} = 35$, $\eta = 0.75$, $W_{\text{empty}} = 4.0\text{ kg}$.
- **Military ALICE Pack (`Base.Bag_ALICEpack`):** $C_{\max} = 28$, $\eta = 0.85$, $W_{\text{empty}} = 2.0\text{ kg}$.
- **Big Hiking Backpack (`Base.Bag_BigHikingBag`):** $C_{\max} = 22$, $\eta = 0.80$, $W_{\text{empty}} = 1.5\text{ kg}$.

```text
FUNCTION Calculate_Container_Effective_Weight(container):
    total_contents_weight = Sum(container.Items.Weight)
    IF total_contents_weight > container.MaxCapacity THEN
        Trigger_Overflow_Penalty(container)
    END IF
    
    reduced_weight = total_contents_weight * (1.0 - container.WeightReductionRatio)
    RETURN container.BaseWeight + reduced_weight
END FUNCTION
```

#### 2.3. Ma trận Kháng Cắn & Kháng Cào Tuyệt Đối (100% Bite/Scratch Protection Matrix)

Khi đòn tấn công của zombie nhắm vào bộ phận cơ thể $bp$, xác suất xuyên thủng lớp phòng hộ của 1.395 trang bị bảo hộ (`clothing.txt`) được tính qua tích xác suất xuyên qua $K$ lớp giáp:

$$P_{\text{penetrate}}(bp) = \prod_{k=1}^{K} \left( 1.0 - \frac{\text{Protection}_{k}(bp)}{100.0} \right)$$

Khi nhân vật trang bị các vật phẩm phòng hộ $100\%$:
- **Mũ bảo hiểm chống bạo động (`Base.Hat_RiotHelmet`):** $\text{Prot}_{\text{Bite}}(\text{HEAD}) = 100\%$, $\text{Prot}_{\text{Bite}}(\text{NECK}) = 100\% \implies P_{\text{penetrate}} = 0.0$.
- **Áo giáp chống đạn cảnh sát (`Base.Vest_BulletPolice`):** $\text{Prot}_{\text{Bite}}(\text{TORSO\_UPPER}) = 100\% \implies P_{\text{penetrate}} = 0.0$.

```text
FUNCTION Evaluate_Armor_Defense(attacker_attack_type, target_part, worn_clothing_layers):
    penetration_chance = 1.0
    
    FOR EACH layer IN worn_clothing_layers.Covering(target_part):
        layer_defense = (attacker_attack_type == BITE) ? layer.BiteDefense : layer.ScratchDefense
        penetration_chance *= (1.0 - (layer_defense / 100.0))
        
        // Suy hao độ bền của lớp giáp khi hấp thụ đòn đánh
        layer.Condition = Max(0, layer.Condition - 1)
        IF layer.Condition == 0 THEN
            layer.CreateHole(target_part)
        END IF
    ENDFOR

    RETURN (RandomFloat(0.0, 1.0) <= penetration_chance) // TRUE = Giáp bị xuyên thủng
END FUNCTION
```

#### 2.4. Động học Dinh dưỡng Thực phẩm B42 & Tiêu thụ Năng lượng Hàng ngày

Tổng năng lượng cung cấp từ 722 món ăn trong `food.txt` được hấp thụ vào kho đệm trao đổi chất ($E_{\text{buffer}}$):

$$E_{\text{intake}} = \text{Calories} + 4.0 \times (\text{Carbohydrates} + \text{Proteins}) + 9.0 \times \text{Lipids}$$

$$\text{Mức tiêu thụ chuẩn:} \quad E_{\text{burn}} = \begin{cases} [1500, 2000]\text{ kcal/ngày} & \text{khi nghỉ ngơi/làm nhẹ} \\ [3000, 4000]\text{ kcal/ngày} & \text{khi chặt cây, chiến đấu, mang vác nặng} \end{cases}$$

- **Món xào B42 (`PanFriedVegetablesForged`):** $516\text{ kcal}$, cung cấp đầy đủ Carbs & Lipids thực vật giúp chặn đứng tình trạng tụt cân nhanh chóng.

---


================================================================================
# DOCUMENT 03: CRAFTING & TECH TREE GUIDE (1,026 RECIPES & 20 WORKSTATIONS)
================================================================================

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

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


---

# DOCUMENT 04: PROFESSIONS & TRAITS BIBLE (25 OCCUPATIONS & 95 TRAITS)

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

# DOCUMENT 05: WORLD MAP & LOOT DISTRIBUTION GUIDE (11 REGIONS, 362 BASEMENTS & PROCEDURAL DISTRIBUTIONS)

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Xác suất Rơi đồ theo Trọng số Ngẫu nhiên (`ProceduralDistributions.lua` Weighted Random Roll Dynamics)

Trong kiến trúc máy chủ phân bổ thế giới của Project Zomboid Build 42 (tập tin `ProceduralDistributions.lua` gồm 53.419 dòng code), mỗi thùng chứa (`IsoObject` loại `container`) khi được khởi tạo lần đầu trong ô lưới thế giới sẽ thực hiện $R$ lần tung xúc xắc độc lập (`rolls`). Xác suất $P(	ext{item}_i)$ để một vật phẩm cụ thể $	ext{item}_i$ xuất hiện trong một lần tung (`single roll`) được tính bằng tỷ lệ trọng số cá thể $W_i$ trên tổng trọng số $\sum_{j=1}^N W_j$ của toàn bộ bảng loot (`Loot Table`):

$$P_{	ext{roll}}(	ext{item}_i) = rac{W_i}{\sum_{j=1}^N W_j}$$

Đối với bảng loot thực hiện $R$ lần tung độc lập, xác suất để người chơi tìm thấy **ít nhất một** đơn vị vật phẩm $	ext{item}_i$ trong thùng chứa đó ($P_{\ge 1}$) tuân theo phân phối nhị thức bù:

$$P_{\ge 1}(	ext{item}_i, R) = 1 - \left(1 - rac{W_i}{\sum_{j=1}^N W_j}
ight)^R$$

Khi người chơi lục soát một chuỗi gồm $K$ thùng chứa cùng loại (ví dụ: lục soát $K$ hòm gỗ công cụ `CrateTools` trong nhà kho Muldraugh), xác suất tích lũy lũy tiến để nhặt được ít nhất một bảo vật cực hiếm (như Búa tạ `Base.Sledgehammer` hoặc Katana `Base.Katana`) được xác định bởi:

$$P_{	ext{accumulative}}(K) = 1 - \prod_{k=1}^K \left(1 - P_{\ge 1}(	ext{item}_i, R_k)
ight) = 1 - \left(1 - rac{W_i}{\sum W}
ight)^{R \cdot K}$$

```text
FUNCTION Calculate_Procedural_Loot_Roll(container, loot_table_id, sandbox_rarity_modifier):
    // 1. Trích xuất bảng loot từ ProceduralDistributions
    loot_table = ProceduralDistributions.list[loot_table_id]
    total_weight = 0.0
    FOR EACH entry IN loot_table.entries:
        total_weight = total_weight + entry.weight
    END FOR

    // 2. Thực thi số lần tung (rolls)
    spawned_items = []
    rolls = loot_table.rolls * sandbox_rarity_modifier
    
    FOR r = 1 TO rolls:
        random_pick = RandomFloat(0.0, total_weight)
        current_weight = 0.0
        FOR EACH entry IN loot_table.entries:
            current_weight = current_weight + entry.weight
            IF random_pick <= current_weight THEN
                spawned_items.Add(entry.item_id)
                BREAK
            END IF
        END FOR
    END FOR

    RETURN spawned_items
```

---

#### 1.2. Thuật toán Thủ tục Khởi tạo Hầm ngầm 362 Lô `.pzby` (`ProceduralBasementEngine`)

Build 42 đưa vào hệ thống 362 bản đồ ngầm định dạng nhị phân `.pzby` (`media/binmap/`). Xác suất một công trình nhà dân hoặc công trình công cộng sở hữu tầng hầm ngầm ($P_{	ext{basement}}$) được điều phối bởi ma trận vùng ($C_{	ext{zone}}$) và hệ số cấp bậc công trình ($C_{	ext{tier}}$):

$$P_{	ext{basement}} = P_{	ext{base}} 	imes C_{	ext{zone}} 	imes C_{	ext{tier}} 	imes \mathbb{I}_{	ext{valid\_ground}}$$

Trong đó:
- $P_{	ext{base}} = 0.08$ ($8\%$ mặc định trong thiết lập Sandbox chuẩn).
- $C_{	ext{zone}} \in [0.5, 2.5]$ (Rosewood: 0.8, Riverside: 0.6, Muldraugh: 1.5, Irvington: 2.0, Louisville: 2.5).
- Mức ánh sáng tự nhiên tại mọi ô lưới tầng hầm ngầm luôn bị triệt tiêu tuyệt đối:

$$L_{	ext{ambient}}(z_{	ext{basement}}) = 0.0 	ext{ Lux}$$

Bán kính tầm nhìn hiệu dụng $R_{	ext{vis}}$ trong bóng tối hầm ngầm bị giới hạn ngặt nghèo bởi nguồn sáng nhân tạo:

$$R_{	ext{vis}} = egin{cases} 1.0	ext{ block} & 	ext{nếu Không có nguồn sáng (Bóng tối tuyệt đối)} \ 6.0	ext{ blocks} & 	ext{nếu Cầm Đuốc thô sơ / Bật lửa} \ 14.0	ext{ blocks} & 	ext{nếu Đèn pin cứu nạn / Đèn đội đầu công nghiệp} \end{cases}$$

```text
FUNCTION Generate_Procedural_Basement(building):
    // Kiểm tra tính hợp lệ của địa hình nền đất
    IF NOT building.HasValidGroundFootprint() THEN
        RETURN NULL
    END IF

    // Xác suất cơ bản theo vùng và loại công trình
    spawn_chance = 0.08 * building.zone.basement_modifier * building.tier_modifier
    IF RandomFloat(0.0, 1.0) <= spawn_chance THEN
        // Chọn ngẫu nhiên 1 trong 362 layout .pzby phù hợp với chức năng công trình
        pzby_layout = BasementPool.QueryValidLayout(building.type, building.footprint_size)
        
        // Tạo liên kết thẳng đứng (Vertical Depth)
        trapdoor_tile = building.FindOptimalTrapdoorLocation()
        stairs_entity = CreateVerticalStairs(trapdoor_tile, target_z_level = -1)
        
        // Khởi tạo các thùng chứa hầm ngầm (Bunker / Arsenal / Sanatorium)
        basement_instance = LoadAndStitchPzby(pzby_layout, target_z_level = -1)
        RETURN basement_instance
    END IF

    RETURN NULL
```

---

#### 1.3. Phương trình Phân bổ Mật độ Xác sống & Hệ số Rủi ro 11 Vùng Knox County (`RegionalZombieDensityMatrix`)

Mật độ zombie trung bình trên mỗi ô vuông lưới $D_{	ext{zombie}}$ tại vùng bản đồ $Z$ được điều phối bởi hệ số đô thị hóa ($M_{	ext{urban}}$), mật độ dân số lịch sử ($M_{	ext{pop}}$), và hàm lũy tiến theo số ngày sống sót ($t_{	ext{day}}$) cho đến mốc cực đại (`Peak Day` = ngày 28):

$$D_{	ext{zombie}}(Z, t_{	ext{day}}) = D_{	ext{base}}(Z) 	imes M_{	ext{urban}}(Z) 	imes \left(1.0 + rac{\min(t_{	ext{day}}, t_{	ext{peak}})}{t_{	ext{peak}}} 	imes (M_{	ext{peak}} - 1.0)
ight)$$

Trong đó $M_{	ext{peak}} = 1.5$ (Sandbox mặc định), $t_{	ext{peak}} = 28	ext{ ngày}$.

| Mã Vùng (`Zone ID`) | Tên Địa Danh Vùng Bản Đồ | Mật độ Cơ sở ($D_{	ext{base}}$) | Đánh giá Cấp độ Rủi ro | Đặc trưng Cốt lõi & Điểm Nóng (POIs) |
| :--- | :--- | :---: | :---: | :--- |
| `zone:rosewood` | Rosewood, KY | $0.35$ | Thấp - Trung bình | Quy hoạch tập trung; Trạm cứu hỏa (Rìu cứu hỏa, Quần áo chống cắn 100%), Đồn CS, Hiệu sách, Nhà tù Rosewood. |
| `zone:riverside` | Riverside, KY | $0.20$ | Rất Thấp | Ven sông Ohio, tài nguyên cá vô tận; Biệt thự rào sắt kín; Cửa hàng kim khí, Hiệu thuốc, Thư viện trường học. |
| `zone:muldraugh` | Muldraugh, KY | $0.85$ | Trung bình - Cao | Dọc quốc lộ Dixie Highway; Chuỗi nhà kho công nghiệp lớn (Tỷ lệ Búa tạ cao nhất game), Cortman Medical. |
| `zone:westpoint` | West Point, KY | $1.60$ | Rất Cao | Đô thị cổ, ngõ hẹp, cận kề Louisville; Tiệm súng Twiggy's rào sắt kiên cố, Đồn CS vũ khí hạng nặng. |
| `zone:brandenburg` | Brandenburg, KY *(Mới B42)* | $1.20$ | Cao | Cảng công nghiệp ven sông cực tây bắc; Xưởng đóng tàu, cưa kim loại (`metalbandsaw`), đe sắt lớn, phế liệu sắt đúc. |
| `zone:irvington_echocreek` | Irvington & Echo Creek *(Mới B42)* | $0.40$ | Thấp - Trung bình | Đại trang trại gia súc B42 (Bàn thuộc da, thùng bơ, máy ép gạch); Bờ suối dồi dào Đất sét (`Clay`) và Đá lửa (`Chert`). |
| `zone:louisville` | Đại đô thị Louisville | $5.00$ | Cực Kỳ Dày Đặc | Checkpoint Quân sự (M16, Balo ALICE, Kevlar); Bảo tàng Cổ vật (Katana trọng số 8.0); Bệnh viện St. Peregrin. |
| `zone:fallas_lake` | Fallas Lake / Ekron | $0.45$ | Trung bình | Tiệm súng nông thôn, Cửa hàng kim khí, Trạm xăng độc lập trung tâm bản đồ. |
| `zone:valley_station` | Valley Station | $1.40$ | Cao | Ngã 3 chiến lược vào Louisville; Trung tâm mua sắm, Tiệm súng ngã tư đường. |
| `zone:march_ridge` | March Ridge | $1.10$ | Cao | Khu căn hộ quân sự cao tầng; Ký túc xá quân đội, Trường tiểu học, Nhà sách. |

---

#### 1.4. Động học Bản đồ Chú thích & Sự kiện Tòa nhà Đặc biệt (`AnnotatedMap_SurvivorHouse_Probability`)

Khi tiêu diệt zombie, xác suất rơi bản đồ thị trấn có nét vẽ tay và tọa độ ghi chú ($P_{	ext{annotated}}$) là $1.5\%$:

$$P(	ext{Drop Annotated Map}) = 0.015 	imes M_{	ext{loot\_rarity}}$$

Khi người chơi kích hoạt đọc bản đồ chú thích (`OnRead_AnnotatedMap`), hệ thống sẽ ghim vĩnh viễn tọa độ mục tiêu lên bản đồ thế giới (`Minimap Marker`) và kích hoạt sự kiện spawn tòa nhà tương ứng:

1. **Nhà Sinh Tồn Cố Thủ (`Survivor House`)**:
   - $100\%$ các ô cửa sổ và cửa ra vào tầng 1 được đóng ván gỗ 3 lớp (`Barricade Plank = 3`, Độ bền vách $= 3 	imes 1000	ext{ HP}$).
   - $100\%$ tủ bếp chuyển thành bàn loot `KitchenCannedFood` (Đồ hộp bảo quản vĩnh viễn).
   - Tủ quần áo có xác suất $60\%$ spawn tủ súng giấu kín hoặc thùng công cụ hạng nặng.
   - Spawn từ $3	ext{--}8$ zombie cư dân cũ đã bị lây nhiễm bên trong không gian kín.
2. **Nhà Tiệc Tùng / Nhà Sinh Nhật (`Party House`)**:
   - Zombie đội nón sinh nhật (`PartyHat`), bàn ăn spawn bánh kem sinh nhật, rượu vang và đĩa nhạc kẹp trong máy hát.

---

### 2. Bảng Hằng Số Phân Bổ Bản Đồ, Trọng Số Loot & Tỷ Lệ Xuất Hiện (System Constants Table)

#### Bảng Tra Cứu Trọng Số Rơi Đồ Quý Hiếm (`ProceduralDistributions.lua` Extraction)

| Tên Vật Phẩm | Mã Script Identifier | Trọng Số (`Weight`) | Bảng Loot Chứa Đồ (`Loot Tables`) | Địa Điểm Xuất Hiện Thực Tế Trên Bản Đồ |
| :--- | :--- | :---: | :--- | :--- |
| **Búa tạ (Sledgehammer)** | `Base.Sledgehammer` | $0.01	ext{--}0.10$ | `CrateTools`, `GarageTools`, `ArmyHangarTools`, `FireStorageTools` | Thùng gỗ kho bãi Muldraugh/Riverside, Nhà để xe, Trạm cứu hỏa, Xe công trình công cộng ven đường. |
| **Máy phát điện (Generator)** | `Base.Generator` | $10.0$ | `CrateGenerator` *(Chuyên dụng)* | Dãy kho bãi công nghiệp, Dãy Mini-storage, Nhà kho sau vườn nhà dân. |
| **Sách Dùng Máy Phát Điện** | `Base.ElectronicsMag4` | $1.0	ext{--}5.0$ | `BookstoreMisc`, `LibraryBooks`, `PostOfficeBooks`, `LivingRoomSideTable` | Hiệu sách Rosewood, Thư viện trường học Riverside, Bưu điện, Tủ đầu giường nhà dân. |
| **Kiếm Katana Nhật Cổ** | `Base.Katana` | $8.0$ *(Bảo tàng)*<br>$0.001$ *(Thường)* | `MedievalWeaponsJapan`<br>`ComicStoreCounter`, `CyberCafeDesk`, `Hobbies` | Bảo tàng Cổ vật Louisville, Tiệm truyện tranh, Quán net cafe, Xác zombie cắm kiếm sau ngày 60. |
| **Súng Trường Tự Động M16** | `Base.AssaultRifle` | $10.0$ *(Quân sự)*<br>$4.0	ext{--}8.0$ | `ArmyStorageGuns`<br>`SWATStorageGuns`, `DrugLabGuns` | Trại kiểm soát quân sự Checkpoint Louisville, Kho vũ khí SWAT, Phòng ngầm điều chế chất cấm. |
| **Súng Bắn Đạn Chùm JS-2000** | `Base.Shotgun` | $10.0$ | `FirearmWeapons`, `DrugLabGuns`, `PawnShopGunsSpecial` | Tiệm súng Twiggy's West Point, Tiệm cầm đồ, Đồn cảnh sát, Tủ súng nhà dân. |
| **Thuốc Kháng Sinh Y Tế** | `Base.Antibiotics` | $4.0$ *(Trạm xá)*<br>$0.5$ *(Dân cư)* | `DoctorTools`, `DoctorOutfit`<br>`BathroomCabinet`, `BathroomShelf` | Bàn bác sĩ trạm xá Cortman, Xe cứu thương, Bệnh viện St. Peregrin, Tủ gương thuốc phòng tắm. |

---


---

# DOCUMENT 06: VEHICLES & MECHANICS HANDBOOK (52 VEHICLE MODELS, 15 MECHANICAL PARTS & HOTWIRING DYNAMICS)

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



# DOCUMENT 07: MEDIA BROADCASTS, KNOX LORE & 64 SKILL VHS BIBLE (INSTRUCTIONAL MEDIA, XP MULTIPLIERS & AEBS ORACLE)

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Cộng hưởng Kinh nghiệm Bản ghi Tri thức & Hệ số Sách Kỹ năng 5 Bậc (Instructional Record & Skill Book XP Multiplier)



Khi nhân vật lĩnh hội một dòng tri thức (Line Entry) có gắn cờ mã kỹ năng (như `COO+1`, `CRP+1`, `FIS+1`, `HUS+1`, `FKN+1`, `BLA+1`, `POT+1`) từ Tín hiệu Phát sóng Trực tiếp hoặc Bản ghi Lưu trữ (trừu tượng hóa từ TV / Retail-VHS / Home-VHS), lượng điểm kinh nghiệm thực nhận $\Delta \text{XP}_{\text{media}}$ cho kỹ năng $k$ ở cấp độ hiện tại $L(k) \in [0, 10]$ phụ thuộc vào bậc hệ số Sách Kỹ năng $M_{\text{book}}(L(k))$ đã đọc, hệ số cấp khởi đầu $M_{\text{init}}(L_0(k))$ (từ Doc 04) và đặc điểm `Fast Learner` ($+30\%$):

$$M_{\text{book}}(L(k)) = \begin{cases} 3.0 & \text{nếu } L(k) \in \{1, 2\} \text{ và đã đọc Sách Tập 1 (Vol 1)} \\ 5.0 & \text{nếu } L(k) \in \{3, 4\} \text{ và đã đọc Sách Tập 2 (Vol 2)} \\ 8.0 & \text{nếu } L(k) \in \{5, 6\} \text{ và đã đọc Sách Tập 3 (Vol 3)} \\ 12.0 & \text{nếu } L(k) \in \{7, 8\} \text{ và đã đọc Sách Tập 4 (Vol 4)} \\ 16.0 & \text{nếu } L(k) \in \{9, 10\} \text{ và đã đọc Sách Tập 5 (Vol 5)} \\ 1.0 & \text{nếu chưa đọc Sách Kỹ năng tương ứng với cấp hiện tại} \end{cases}$$

$$\Delta \text{XP}_{\text{media}}(k) = \Delta \text{XP}_{\text{line\_base}}(k) \times M_{\text{book}}(L(k)) \times \left(1 + 0.30 \cdot \mathbb{I}_{\text{FastLearner}}\right) \times M_{\text{init}}(L_0(k))$$

(Ví dụ kiểm chứng từ tài liệu: Khi kết hợp Sách Tập 1 ($\times 3$) với `Fast Learner` ($+30\%$), hệ số nhân gộp đạt $3.0 \times 1.30 = 3.9\times$, biến một chương trình Mộc cơ bản $75\text{ XP}$ thành $225\text{--}300\text{ XP}$).

#### 1.2. Phương trình Quy tắc Chồng XP Độc lập & Loại trừ Trùng lặp (`Live Broadcast` vs `Standard Archive` vs `Unique Field Record`)



Gọi $E_{\text{id}}$ là định danh của một phân đoạn nội dung tri thức. Quy tắc kiểm tra chống nhận trùng điểm kinh nghiệm (Idempotency & Stacking Rule) giữa 3 nguồn phát $\text{Source} \in \{\text{LIVE\_BROADCAST}, \text{RETAIL\_ARCHIVE}, \text{HOME\_FIELD\_RECORD}\}$ được xác định bởi hàm chỉ thị $\mathbb{I}_{\text{grant}}(p, E_{\text{id}}, \text{Source})$ cho người chơi $p$:

$$\mathbb{I}_{\text{grant}}(p, E_{\text{id}}, \text{Source}) = \begin{cases} 1 & \text{nếu } \text{Source} = \text{LIVE\_BROADCAST} \land E_{\text{id}} \notin \mathcal{H}_{\text{watched}}(p) \\ 1 & \text{nếu } \text{Source} = \text{RETAIL\_ARCHIVE} \land E_{\text{id}} \notin \mathcal{H}_{\text{watched}}(p) \\ 0 & \text{nếu } \text{Source} = \text{RETAIL\_ARCHIVE} \land E_{\text{id}} \in \mathcal{H}_{\text{watched}}(p) \ (\text{đã xem trên Live Broadcast}) \\ 1 & \text{nếu } \text{Source} = \text{HOME\_FIELD\_RECORD} \land E_{\text{id}} \notin \mathcal{H}_{\text{home\_watched}}(p) \ (\text{Độc lập 100\% với Live Broadcast}) \end{cases}$$

```text
FUNCTION Process_Instructional_Media_Line(player, media_item, line_entry):
    // 1. Kiểm tra quy tắc chồng XP (Stacking Rule: Live vs Retail vs Home)
    IF media_item.Category IN ["LIVE_BROADCAST", "RETAIL_ARCHIVE"] THEN
        // Retail VHS trùng nội dung với TV: nếu đã xem tập đó trên TV thì KHÔNG nhận thêm XP
        IF player.HasConsumedBroadcastLine(line_entry.SharedEpisodeLineID) == TRUE THEN
            RETURN XP_GRANT_BLOCKED_ALREADY_WATCHED
        END IF
        player.MarkBroadcastLineConsumed(line_entry.SharedEpisodeLineID)
    ELSE IF media_item.Category == "HOME_FIELD_RECORD" THEN
        // Home VHS luôn trao XP độc lập với TV, chỉ kiểm tra xem chính cuộn băng đó đã xem chưa
        IF player.HasConsumedHomeRecordLine(line_entry.UniqueHomeLineID) == TRUE THEN
            RETURN XP_GRANT_BLOCKED_ALREADY_WATCHED
        END IF
        player.MarkHomeRecordLineConsumed(line_entry.UniqueHomeLineID)
    END IF

    // 2. Mở khóa Công thức Chế tác Đặc biệt (nếu dòng thoại gắn Recipe Unlock)
    FOR EACH recipe_id IN line_entry.UnlockedRecipes DO
        player.UnlockRecipe(recipe_id)
    END FOR

    // 3. Tính toán XP với hệ số Sách Kỹ năng 5 bậc (x3, x5, x8, x12, x16)
    skill_id = line_entry.TargetSkill
    current_level = player.GetSkillLevel(skill_id)
    book_mult = player.GetActiveSkillBookMultiplier(skill_id, current_level) // 1.0, 3.0, 5.0, 8.0, 12.0, 16.0
    
    fast_learner_mult = 1.0
    IF player.HasTrait("base:fastlearner") THEN
        fast_learner_mult = 1.30
    END IF

    total_xp = line_entry.BaseXP * book_mult * fast_learner_mult
    player.AddSkillXP(skill_id, total_xp)
    RETURN XP_GRANT_SUCCESS
END FUNCTION

```

#### 1.3. Thuật toán Khởi tạo Tần số Điềm báo Khẩn cấp & Dự báo Siêu sự kiện (`ISDynamicRadio` / AEBS Oracle Engine)



Khi khởi tạo một thế giới mới, tần số của Hệ thống Tín hiệu Dự báo Khẩn cấp (`AEBS` $\rightarrow$ trừu tượng hóa thành `World_Event_Telemetry_Oracle`) được chọn ngẫu nhiên rời rạc trong dải $[88000, 108000]\text{ Hz}$ (tương đương $88.0\text{ MHz}$ đến $108.0\text{ MHz}$) với bước nhảy $\Delta f = 200\text{ Hz}$ ($0.2\text{ MHz}$), tổng cộng có $101$ tần số khả dĩ:

$$f_{\text{AEBS}} = 88000 + 200 \times k, \quad k \sim \mathcal{U}_{\text{int}}(0, 100) \quad (\text{Đơn vị hiển thị: } \frac{f_{\text{AEBS}}}{1000} \in [88.0, 108.0]\text{ MHz})$$

```text
FUNCTION Initialize_World_Telemetry_Oracle_Frequency():
    // Tương đương hàm radio:getRandomFrequency(88000, 108000) với bước nhảy 0.2 MHz (200 đơn vị)
    step_index = RandomInt(0, 100)
    aebs_frequency_khz = 88000 + (step_index * 200)
    RETURN aebs_frequency_khz
END FUNCTION

FUNCTION Broadcast_Hourly_Telemetry_Oracle(world_time_hour, world_day):
    // Phát định kỳ vào đầu mỗi giờ (XX:00)
    IF world_time_hour.Minutes != 00 THEN RETURN END IF

    EmitSignalHeader("<bzzt> Fiver Zero Two. <wzzt> Automated Broadcast System. <bzzt>")
    EmitWeatherForecast(window_hours_min = 24, window_hours_max = 48) // Gió, sương mù, bão trước 24-48h

    // Cảnh báo Siêu sự kiện Trên không (Helicopter / Aerial Acoustic Lure)
    IF IsAerialLureScheduledToday(world_day) AND world_time_hour.Hour >= 9 THEN
        EmitAlert("Air Activity detected.") // Sự kiện sẽ kích hoạt trong khung 12:00 - 19:00 cùng ngày
    END IF

    // Cảnh báo 3 giai đoạn sụp đổ lưới năng lượng (Power Grid Blackout)
    IF HoursUntilPowerShutoff() >= 48 AND HoursUntilPowerShutoff() <= 72 THEN
        EmitAlert("Knox Power Grid: Power fluctuations detected.")
    ELSE IF HoursUntilPowerShutoff() > 0 AND HoursUntilPowerShutoff() < 48 THEN
        EmitAlert("Knox Power Grid: Systems failing. Network compromised.")
    ELSE IF HoursUntilPowerShutoff() <= 0 THEN
        EmitAlert("Knox Power Grid: Blackout.")
    END IF
END FUNCTION

```

---

### 2. Bảng Hằng Số Truyền Thông, Hệ Số Sách Kỹ Năng & Lịch Trình Sự Kiện (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Media Database** | `RadioData_Broadcast_Entries` | `> 1120` | `entries` | Tổng số phân đoạn phát sóng trong `RadioData.xml` / `RadioData.json`.

 |
| **Media Database** | `RadioData_Dialogue_Lines` | `13166` | `lines` | Tổng số câu thoại phát sóng trên toàn hệ thống.

 |
| **Media Database** | `Total_Broadcast_Channels` | `18` | `channels` | Tổng số kênh truyền hình, phát thanh thương mại, dân sự và quân sự.

 |
| **Media Database** | `Total_Recorded_Media_Items` | `352` | `items` | Tổng số bản ghi lưu trữ trong `recorded_media.lua` ($155 + 128 + 69$).

 |
| **Media Database** | `Retail_VHS_Count` | `155` | `tapes` | Số lượng bản ghi thương mại sản xuất hàng loạt (Retail-VHS).

 |
| **Media Database** | `Home_VHS_Count` | `128` | `tapes` | Số lượng bản ghi thực địa/gia đình độc bản (Home-VHS).

 |
| **Media Database** | `Audio_CD_Count` | `69` | `discs` | Số lượng đĩa âm thanh giải trí (CDs).

 |
| **Media Database** | `Skill_And_Recipe_VHS_Count` | `64` | `tapes` | Số lượng bản ghi VHS trao điểm XP kỹ năng và mở khóa công thức chế tạo.

 |
| **Skill Book Mult** | `SkillBook_Vol1_Multiplier` | `3.0` (`3.9` w/ Fast Learner) | `multiplier (Lv 1-2)` | Hệ số nhân XP cho Cấp 1–2 khi đọc xong Sách Tập 1.

 |
| **Skill Book Mult** | `SkillBook_Vol2_Multiplier` | `5.0` | `multiplier (Lv 3-4)` | Hệ số nhân XP cho Cấp 3–4 khi đọc xong Sách Tập 2.

 |
| **Skill Book Mult** | `SkillBook_Vol3_Multiplier` | `8.0` | `multiplier (Lv 5-6)` | Hệ số nhân XP cho Cấp 5–6 khi đọc xong Sách Tập 3.

 |
| **Skill Book Mult** | `SkillBook_Vol4_Multiplier` | `12.0` | `multiplier (Lv 7-8)` | Hệ số nhân XP cho Cấp 7–8 khi đọc xong Sách Tập 4.

 |
| **Skill Book Mult** | `SkillBook_Vol5_Multiplier` | `16.0` | `multiplier (Lv 9-10)` | Hệ số nhân XP cho Cấp 9–10 khi đọc xong Sách Tập 5.

 |
| **Broadcast Schedule** | `LifeAndLiving_Channel_ID` | `203` | `channel_id` | Kênh đào tạo kỹ năng sinh tồn chủ lực (phát từ Ngày 0 đến Ngày 8).

 |
| **Broadcast Schedule** | `LifeAndLiving_Daily_Slots` | `06:00 / 12:00 / 18:00` | `24h clock` | 3 khung giờ cố định mỗi ngày: `06:00` (Cooking), `12:00` (Carpentry), `18:00` (Exposure Survival).

 |
| **Broadcast Schedule** | `TV_Signal_Termination_Day` | `Day 8 (after 06:00)` | `world_day` | Thời điểm tín hiệu truyền hình trực tiếp tắt vĩnh viễn, chuyển sang nhiễu trắng (Static).

 |
| **Broadcast Schedule** | `Woodcraft_Single_Ep_XP` | `75` (`225 - 300` w/ Vol 1) | `XP` | Lượng XP gốc của 1 tập chương trình Mộc và lượng XP sau khi nhân Sách Tập 1.

 |
| **Broadcast Channels** | `Other_TV_Channel_IDs` | `200, 201, 204..210` | `channel_id` | Các kênh phụ trợ: `200` (Triple-N), `201` (WBLN), `204` (TURBO: Boredom `-1`), `205` (PawsTV), `206` (KPATV), `207` (Music), `208` (Sports), `209` (Brennan Movie), `210` (GBC).

 |
| **Broadcast Channels** | `Civilian_Radio_Frequency` | `91.2` | `MHz` | Tần số đài nghiệp dư dân sự phát tín hiệu cầu cứu sau khi sụp đổ.

 |
| **Special VHS Stats** | `TheDogGoblin_Horror_Effect` | `Stress: +0.2 (STS+0.2) | +Panic` | `stat_delta` | Biến đổi tâm lý khi xem bản ghi kinh dị *The Dog Goblin*.

 |
| **Special VHS Stats** | `BasicGunHandling_Skill_Grant` | `Aiming +3, Reloading +3` | `skill_levels` | Bản ghi thực địa quý nhất (`AIM+2, REL+2, AIM+1, REL+1`), nâng thẳng Bắn súng & Nạp đạn lên Cấp 3–4.

 |
| **AEBS Oracle** | `AEBS_Frequency_Range` | `88000 - 108000` (`88.0 - 108.0`) | `Hz / MHz` | Dải tần số ngẫu nhiên của Đài Khẩn cấp Tự động trong mỗi thế giới mới.

 |
| **AEBS Oracle** | `AEBS_Frequency_Step` | `0.2` (`200`) | `MHz (Hz)` | Bước nhảy chia lưới tần số của hàm `getRandomFrequency(88000, 108000)`.

 |
| **AEBS Oracle** | `AEBS_Broadcast_Interval` | `XX:00` (Mỗi giờ) | `hourly_tick` | Chu kỳ phát bản tin dự báo tự động vào đúng phút thứ 00 mỗi giờ.

 |
| **AEBS Oracle** | `AirActivity_Warning_Window` | `Alert >= 09:00 | Event: 12:00-19:00` | `24h clock` | Khung giờ phát cảnh báo `"Air Activity detected."` và khung giờ sự kiện bay qua thực tế.

 |
| **AEBS Oracle** | `Power_Fluctuation_Lead_Time` | `48 - 72` | `hours (giờ)` | Thời gian báo trước của cảnh báo giai đoạn 1 trước khi mất điện toàn cục.

 |
| **AEBS Oracle** | `Weather_Forecast_Lead_Time` | `24 - 48` | `hours (giờ)` | Thời gian dự báo trước về gió, sương mù và bão tuyết/bão nhiệt đới.

 |
| **World Collapse** | `World_Collapse_Timeline_Days` | `14` (`Day 0 = July 9, 1993`) | `days` | Chu kỳ 14 ngày sụp đổ hoàn toàn của hệ thống hạ tầng và phát sóng thế giới (Ngày 7 bùng phát chủng lây qua không khí).

 |
| **Stealth Tactic** | `Safe_TV_Volume_Level` | `1 - 2` | `volume_bars` | Mức âm lượng an toàn khi nghe/xem thiết bị phát để không thu hút thực thể thù địch bên ngoài.

 |

---

# DOCUMENT 08: FARMING, TRAPPING, FISHING & WILDERNESS SURVIVAL HANDBOOK (43 CROPS/HERBS, 6 TRAPS, 21 FISH, 18 GENES & 27 FORAGE CATEGORIES)

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

# DOCUMENT 09: CLIMATE, SEASONS & WORLD EROSION BIBLE (12 MONTHS, SEVERE WEATHER, 37.0°C CORE EQUILIBRIUM & 100-DAY EROSION CURVE)

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

# DOCUMENT 10: SANDBOX CUSTOMIZATION & RULES BIBLE (5 PRESETS, POPULATION DIRECTOR, 21 LOOT RARITY & 35-SKILL MULTIPLIERCONFIG)

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

# DOCUMENT 11: COMBAT, BALLISTICS & STEALTH BIBLE (6 MELEE CLASSES, 22 FIREARMS, BERNOULLI DECAY & JAW STAB DYNAMICS)

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Sát thương Cận chiến Thực tế & Sự Sụp đổ do Moodle (Melee Final Damage & Moodle Collapse)

Trong kiến trúc Bullet Physics C++ và runtime Java của Project Zomboid Build 42, sát thương gây ra lên một cá thể xác sống khi trúng đòn cận chiến được xác định theo phương trình tích đa nhân tử:

$$	ext{Damage}_{	ext{Final}} = 	ext{BaseDamage} 	imes M_{	ext{Strength}} 	imes M_{	ext{Skill}} 	imes M_{	ext{Moodle}} 	imes M_{	ext{Position}} 	imes M_{	ext{Crit}}$$

Trong đó:
1. **$	ext{BaseDamage}$**: Biến ngẫu nhiên đều phân bố trong khoảng $[	ext{MinDamage}, 	ext{MaxDamage}]$ của vũ khí.
2. **$M_{	ext{Strength}}$ (Hệ số Sức mạnh Thể chất)**:
   $$M_{	ext{Strength}} = egin{cases} 0.60 + (	ext{Strength} 	imes 0.075) & 	ext{nếu } 	ext{Strength} \in [0, 4] \quad (	ext{Phạt } -40\% 	ext{ đến } -10\%) \ 1.00 & 	ext{nếu } 	ext{Strength} = 5 \quad (	ext{Mức bình thường}) \ 1.10 + ((	ext{Strength} - 6) 	imes 0.10) & 	ext{nếu } 	ext{Strength} \in [6, 8] \quad (+10\% 	ext{ đến } +30\%) \ 1.40 + ((	ext{Strength} - 9) 	imes 0.20) & 	ext{nếu } 	ext{Strength} \in [9, 10] \quad (	ext{Stout / Strong: } +40\% 	ext{ đến } +60\%) \end{cases}$$
3. **$M_{	ext{Skill}}$ (Hệ số Kỹ năng Chuyên môn Vũ khí: Axe, Long Blade, Blunt, Spear, Short Blade)**:
   $$M_{	ext{Skill}} = 1.0 + (	ext{SkillLevel} 	imes 0.10) \quad (	ext{Cấp 10 mang lại } +100\% 	ext{ sát thương})$$
4. **$M_{	ext{Position}}$ (Vị thế Không gian Mục tiêu)**:
   $$M_{	ext{Position}} = egin{cases} 1.0 & 	ext{khi Zombie đứng thẳng} \ 1.5 & 	ext{khi Zombie ngã bò trên mặt đất} \ 2.5	ext{--}3.0 & 	ext{khi Dẫm chân lên đầu (Head Stomp - Tiêu diệt tức thì)} \end{cases}$$
5. **$M_{	ext{Crit}}$ (Hệ số Bạo kích Chí mạng)**:
   $$M_{	ext{Crit}} = egin{cases} 	ext{CritDmgMultiplier} \in [2.0, 6.0] & 	ext{với xác suất } P_{	ext{crit}} \ 1.0 & 	ext{với xác suất } 1 - P_{	ext{crit}} \end{cases}$$
6. **$M_{	ext{Moodle}}$ (Ma trận Trừng phạt Trạng thái Cơ thể - Kẻ Sát Nhân Thể Lực)**:
   $$M_{	ext{Moodle}} = M_{	ext{Exhaustion}} 	imes M_{	ext{Drowsiness}} 	imes M_{	ext{Pain}}$$
   - **Mệt mỏi (Exhaustion)**: Cấp 1 (Moderate) phạt $-50\%$, Cấp 2 (High) phạt $-80\%$, Cấp 3 (Excessive) phạt $-90\%$, Cấp 4 (Total Exhaustion) phạt $-95\%$ (Sát thương tụt dưới $0.1$).
   - **Buồn ngủ (Drowsiness)**: Phạt lũy tiến từ $-50\%$ đến $-95\%$ sát thương.
   - **Hoảng sợ (Panic)**: Tuyệt đối **KHÔNG GIẢM SÁT THƯƠNG CẬN CHIẾN**, chỉ giảm xác suất bạo kích $P_{	ext{crit}}$ từ $-10\%$ đến $-30\%$.

```text
FUNCTION Calculate_Melee_Damage(player, weapon, target_zombie, is_head_stomp):
    // 1. Roll sát thương gốc
    base_dmg = RandomFloat(weapon.minDamage, weapon.maxDamage)
    
    // 2. Hệ số Strength và Weapon Skill
    str_level = player.GetPerkLevel(Perks.Strength)
    skill_level = player.GetPerkLevel(weapon.perkType)
    m_str = Calculate_Strength_Multiplier(str_level)
    m_skill = 1.0 + (skill_level * 0.10)
    
    // 3. Hệ số Moodle (Exhaustion là chí tử)
    m_exhaust = Lookup_Exhaustion_Penalty(player.GetMoodleLevel(Moodles.Endurance))
    m_drowsy = Lookup_Drowsiness_Penalty(player.GetMoodleLevel(Moodles.Tired))
    m_moodle = m_exhaust * m_drowsy

    // 4. Vị thế mục tiêu
    IF is_head_stomp THEN
        m_pos = 2.75
    ELSE IF target_zombie.IsProne() THEN
        m_pos = 1.5
    ELSE
        m_pos = 1.0
    END IF

    // 5. Kiểm tra bạo kích
    panic_penalty = player.GetMoodleLevel(Moodles.Panic) * 0.075
    effective_crit_chance = Max(0.0, weapon.critChance - panic_penalty)
    IF RandomFloat(0.0, 1.0) <= effective_crit_chance THEN
        m_crit = weapon.critMultiplier
    ELSE
        m_crit = 1.0
    END IF

    final_dmg = base_dmg * m_str * m_skill * m_moodle * m_pos * m_crit
    RETURN final_dmg
```

---

#### 1.2. Công thức Độ bền Vũ khí theo Phân phối Bernoulli & Kỹ năng Bảo trì (`Maintenance`)

Mỗi cú đánh trúng đích, xác suất mất 1 điểm độ bền ($P_{	ext{decay}}$) được xác định bằng phân phối Bernoulli độc lập:

$$P_{	ext{decay}} = rac{1}{	ext{ConditionLowerChanceOneIn} + (	ext{MaintenanceLevel} 	imes 2) + \lfloorrac{	ext{WeaponSkillLevel}}{2}
floor}$$

Số lần đánh trung bình trước khi vũ khí hỏng hoàn toàn từ độ bền tối đa $C_{	ext{max}}$:

$$E[	ext{Hits}] = C_{	ext{max}} 	imes \left( 	ext{ConditionLowerChanceOneIn} + (	ext{MaintenanceLevel} 	imes 2) + \lfloorrac{	ext{WeaponSkillLevel}}{2}
floor 
ight)$$

---

#### 1.3. Phương trình Vật lý Đạn đạo Súng Đạn & Độ Chính Xác Aiming 0 $
ightarrow$ 10

Xác suất bắn trúng thực tế của viên đạn ($	ext{HitChance}_{	ext{Actual}}$) phụ thuộc vào cấp độ ngắm bắn `Aiming`, cự ly bắn $d$, trạng thái di chuyển và mức độ hoảng sợ:

$$	ext{HitChance}_{	ext{Actual}} = \left( 	ext{BaseHitChance} + (	ext{AimingLevel} 	imes 0.07) - 	ext{Penalty}_{	ext{dist}}(d) - 	ext{Penalty}_{	ext{move}} 
ight) 	imes M_{	ext{Panic}}$$

Trong đó:
- **Aiming Cấp 0–2**: Súng ngắn/súng trường phạt $-70\%	ext{ đến }-80\%$ trúng đích khi $d > 5	ext{ m}$. Chỉ có **Shotgun** (JS-2000, Double Barrel) mới đảm bảo trúng $65\%	ext{--}70\%$ vì cơ chế bắn chùm đạn ghém (*Pellets* trúng cùng lúc 3–4 mục tiêu), cấp trọn vẹn điểm XP cho từng con trúng đạn.
- **Aiming Cấp 8–10**: Mở khóa cơ chế **Đạn Xuyên Mục Tiêu (Bullet Piercing)**: 1 viên đạn súng trường cỡ `.308 Win` xuyên qua 2 mục tiêu đứng thẳng hàng.
- **Bán kính phát tán âm thanh súng đạn ($R_{	ext{sound}}$)**: Lan truyền sóng âm từ $60	ext{ ô}$ (Revolver .38 Special) đến $200	ext{ ô gạch}$ (Shotgun JS-2000).

---

#### 1.4. Động lực học Ẩn nấp: Bán kính Tiếng bước chân, Nón Tầm nhìn Zombie & Đòn Ám sát Một Hit Tức Tử (`Jaw Stab`)

Bán kính vòng tròn tiếng ồn bước chân phát ra ($R_{	ext{footstep}}$) phụ thuộc vào tốc độ di chuyển, loại giày và kỹ năng `Lightfoot`:

$$R_{	ext{footstep}} = R_{	ext{base}}(	ext{Stance}, 	ext{Footwear}) 	imes (1.0 - 	ext{LightfootLevel} 	imes 0.05)$$

Cấu trúc 3 vùng giác quan của Zombie:
1. **Nón Tầm nhìn Chính diện ($120^\circ$)**: Tầm xa phát hiện $15	ext{--}20	ext{ ô gạch}$ ban ngày, nhận diện mục tiêu đứng thẳng sau $0.5	ext{s}$.
2. **Tầm nhìn Ngoại vi ($180^\circ$)**: Nhận diện chuyển động nhanh; nếu ngồi rón rén (`Crouch`), tỷ lệ phát hiện giảm $70\%$.
3. **Điểm Mù Sau Lưng ($180^\circ$ Bán cầu sau)**: Zombie hoàn toàn **MÙ** ở nửa bán cầu phía sau lưng.

**Đòn Ám Sát Một Hit Tức Tử (Jaw Stab / Stealth Kill)**:
Khi tiếp cận zombie từ phía sau trong điểm mù ở cự ly $\le 0.5	ext{ ô gạch}$ bằng dao ngắn (`Small Blade`):
- Nhân vật tự động đặt tay trái tì lên vai zombie.
- Nhấp chuột tấn công $
ightarrow$ Kích hoạt đòn **Đâm Xuyên Cằm (Jaw Stab)**:
  - **Tử vong tức thì $100\%$ (Instant Kill)** mọi loại zombie.
  - Hoàn toàn im lặng ($R_{	ext{sound}} = 0	ext{ ô}$).
  - **Bảo toàn nguyên vẹn $100\%$ trang phục** (không rách, không vấy máu áo giáp Kevlar / quân phục).

---

### 2. Bảng Hằng Số Vật Lý Chiến Đấu, Súng Đạn & Độ Bền (System Constants Table)

#### Bảng Xếp Hạng Độ Bền Thực Tế & Tỷ Lệ Mất Độ Bền Cận Chiến

| Tên Vũ Khí | Danh Mục Hệ Vũ Khí | Độ Bền Tối Đa ($C_{	ext{max}}$) | Tỷ Lệ Gốc (`1 in X`) | Tỷ Lệ Tại Maintenance 5 | Số Zombie Hạ Gục Ước Tính |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Xà beng (Crowbar)** | Long Blunt | **15** | **1 in 70** | **1 in 80–85** | **> 1.000 zombie** (Bền nhất game, cày Maintenance tối ưu). |
| **Rìu cứu hỏa (Fire Axe)** | Axe | **15** | **1 in 35** | **1 in 45–50** | **400–600 zombie** (Sát thương cực lớn, phá cửa). |
| **Gậy bóng chày đinh (Spiked Bat)** | Long Blunt | **15** | **1 in 25** | **1 in 35** | **350–450 zombie** (Dễ chế tạo, tầm đánh an toàn). |
| **Thanh kiếm Katana Nhật** | Long Blade | **10** | **1 in 15** | **1 in 25** | **150–250 zombie** (Sát thương 8.0-8.0 đỉnh cao, chém 3 con). |
| **Giáo gỗ tự chế (Crafted Spear)** | Spear | **5** | **1 in 2** | **1 in 12** | **10–60 zombie** (Crit 1-hit tầm xa, rất nhanh gãy). |
| **Búa thợ mộc (Hammer)** | Small Blunt | **10** | **1 in 30** | **1 in 40** | **250–350 zombie** (Dắt thắt lưng, kiêm công cụ xây dựng). |
| **Dao săn (Hunting Knife)** | Small Blade | **10** | **1 in 25** | **1 in 35** | **Ám sát Jaw Stab vô hạn** (Tốn cực ít thể lực). |

---

#### Bảng Thông Số Đạn Đạo 22 Khẩu Súng Build 42

| Tên Khẩu Súng (ID Script) | Cỡ Đạn (Caliber) | Hộp Tiếp Đạn | Sát Thương | Tầm Bắn | Bán Kính Âm Thanh | Độ Chính Xác Gốc | Độ Giật | Tỷ Lệ Kẹt Đạn |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `Base.AssaultRifle` (M16) | `5.56mm` | 30 viên | $1.0	ext{--}1.6$ | $30	ext{ ô}$ | **$150	ext{ ô gạch}$** | $50\%$ | 15 | $1\%$ |
| `Base.AssaultRifle2` (M14) | `.308 Win` | 20 viên | $1.2	ext{--}2.0$ | **$40	ext{ ô}$** | **$170	ext{ ô gạch}$** | $50\%$ | 20 | $1\%$ |
| `Base.Shotgun` (JS-2000) | `Shotgun Shells` | 6 viên | **$1.5	ext{--}2.2$** | $12	ext{ ô}$ | **$200	ext{ ô gạch}$** | **$65\%$ (Bắn chùm)** | 30 | $2\%$ |
| `Base.ShotgunSawnoff` (Cưa nòng) | `Shotgun Shells` | 6 viên | $1.3	ext{--}2.0$ | $10	ext{ ô}$ | **$200	ext{ ô gạch}$** | **$70\%$** | 30 | $2\%$ |
| `Base.DoubleBarrelShotgun` | `Shotgun Shells` | 2 viên | **$1.5	ext{--}2.2$** | $15	ext{ ô}$ | **$200	ext{ ô gạch}$** | **$65\%$** | 30 | $0\%$ (Không kẹt) |
| `Base.DoubleBarrelSawnoff` | `Shotgun Shells` | 2 viên | $1.2	ext{--}1.8$ | $8	ext{ ô}$ | **$200	ext{ ô gạch}$** | **$70\%$ (Góc rộng)** | 34 | $0\%$ (Không kẹt) |
| `Base.JS3T_Shotgun` (B42 Mới) | `Shotgun Shells` | 8 viên | $1.5	ext{--}2.2$ | $12	ext{ ô}$ | **$200	ext{ ô gạch}$** | $65\%$ | 28 | $2\%$ |
| `Base.HuntingRifle` (MSR700) | `.308 Win` | 3 viên nạp tay | $1.2	ext{--}2.0$ | **$40	ext{ ô}$** | **$170	ext{ ô gạch}$** | $50\%$ | 23 | $1\%$ |
| `Base.MSR7T_Rifle` (B42 Mới) | `.308 Win` | 5 viên | $1.2	ext{--}2.0$ | **$40	ext{ ô}$** | **$170	ext{ ô gạch}$** | $50\%$ | 21 | $1\%$ |
| `Base.VarmintRifle` (MSR788) | `5.56mm` | 3 viên | $1.0	ext{--}1.6$ | $30	ext{ ô}$ | $150	ext{ ô gạch}$ | $50\%$ | 17 | $1\%$ |
| `Base.JS14_Rifle` (B42 Mới) | `5.56mm` | 20 viên | $1.0	ext{--}1.6$ | $30	ext{ ô}$ | $150	ext{ ô gạch}$ | $45\%$ | 15 | $2\%$ |
| `Base.L94_Rifle` (B42 Đòn bẩy) | `.30-30 Win` | 6 viên | $1.2	ext{--}2.0$ | $30	ext{ ô}$ | $160	ext{ ô gạch}$ | $50\%$ | 20 | $1\%$ |
| `Base.L92_Carbine` (B42 Đòn bẩy)| `.357 Mag` | 8 viên | $1.0	ext{--}1.6$ | $24	ext{ ô}$ | $120	ext{ ô gạch}$ | $50\%$ | 12 | $1\%$ |
| `Base.TrapperCarbine` (B42 Mới) | `.45 Auto` | 10 viên | $1.0	ext{--}1.4$ | $20	ext{ ô}$ | **$70	ext{ ô gạch}$ (Rất êm)**| $50\%$ | 12 | $2\%$ |
| `Base.Pistol` (Beretta M9) | `9mm` | 15 viên | $0.6	ext{--}1.0$ | $15	ext{ ô}$ | $100	ext{ ô gạch}$ | $50\%$ | 12 | $1\%$ |
| `Base.Pistol2` (Colt M1911) | `.45 Auto` | 7 viên | $0.9	ext{--}1.2$ | $12	ext{ ô}$ | **$70	ext{ ô gạch}$ (Trầm)** | $50\%$ | 15 | $1\%$ |
| `Base.Pistol3` (Desert Eagle) | `.44 Mag` | 8 viên | $1.0	ext{--}1.6$ | $17	ext{ ô}$ | $120	ext{ ô gạch}$ | $50\%$ | 17 | $1\%$ |
| `Base.Revolver` (Magnum .357) | `.357 Mag` | 6 viên ổ xoay | $0.95	ext{--}1.4$| $12	ext{ ô}$ | $100	ext{ ô gạch}$ | $50\%$ | 16 | $0\%$ (Không kẹt) |
| `Base.Revolver_Long` (.44) | `.44 Mag` | 6 viên ổ xoay | $1.0	ext{--}1.6$ | $18	ext{ ô}$ | $120	ext{ ô gạch}$ | $50\%$ | 17 | $0\%$ (Không kẹt) |
| `Base.Revolver_Short` (.38) | `.38 Special` | 6 viên ổ xoay | $0.5	ext{--}0.8$ | $8	ext{ ô}$ | **$60	ext{ ô gạch}$ (Êm nhất)**| $50\%$ | 11 | $0\%$ (Không kẹt) |

---

#### Bảng Bán Kính Tiếng Bước Chân Theo Tư Thế & Giày Dép

| Trạng Thái Di Chuyển | Đi Chân Trần (`Barefoot`) | Mang Giày Thể Thao (`Sneakers`) | Mang Ủng Quân Đội (`Military Boots`) |
| :--- | :---: | :---: | :---: |
| **Ngồi rón rén (Sneak Walk)** | **$1	ext{ ô gạch}$** | $2	ext{ ô gạch}$ | $3	ext{ ô gạch}$ |
| **Đi bộ bình thường (Normal Walk)**| $4	ext{ ô gạch}$ | $6	ext{ ô gạch}$ | $8	ext{ ô gạch}$ |
| **Chạy nước rút (Sprinting)** | $10	ext{ ô gạch}$ | $14	ext{ ô gạch}$ | **$20	ext{ ô gạch}$** |

---


---

# DOCUMENT 12: BASE BUILDING, FORTIFICATIONS & ARCHITECTURE BIBLE (32-FLOOR DEPTH, DUAL BARRICADES, 3X3 PLUMBING & GENERATOR GRIDS)

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

# DOCUMENT 13: MEDICAL, SURGERY & HEALTH BIBLE (17 BODY PARTS, 6 WOUND KINETICS, DUAL INFECTION & SURGERY EQUATIONS)

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

# DOCUMENT 14: META SECRETS, EMERGENT STORYTELLING & WORLD MYSTERIES BIBLE (80KG METABOLISM, 7 WEIGHT TIERS, STASHSYSTEM & 164 STORIES)

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


# DOCUMENT 15: AI PATHFINDING, DUAL-LAYER NAVMESH & HORDE SIMULATION BIBLE (64-LEVEL Z-AXIS, SSFA FUNNEL & 50,000 VIRTUAL HORDE)

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Hình học Viên nang Va chạm & Ánh xạ Trục Tọa độ Đứng 64 Tầng (`PolygonalMap2.java`)

Trong không gian đa giác liên tục (`PolygonalMap2`), mỗi thực thể hình người hoặc xác sống được đại diện bởi một viên nang va chạm (collision capsule) có bán kính cơ sở $R = 0.3\text{f}$ và hệ số đệm đường chéo $R_{\text{diag}}$ để tránh kẹt tại các đỉnh góc tường nhọn:

$$R = 0.3\text{f}, \quad R_{\text{diag}} = \sqrt{2 \times R^2} = \sqrt{0.18\text{f}} \approx 0.42426\text{f}$$

Để tích hợp đồng thời hệ thống 362 tầng hầm ngầm (xuống tới tầng $-32$) và các tòa tháp cao tầng (lên tới tầng $+31$) trong một mảng chỉ mục không âm gồm $\text{LEVELS\_PER\_CHUNK} = 64$ tầng, tọa độ trục đứng nội bộ $\text{TargetZ}_{\text{Internal}} \in [0, 63]$ được ánh xạ tuyến tính từ tọa độ thế giới $\text{TargetZ}_{\text{Game}} \in [-32, +31]$ qua mốc mặt đất $\text{GROUND\_LEVEL} = 32$:

$$\text{TargetZ}_{\text{Internal}} = \text{TargetZ}_{\text{Game}} + 32.0\text{f}$$

#### 1.2. Phương trình Diện tích Có hướng & Thuật toán Phễu Kéo Căng Dây (`LowLevelAStar.java` / Simple Stupid Funnel Algorithm - SSFA)

Khi động vật di chuyển qua chuỗi tam giác lân cận trên lưới `Mesh.java`, thuật toán SSFA tối ưu hóa chuỗi các cạnh cổng (Portal Edges: $[\vec{P}_{\text{left}}^{(i)}, \vec{P}_{\text{right}}^{(i)}]$) thành đường thẳng mượt nhất thông qua định thức tích có hướng 2D ($\text{triarea2}$) giữa đỉnh phễu $\vec{A} = (a_x, a_y)$ và hai điểm $\vec{B} = (b_x, b_y)$, $\vec{C} = (c_x, c_y)$:

$$\text{triarea2}(\vec{A}, \vec{B}, \vec{C}) = (b_x - a_x)(c_y - a_y) - (c_x - a_x)(b_y - a_y)$$

Dấu của $\text{triarea2}(\vec{\text{Apex}}, \vec{P}_{\text{current}}, \vec{P}_{\text{next}})$ quyết định việc thu hẹp góc phễu hoặc chốt điểm uốn (Apex Reset) khi cạnh trái cắt qua cạnh phải:

```text
FUNCTION Execute_Funnel_String_Pulling(start_pos, end_pos, portal_left_list, portal_right_list):
    path_points = [start_pos]
    portal_apex  = start_pos
    portal_left  = portal_left_list[0]
    portal_right = portal_right_list[0]
    apex_index = 0; left_index = 0; right_index = 0

    FOR i = 1 TO Length(portal_left_list) - 1 DO
        next_left  = portal_left_list[i]
        next_right = portal_right_list[i]

        // 1. Cập nhật cạnh phải của phễu
        IF triarea2(portal_apex, portal_right, next_right) <= 0.0 THEN
            IF portal_apex == portal_right OR triarea2(portal_apex, portal_left, next_right) > 0.0 THEN
                portal_right = next_right
                right_index  = i
            ELSE
                // Cạnh phải cắt qua cạnh trái -> Chốt đỉnh Left làm điểm uốn mới
                path_points.Append(portal_left)
                portal_apex  = portal_left
                apex_index   = left_index
                portal_left  = portal_apex
                portal_right = portal_apex
                left_index   = apex_index
                right_index  = apex_index
                i = apex_index
                CONTINUE
            END IF
        END IF

        // 2. Cập nhật cạnh trái của phễu
        IF triarea2(portal_apex, portal_left, next_left) >= 0.0 THEN
            IF portal_apex == portal_left OR triarea2(portal_apex, portal_right, next_left) < 0.0 THEN
                portal_left = next_left
                left_index  = i
            ELSE
                // Cạnh trái cắt qua cạnh phải -> Chốt đỉnh Right làm điểm uốn mới
                path_points.Append(portal_right)
                portal_apex  = portal_right
                apex_index   = right_index
                portal_left  = portal_apex
                portal_right = portal_apex
                left_index   = apex_index
                right_index  = apex_index
                i = apex_index
                CONTINUE
            END IF
        END IF
    END FOR

    path_points.Append(end_pos)
    RETURN path_points
END FUNCTION

```

#### 1.3. Phương trình Lọc Ngưỡng Sóng Âm Di cư Đàn Ảo (`ZombiePopulationManager.java#addWorldSound`)

Một nguồn âm thanh thế giới $S_{\text{sound}}$ chỉ kích hoạt hàm native C++ `n_worldSound()` để điều hướng hàng chục nghìn thực thể ảo (`Virtual Zombies` nằm ở khoảng cách $> 3\text{ chunks}$ ngoài vùng tải) khi thỏa mãn đồng thời 3 điều kiện lọc nghiêm ngặt:

$$\text{TriggerVirtualHordeMigration}(S_{\text{sound}}) = \text{TRUE} \iff \begin{cases} 1.\ R_{\text{sound}}(S_{\text{sound}}) \ge 50\text{ tiles} \\ 2.\ \text{sourceIsZombie}(S_{\text{sound}}) = \text{FALSE} \\ 3.\ \text{stressZombies}(S_{\text{sound}}) = \text{TRUE} \end{cases}$$

```text
FUNCTION Process_World_Sound_For_Virtual_Population(sound):
    // Lọc bỏ toàn bộ âm thanh cục bộ < 50 ô (bước chân, đập cửa kính, nói chuyện)
    IF sound.radius < 50 THEN RETURN END IF
    IF sound.sourceIsZombie == TRUE THEN RETURN END IF
    IF sound.stressZombies == FALSE THEN RETURN END IF

    // Gọi tầng Native C++ tính toán vector di cư cho ~50,000 Virtual Zombies (32-bit structs)
    NativePopMan.n_worldSound(sound.x, sound.y, sound.z, sound.radius)
END FUNCTION

```

---

### 2. Bảng Hằng Số Vật Lý, Lưới Điều Hướng 64 Tầng & Quần Thể Ảo (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **PolygonalMap2** | `RADIUS` | `0.3f` | `blocks (tiles)` | Bán kính viên nang va chạm (collision capsule) của nhân vật và zombie.

 |
| **PolygonalMap2** | `RADIUS_DIAGONAL` | `sqrt(0.18f) = 0.42426f` | `blocks (tiles)` | Khoảng cách đệm an toàn khi di chuyển chéo qua góc tường nhọn.

 |
| **PolygonalMap2** | `PATHS_UNDER_VEHICLES` | `true` | `boolean` | Cho phép đồ thị tìm đường vẽ nhánh luồn xuyên qua gầm xe hơi đang đỗ.

 |
| **PolygonalMap2** | `CLOSE_TO_WALLS` | `true` | `boolean` | Cho phép thực thể bám sát mép tường để tối ưu hóa quãng đường ngắn nhất.

 |
| **64-Level Z-Axis** | `LEVELS_PER_CHUNK` | `64` | `floors (tầng)` | Tổng số tầng chiều cao trong một chunk mô phỏng của Build 42.

 |
| **64-Level Z-Axis** | `GROUND_LEVEL` | `32` (`+32.0f` offset) | `z-index` | Mốc mặt đất ($Z=0$ game $\to Z=32$ internal; hầm sâu nhất $-32 \to 0$; đỉnh $+31 \to 63$).

 |
| **Node & Cxn Flags** | `NODE_FLAG_CRAWL` | `1` | `bitmask_flag` | Cờ đánh dấu node cho phép zombie bò lết chui qua gầm phương tiện.

 |
| **Node & Cxn Flags** | `CXN_FLAG_THUMP` | `2` | `bitmask_flag` | Cờ kết nối chuyển trạng thái sang đập phá cửa (`ThumpDoor`) khi cửa bị khóa.

 |
| **Node & Cxn Flags** | `NODE_FLAG_STAIR` | `16` (`6` stair nodes) | `bitmask_flag` | Cờ đánh dấu 6 node cầu thang (`stairsTW/MW/BW`, `stairsTN/MN/BN`).

 |
| **Dual-Layer A*** | `Macro_HLAStar_Thresholds` | `> 1 chunk (> 10 tiles) | Long: > 50 tiles` | `tiles (blocks)` | Ngưỡng khoảng cách kích hoạt tìm đường vĩ mô `HLAStar` theo vùng `HLChunkRegion`.

 |
| **Virtual PopMan** | `Virtualize_Chunk_Distance` | `> 3` | `chunks` | Khoảng cách người chơi rời xa khiến `IsoZombie` bị xóa khỏi RAM Java và nén thành struct 32-bit C++.

 |
| **Virtual PopMan** | `Active_Real_vs_Virtual_Cap` | `Real: 200 - 500 | Virtual: 50,000+` | `entities` | Số lượng thực thể Java thực tế quanh người chơi so với tổng quần thể ảo trong C++.

 |
| **Virtual PopMan** | `Virtual_Sound_Trigger_Floor` | `>= 50` (Ignored if `< 50`) | `tiles (blocks)` | Bán kính âm thanh tối thiểu để đánh thức và kéo đàn zombie ảo di cư (`n_worldSound`).

 |

---
