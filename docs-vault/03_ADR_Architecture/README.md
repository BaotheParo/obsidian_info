# 03_ADR_Architecture

Thư mục lưu trữ các bản ghi quyết định kiến trúc (**Architecture Decision Records - ADR**).

Các bản ghi này có hai mục đích chính:
1. **Quản trị kỹ thuật & bảo trì:** Ghi lại lý do tại sao hệ thống lại được thiết kế như vậy, các đánh đổi (trade-offs) về mặt hiệu năng và độ phức tạp.
2. **Portfolio & Phỏng vấn tuyển dụng (Senior / Architect):** Minh chứng sống cho tư duy kỹ thuật sâu sắc, khả năng xử lý bài toán tải cao (High Throughput), độ trễ thấp (Low Latency 50ms/tick) và thiết kế hệ thống phân tán.

## Quy ước đặt tên file
`ADR_<Số_thứ_tự>_<Chủ_đề>.md` (Ví dụ: `ADR_001_Hexagonal_Domain_Separation.md`, `ADR_002_Lockfree_Tick_Queue.md`)

## Template sử dụng
Sử dụng template `[[00_Templates/Tpl_ADR]]`.

---

## Danh mục Architecture Decision Records (ADR)
1. `[[03_ADR_Architecture/ADR_002_Spatial_Temporal_Quantization_and_Stateless_RNG]]`: Lượng tử hóa Không gian - Thời gian, Stateless Counter-Based RNG (SplitMix64) và Quy tắc làm tròn bất đối xứng chống Bào mòn Số dư.
2. `[[03_ADR_Architecture/ADR_003_Topological_Dungeon_Bitmask_and_Tactical_Event_Queue]]`: Xác thực Đồ thị Hầm ngục bằng Bitmask, Hàng đợi Sự kiện Breadth-First (TacticalEventBus) và Chống Lạm phát Năng lực.


