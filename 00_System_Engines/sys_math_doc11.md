---
id: "sys_math:doc11"
type: "system_mathematics_spec"
document_source: "Doc 11: Project Zomboid B42 Combat, Ballistics & Stealth Bible (6 Melee Classes, 22 Firearms, Bernoulli Durability Decay & Jaw Stab Physics)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[doc_adnd_06_combat_injury]]",
  "[[doc_srd_02_equipment_mastery_craft]]",
  "[[sys_pipeline_doc11]]"
]
key_equations: [
  "Melee Final Damage & Moodle Exhaustion Collapse (-95% damage debuff)",
  "Weapon Durability Bernoulli Decay P = 1 / (BaseChance + Maint*2 + Skill/2)",
  "Firearms Ballistics, Aiming 0-10 Progression & Shotgun Spread Mechanics",
  "Stealth Footstep Audio Attenuation & Jaw Stab Instant Kill Physics"
]
---

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

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Quân giới & Khí tài chiến đấu: [[03_Arsenal_And_Bastions]]
- Quy tắc giao tranh & Chấn thương AD&D / D&D 5E: [[doc_adnd_06_combat_injury]], [[doc_srd_02_equipment_mastery_craft]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc11]]
