# 02_GDD_Specs

Thư mục lưu trữ đặc tả kỹ thuật tính năng (Feature Specifications) theo phương pháp **Spec-Driven Development (SDD)**. 

Tất cả các tài liệu ở đây được viết để các AI Agent (hoặc Developer) đọc hiểu, sinh mã nguồn Pure Java trong `DomainCore`, và tạo bộ kiểm thử JUnit 5 trước khi hiện thực.

## Quy tắc bắt buộc
1. Mọi tính năng mới phải có file Spec ở đây trước khi viết code.
2. Không import thư viện nền tảng (Minecraft/PaperMC/Bukkit) vào trong logic đặc tả.
3. Phải xác định rõ State Machine, Invariants, và các trường hợp Anti-Dupe / Anti-Exploit.

## Quy ước đặt tên file
`SPEC_<Mã_tính_năng>_<Tên_tính_năng>.md` (Ví dụ: `SPEC_001_Combat_Resolution.md`, `SPEC_002_Currency_Vault.md`)

## Template sử dụng
Sử dụng template `[[Tpl_Feature_Spec]]`.
