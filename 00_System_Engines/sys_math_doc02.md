---
id: "sys_math:doc02"
type: "system_mathematics_spec"
document_source: "Doc 02: Project Zomboid B42 Complete Item Encyclopedia (5,105 Items Database)"
associated_master_moc: "[[00_Master_MOC]]"
key_equations: [
  "Melee Weapon Crit Multiplier & Kinetic Distribution",
  "Container Weight Reduction & Capacity Dynamics",
  "Armor 100% Bite/Scratch Protection Matrix",
  "Daily Caloric Intake & Macronutrient Energy Density"
]
---
# ## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

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
