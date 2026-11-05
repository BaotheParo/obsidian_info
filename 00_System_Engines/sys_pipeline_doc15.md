---
id: "sys_pipeline:doc15"
type: "system_tick_pipeline_spec"
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
  "[[sys_math_doc15]]"
]
state_machines: [
  "Dual-Layer Hierarchical Navigation FSM (Macro HLAStar + Micro VGAStar)",
  "Polygon Obstacle Traversal & Anti-Stuck Sensor (WalkingOnTheSpot Repath)",
  "Animal Triangulated NavMesh SSFA Funnel Algorithm Pipeline",
  "Virtual Horde Population Manager & 50-Tile Sound Gate Trigger"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Điều Hướng Đa Giác 64 Tầng, Phễu Tam Giác SSFA & Di Cư Đàn Ảo 50.000 Thực Thể (`Dual-Layer Pathfinding, Animal NavMesh & Virtual Horde Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> ENTITY_NAVIGATION_REQUEST : Thực thể (Zombie / Animal) yêu cầu đường đi tới Target (X, Y, Z_game)

    state "1. Ánh xạ 64 Tầng & Tìm đường 2 Lớp (PolygonalMap2: HLAStar + VGAStar)" as ZombieNavPhase {
        ENTITY_NAVIGATION_REQUEST --> Z_AXIS_64_MAPPING : Chuyển đổi trục đứng: TargetZ_Internal = TargetZ_Game + 32.0f\n(LEVELS_PER_CHUNK = 64 | GROUND_LEVEL = 32 | Hầm -32 -> 0 | Tháp +31 -> 63)
        
        Z_AXIS_64_MAPPING --> MACRO_HLASTAR_CHECK : Nếu khoảng cách > 1 chunk (> 10..50 tiles) -> Chạy HLAStar.java\nTìm chuỗi HLChunkRegion & Điểm chuyển tầng HLLevelTransition (6 nút Stairs: NODE_FLAG_STAIR = 16)
        
        MACRO_HLASTAR_CHECK --> MICRO_VGASTAR_NATIVE : Đẩy xuống VisibilityGraph & PathfindNative (JNI C++ Đa luồng)\nCắt tia Liang-Barsky + Bám tường (CLOSE_TO_WALLS = true)\nĐệm viên nang RADIUS = 0.3f (Đường chéo RADIUS_DIAGONAL = 0.42426f)
        
        MICRO_VGASTAR_NATIVE --> OBSTACLE_AND_STUCK_FSM : Tương tác vật cản trên đường đi (PathFindBehavior2):\n- Gầm xe ô tô (PATHS_UNDER_VEHICLES = true): Bật NODE_FLAG_CRAWL = 1 bò qua gầm xe\n- Cửa khóa (!couldBeOpen): Bật CXN_FLAG_THUMP = 2 -> Vây đập sập cửa\n- Cửa sổ / Rào thấp: Trèo qua (Nếu còn kính vỡ !isGlassRemoved -> Cứa rách thịt; Nếu đông -> Fence Lunge)\n- Cảm biến chống kẹt (WalkingOnTheSpot): Đứng yên nhiều nhịp -> Hủy đường (isCancel = true) & Đập vật cản
    }

    state "2. Lưới Tam giác Động vật & Thuật toán Phễu Kéo Căng Dây (AnimalPathfind SSFA)" as AnimalNavPhase {
        ENTITY_NAVIGATION_REQUEST --> TRIANGULATED_MESH_A_STAR : Động vật trên đồng cỏ (Zone) -> Chia lưới tam giác (Mesh.java)\nChạy LowLevelAStar tìm chuỗi Cạnh Cổng (Portal Edges: portalLeft, portalRight)
        TRIANGULATED_MESH_A_STAR --> FUNNEL_STRING_PULLING : Chạy thuật toán Phễu (SSFA) với tích có hướng triarea2(Apex, Left, Right)\nKẹp góc phễu & Chốt điểm uốn -> Xuất quỹ đạo đường thẳng kéo căng mượt mà\nKết hợp OffMeshConnection (Hươu nhảy rào thấp, Thỏ nhảy mương, Bò qua cổng chuồng)
    }

    state "3. Ảo hóa Quần thể 50.000 Zombie & Ngưỡng Sóng âm 50-Tile (ZombiePopulationManager)" as PopManPhase {
        [*] --> CHUNK_DISTANCE_MONITOR : Giám sát khoảng cách giữa Người chơi và toàn bộ 50.000+ Zombie
        CHUNK_DISTANCE_MONITOR --> VIRTUALIZE_TO_CPP_32BIT : Nếu khoảng cách > 3 Chunks -> Xóa IsoZombie khỏi RAM Java (removeFromWorld)\nNén thành Struct nhị phân 32-bit trong C++ (Chỉ giữ 200 - 500 Real Zombies quanh người chơi)
        
        VIRTUALIZE_TO_CPP_32BIT --> WORLD_SOUND_FILTER_50 : Phát sinh âm thanh thế giới (addWorldSound):\n- Nếu sound.radius < 50 HOẶC sourceIsZombie HOẶC !stressZombies -> BỎ QUA (Đàn ảo đứng yên)\n- Nếu sound.radius >= 50 (VD: Súng .38 = 60 ô, Shotgun = 200 ô, Trực thăng > 500 ô)\n-> Gọi Native n_worldSound() -> Hàng ngàn Virtual Zombies di cư ngầm về nguồn âm!
    }

```

---
