---
id: "sys_pipeline:doc11"
type: "system_tick_pipeline_spec"
document_source: "Doc 11: Project Zomboid B42 Combat, Ballistics & Stealth Bible (6 Melee Classes, 22 Firearms, Bernoulli Durability Decay & Jaw Stab Physics)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[doc_adnd_06_combat_injury]]",
  "[[doc_srd_02_equipment_mastery_craft]]",
  "[[sys_math_doc11]]"
]
state_machines: [
  "Melee Combat, Durability Decay & Prone Stomp Tick Pipeline",
  "Firearms Ballistics, Jamming Clearing & Acoustic Wave Pipeline",
  "Stealth Blind Spot Infiltration & Jaw Stab Execution FSM"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Máy Trạng Thái Tác Chiến Cận Chiến, Suy Giảm Độ Bền & Đòn Kết Liễu Dẫm Đầu (`Melee Combat & Durability Tick Pipeline`)

```mermaid
flowchart TD
    AttackInput["Nhấp Chuột Trái Tấn Công (Melee Swing)"] --> CheckRange{"Mục Tiêu Nằm Trong Tầm Đánh (Range)?"}
    
    CheckRange -- "Trượt (Out of Range)" --> MissAction["Vung Hụt: Tiêu Hao Thể Lực Chuẩn<br/>Không Mất Độ Bền Vũ Khí"]
    CheckRange -- "Trúng Đích" --> CheckState{"Trạng Thái Zombie?"}
    
    CheckState -- "Đang Đứng" --> CalcDamage["Tính Sát Thương = Base * M_Str * M_Skill * M_Moodle"]
    CheckState -- "Ngã Trên Đất" --> ApplyProneBonus["Nhân Hệ Số Vị Thế x 1.5 Sát Thương"]
    CheckState -- "Dẫm Đầu (Head Stomp)" --> InstantLethal["Nhân Hệ Số x 2.75 Sát Thương<br/>Tiêu Diệt Tức Thì / Không Tốn Độ Bền Vũ Khí"]
    
    ApplyProneBonus --> CalcDamage
    CalcDamage --> CheckCrit{"Tung Xúc Xắc Bạo Kích (P_crit - PanicPenalty)?"}
    
    CheckCrit -- "Trúng Bạo Kích" --> ApplyCritMultiplier["Nhân Hệ Số Crit (x 2.0 - x 6.0)"]
    CheckCrit -- "Không Bạo Kích" --> ApplyNormalDamage["Gây Sát Thương Lên Máu Zombie"]
    
    ApplyCritMultiplier --> ApplyNormalDamage
    ApplyNormalDamage --> RollDecay{"Tung Xúc Xắc Bernoulli Mất Độ Bền<br/>P = 1 / (BaseChance + Maint*2 + Skill/2)"}
    
    RollDecay -- "Không Mất" --> MaintainDurability["Bảo Toàn Độ Bền Vũ Khí"]
    RollDecay -- "Mất 1 Độ Bền" --> LowerCondition["Condition = Condition - 1"]
    
    LowerCondition --> CheckBreak{"Condition <= 0?"}
    CheckBreak -- "Vũ Khí Gãy" --> WeaponDestroyed["Phát Âm Thanh Vũ Khí Gãy<br/>Rơi Mảnh Vụn / Chuyển Vũ Khí Phụ"]
    CheckBreak -- "Còn Dùng Được" --> EndMelee["Hoàn Tất Đòn Đánh"]
    MaintainDurability --> EndMelee
    InstantLethal --> EndMelee
```

---

### 2. Luồng Xử Lý Đạn Đạo Súng Đạn, Kẹt Đạn & Phát Tán Sóng Âm 200 Ô (`Firearms Ballistics, Jamming & Acoustic Event Pipeline`)

```mermaid
flowchart TD
    FireInput["Bóp Cò Bắn Súng (Fire Trigger)"] --> CheckChamber{"Đạn Có Sẵn Trong Buồng Đạn?"}
    
    CheckChamber -- "Hết Đạn" --> DryFire["Tiếng Click Cơ Học Khô Khốc<br/>Yêu Cầu Nạp Băng Đạn Mới (Phím R)"]
    CheckChamber -- "Có Đạn" --> CheckJam{"Kiểm Tra Tỷ Lệ Kẹt Đạn (JamRate 1-2%)?"}
    
    CheckJam -- "Bị Kẹt Đạn (Jammed)" --> GunJammed["Súng Bị Kẹt Đạn!<br/>Khóa Cò / Cần Bấm Phím X (Rack Slide) Xử Lý"]
    CheckJam -- "Bắn Thành Công" --> EmitAcousticWave["Phát Tán Sóng Âm Tiếng Nổ Lưới Ô<br/>(Revolver .38: 60 ô | JS-2000 Shotgun: 200 ô)"]
    
    EmitAcousticWave --> AlertZombies["Đánh Thức Toàn Bộ Zombie Trong Bán Kính<br/>Chuyển AI Sang Trạng Thái Truy Đuổi (State: Hunt)"]
    EmitAcousticWave --> CalcAccuracy["Tính Tỷ Lệ Trúng = BaseHit + Aiming*7% - Dist - Move"]
    
    CalcAccuracy --> ApplyPanicPenalty["Giảm Tỷ Lệ Trúng Theo Mức Độ Panic"]
    ApplyPanicPenalty --> ResolveShot{"Xúc Xắc Bắn Trúng?"}
    
    ResolveShot -- "Bắn Trượt" --> BulletWhiz["Đạn Bay Xuyên Không Gian Gây Tiếng Rít"]
    ResolveShot -- "Bắn Trúng" --> ApplyBallisticDamage["Gây Sát Thương Đạn Đạo (1.0 - 2.2)"]
    
    ApplyBallisticDamage --> CheckShotgunSpread{"Súng Shotgun Bắn Chùm?"}
    CheckShotgunSpread -- "Shotgun" --> SpreadTo3Zombies["Gây Sát Thương Đồng Thời 3-4 Zombie<br/>Mỗi Con Trúng Đạn Nhận Điểm XP Aiming Độc Lập"]
    CheckShotgunSpread -- "Súng Đơn" --> CheckPiercing{"Aiming >= 8 & Đạn .308 Win?"}
    
    CheckPiercing -- "Đạn Xuyên" --> PierceSecondTarget["Viên Đạn Xuyên Qua Đầu Zombie Thứ 2"]
    CheckPiercing -- "Không Xuyên" --> EndShot["Hoàn Tất Phát Bắn"]
    SpreadTo3Zombies --> EndShot
    PierceSecondTarget --> EndShot
```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Quân giới & Khí tài chiến đấu: [[03_Arsenal_And_Bastions]]
- Quy tắc giao tranh AD&D / D&D 5E: [[doc_adnd_06_combat_injury]], [[doc_srd_02_equipment_mastery_craft]]
- Phương trình toán học đi kèm: [[sys_math_doc11]]
