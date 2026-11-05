---
id: "ADR-20261018-Spatial-Temporal-Quantization-and-Stateless-RNG"
title: "Lượng tử hóa Không gian - Thời gian, Stateless RNG và Phòng vệ Cắt số dư trong Domain Core"
status: "accepted"
date: "2026-10-18"
deciders:
  - "System Architect"
  - "Lead Co-Dev / Strategic Advisor"
tags:
  - adr
  - architecture
  - performance
  - low-latency
  - deterministic
  - anti-exploit
  - portfolio-showcase
---

# Architecture Decision Record: Lượng tử hóa Không gian - Thời gian, Stateless RNG và Phòng vệ Cắt số dư trong Domain Core

> **Trạng thái:** `ACCEPTED`  
> **Phạm vi tác động:** `Pure Java Core` | `Inbound Ports (Quantizer)` | `Economy Engine` | `Simulation Replay`

---

## 1. Ngữ cảnh & Đặt vấn đề (Context & Problem Statement)

Khi phân định ranh giới Hexagonal Architecture giữa **PaperMC / WebGL Adapters (Xử lý Continuous)** và **Pure Java Domain Core (Xử lý Discrete)**, hệ thống phát sinh 4 thách thức kỹ thuật sống còn:

1. **Rò rỉ Tọa độ Thực (Floating-point Leakage):** PaperMC sử dụng tọa độ liên tục `double x, y, z` và thời gian hệ thống `System.currentTimeMillis()`. Nếu để dữ liệu này tràn vào Domain, tính tất định (Determinism) bị phá vỡ hoàn toàn do sai số IEEE 754 và chênh lệch clock máy chủ.
2. **Lỗ hổng Bào mòn Số dư (Integer Truncation Exploit):** Phép chia số nguyên trong Java luôn làm tròn xuống (`truncate towards zero`). Power Gamers có thể chia nhỏ giao dịch bán lẻ để triệt tiêu hoàn toàn thuế hoặc phí giao dịch.
3. **Vỡ tính Tất định do Mutable RNG:** Việc sử dụng các bộ sinh số có trạng thái ngầm (`java.util.Random`, `SplittableRandom`) khiến thứ tự lặp duyệt quái vật trong `HashMap` làm thay đổi kết quả toàn bộ trận đánh, phá hủy tính năng Anti-cheat Replay.
4. **Vòng lặp R&D ban đầu quá chậm:** Việc triển khai PaperMC Server ngay từ đầu tốn nhiều công sức dựng môi trường thay vì tập trung thẩm định cỗ máy toán học và cân bằng kinh tế.

---

## 2. Quyết định Kiến trúc (Architectural Decisions)

### 2.1. Điều chỉnh Chiến lược: Headless CLI & Monte Carlo Simulation (Phase 1)
- **Quyết định:** Giai đoạn 1 tập trung toàn lực vào **Pure Java Domain Core** chạy trên nền Console / JUnit 5.
- Thực hiện các kịch bản kiểm thử Monte Carlo chạy 100,000 trận đánh hoặc 10,000 giao dịch chợ đen trong thời gian dưới $0.5\text{s}$.
- Chỉ tích hợp PaperMC sau khi hệ thống kinh tế và FSM đã được chứng minh là cân bằng và bất khả xâm phạm trước các macro exploit.

### 2.2. Tầng Lượng tử hóa Không gian & Thời gian (Spatial-Temporal Quantizer)
- **Quyết định:** Đặt một bộ lọc `Quantizer` tại toàn bộ Inbound Ports:
  - **Không gian:** Chuyển đổi vector `double` thành các Enum rời rạc: `RelativeAngle { FRONT, FLANK, BACKSTAB }`, `DistanceBand { MELEE, NEAR, FAR, OUT_OF_RANGE }`, `Elevation { LOW_GROUND, LEVEL, HIGH_GROUND }`.
  - **Thời gian:** Cấm tuyệt đối `System.currentTimeMillis()` bên trong Domain Core. Thời gian được đo đếm thuần túy bằng `long logicalTick`.

### 2.3. Quy tắc Làm tròn Bất đối xứng & Đơn vị Tiền tệ Nội bộ (Micro-Currency)
- **Quyết định:**
  - Mọi khoản **Thuế / Chi phí trừ** của người chơi bắt buộc phải làm tròn lên bằng `Math.ceilDiv()`. Dù giao dịch 1 đồng với thuế 0.01% vẫn bị thu tối thiểu 1 đơn vị.
  - Mọi khoản **Sát thương / Phần thưởng** làm tròn xuống (`floor`).
  - Trong RAM và Database: Lưu trữ theo đơn vị `Micro-Units` ($1\text{ Vàng Hiển thị} = 10,000\text{ Micro-Gold}$). Chỉ chia $10,000$ khi đẩy ra Outbound Adapter.

### 2.4. Pure Functional Stateless RNG (SplitMix64 Counter-Based)
- **Quyết định:** Thay thế toàn bộ đối tượng sinh số ngẫu nhiên có trạng thái mutable bằng thuật toán băm không trạng thái `DeterministicDice.roll(matchSeed, tick, actorId, rollIndex, min, max)`.
- Đảm bảo $100\%$ Thread-safe, không tốn RAM lưu trữ trạng thái RNG, và có thể tua tới bất kỳ tick nào để replay kết quả.

---

## 3. Hệ quả Đánh đổi (Consequences & Trade-offs)

### 3.1. Điểm mạnh (Positive)
- **An toàn tuyệt đối trước Exploit:** Triệt tiêu hoàn toàn khả năng lách thuế bằng macro chia nhỏ giao dịch.
- **Tất định 100% (Bit-exact Replay):** Bất kỳ trận đánh nào có tranh chấp đều có thể tái lập chính xác từng con số chỉ với `(MatchSeed, CommandList)`.
- **Tốc độ Test siêu tốc:** Bộ test Monte Carlo chạy trên JVM sạch cho phép kiểm tra hàng triệu phép tính trong vài giây.

### 3.2. Điểm đánh đổi & Biện pháp khắc phục (Trade-offs & Mitigations)
- **Độ phân giải không gian bị giảm:** Tọa độ bị gom cụm thành các dải (Bands) thay vì từng milimet.
  - *Biện pháp:* Dải cự ly $2.5\text{m}$ (Melee) và góc $60^\circ$ (Backstab) là quá đủ cho gameplay hành động mà không gây cảm giác giật cục.

---

## 4. Điểm sáng Kỹ thuật cho Phỏng vấn / Portfolio (Interview Showcase)

> **Keyword:** *High-Performance Fixed-Point Math, Exploit-Proof Asymmetric Rounding, Stateless PRNG (SplitMix64), Spatial-Temporal Quantization for Hexagonal Game Engine.*

- **Câu hỏi phỏng vấn dự kiến:** *"Tại sao trong một Game Engine thời gian thực, bạn lại cấm dùng `System.currentTimeMillis()` và dấu phẩy động trong Domain?"*
  - **Câu trả lời chuẩn:** Dấu phẩy động không có tính tất định trên các kiến trúc CPU khác nhau (khác biệt thanh ghi FPU/SSE), còn đồng hồ hệ thống có thể bị trôi (clock drift/NTP sync) hoặc làm sai lệch kết quả Replay. Việc lượng tử hóa không gian thành Enum và thời gian thành `logicalTick` đảm bảo Domain Core là một State Machine tất định $100\%$, kiểm thử siêu tốc trên JUnit và sẵn sàng chạy ở bất kỳ môi trường nào.
