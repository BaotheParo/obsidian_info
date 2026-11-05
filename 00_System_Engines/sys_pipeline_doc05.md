---
id: "sys_pipeline:doc05"
type: "system_tick_pipeline_spec"
document_source: "Doc 05: Project Zomboid B42 World Map & Loot Distribution Guide (11 Regions, 362 Basements & ProceduralDistributions.lua)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[01_World_And_Factions]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[sys_math_doc05]]"
]
state_machines: [
  "World Chunk Initialization & Procedural Basement Generation Pipeline",
  "Optimal 7-Day Survival Route State Progression"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khởi Tạo Thùng Chứa Thế Giới, Tính Trọng Số Loot & Sinh Tầng Hầm Ngầm (`World Map & Procedural Basement Tick Pipeline`)

```mermaid
flowchart TD
    StartChunk["Chunk Thế Giới Được Nạp (10x10x8 Tiles)"] --> CheckBasement{"Kiểm Tra Xác Suất Sinh Hầm Ngầm<br/>(P = 8% * ZoneMod * TierMod)"}
    
    CheckBasement -- "Xúc Xắc Thất Bại" --> SpawnSurfaceBuilding["Chỉ Khởi Tạo Công Trình Mặt Đất (z = 0)"]
    CheckBasement -- "Xúc Xắc Thành Công" --> QueryPzby["Truy Vấn 1 Trong 362 Layout .pzby Phù Hợp"]
    
    QueryPzby --> StitchVertical["Ghép Nối Lô Ngầm (z = -1)<br/>Tạo Cầu Thang Ngầm / Cửa Nắp Sàn"]
    StitchVertical --> ApplyDarkness["Gán Mức Sáng Ambient = 0.0 Lux<br/>Triệt Tiêu Hoàn Toàn Tầm Nhìn Tự Nhiên"]
    
    SpawnSurfaceBuilding --> ScanContainers["Quét Toàn Bộ Đối Tượng IsoObject (Container)"]
    ApplyDarkness --> ScanContainers
    
    ScanContainers --> ContainerLoop["Vòng Lặp Qua Từng Thùng Chứa"]
    ContainerLoop --> CheckSpecialEvent{"Kiểm Tra Sự Kiện Tòa Nhà<br/>(Survivor House 5%? Party House?)"}
    
    CheckSpecialEvent -- "Survivor House" --> ForceBarricades["Gia Cố Cửa/Cửa Sổ 3 Ván Gỗ (3000 HP)<br/>100% Đồ Hộp Tủ Bếp, 60% Súng Tủ Đồ"]
    CheckSpecialEvent -- "Bình Thường" --> LoadProceduralTable["Đọc Bảng Loot Từ ProceduralDistributions.lua"]
    
    ForceBarricades --> RollLoot["Thực Thi R Lần Tung (Rolls) Theo Trọng Số W_i / Tổng W"]
    LoadProceduralTable --> RollLoot
    
    RollLoot --> PopulateInventory["Nạp Vật Phẩm Vào Thùng Chứa Thế Giới"]
    PopulateInventory --> EndChunk["Hoàn Tất Khởi Tạo Chunk"]
```

---

### 2. Sơ Đồ Pipeline Lộ Trình 7 Ngày Đầu Tiên Tối Ưu (`Day 1 - Day 7 Survival Route Execution`)

```mermaid
graph TD
    Day1["Ngày 1: Sinh Tồn Tại Chỗ<br/>- Dọn dẹp nhà xuất phát<br/>- Thu thập: 2 chai nước, dao bếp, chảo xào, balo học sinh<br/>- Đọc sách kỹ năng Tập 1 ngay trong ngày"]
    
    Day2["Ngày 2: Đột Kích Trạm Cứu Hỏa / Cảnh Sát<br/>- Tiếp cận Rosewood hoặc Riverside<br/>- Thu thập: Rìu cứu hỏa (Vũ khí chính), Áo chống cắn 100%, Balo ALICE"]
    
    Day3["Ngày 3: Kiếm Xe Hơi & Xăng Dầu<br/>- Kiểm tra bãi đỗ xe tìm chìa khóa / Nghề Burglar bẻ khóa nổ máy<br/>- Dùng bình xăng rỗng hút xăng (Siphon) sang xe tốt"]
    
    Day4["Ngày 4: Cửa Hàng Kim Khí & Hiệu Sách<br/>- Thu thập: Búa, Cưa gỗ, Đinh sắt, Xẻng, Sách kỹ năng Vol 1-3<br/>- Tối thượng: Sách 'Cách dùng máy phát điện' (ElectronicsMag4)"]
    
    Day5["Ngày 5: Truy Lùng Búa Tạ & Máy Phát Điện<br/>- Đột kích nhà kho công nghiệp (Muldraugh / Riverside Warehouses)<br/>- Tìm Búa tạ (Sledgehammer) phá cửa tiệm súng<br/>- Nhặt ít nhất 2 Máy phát điện (Căn cứ & Cây xăng)"]
    
    Day6_7["Ngày 6 - 7: Rút Về Căn Cứ Ven Sông & Thiết Lập Cây Công Nghệ B42<br/>- Thiết lập biệt thự ven sông có rào sắt an toàn<br/>- Khai thác Đất sét bờ suối, nặn bàn gốm, đốt than, xây lò nung<br/>- Hoàn tất nông nghiệp & trữ nước trước Ngày 30 cúp điện cúp nước"]

    Day1 --> Day2
    Day2 --> Day3
    Day3 --> Day4
    Day4 --> Day5
    Day5 --> Day6_7
```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Thế giới, Địa lý & Điển tích: [[01_World_And_Factions]]
- Công trình & Căn cứ: [[03_Arsenal_And_Bastions]]
- Phương trình toán học đi kèm: [[sys_math_doc05]]
