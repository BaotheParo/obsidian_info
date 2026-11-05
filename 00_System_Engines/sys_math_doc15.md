---
id: "sys_math:doc15"
type: "system_mathematics_spec"
document_source: "Doc 15: Project Zomboid B42 AI Pathfinding, Dual-Layer NavMesh & Horde Simulation Bible (64-Level Z-Axis, SSFA Funnel & 50,000 Virtual Horde)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[02_Bestiary_And_AI]]",
  "[[sys_ai_sensory]]",
  "[[doc_tmkwtd_01_core_ai_heuristics]]",
  "[[doc_ldmc_01_hordes_stress_templates]]",
  "[[sys_ai_aggressive_pack]]",
  "[[sys_pipeline_doc15]]"
]
key_equations: [
  "Collision Capsule Geometry & 64-Level Vertical Coordinate Mapping (Ground Z = 32.0f)",
  "Triangulated NavMesh SSFA Funnel Algorithm & Cross-Product Determinant triarea2()",
  "Off-Screen Virtual Horde Despawn Pooling (> 3 Chunks to 32-bit C++ Struct)",
  "Macro 50-Tile Sound Wave Gate & Worldwide Native Horde Migration (n_worldSound)"
]
---

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
