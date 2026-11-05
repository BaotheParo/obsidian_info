---
id: "sys_math:doc01"
type: "system_mathematics_spec"
document_source: "Doc 01: Project Zomboid B42 Architecture and Design Bible (Complete Re-extraction)"
associated_master_moc: "[[00_Master_MOC]]"
key_equations: [
  "AI Sensory Perception Cone & Hearing Wave",
  "Anatomy 17-parts Bleed Rate & Infection Probability",
  "Weapon Durability Bernoulli Decay",
  "Internal Economic Balance 4-elements",
  "Thermoregulation Multilayer Core Heat Equilibrium",
  "Macronutrient Nutrition & Body Weight Dynamics",
  "Progressive Repair Degradation (fixing.txt)",
  "Mendelian Animal Genetics & Allele Inheritance",
  "Polyglot Native UDP & Client Prediction Dynamics"
]
---
# ## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

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
