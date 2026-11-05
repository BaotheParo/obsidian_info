---
id: "sys_pipeline:doc07"
type: "system_tick_pipeline_spec"
document_source: "Doc 07: Project Zomboid B42 Media Broadcasts, Knox Lore & Skill VHS Bible (AEBS Oracle, 5-Tier Skill Books, 64 Instructional VHS & Idempotent XP)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[01_World_And_Factions]]",
  "[[doc_prp_01_goals_milestones]]",
  "[[sys_math_doc07]]"
]
state_machines: [
  "Early 9-Day Live Broadcast Program State Machine (Channel 203)",
  "Automated Emergency Broadcast System (AEBS) Hourly Tick Pipeline",
  "Idempotent Instructional XP Ingestion & Skill Book Multiplier Pipeline"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Phát Sóng 14 Ngày, Dự Báo AEBS Mỗi Giờ & Xử Lý Chồng XP Bản Ghi (`Media, AEBS Oracle & Skill Leveling Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> WORLD_INIT_DAY_0 : Khởi tạo Thế giới (Ngày 0 = 09/07)

    state "1. Bộ Lập Lịch Phát Sóng & Điềm Báo AEBS (Hourly & 14-Day Cron Pipeline)" as BroadcastCron {
        WORLD_INIT_DAY_0 --> LOCK_AEBS_FREQ : Chạy getRandomFrequency(88000, 108000) bước nhảy 200 (0.2 MHz)\nKhóa 1 trong 101 tần số AEBS cho toàn bộ Save Game
        LOCK_AEBS_FREQ --> HOURLY_AEBS_TICK : Đầu mỗi giờ (XX:00) phát bản tin AEBS:\n- Dự báo Gió / Sương mù / Bão (trước 24-48h)\n- Cảnh báo Lưới điện 3 giai đoạn (Dao động 48-72h -> Sụp đổ -> Blackout)\n- Nếu >= 09:00 có "Air Activity detected." -> Kích hoạt Siêu sự kiện Trên không (12:00-19:00)
        
        WORLD_INIT_DAY_0 --> LIFE_AND_LIVING_9DAYS : Kênh 203 phát sóng Ngày 0..Ngày 8 (06:00, 12:00, 18:00)\nNgười chơi vặn Volume = 1..2 vạch để tránh lộ SoundRadius
        LIFE_AND_LIVING_9DAYS --> SIGNAL_COLLAPSE_DAY_8 : Sau 06:00 Ngày 8 (Ngày 7 bùng phát chủng không khí)\nToàn bộ kênh TV tắt thành Nhiễu trắng (Static White Noise)
    }

    state "2. Máy Trạng Thái Tiêu Thụ Bản Ghi & Nhân XP 5 Bậc (Media Consumption Engine)" as MediaXPEngine {
        [*] --> READ_SKILL_BOOK_FIRST : Đọc Sách Kỹ năng Tập 1..5 trước khi xem/nghe\nKích hoạt Multiplier: Vol1(x3), Vol2(x5), Vol3(x8), Vol4(x12), Vol5(x16)
        READ_SKILL_BOOK_FIRST --> CONSUME_MEDIA_LINE : Nhân vật nghe từng dòng thoại (13.166 lines / 352 media items)
        
        CONSUME_MEDIA_LINE --> CHECK_STACKING_RULE : Kiểm tra Nguồn Phát (TV vs 155 Retail VHS vs 128 Home VHS)
        CHECK_STACKING_RULE --> BLOCK_DUPLICATE_XP : Nếu là Retail VHS VÀ đã xem tập đó trên TV\n-> Bỏ qua XP (0% XP)
        CHECK_STACKING_RULE --> GRANT_BOOSTED_XP : Nếu là TV lần đầu HOẶC Retail VHS chưa xem TV\nHOẶC là Home VHS (Độc lập 100% với TV)
        
        GRANT_BOOSTED_XP --> APPLY_FORMULA_AND_RECIPES : XP_final = Base_XP * M_book(x3..x16) * (1 + 0.30 * FastLearner) * M_init\nMở khóa Recipe (VD: MakeMildewCure, MetalWallLvl1, CraftMakeshiftRadio)\nHoặc biến đổi Tâm lý (The Dog Goblin: Stress +0.2, +Panic)
    }

```

---
