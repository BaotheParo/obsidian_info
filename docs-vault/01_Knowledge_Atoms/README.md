# 01_Knowledge_Atoms

Thư mục lưu trữ các hạt tri thức nguyên tử (Atomic Knowledge) được chắt lọc từ các đầu sách kinh điển và nghiên cứu học thuật:
- **Game Mechanics: Advanced Game Design** (Ernest Adams, Joris Dormans)
- **Designing Data-Intensive Applications (DDIA)** (Martin Kleppmann)
- **Dungeons & Dragons 5th Edition (D&D 5e)** Core Rules
- **Game Programming Patterns** (Robert Nystrom)

## Quy ước đặt tên file
`KA_<Chủ_đề>_<Tên_cơ_chế>.md` (Ví dụ: `KA_Mechanics_Diminishing_Returns.md`, `KA_DDIA_Event_Driven_State_Sync.md`)

## Template sử dụng
Sử dụng template `[[00_Templates/Tpl_Knowledge_Atom]]`.

---

## Danh mục Knowledge Atoms đã chắt lọc

### Game Mechanics (Ernest Adams & Joris Dormans) - Chapter 1: Designing Game Mechanics
1. `[[01_Knowledge_Atoms/KA_Mechanics_Discrete_vs_Continuous]]`: Phân biệt Cơ chế Rời rạc vs Cơ chế Liên tục và ranh giới kiến trúc Hexagonal (Domain Core vs PaperMC/Client).
2. `[[01_Knowledge_Atoms/KA_Mechanics_Five_Types_Classification]]`: Phân loại 5 dạng cơ chế game (Physics, Economy, Progression, Tactical, Social) và ma trận phân tầng.
3. `[[01_Knowledge_Atoms/KA_Mechanics_Three_Sources_of_Unpredictability]]`: 3 nguồn gốc bất định (Chance, Player Choice, Complex Rules) và thiết kế Deterministic Domain Engine.
4. `[[01_Knowledge_Atoms/KA_Process_Prototyping_and_Make_The_Toy_First]]`: Quy trình 3 giai đoạn (Concept - Elaboration - Tuning), triết lý "Make the toy first" và kiểm thử kinh tế chống exploit.

### Game Mechanics (Ernest Adams & Joris Dormans) - Chapter 2: Emergence and Progression
5. `[[01_Knowledge_Atoms/KA_Mechanics_Emergence_vs_Progression]]`: Emergence vs Progression, phân biệt Process-Intensive vs Data-Intensive, Rào cản phức hợp (Complexity Barrier) và tối ưu hóa Bitmask.
6. `[[01_Knowledge_Atoms/KA_Progression_Lock_Key_and_Hub_Spoke]]`: Cơ chế Lock-and-Key đa năng, Điểm thắt cổ chai (Bottlenecks) và Kiến trúc không gian Hub-and-Spoke trong Dungeon State Machine.
7. `[[01_Knowledge_Atoms/KA_Architecture_Hybrid_Emergence_Progression]]`: Mô hình tích hợp lai (Hybrid Integration), bài học tiến hóa từ StarCraft 1 sang StarCraft 2 và Domain Event Bridge điều phối môi trường.

### Game Mechanics (Ernest Adams & Joris Dormans) - Chapter 3: Complex Systems and the Structure of Emergence
8. `[[01_Knowledge_Atoms/KA_Complexity_Edge_of_Chaos_and_Feedback_Loops]]`: Vùng Ranh giới Hỗn loạn (Edge of Chaos), Phản hồi Dương vs Âm, Dynamic Bounty & Upkeep Governor.
9. `[[01_Knowledge_Atoms/KA_Emergence_Local_Rules_and_Flocking_AI]]`: Quy tắc cục bộ (Cellular Automata), Giải mã ảo giác AI săn mồi Pac-Man và D&D Pack Tactics không tốn chi phí mạng.
10. `[[01_Knowledge_Atoms/KA_Emergence_Taxonomy_and_Expressive_Consistency]]`: 4 cấp độ Emergence của Jochen Fromm, triết lý "Consistency over Realism" và Hệ thống quy tắc biểu cảm trực giao (Orthogonal Mechanics).
