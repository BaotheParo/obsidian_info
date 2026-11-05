---
id: "sys_math:doc05"
type: "system_mathematics_spec"
document_source: "Doc 05: Project Zomboid B42 World Map & Loot Distribution Guide (11 Regions, 362 Basements & ProceduralDistributions.lua)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[01_World_And_Factions]]",
  "[[02_Bestiary_And_AI]]",
  "[[03_Arsenal_And_Bastions]]",
  "[[sys_pipeline_doc05]]"
]
key_equations: [
  "ProceduralDistributions.lua Weighted Random Loot Roll Dynamics",
  "ProceduralBasementEngine (362 .pzby lots, 0 Lux Ambient Darkness)",
  "Regional Zombie Density & Danger Matrix (11 Knox Regions)",
  "Annotated Map & Survivor House Generation Probability"
]
---

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

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Thế giới, Địa lý & Điển tích: [[01_World_And_Factions]]
- Quái vật & AI: [[02_Bestiary_And_AI]]
- Công trình & Căn cứ: [[03_Arsenal_And_Bastions]]
- Luồng máy trạng thái đi kèm: [[sys_pipeline_doc05]]
