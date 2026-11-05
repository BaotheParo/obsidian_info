---
id: "ADR-20261019-Topological-Dungeon-Bitmask-and-Tactical-Event-Queue"
title: "Xác thực Đồ thị Hầm ngục bằng Bitmask, Hàng đợi Sự kiện Breadth-First và Chống Lạm phát Năng lực"
status: "accepted"
date: "2026-10-19"
deciders:
  - "System Architect"
  - "Lead Co-Dev / Strategic Advisor"
tags:
  - adr
  - architecture
  - performance
  - low-latency
  - event-driven
  - anti-exploit
  - game-programming-patterns
  - portfolio-showcase
---

# Architecture Decision Record: Xác thực Đồ thị Hầm ngục bằng Bitmask, Hàng đợi Sự kiện Breadth-First và Chống Lạm phát Năng lực

> **Trạng thái:** `ACCEPTED`  
> **Phạm vi tác động:** `Pure Java Core` | `Dungeon Engine` | `Tactical Event Bus` | `Combat Reaction Matrix`

---

## 1. Ngữ cảnh & Đặt vấn đề (Context & Problem Statement)

Khi triển khai các cơ chế thăng tiến (Progression) và chiến đấu phức hợp (Emergence) từ Chương 2 sách *Game Mechanics*, hệ thống đối mặt với 4 rủi ro kỹ thuật và vận hành chí tử:

1. **Bẫy Đệ quy Sự kiện Vô tận (Synchronous Event Cascade):** Nếu dùng mô hình Event Dispatcher đồng bộ trực tiếp trên Call Stack, khi một đòn đánh kích hoạt nổ diện rộng, đòn nổ diện rộng lại giết quái kích hoạt chuyển Phase của Boss, chuỗi gọi đệ quy sâu sẽ ném `StackOverflowError` hoặc `ConcurrentModificationException` làm sập Main Thread của PaperMC.
2. **Lỗ hổng Nhảy cóc Đồ thị (Topological Bypass):** Hacker có thể gửi packet `UnlockGateCommand` cho phòng Boss cuối ngay khi đứng ở sảnh vào nếu hệ thống chỉ kiểm tra sở hữu chìa khóa mà không kiểm tra vị trí phòng kề cận trên đồ thị hầm ngục.
3. **Cái bẫy Dữ liệu (Data-Intensive Trap):** Chi phí thiết kế nội dung thủ công quá lớn nhưng người chơi tiêu thụ quá nhanh.
4. **Lạm phát Chỉ số Tuyến tính (Power Creep):** Tăng chỉ số thuần túy phá hủy nền kinh tế và tính cân bằng gameplay.

---

## 2. Quyết định Kiến trúc (Architectural Decisions)

### 2.1. Hàng đợi Sự kiện Theo Chiều rộng (Breadth-First TacticalEventBus) với Cầu chì Ngắt mạch
- **Quyết định:** Sử dụng mẫu thiết kế **Event Queue** (*Game Programming Patterns*).
- Mọi Domain Event khi sinh ra trong quá trình tính toán combat **chỉ được đẩy vào đuôi của `ArrayDeque<DomainEvent>`** ($\mathcal{O}(1)$), tuyệt đối không gọi subscriber đệ quy trên Call Stack.
- Khi luồng tính toán đòn đánh hiện tại chốt xong $100\%$ dữ liệu, vòng lặp `flushEvents()` mới tuần tự phân phối từng sự kiện ra ngoài.
- **Cầu chì bảo vệ (Circuit Breaker):** Giới hạn `MAX_CASCADE_DEPTH = 8`. Nếu phát hiện chuỗi phản ứng dây chuyền vượt quá 8 cấp độ, cầu chì ngắt ngay lập tức với `IllegalStateException`, bảo vệ an toàn cho chu kỳ $50\text{ms}$/tick của máy chủ.

### 2.2. Kiểm định Tính Kề cận Đồ thị (Topological Adjacency) bằng Bitmask $\mathcal{O}(1)$
- **Quyết định:** Cấu trúc `DungeonProgress` lưu trữ `int currentRoomId` kết hợp 3 thanh ghi Bitmask riêng biệt:
  - `unlockedGatesBitmask`: Cổng đã mở.
  - `consumableKeysBitmask`: Chìa khóa dùng 1 lần (xóa bit bằng phép `AND NOT`).
  - `capabilitiesBitmask`: Chìa khóa năng lực vĩnh viễn (giữ nguyên bit).
- Mọi thao tác mở cổng bắt buộc kiểm tra điều kiện kề cận:
  $$\text{currentRoomBit} \ \& \ \text{adjacentRoomsMask} \ne 0$$
- Đảm bảo hacker không thể tương tác với bất kỳ cổng nào nếu chưa di chuyển hợp lệ đến phòng kề cận.

### 2.3. Chống Lạm phát bằng Chìa khóa Năng lực (Capability Keys)
- **Quyết định:** Phần thưởng săn Boss hiếm được thiết kế theo triết lý *Zelda Gale Boomerang*: Mở khóa **Năng lực / Quyền tiếp cận (Capability Access)** thay vì tăng chỉ số tuyến tính.
- Món đồ quý trên Chợ Đen mang lại giá trị mở khóa vùng tài nguyên mới (Progression Unlock), triệt tiêu nguy cơ Power Creep.

### 2.4. Biến số Môi trường Thủ tục (Procedural Environmental Mutators)
- **Quyết định:** Trên cùng một bản đồ Hub-and-Spoke cố định, Domain Core sinh ra các hiệu ứng môi trường ngẫu nhiên (dung nham dâng, sương mù làm chậm, bão sét) theo Seed tất định của ngày.
- Tái sử dụng tối đa cấu trúc dữ liệu bản đồ nhưng tạo ra hàng chục kịch bản chiến thuật phong phú.

---

## 3. Hệ quả Đánh đổi (Consequences & Trade-offs)

### 3.1. Điểm mạnh (Positive)
- **Bất khả xâm phạm trước Stack Overflow:** Tách biệt hoàn toàn Call Stack của CombatResolver và ProgressionSubscribers.
- **Bảo mật Đồ thị Tuyệt đối:** Không cần duyệt đồ thị đệ quy; chỉ mất 6 phép toán bitwise để xác thực toàn bộ luật di chuyển và mở cổng.
- **Tuổi thọ Nội dung Vô hạn:** Vòng lặp gameplay duy trì sự tươi mới nhờ Environmental Mutators mà không tốn công sức vẽ thêm map.

### 3.2. Điểm đánh đổi & Biện pháp (Trade-offs & Mitigations)
- **Độ trễ xử lý Event:** Sự kiện không được chạy tức thì trong microsecond sinh ra mà phải chờ kết thúc hàm hiện tại.
  - *Biện pháp:* Toàn bộ việc flush diễn ra trên `ArrayDeque` trong cùng 1 tick CPU (RAM), độ trễ trễ microsecond là không đáng kể so với lợi ích ổn định hệ thống.

---

## 4. Điểm sáng Kỹ thuật cho Phỏng vấn / Portfolio (Interview Showcase)

> **Keyword:** *In-Memory Event Queue with Cascade Circuit Breaker, Topological Bitmask Validation, Capability-Driven Progression, Procedural Environmental Mutators.*

- **Câu hỏi phỏng vấn dự kiến:** *"Làm sao bạn giải quyết bài toán Event Storm hoặc đệ quy vô hạn khi các Domain Event kích hoạt chéo lẫn nhau trong Game Engine thời gian thực?"*
  - **Câu trả lời chuẩn:** Thay vì sử dụng Synchronous Observer pattern truyền thống trên Call Stack, tôi thiết kế `TacticalEventBus` sử dụng Breadth-First Event Queue (`ArrayDeque`). Mọi event mới đều được xếp hàng ở đuôi hàng đợi, cho phép luồng logic ban đầu kết thúc trọn vẹn và giải phóng Stack frame trước khi xử lý cascading side-effects. Đồng thời, tôi cài đặt một Circuit Breaker giới hạn độ sâu tối đa (8 tầng) để ngắt an toàn mọi vòng lặp logic bất thường, bảo vệ ngân sách 50ms/tick của Game Server.
