---
id: "sys_pipeline:doc12"
type: "system_tick_pipeline_spec"
document_source: "Doc 12: Project Zomboid B42 Base Building, Fortifications & Architecture Bible (32-Floor Depth, Dual Barricades, 3x3 Plumbing & Generator Grids)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[doc_dmg_04_bastions]]",
  "[[sys_math_doc12]]"
]
state_machines: [
  "32-Floor Vertical Construction & Multi-Stage Wall Upgrades Pipeline",
  "Rainwater Harvesting, 3x3 Z+1 Plumbing Auto-Purification Engine",
  "Generator Electrical Grid, Fuel Decay & Fire/CO Hazard FSM"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Xây Dựng 32 Tầng, Gia Cố 2 Mặt, Lọc Nước $3\times 3$ ($Z+1$) & Giám Sát An Toàn Máy Phát (`Architecture, Plumbing & Generator Grid Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> FORTIFICATION_PLANNING : Quy hoạch Pháo đài (Giới hạn độ cao 32 Tầng + 362 Hầm ngầm giữ tươi rau củ +50%)

    state "1. Pipeline Thi công Kiến trúc & Gia cố 2 Mặt (ISBuildUtil & ISBarricadeAction)" as BuildPhase {
        FORTIFICATION_PLANNING --> CONSTRUCT_PERIMETER : Tính HP = BaseHP + (Carpentry * 50) + [100 nếu có Handy]\n- Vòng ngoài: Log Wall (400 -> 1.000 HP, 4 gỗ + 2 dây, không tốn đinh) hoặc Metal Pole Fence (500 -> 1.200 HP)\n- Cổng xe: Double Airlock Gate (2 lớp cổng đôi rộng 4 ô, 400-900 HP, cách nhau 6m)\n- Tường nhà: Frame -> Wall Lv3 (300-900 HP) -> Trát vữa Plaster -> Lăn sơn Paint
        
        CONSTRUCT_PERIMETER --> DUAL_SIDE_BARRICADE : Gia cố Cửa sổ / Cửa chính (Mặt trong + Mặt ngoài):\n- Metal Bars (3.000 HP/mặt = 6.000 HP): Nhìn & Đâm giáo/Bắn súng xuyên qua\n- Sheet Metal (5.000 HP/mặt = 10.000 HP): Chặn 100% ánh sáng & tầm nhìn\n- Wooden Planks (1.000 HP/tấm, 4 tấm/mặt = 8.000 HP): 1-2 tấm bắn qua được, 3-4 tấm kín mít
        
        DUAL_SIDE_BARRICADE --> VERTICAL_ISOLATION_META : Đắp dốc (ISBuildRampCursor) lái xe lên sân thượng + Rải đất trồng rau (ISNaturalFloor)\nDùng Sledgehammer phá cầu thang tầng 1 -> Leo lên tầng 2 bằng Dây vải (Sheet Rope)
    }

    state "2. Pipeline Nối ống Tự lọc Nước mưa (ISPlumbItem 3x3 Z+1 Verification)" as PlumbPhase {
        FORTIFICATION_PLANNING --> CHECK_PLUMB_CONDITIONS : Người chơi cầm Pipe Wrench chuột phải vào Bồn rửa (Sink)
        CHECK_PLUMB_CONDITIONS --> VERIFY_ENCLOSED_AND_ZPLUS1 : Kiểm tra 4 điều kiện bắt buộc:\n1. Có Pipe Wrench\n2. Phòng khép kín hoàn chỉnh (square:getRoom() ~= nil, kín tường & trần)\n3. Có Rain Barrel (160L ở Carp 4 / 400L ở Carp 7) tại tầng Z+1 trong lưới 3x3 ô quanh bồn rửa\n4. Trong thùng đang có >= 1 đơn vị nước
        VERIFY_ENCLOSED_AND_ZPLUS1 --> AUTO_PURIFY_ACTIVE : Hợp lệ -> Kích hoạt [Plumb Sink]\nNước mưa bẩn (Tainted Water) chảy qua vòi tự động lọc sạch 100% vi khuẩn không cần đun sôi!
    }

    state "3. Pipeline Điện trường Máy phát 20-Tile & Cảnh báo Tử thần (MOGenerator Tick)" as GenPhase {
        FORTIFICATION_PLANNING --> GENERATOR_RUNNING_TICK : Bật Máy phát (Bình 100% = 10-12L = 2.5 can xăng | Tiêu hao 0.1 L/h = 10-14 ngày)\nPhủ sóng điện trường: Bán kính ngang 20 ô gạch x Chiều cao 3 tầng (Z-1, Z, Z+1)
        
        GENERATOR_RUNNING_TICK --> CHECK_INDOOR_CO_POISON : Kiểm tra vị trí đặt máy phát
        CHECK_INDOOR_CO_POISON --> LETHAL_CO_ASPHYXIATION : Nếu đặt TRONG NHÀ KÍN hoặc TẦNG HẦM (getRoom() ~= nil)\n-> Khí độc CO rút máu liên tục -> Ngủ say là chết ngạt không có âm thanh cảnh báo!
        
        GENERATOR_RUNNING_TICK --> CHECK_CONDITION_20PCT : Hao mòn độ bền mỗi giờ (Bảo trì bằng Electronics Scrap khi < 50%)
        CHECK_CONDITION_20PCT --> CATASTROPHIC_FIRE_EXPLOSION : Nếu Condition < 20% -> Xì khói đen & PHÁT NỔ TỨC THÌ\nThiêu rụi toàn bộ căn cứ!
    }

```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Căn cứ, Pháo đài & Công trình Bastion: [[03_Arsenal_And_Bastions]], [[doc_dmg_04_bastions]]
- Phương trình toán học đi kèm: [[sys_math_doc12]]
