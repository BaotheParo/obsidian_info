---
id: "sys_pipeline:doc06"
type: "system_tick_pipeline_spec"
document_source: "Doc 06: Project Zomboid B42 Vehicles & Mechanics Handbook (52 Models, 15 Mechanical Parts & Hotwiring Dynamics)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[crv_xe_tho_boc_sat_15_parts]]",
  "[[sys_math_doc06]]"
]
state_machines: [
  "Vehicle Engine Ignition, Physics Stall & Collision FSM",
  "15-Part Mechanical Anatomy & Maintenance Pipeline"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khởi Động, Vật Lý Va Chạm, Phát Tán Âm Thanh Ống Xả & Bảo Trì 15 Bộ Phận (`Vehicle Physics & Mechanics Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> VEHICLE_ENTERED : Người chơi leo vào ghế (Ưu tiên đập kính ghế phụ nếu cửa khóa)

    state "1. Chiếm quyền & Đánh lửa Khởi động (Hotwire & Ignition Pipeline)" as IgnitionPhase {
        VEHICLE_ENTERED --> CHECK_AUTH : Kiểm tra Chìa khóa / Trạng thái Hotwired
        CHECK_AUTH --> HOTWIRE_PROCESS : Chưa mở khóa -> Yêu cầu: Nghề Burglar\nHOẶC (Electricity >= 1 & Mechanics >= 2 & Screwdriver)\nThao tác đấu dây: 5 - 10 giây
        HOTWIRE_PROCESS --> IGNITION_ATTEMPT : Đấu dây vĩnh viễn thành công
        CHECK_AUTH --> IGNITION_ATTEMPT : Đã có chìa / Đã Hotwire
        
        IGNITION_ATTEMPT --> BATTERY_AND_QUALITY_CHECK : Yêu cầu Battery >= 10% & EngineCondition > 0%\n- Nếu EngineQuality > 85%: Nổ máy ngay lần 1 (kể cả âm độ)\n- Nếu EngineQuality < 50%: Phải đề 3 - 5 lần mới nổ
    }

    state "2. Vòng lặp Vận hành, Sóng âm & Va chạm (PZBullet64 Runtime Tick)" as DrivingPhase {
        BATTERY_AND_QUALITY_CHECK --> ENGINE_RUNNING_TICK : Động cơ nổ máy -> Tiêu hao xăng (Rò rỉ nếu Gas Tank thủng)
        
        ENGINE_RUNNING_TICK --> MUFFLER_SOUND_WAVE : Tính SoundRadius theo Muffler Condition:\n- Muffler > 80%: Bán kính 20 - 30m\n- Muffler < 20%: Bán kính 100 - 150m (Gom zombie cả thị trấn)
        
        ENGINE_RUNNING_TICK --> CHASSIS_DYNAMICS : - Lốp chuẩn 30-35 psi (Nổ/non hơi -> Giảm 70% tốc độ)\n- Phanh hỏng -> Quãng đường phanh x4\n- Giảm xóc hỏng -> Lật nghiêng khi cua gấp\n- EngineCondition < 20% -> Chết máy ngẫu nhiên
        
        ENGINE_RUNNING_TICK --> COLLISION_RESOLUTION : Va chạm với bầy Zombie
        COLLISION_RESOLUTION --> FRONTAL_IMPACT : Đâm đầu xe -> Sau 3-5 lần Hood = 0%\n-> Động cơ nhận 100% sát thương trực tiếp -> Chết máy!
        COLLISION_RESOLUTION --> REVERSE_SPIN_SWEEP : Lùi đuôi xe (Reverse Spin) -> Bánh xe nghiền nát đầu zombie\nĐộng cơ an toàn 100% (Chỉ móp cốp giảm dung tích Trunk)
    }

    state "3. Thoát hiểm Bao vây & Bảo trì Cơ khí 24h (Escape & 24h Mechanics Cooldown)" as MaintenancePhase {
        FRONTAL_IMPACT --> SURROUNDED_STALLED : Xe chết máy giữa 10 - 20 con zombie
        SURROUNDED_STALLED --> INSTANT_DEATH_2S : Mở cửa nhảy ra ngoài ngay -> Bị dìm chết trong 2 giây!\n(Hoặc kính chắn gió/cửa sổ vỡ -> Bị cắn xuyên cửa vào mặt)
        SURROUNDED_STALLED --> SEAT_SWITCH_ESCAPE : Kính còn nguyên 100% -> Bấm 'Z' chuyển sang ghế phụ đối diện\nMở cửa phía ít zombie để thoát thân
        
        VEHICLE_ENTERED --> MECHANICS_SERVICE : Mở giao diện Cơ khí (15 Bộ phận / 3 Phân hạng)
        MECHANICS_SERVICE --> CHECK_24H_COOLDOWN : Tháo/Lắp Đèn, Đài, Ắc quy, 4 Bánh trên 5-10 xe/ngày\nNếu (T_now - T_last_part_id) >= 24 giờ -> Cộng XP * M_book(x3..x5)\nSau 3-5 ngày đạt Mechanics Cấp 4-6
    }

```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Quân giới, Khí tài & Công trình: [[03_Arsenal_And_Bastions]]
- Đoàn xe thồ bọc sắt Đại Việt: [[crv_xe_tho_boc_sat_15_parts]]
- Phương trình toán học đi kèm: [[sys_math_doc06]]
