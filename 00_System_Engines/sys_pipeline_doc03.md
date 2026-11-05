---
id: "sys_pipeline:doc03"
type: "system_tick_pipeline_spec"
document_source: "Doc 03: Project Zomboid B42 Crafting & Tech Tree Guide (1,026 craftRecipes & 20 Workstations)"
associated_master_moc: "[[00_Master_MOC]]"
state_machines: [
  "5-Age Tech Tree Transition FSM (Stone -> Pottery -> Bloomery -> Machinery -> Civilization)",
  "Workstation Entity Execution & Tool Degradation Pipeline"
]
tick_rate_hz: 60
---
# ## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Thực Thi Công Thức Chế Tác & Tiến Hóa 5 Thời Kỳ (Workstation CraftRecipe & 5-Age Progression Pipeline)

```mermaid
stateDiagram-v2
    [*] --> AGE_0_PRIMITIVE : Khởi đầu sinh tồn (Chưa có công cụ kim loại)

    state "1. Chuỗi Tiến hóa 5 Thời kỳ (5-Age Tech Tree Dependency Graph)" as TechProgression {
        AGE_0_PRIMITIVE --> AGE_1_POTTERY : Ghè đá lửa (230 ticks) + Thu hái Đất sét bờ sông\nXây Bàn xoay gốm + Máy ép gạch (50 ticks) + Hố đốt than
        AGE_1_POTTERY --> AGE_2_METALLURGY : Xây Lò nung vòm (Dome Kiln) -> Nung Gạch chịu nhiệt (Firebricks)\n& Khuôn đúc (100 ticks) + Giải quyết cắt nước ngày 30
        AGE_2_METALLURGY --> AGE_3_GLASS_TEXTILE : Xây Lò luyện quặng (Furnace I) + Lò rèn (Forge I) + Đe sắt\nNấu chảy cát + tro Soda (>1200°C, 20 ticks) -> Thổi lọ thủy tinh (100 ticks)\nĐập/Chải/Quay/Dệt vải lanh + Thuộc da chống cắn 80-100%
        AGE_3_GLASS_TEXTILE --> AGE_4_RECONSTRUCTION : Nâng cấp Forge II/III + Bàn cưa vòng + Máy khoan đứng\nRèn Giáp tấm, Gorget che cổ (100% Bite Def), Kiếm ngắn (500 ticks)\nĐúc thuốc đạn (Lưu huỳnh + Than + Chì) + Phơi thịt khô Jerky (6-12 tháng)
    }

    state "2. Máy Trạng Thái Thực Thi craftRecipe (Per-Recipe Tick Pipeline)" as RecipeEngine {
        [*] --> VALIDATE_CONTEXT : Người chơi gửi yêu cầu chế tác (1 trong 1.026 công thức)
        VALIDATE_CONTEXT --> CHECK_THERMAL_AND_TOOLS : Kiểm tra khoảng cách tới Workstation (20 trạm)\nKiểm tra nguyên liệu & công cụ mode:keep
        CHECK_THERMAL_AND_TOOLS --> RUNNING_CRAFT_TICKS : Hợp lệ -> Khóa hành động trong T_craft (20..600 ticks)
        RUNNING_CRAFT_TICKS --> TOOL_DEGRADATION_ROLL : Hoàn tất T_craft -> Tiêu hao nguyên liệu thường\nGiữ lại công cụ mode:keep
        TOOL_DEGRADATION_ROLL --> APPLY_TOOL_WEAR : Kiểm tra cờ flags[MayDegradeLight] / flags[MayDegradeHeavy]\nTrừ độ bền công cụ (Phục hồi vũ khí cùn tại Grindstone)
        APPLY_TOOL_WEAR --> GRANT_OUTPUT_AND_XP : Sinh vật phẩm Output + Cộng trực tiếp xpAward = Skill:XP\n(Nhân với hệ số Sách kỹ năng Tập 1..5 nếu đã đọc)
    }

```

---
