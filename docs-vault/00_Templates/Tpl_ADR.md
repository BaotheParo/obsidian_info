---
id: "ADR-{{date:YYYYMMDD}}-{{title}}"
title: "{{title}}"
status: "proposed" # proposed | accepted | rejected | deprecated | superseded
date: "{{date:YYYY-MM-DD}}"
deciders:
  - "System Architect (Lead Developer)"
tags:
  - adr
  - architecture
  - performance
  - hexagonal
  - portfolio-showcase
---

# Architecture Decision Record: {{title}}

> **Trạng thái:** `PROPOSED` | `ACCEPTED` | `DEPRECATED` | `SUPERSEDED by [[ADR-XXXX]]`  
> **Phạm vi tác động:** `Pure Java Core` | `PaperMC Adapter` | `WebGL Client` | `Network Protocol`  

---

## 1. Ngữ cảnh & Đặt vấn đề (Context & Problem Statement)
- Mô tả bối cảnh kỹ thuật hiện tại: Đang gặp khó khăn gì trong việc mở rộng hoặc duy trì hệ thống?
- Tại sao phải đưa ra quyết định kiến trúc này ngay bây giờ?
- Vấn đề nếu không giải quyết: Tắc nghẽn CPU, rò rỉ bộ nhớ, coupling chặt chẽ với PaperMC khiến không tái sử dụng được logic cho WebGL Client.

---

## 2. Các Ràng buộc Hệ thống (System Constraints & SLOs)

Một Game Server thời gian thực vận hành trên các giới hạn tài nguyên khắt khe:
1. **Ngân sách vòng lặp thời gian thực (20 TPS - 50ms/Tick Budget):**
   - Vòng lặp chính của PaperMC thực thi mỗi **50ms**.
   - Toàn bộ thời gian xử lý gameplay trong một tick phải hoàn thành trong **< 15ms - 25ms** để nhường ngân sách cho I/O, Entity Tracker, Pathfinding và Chunks của engine nền.
   - Thao tác quyết định này có làm block Main Server Thread hay không?
2. **Áp lực Garbage Collection (GC Allocation Pressure):**
   - Tránh cấp phát Object rác liên tục trong vòng lặp tick (Zero-allocation / Object Pooling / Primitive Arrays khi cần thiết).
3. **Phân tách ranh giới Hexagonal Architecture:**
   - Pure Java Core Engine phải độc lập 100% với môi trường Minecraft (`org.bukkit.*`, `net.minecraft.*`).
   - Có thể chạy Unit Test độc lập trên JVM thông thường mà không cần khởi động MockBukkit hay Minecraft Server.

---

## 3. Các Phương án Cân nhắc (Options Considered)

### 3.1. Phương án A: [Tên phương án] (Ví dụ: Trực tiếp can thiệp Bukkit Event Loop)
- **Mô tả:** ...
- **Ưu điểm:**
  - Thời gian hiện thực nhanh ban đầu.
  - Tận dụng sẵn API của PaperMC.
- **Nhược điểm & Rủi ro:**
  - Dính chặt vào Bukkit API, không thể port logic lên WebGL Client hoặc Standalone Engine.
  - Khó Unit Test, phụ thuộc vào MockBukkit cồng kềnh.
  - Nguy cơ block Main Thread khi logic phức tạp.

### 3.2. Phương án B: [Tên phương án] (Ví dụ: Hexagonal Ports & Adapters + Off-tick Asynchronous State)
- **Mô tả:** Tách Domain Core thành thư viện Pure Java độc lập. PaperMC chỉ đóng vai trò Adapter trung gian nhận/gửi I/O qua các Port.
- **Ưu điểm:**
  - Code sạch, 100% testable với JUnit 5 tốc độ cao (< 1s cho 10,000 tests).
  - Tái sử dụng trọn vẹn logic cho WebGL Serverless hoặc Client-side Prediction.
  - Dễ dàng thay thế database hoặc network transport mà không đụng vào gameplay logic.
- **Nhược điểm & Rủi ro:**
  - Cần viết thêm Boilerplate DTO/Mapper giữa Bukkit Entities và Domain Models.
  - Cần quản lý đồng bộ dữ liệu giữa Asynchronous Thread và Main Thread cẩn thận.

---

## 4. Quyết định Chọn & Lý do (Decision & Rationale)

**Phương án được chọn:** `Phương án B: ...`

**Lý do lựa chọn:**
1. Đáp ứng trọn vẹn ràng buộc ngân sách 50ms/tick nhờ khả năng phân tách tính toán nặng sang worker thread hoặc gom batch.
2. Đảm bảo nguyên lý thiết kế hệ thống bền vững (Clean Architecture / Hexagonal).
3. Cho phép kiểm thử tự động (Unit Test / Mutation Testing) một cách triệt để.

---

## 5. Hệ quả Đánh đổi (Consequences & Trade-offs)

### 5.1. Tác động Tích cực (Positive)
- **Khả năng bảo trì:** Logic game được cô lập hoàn toàn, dễ refactor.
- **Hiệu năng:** Kiểm soát được chi phí CPU trên mỗi tick thông qua profiling trực tiếp trên JVM.
- **Tái sử dụng:** Core Engine có thể deploy ở nhiều môi trường (Standalone Java Daemon, PaperMC Plugin, headless simulation).

### 5.2. Tác động Tiêu cực & Biện pháp Giảm thiểu (Negative & Mitigations)
- **Độ trễ do Thread Handoff:** Nếu logic đẩy sang async thread rồi trả về Main Thread, sẽ tốn ít nhất 1-2 tick độ trễ.
  - *Biện pháp giảm thiểu:* Chỉ đẩy các tác vụ nặng (như tính toán thuật toán đường đi, pathfinding lưới lớn, nén nén packet) sang async; các phép tính toán tức thời (combat hit check) giữ ở Domain chạy sync trên Main Thread với độ phức tạp $\mathcal{O}(1)$.
- **Chi phí viết Adapter Layer:** Tốn thời gian tạo Mapper và Port.
  - *Biện pháp giảm thiểu:* Tận dụng Java Records và Code Generator hoặc AI Agent tự động sinh code theo Spec.

---

## 6. Góc nhìn Tuyển dụng & Portfolio CV (Interview Talking Points)

> Mục này đúc kết các luận điểm kỹ thuật đắt giá để đưa vào CV hoặc thảo luận trong vòng phỏng vấn System Design / Senior Java Engineer.

- **Kỹ thuật nổi bật:**
  - Thiết kế kiến trúc Hexagonal (Ports & Adapters) cho Real-time Game Server.
  - Tối ưu hóa chu kỳ Tick 50ms, giảm thiểu GC pauses bằng các kỹ thuật Low Latency Java.
  - Xử lý bài toán Concurrency và Data Consistency giữa Main Engine Thread và Network Worker Threads.
- **Câu hỏi phỏng vấn dự kiến:**
  - *"Làm thế nào để đảm bảo Domain Core không bị block bởi I/O khi lưu trữ dữ liệu người chơi?"*
    - *Trả lời mẫu:* Sử dụng Write-Behind Caching kết hợp với Outbound Port không đồng bộ (Disruptor / RingBuffer / CompletableFuture queue) để việc ghi DB không bao giờ diễn ra trên Tick Thread.
