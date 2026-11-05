# AGENTS.md - Chỉ Dẫn Kỹ Thuật Dành Cho AI Agents

> **Dự án:** Long-term R&D Game Server Architecture  
> **Kiến trúc:** Hexagonal Architecture (Ports & Adapters)  
> **Các thành phần:** Pure Java Core Engine | PaperMC Server Adapter | WebGL Client Adapter  
> **Quy định:** Văn bản này là **Hiến pháp Kỹ thuật (Technical Constitution)** bắt buộc cho mọi AI Agent (và Developer) tham gia vào việc phân tích, thiết kế, sinh mã hoặc kiểm thử mã nguồn trong repository này.

---

## 1. Nguyên Tắc Cốt Lõi: Spec-Driven Development (SDD)

1. **ĐỌC SPEC TRƯỚC KHI VIẾT CODE:**
   - Trước khi tạo bất kỳ class, method hay sửa đổi logic nào, Agent **BẮT BUỘC** phải tìm và đọc tài liệu đặc tả tương ứng trong:
     ```
     docs-vault/02_GDD_Specs/SPEC_*.md
     ```
   - Nếu tính năng chưa có file Spec tương ứng trong `02_GDD_Specs/`, Agent **phải dừng lại và yêu cầu tạo/duyệt Spec trước**, hoặc hỗ trợ người dùng sinh file Spec theo mẫu `[[Tpl_Feature_Spec]]`.

2. **TUÂN THỦ BẤT BIẾN (INVARIANTS):**
   - Không được phép tự ý thay đổi State Machine, luồng tính toán toán học, hoặc cơ chế chống dupe/exploit đã được chốt trong Spec nếu chưa có sự đồng ý của System Architect.

---

## 2. Ranh Giới Kiến Trúc Hexagonal (Bất Khả Xâm Phạm)

Hệ thống được tổ chức theo cấu trúc 3 tầng phân lập:

```
[ Inbound Adapters ]          [ Domain Core Engine ]          [ Outbound Adapters ]
- PaperMC Events      --->      (Pure Java 17/21+)      --->    - SQL / Storage Adapter
- WebGL WebSocket     --->      - Entities & Records    --->    - Redis Sync Adapter
- Headless Simulation --->      - State Machines        --->    - Network Broadcaster
                                - Domain Services
                                - Inbound / Outbound Ports
```

### Quy tắc Vàng về Dependencies:
> 🚫 **TUYỆT ĐỐI CẤM:** Không được import bất kỳ package nào thuộc `org.bukkit.*`, `net.minecraft.*`, `io.papermc.*`, hoặc các thư viện nền tảng game engine vào tầng **Pure Java Domain**.

- Tầng **Domain Core** chỉ sử dụng:
  - Standard Java SDK (`java.lang.*`, `java.util.*`, `java.time.*`, `java.math.*`).
  - Thư viện tiện ích thuần không phụ thuộc I/O (ví dụ: annotations như JetBrains `@NotNull` nếu cần).
- Mọi tương tác với thế giới Minecraft (Player inventory, Location, Particles, Sounds, Chunks) phải đi qua:
  - **Inbound Ports:** Nhận Commands/Requests từ PaperMC Listener.
  - **Outbound Ports:** Bắn Domain Events để PaperMC Listener tự render hiệu ứng hoặc update Minecraft Inventory.

---

## 3. Ràng Buộc Hiệu Năng & Ngân Sách Vòng Lặp (50ms/Tick Budget)

Game Server vận hành ở tần số chuẩn **20 TPS (Ticks Per Second)**. Mỗi tick tương ứng với đúng **50 milliseconds**:

1. **Ngân sách thời gian cho Domain Logic:**
   - Mỗi chu kỳ tick của toàn bộ Domain Core cho hàng trăm player phải gói gọn trong **< 10ms - 15ms**, dành phần còn lại cho Minecraft World Chunks, Physics, AI Mobs và Network Flush.
   - Bất kỳ hàm tính toán đơn lẻ nào trong Domain (ví dụ: Combat formula, Trade resolution) phải có thời gian thực thi trung bình **dưới 0.05ms (50 microseconds)**. Độ phức tạp ưu tiên $\mathcal{O}(1)$ hoặc tối đa $\mathcal{O}(\log N)$.

2. **Chính sách Không Block (Zero Blocking I/O):**
   - CẤM gọi blocking network calls, SQL queries, đọc file từ đĩa, hoặc `Thread.sleep()` bên trong vòng lặp tick của Domain.
   - Mọi I/O phải được ủy thác qua Outbound Port xử lý không đồng bộ (Asynchronous Worker / Write-Behind Cache).

3. **Ý thức về Garbage Collection (GC Allocation Pressure):**
   - Hạn chế tạo rác Object ngắn hạn trong các vòng lặp hot-path chạy mỗi tick.
   - Ưu tiên sử dụng Java Records cho dữ liệu bất biến, cân nhắc Primitive Collections / FastUtil hoặc Object Pooling cho các cấu trúc dữ liệu đồ sộ chạy liên tục.

---

## 4. Chuẩn Mực Kiểm Thử Bắt Buộc (JUnit 5 Suite)

Không có mã nguồn Domain nào được coi là hoàn tất nếu thiếu bộ Unit Test tương ứng:

1. **Coverage & Cấu trúc Test:**
   - Mọi tính năng Domain phải có test suite JUnit 5 đạt độ phủ nhánh logic cao (100% Core Branch Coverage).
   - Viết test theo cấu trúc rõ ràng: `// Given`, `// When`, `// Then`.

2. **Kiểm thử Tính tất định (Deterministic Tests):**
   - Các thuật toán có yếu tố ngẫu nhiên (RNG) phải nhận Seed hoặc Mock `RandomProvider` để đảm bảo test case có thể tái lập (replayable) 100%.

3. **Kiểm thử Chống Exploit & Concurrency:**
   - Bắt buộc có test case cho:
     - Số âm (Negative balance / Wager).
     - Tràn số nguyên (`Long.MAX_VALUE`).
     - Race conditions (2 threads cố gắng rút tiền đồng thời).
     - Thao tác khi state đang ở trạng thái không hợp lệ.

4. **Kiểm thử Ngân sách Hiệu năng (Performance Budget Assertion):**
   - Có kịch bản benchmark tối thiểu 10,000 - 100,000 iterations để verify không có hàm nào vi phạm ngân sách xử lý.

---

## 5. Quy Chuẩn Lập Trình (Coding Standards)

- **Ngôn ngữ:** Java 17+ (ưu tiên Java 21 LTS).
- **Mô hình hóa dữ liệu:**
  - Dùng `record` cho Data Transfer Objects, Commands, Events, và Value Objects.
  - Dùng `sealed interface` / `permits` cho Domain Events và State definitions để tận dụng Exhaustive Pattern Matching (`switch`).
- **Phòng thủ Tràn số:** Dùng `Math.addExact`, `Math.multiplyExact`, `Math.subtractExact` cho mọi phép toán liên quan đến tiền tệ, điểm kinh nghiệm, tài nguyên.
- **Tài liệu hóa:**
  - Khi đưa ra quyết định kỹ thuật lớn: Tạo file ADR trong `docs-vault/03_ADR_Architecture/` theo mẫu `[[Tpl_ADR]]`.
  - Khi đúc kết cơ chế thiết kế game: Tạo file trong `docs-vault/01_Knowledge_Atoms/` theo mẫu `[[Tpl_Knowledge_Atom]]`.
