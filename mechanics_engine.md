# TOÀN THƯ CƠ CHẾ ĐỘNG CƠ HẠT NHÂN (MECHANICS ENGINE COMPENDIUM)
> **Dự Án:** Đại Việt Thế Kỷ XIV–XV RPG (Project Zomboid Build 42 Engine Re-engineering)
> **Cơ Sở Dữ Liệu Nguồn:** `mechanics_engine.db` (SQLite 3 - UTF-8 Encoded)
> **Tổng Dữ Liệu Tinh Lọc:** 270.000 dòng JSON thô $\rightarrow$ 35 Khuôn Mẫu Cận Chiến, 3 Hỏa Khí, 43 Cây Trồng, 25 Thân Phận, 95 Đặc Điểm, 1.026 Công Thức, 64 Bí Tịch & Quan Tượng Đài.

---

## 1. TỔNG QUAN CƠ SỞ DỮ LIỆU `mechanics_engine.db`

Cơ sở dữ liệu được tổ chức theo mô hình bảng quan hệ chuẩn hóa (Relational Schema), đánh index toàn bộ các trường danh mục chính để hỗ trợ truy vấn tức thời (O(1) / O(log N)):

| Tên Bảng (Table) | Số Lượng Bản Ghi | Khóa Chính (Primary Key) | Chỉ Mục (Index) | Phạm Vi Áp Dụng Trọng Tâm |
| :--- | :---: | :--- | :--- | :--- |
| `astronomical_telemetry` | **78** | `signal_key` | `Primary Key` | Dự báo thời tiết 48h, bão lốc, sập thành trì |
| `crops` | **43** | `name` | `Primary Key` | Nhu cầu nước, chu kỳ sinh trưởng, tính chịu rét |
| `items` | **5,092** | `id` | `idx_items_cat, idx_items_disp` | Trọng lượng, phân loại cổ phong, giá trị kim loại |
| `melee_weapons` | **387** | `id` | `idx_melee_cat, idx_melee_dmg` | Cơ chế sát thương cận chiến, độ bền, tầm với |
| `professions` | **25** | `id` | `Primary Key` | Điểm Point-Buy, hệ số nhân XP, Trait khởi đầu |
| `ranged_weapons` | **22** | `id` | `Primary Key` | Đạn đạo, nạp đạn, tầm nổ âm thanh |
| `recipes` | **1,026** | `name (composite)` | `idx_rec_cat, idx_rec_ancient` | Cây công nghệ 5 thời kỳ, thời gian chế tác |
| `skill_tomes` | **64** | `id` | `Primary Key` | Trao điểm XP vĩnh viễn, mở khóa công thức |
| `traits` | **95** | `id` | `Primary Key` | Điểm cân bằng nhân vật, bổ trợ thuộc tính |
| `trapping_gear` | **6** | `name` | `Primary Key` | Độ bền bẫy, cơ chế bẫy thú hoang dã |
| `wildlife_game` | **6** | `name` | `Primary Key` | Khung giờ hoạt động, tập tính dã ngoại |

---

## 2. KHO VŨ KHÍ & KHÍ TÀI ĐẠI VIỆT THẾ KỶ XIV–XV

### 2.1. Bảng 35 Khuôn Mẫu Vũ Khí Cận Chiến Cốt Lõi (Core Melee Archetypes)
Toàn bộ 387+ vũ khí cận chiến thô trong Build 42 đã được rút gọn bằng thuật toán SQL Grouping theo phân hệ vũ khí và 5 bậc sát thương thực nghiệm:

| ID Khuôn Mẫu | Tên Vũ Khí Đại Việt | Phân Hệ | Bậc | Trọng Lượng | Sát Thương [Min - Max] | Bạo Kích | Nhân Bạo Kích | Độ Bền | Tốc Độ | Số Mẫu Đại Diện |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `wpn_arch_axe_t1` | **Rìu Tiền Tiêu Nhẹ** | Rìu | Bậc 1 | 1.44 kg | 0.50 – 0.93 | 20.6% | x3.1 | 8 | 1.07 | 9 mẫu |
| `wpn_arch_axe_t2` | **Rìu Tiều Phu Khai Hoang** | Rìu | Bậc 2 | 1.67 kg | 0.64 – 1.38 | 15.6% | x4.8 | 9 | 1.03 | 9 mẫu |
| `wpn_arch_axe_t3` | **Rìu Chiến Phá Trận** | Rìu | Bậc 3 | 2.39 kg | 0.69 – 1.54 | 13.3% | x4.8 | 8 | 0.98 | 9 mẫu |
| `wpn_arch_axe_t4` | **Rìu Đại Đao Hai Tay** | Rìu | Bậc 4 | 3.06 kg | 0.91 – 1.88 | 24.4% | x4.7 | 11 | 0.99 | 9 mẫu |
| `wpn_arch_axe_t5` | **Việt Phủ Nghi Lễ** | Rìu | Bậc 5 | 4.22 kg | 1.32 – 2.60 | 40.0% | x6.2 | 11 | 0.91 | 9 mẫu |
| `wpn_arch_longblade_t1` | **Đoản Đao Cận Thân** | Trường Đao & Kiếm | Bậc 1 | 1.75 kg | 0.90 – 1.70 | 20.0% | x2.0 | 10 | 1.00 | 2 mẫu |
| `wpn_arch_longblade_t2` | **Phác Đao Tuần Tiễu** | Trường Đao & Kiếm | Bậc 2 | 1.75 kg | 1.15 – 1.80 | 20.0% | x2.5 | 11 | 1.00 | 2 mẫu |
| `wpn_arch_longblade_t3` | **Kiếm Bách Luyện Thép** | Trường Đao & Kiếm | Bậc 3 | 1.50 kg | 1.25 – 2.00 | 20.0% | x3.5 | 10 | 1.00 | 2 mẫu |
| `wpn_arch_longblade_t4` | **Trảm Mã Đại Đao** | Trường Đao & Kiếm | Bậc 4 | 1.75 kg | 1.25 – 2.00 | 15.0% | x3.5 | 11 | 1.00 | 2 mẫu |
| `wpn_arch_longblade_t5` | **Liễu Diệp Bảo Kiếm** | Trường Đao & Kiếm | Bậc 5 | 2.17 kg | 2.83 – 3.75 | 22.5% | x4.8 | 11 | 1.00 | 6 mẫu |
| `wpn_arch_spear_t1` | **Ngọn Tầm Vông Vót Nhọn** | Thương & Giáo | Bậc 1 | 1.30 kg | 0.46 – 0.68 | 8.0% | x1.0 | 6 | 0.92 | 5 mẫu |
| `wpn_arch_spear_t2` | **Thương Tre Bọc Sắt** | Thương & Giáo | Bậc 2 | 1.80 kg | 0.90 – 1.32 | 24.0% | x7.2 | 7 | 0.94 | 5 mẫu |
| `wpn_arch_spear_t3` | **Trường Thương Bộ Binh** | Thương & Giáo | Bậc 3 | 2.60 kg | 1.00 – 1.58 | 20.0% | x4.8 | 5 | 0.99 | 5 mẫu |
| `wpn_arch_spear_t4` | **Bát Xà Đại Mâu** | Thương & Giáo | Bậc 4 | 2.52 kg | 1.04 – 1.60 | 20.0% | x5.0 | 7 | 1.00 | 5 mẫu |
| `wpn_arch_spear_t5` | **Thương Câu Liêm Phá Ngựa** | Thương & Giáo | Bậc 5 | 2.83 kg | 1.17 – 1.68 | 20.0% | x4.9 | 7 | 1.00 | 9 mẫu |
| `wpn_arch_blunt_t1` | **Côn Gỗ Luyện Võ** | Trường Côn & Chùy | Bậc 1 | 1.57 kg | 0.25 – 0.51 | 11.3% | x1.5 | 5 | 1.14 | 19 mẫu |
| `wpn_arch_blunt_t2` | **Đòn Gánh Thiết Mộc** | Trường Côn & Chùy | Bậc 2 | 1.98 kg | 0.48 – 0.88 | 24.7% | x2.2 | 8 | 1.02 | 19 mẫu |
| `wpn_arch_blunt_t3` | **Thiết Bổng Tráng Sĩ** | Trường Côn & Chùy | Bậc 3 | 2.29 kg | 0.73 – 1.28 | 30.8% | x3.4 | 12 | 0.97 | 19 mẫu |
| `wpn_arch_blunt_t4` | **Chùy Gai Phá Giáp** | Trường Côn & Chùy | Bậc 4 | 3.22 kg | 0.97 – 1.85 | 34.2% | x3.3 | 11 | 0.90 | 19 mẫu |
| `wpn_arch_blunt_t5` | **Lang Nha Bổng** | Trường Côn & Chùy | Bậc 5 | 6.20 kg | 1.79 – 2.88 | 31.0% | x2.4 | 10 | 0.87 | 20 mẫu |
| `wpn_arch_smallblunt_t1` | **Dùi Cui Gỗ Nghiến** | Đoản Côn & Búa Nhỏ | Bậc 1 | 1.21 kg | 0.24 – 0.44 | 10.3% | x1.6 | 6 | 1.17 | 19 mẫu |
| `wpn_arch_smallblunt_t2` | **Đoản Bổng Dân Binh** | Đoản Côn & Búa Nhỏ | Bậc 2 | 1.07 kg | 0.38 – 0.76 | 15.5% | x2.4 | 6 | 1.07 | 19 mẫu |
| `wpn_arch_smallblunt_t3` | **Chùy Đồng Hộ Thân** | Đoản Côn & Búa Nhỏ | Bậc 3 | 1.87 kg | 0.48 – 0.97 | 18.8% | x3.0 | 7 | 1.04 | 19 mẫu |
| `wpn_arch_smallblunt_t4` | **Búa Rèn Công Binh** | Đoản Côn & Búa Nhỏ | Bậc 4 | 1.49 kg | 0.54 – 1.03 | 23.2% | x2.6 | 7 | 1.03 | 19 mẫu |
| `wpn_arch_smallblunt_t5` | **Song Tiết Côn** | Đoản Côn & Búa Nhỏ | Bậc 5 | 1.76 kg | 0.63 – 1.20 | 25.7% | x3.6 | 10 | 1.05 | 22 mẫu |
| `wpn_arch_smallblade_t1` | **Dao Nhỏ Gọt Vỏ** | Dao Găm & Trủy Thủ | Bậc 1 | 0.30 kg | 0.10 – 0.10 | 3.6% | x5.5 | 3 | 1.00 | 11 mẫu |
| `wpn_arch_smallblade_t2` | **Trủy Thủ Ám Sát** | Dao Găm & Trủy Thủ | Bậc 2 | 0.55 kg | 0.14 – 0.35 | 11.2% | x8.2 | 3 | 1.00 | 11 mẫu |
| `wpn_arch_smallblade_t3` | **Dao Găm Đi Rừng** | Dao Găm & Trủy Thủ | Bậc 3 | 0.50 kg | 0.27 – 0.47 | 10.5% | x5.0 | 6 | 1.04 | 11 mẫu |
| `wpn_arch_smallblade_t4` | **Đao Chặt Xương** | Dao Găm & Trủy Thủ | Bậc 4 | 0.51 kg | 0.31 – 0.65 | 21.4% | x4.0 | 9 | 1.00 | 11 mẫu |
| `wpn_arch_smallblade_t5` | **Đao Lạc Diệp** | Dao Găm & Trủy Thủ | Bậc 5 | 0.83 kg | 0.54 – 1.02 | 30.0% | x4.1 | 7 | 1.00 | 15 mẫu |
| `wpn_arch_other_t1` | **Liềm Cắt Cỏ Thép** | Dụng Cụ Ứng Biến | Bậc 1 | 1.50 kg | 0.00 – 0.00 | 0.0% | x0.0 | 1 | 1.00 | 9 mẫu |
| `wpn_arch_other_t2` | **Cuốc Vỡ Đất** | Dụng Cụ Ứng Biến | Bậc 2 | 1.50 kg | 0.00 – 0.00 | 0.0% | x0.0 | 1 | 1.00 | 9 mẫu |
| `wpn_arch_other_t3` | **Xẻng Sắt Công Sự** | Dụng Cụ Ứng Biến | Bậc 3 | 1.50 kg | 0.00 – 0.00 | 0.0% | x0.0 | 1 | 1.00 | 9 mẫu |
| `wpn_arch_other_t4` | **Cào Cỏ Răng Sắt** | Dụng Cụ Ứng Biến | Bậc 4 | 0.72 kg | 0.03 – 0.03 | 0.0% | x0.0 | 1 | 1.00 | 9 mẫu |
| `wpn_arch_other_t5` | **Trượng Gỗ Thần Trấn** | Dụng Cụ Ứng Biến | Bậc 5 | 0.42 kg | 0.17 – 0.26 | 5.0% | x0.9 | 2 | 0.98 | 10 mẫu |

### 2.2. Bảng Hỏa Khí Cổ Truyền Đại Việt (Gunpowder Arsenal)
Ánh xạ từ động cơ đạn đạo thực nghiệm của Build 42, loại bỏ toàn bộ súng đạn hiện đại để xây dựng hỏa lực thời Trần – Hồ:

| Tên Hỏa Khí Đại Việt | Nguyên Mẫu Động Cơ PZ | Cỡ Nạp & Loại Đạn | Tầm Nổ Âm Thanh | Cự Ly Hiệu Quả | Sát Thương | Sức Dừng (Knockdown) | Tỷ Lệ Tịt Ngòi Khi Ẩm |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Súng Hỏa Mai Điểu Thương** | Varmint / Single-shot Hunting Rifle | 1 viên (Chì đúc + Dây cháy) | **60 ô gạch** | 25 ô | 1.8 – 2.5 | 1.2 | 15.0% |
| **Ống Hỏa Hổ Phun Đạn Rải** | Double Barrel Shotgun / Sawn-off | 2 nòng (Mảnh gang + Độc dược) | **200 ô gạch** | 12 ô (Quét diện) | 2.2 – 3.8 | 3.0 | 8.0% |
| **Pháo Thần Cơ Cầm Tay Hồ Nguyên Trừng** | Heavy Slug Shotgun / Magnum | 1 phát (Đạn đồng đặc phá giáp) | **350 ô gạch** | 35 ô | 4.0 – 6.5 | 5.0 | 5.0% |
| **Hỏa Xung Kỵ Binh Điểm Hỏa** | Revolver .38 / .45 Single Action | 1 phát (Nạp nhanh báng ngắn) | **70 ô gạch** | 15 ô | 1.4 – 2.0 | 1.0 | 12.0% |

### 2.3. Bảng Tham Chiếu Toàn Bộ 22 Khẩu Súng Gốc Của Động Cơ (`ranged_weapons`)

| Mã Súng (ID) | Loại Đạn | Hộp Đạn | Tầm Âm Thanh | Âm Lượng | Độ Giật | Tỷ Lệ Trúng | Sát Thương [Min - Max] | Tầm Xa |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `Revolver_CapGun` | `base:cap_gun_cap` | 0 | **30 ô** | 15 dB | 11 | 0% | 0.0 – 0.0 | 0.6 ô |
| `Rifle_CapGun` | `base:cap_gun_cap` | 0 | **30 ô** | 15 dB | 17 | 0% | 0.0 – 0.0 | 0.6 ô |
| `Revolver_Short` | `base:bullets_38` | 0 | **60 ô** | 25 dB | 11 | 50% | 0.5 – 0.8 | 8.0 ô |
| `Pistol2` | `base:bullets_45` | 15 | **70 ô** | 50 dB | 15 | 50% | 0.9 – 1.2 | 12.0 ô |
| `TrapperCarbine` | `base:bullets_45` | 0 | **70 ô** | 50 dB | 12 | 50% | 1.0 – 1.4 | 20.0 ô |
| `Pistol` | `base:bullets_9mm` | 15 | **100 ô** | 30 dB | 12 | 50% | 0.6 – 1.0 | 15.0 ô |
| `Revolver` | `base:bullets_357` | 15 | **100 ô** | 65 dB | 16 | 50% | 0.9 – 1.4 | 12.0 ô |
| `Pistol3` | `base:bullets_44` | 15 | **120 ô** | 75 dB | 17 | 50% | 1.0 – 1.6 | 17.0 ô |
| `Revolver_Long` | `base:bullets_44` | 15 | **120 ô** | 75 dB | 17 | 50% | 1.0 – 1.6 | 18.0 ô |
| `L92_Carbine` | `base:bullets_357` | 0 | **120 ô** | 75 dB | 12 | 50% | 1.0 – 1.6 | 24.0 ô |
| `AssaultRifle` | `base:bullets_556` | 0 | **150 ô** | 35 dB | 15 | 50% | 1.0 – 1.6 | 30.0 ô |
| `VarmintRifle` | `base:bullets_556` | 0 | **150 ô** | 200 dB | 17 | 50% | 1.0 – 1.6 | 30.0 ô |
| `JS14_Rifle` | `base:bullets_556` | 0 | **150 ô** | 35 dB | 15 | 45% | 1.0 – 1.6 | 30.0 ô |
| `L94_Rifle` | `base:bullets_3030` | 0 | **160 ô** | 200 dB | 20 | 50% | 1.2 – 2.0 | 30.0 ô |
| `AssaultRifle2` | `base:bullets_308` | 20 | **170 ô** | 30 dB | 20 | 50% | 1.2 – 2.0 | 40.0 ô |
| `HuntingRifle` | `base:bullets_308` | 0 | **170 ô** | 200 dB | 23 | 50% | 1.2 – 2.0 | 40.0 ô |
| `MSR7T_Rifle` | `base:bullets_308` | 0 | **170 ô** | 200 dB | 21 | 50% | 1.2 – 2.0 | 40.0 ô |
| `DoubleBarrelShotgun` | `base:shotgun_shells` | 0 | **200 ô** | 200 dB | 30 | 65% | 1.5 – 2.2 | 15.0 ô |
| `DoubleBarrelShotgunSawnoff` | `base:shotgun_shells` | 0 | **200 ô** | 200 dB | 34 | 70% | 1.2 – 1.8 | 8.0 ô |
| `Shotgun` | `base:shotgun_shells` | 0 | **200 ô** | 200 dB | 30 | 65% | 1.5 – 2.2 | 12.0 ô |
| `ShotgunSawnoff` | `base:shotgun_shells` | 6 | **200 ô** | 250 dB | 30 | 70% | 1.3 – 2.0 | 10.0 ô |
| `JS3T_Shotgun` | `base:shotgun_shells` | 0 | **200 ô** | 200 dB | 28 | 65% | 1.5 – 2.2 | 12.0 ô |

---

## 3. KHO NÔNG NGHIỆP, SINH THÁI & DÃ NGOẠI

### 3.1. Bảng 43 Loại Cây Trồng & Nam Dược Cổ Truyền (`crops`)

| Tên Cây Trồng / Dược Thảo | Phân Loại | Nước Tối Thiểu | Nước Lý Tưởng | Ngày Thu Hoạch | Sản Lượng [Min - Max] | Cấp Yêu Cầu | Chịu Lạnh Tuyệt Đối | Các Tháng Gieo Trồng |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Kale** | Rau Củ Lương Thực | 85 | 80 | 12.2 ngày | 2 – 4 | Cấp 5 | ✅ Có | Tháng 2, 3, 4, 5, 6, 7, 8 |
| **Barley** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 6 | ✅ Có | Tháng 8, 9, 10 |
| **Flax** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 6 | ✅ Có | Tháng 8, 9, 10 |
| **Garlic** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ✅ Có | Tháng 7, 8, 9 |
| **Rye** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 6 | ✅ Có | Tháng 8, 9, 10 |
| **Wheat** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 6 | ✅ Có | Tháng 8, 9, 10 |
| **BellPepper** | Rau Củ Lương Thực | 75 | 85 | 12.2 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 4, 5, 6 |
| **Greenpeas** | Rau Củ Lương Thực | 75 | 85 | 12.2 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 2, 3, 4 |
| **Lettuce** | Rau Củ Lương Thực | 75 | 85 | 12.2 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 2, 3, 4, 7, 8, 9 |
| **Turnip** | Rau Củ Lương Thực | 65 | 70 | 12.2 ngày | 3 – 4 | Cấp 5 | ❌ Không | Tháng 5, 6, 7, 8, 9, 10 |
| **Corn** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **Cucumber** | Rau Củ Lương Thực | 75 | 85 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **Hemp** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 6 | ❌ Không | Tháng 8, 9, 10 |
| **Hops** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 3 – 4 | Cấp 6 | ❌ Không | Tháng 8, 9, 10 |
| **Onion** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 2, 3 |
| **Pumpkin** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **Soybeans** | Rau Củ Lương Thực | 65 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **SweetPotato** | Rau Củ Lương Thực | 65 | 70 | 18.0 ngày | 3 – 4 | Cấp 5 | ❌ Không | Tháng 5, 6, 7 |
| **Watermelon** | Rau Củ Lương Thực | 35 | 80 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **Zucchini** | Rau Củ Lương Thực | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 3, 4, 5 |
| **WildGarlic** | Nam Dược / Thảo Mộc | 35 | 70 | 18.0 ngày | 2 – 4 | Cấp 5 | ✅ Có | Tháng 7, 8, 9, |
| **Leek** | Nam Dược / Thảo Mộc | 35 | 70 | 50.1 ngày | 2 – 4 | Cấp 3 | ✅ Có | Tháng 3, 4, 5, 6, 7, 8, 9 |
| **Basil** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **BlackSage** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8 |
| **BroadleafPlantain** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8 |
| **Chamomile** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **Chives** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8 |
| **Cilantro** | Nam Dược / Thảo Mộc | 70 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **Comfrey** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8 |
| **CommonMallow** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8 |
| **Lavender** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **LemonGrass** | Nam Dược / Thảo Mộc | 35 | 70 | 20.2 ngày | 2 – 4 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8, 9 |
| **Marigold** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **Mint** | Nam Dược / Thảo Mộc | 70 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 2, 3, 4 |
| **Oregano** | Nam Dược / Thảo Mộc | 60 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 2, 3, 4 |
| **Parsley** | Nam Dược / Thảo Mộc | 70 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 2, 3, 4 |
| **Poppies** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **Roses** | Nam Dược / Thảo Mộc | 80 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 3, 4, 5 |
| **Thyme** | Nam Dược / Thảo Mộc | 35 | 70 | 20.2 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 2, 3, 4 |
| **Habanero** | Nam Dược / Thảo Mộc | 75 | 85 | 20.3 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 4, 5, 6 |
| **Jalapeno** | Nam Dược / Thảo Mộc | 75 | 85 | 20.3 ngày | 2 – 4 | Cấp 5 | ❌ Không | Tháng 4, 5, 6 |
| **Sage** | Nam Dược / Thảo Mộc | 35 | 70 | 30.0 ngày | 4 – 6 | Cấp 3 | ❌ Không | Tháng 2, 3, 4 |
| **Rosemary** | Nam Dược / Thảo Mộc | 35 | 70 | 60.2 ngày | 2 – 4 | Cấp 3 | ❌ Không | Tháng 3, 4, 5, 6, 7, 8, 9 |

### 3.2. Bảng Trang Bị Bẫy Thú Dân Gian (`trapping_gear`) & Động Vật Hoang Dã (`wildlife_game`)

| Tên Bẫy Thú Dân Gian | Mã Định Danh Engine | Độ Bền Bẫy (Strength) | Loài Thú Săn Bắt | Tên Loại Động Vật | Khung Giờ Hoạt Động (Săn Bẫy) |
| :--- | :--- | :---: | :--- | :--- | :---: |
| **crateTrap** | `Base.TrapCrate` | 15 | **rabbit** | `rabbit` | 19:00 – 5:00 |
| **trapBox** | `Base.TrapBox` | 15 | **squirrel** | `squirrel` | 19:00 – 5:00 |
| **cageTrap** | `Base.TrapCage` | 20 | **bird** | `bird` | 0:00 – 0:00 |
| **snareTrap** | `Base.TrapSnare` | 10 | **mouse** | `mouse` | 0:00 – 0:00 |
| **stickTrap** | `Base.TrapStick` | 15 | **rat** | `rat` | 0:00 – 0:00 |
| **mouseTrap** | `Base.TrapMouse` | 50 | **raccoon** | `raccoon` | 19:00 – 5:00 |

---

## 4. HỆ THỐNG THÂN PHẬN & ĐẶC ĐIỂM NHÂN VẬT

### 4.1. Bảng 25 Thân Phận (Nghề Nghiệp Khởi Đầu - `professions`)

| Mã Thân Phận | Tên Nghề Nghiệp | Chi Phí Điểm (Point Cost) | Kỹ Năng Được Thưởng (XP Boosts) | Đặc Điểm Tặng Kèm (Granted Traits) |
| :--- | :--- | :---: | :--- | :--- |
| `base:unemployed` | **Custom Occupation** | **+8** | Không có | Không có |
| `base:burgerflipper` | **Burger Flipper** | **+2** | Cooking: +2, Maintenance: +1, SmallBlade: +1 | base:cook2 |
| `base:tailor` | **Tailor** | **+2** | Tailoring: +4 | Không có |
| `base:doctor` | **Doctor** | **+0** | Doctor: +6, SmallBlade: +1 | Không có |
| `base:farmer` | **Farmer** | **+0** | Farming: +4, Husbandry: +1, Strength: +1 | Không có |
| `base:fireofficer` | **Firefighter** | **+0** | Sprinting: +1, Strength: +1, Fitness: +1, Axe: +1 | Không có |
| `base:lumberjack` | **Lumberjack** | **+0** | Axe: +2, Strength: +1, Maintenance: +1 | base:axeman |
| `base:nurse` | **Nurse** | **+0** | Doctor: +3, Lightfoot: +1, Fitness: +1 | base:nightowl |
| `base:rancher` | **Rancher** | **+0** | Husbandry: +4, Butchering: +3, Fitness: +1 | Không có |
| `base:carpenter` | **Carpenter** | **-2** | Woodwork: +4, Carving: +1, SmallBlunt: +1, Masonry: +1, Maintenance: +1 | Không có |
| `base:chef` | **Chef** | **-2** | Cooking: +4, Maintenance: +1, SmallBlade: +1, Butchering: +2 | base:cook2 |
| `base:constructionworker` | **Construction Worker** | **-2** | SmallBlunt: +2, Blunt: +1, Masonry: +2, Woodwork: +1, Maintenance: +1 | Không có |
| `base:electrician` | **Electrician** | **-2** | Electricity: +5 | Không có |
| `base:fisherman` | **Fishing Guide** | **-2** | Fishing: +3, PlantScavenging: +1, Butchering: +1 | Không có |
| `base:repairman` | **DIY Expert** | **-2** | Woodwork: +1, Maintenance: +2, SmallBlunt: +1, Masonry: +1 | base:inventive_prof |
| `base:securityguard` | **Security Guard** | **-2** | Sprinting: +2, Lightfoot: +1, SmallBlunt: +1 | base:nightowl |
| `base:engineer` | **Engineer** | **-4** | Electricity: +1, Woodwork: +1, Masonry: +1 | Không có |
| `base:mechanics` | **Mechanic** | **-4** | Mechanics: +4, MetalWelding: +1 | base:mechanics2 |
| `base:metalworker` | **Welder** | **-4** | MetalWelding: +4 | Không có |
| `base:parkranger` | **Park Ranger** | **-4** | Trapping: +1, Doctor: +1, PlantScavenging: +1, FlintKnapping: +1, Carving: +1 | base:herbalist_prof |
| `base:policeofficer` | **Police Officer** | **-4** | Aiming: +4, Reloading: +1, Nimble: +1 | Không có |
| `base:burglar` | **Burglar** | **-6** | Nimble: +2, Sneak: +2, Lightfoot: +2 | base:burglar |
| `base:fitnessinstructor` | **Fitness Instructor** | **-6** | Fitness: +3, Sprinting: +2, Strength: +1 | base:nutritionist2 |
| `base:smither` | **Blacksmith** | **-6** | Blacksmith: +4, Maintenance: +1, SmallBlunt: +1 | base:blacksmith2 |
| `base:veteran` | **Veteran** | **-8** | Aiming: +2, Reloading: +2 | base:desensitized |

### 4.2. Bảng 95 Đặc Điểm Cốt Lõi (Traits & Feats - `traits`)

| Mã Đặc Điểm | Tên Đặc Điểm | Chi Phí Điểm | Mô Tả Tác Động Hệ Thống | Kỹ Năng Tăng Thêm |
| :--- | :--- | :---: | :--- | :--- |
| `base:athletic` | **Athletic** | **+10** | Faster running speed.<br>Can run for longer without tiring. | Fitness: +4 |
| `base:strong` | **Strong** | **+10** | Extra knockback from melee weapons.<br>Increased carrying weight. | Strength: +4 |
| `base:wildernessknowledge` | **Bushcrafter** | **+8** | Can find medicinal herbs and craft medicines and poultices from them, and make simple stone and bone tools. <br> Start fires faster. | PlantScavenging: +1, FlintKnapping: +1, Maintenance: +1, Carving: +1 |
| `base:handy` | **Handy** | **+8** | Faster and stronger constructions. | Carving: +1, Maintenance: +1, Masonry: +1, Woodwork: +1 |
| `base:hunter` | **Hunter** | **+8** | Không có mô tả chi tiết. | Aiming: +1, Tracking: +1, Sneak: +1, SmallBlade: +1, Butchering: +1 |
| `base:thickskinned` | **Thick-skinned** | **+8** | Lower chance of being scratched or bitten. | - |
| `base:blacksmith` | **Blacksmith Knowledge** | **+6** | Can use an anvil to create metal items. | Blacksmith: +2, Maintenance: +1 |
| `base:brawler` | **Brawler** | **+6** | Không có mô tả chi tiết. | Axe: +1, Blunt: +1 |
| `base:fasthealer` | **Fast Healer** | **+6** | Recovers quickly from injuries and illness. | - |
| `base:fastlearner` | **Fast Learner** | **+6** | Increased XP gains. | - |
| `base:fit` | **Fit** | **+6** | Không có mô tả chi tiết. | Fitness: +2 |
| `base:formerscout` | **Former Scout** | **+6** | Start fires faster. | Doctor: +1, PlantScavenging: +1, Fishing: +1 |
| `base:keenhearing` | **Keen Hearing** | **+6** | Larger perception radius. | - |
| `base:stout` | **Stout** | **+6** | Extra knockback from melee weapons.<br>Increased carrying weight. | Strength: +2 |
| `base:gymnast` | **Gymnast** | **+5** | Không có mô tả chi tiết. | Lightfoot: +1, Nimble: +1 |
| `base:hiker` | **Hiker** | **+5** | Không có mô tả chi tiết. | PlantScavenging: +1, Trapping: +1 |
| `base:target_shooter` | **Target Shooter** | **+5** | Không có mô tả chi tiết. | Aiming: +1 |
| `base:adrenalinejunkie` | **Adrenaline Junkie** | **+4** | Moves faster when highly panicked. | - |
| `base:fishing` | **Angler** | **+4** | Không có mô tả chi tiết. | Fishing: +1 |
| `base:baseballplayer` | **Baseball Player** | **+4** | Không có mô tả chi tiết. | Blunt: +1 |
| `base:brave` | **Brave** | **+4** | Less prone to becoming panicked. | - |
| `base:eagleeyed` | **Eagle Eyed** | **+4** | Faster visibility fade.<br>Higher visibility arc.<br>Weapon sights more effective at long range. | - |
| `base:graceful` | **Graceful** | **+4** | Makes less noise when moving. | - |
| `base:herbalist` | **Herbalist** | **+4** | Can find medicinal herbs and craft medicines and poultices from them. | PlantScavenging: +1 |
| `base:inconspicuous` | **Inconspicuous** | **+4** | Less likely to be spotted by zombies. | - |
| `base:organized` | **Organized** | **+4** | Increased container inventory capacity. | - |
| `base:resilient` | **Resilient** | **+4** | Less prone to disease.<br>Slower rate of zombification. | - |
| `base:jogger` | **Runner** | **+4** | Không có mô tả chi tiết. | Sprinting: +1 |
| `base:tailor` | **Sewer** | **+4** | Không có mô tả chi tiết. | Tailoring: +1 |
| `base:tinkerer` | **Tinkerer** | **+4** | Không có mô tả chi tiết. | Maintenance: +1 |
| `base:nightvision` | **Cat's Eyes** | **+3** | Better vision at night. | - |
| `base:crafty` | **Crafty** | **+3** | Increased XP gains for Crafting skills. | - |
| `base:cook` | **Keen Cook** | **+3** | Knows cooking recipes. | Cooking: +2, Butchering: +1 |
| `base:mechanics` | **Vehicle Knowledge** | **+3** | Has knowledge of common and commercial vehicle models, and repairs. | Mechanics: +1 |
| `base:needslesssleep` | **Wakeful** | **+3** | Needs less sleep. | - |
| `base:artisan` | **Artisan** | **+2** | Better at pottery and glass crafts. | Glassmaking: +1, Pottery: +1 |
| `base:dextrous` | **Dextrous** | **+2** | Transfers inventory items quickly.<br>Faster rope climbing speed, and less chance of falling.<br>Faster weapon shouldering time. | - |
| `base:fastreader` | **Fast Reader** | **+2** | Takes less time to read books. | - |
| `base:firstaid` | **First Aider** | **+2** | Không có mô tả chi tiết. | Doctor: +1 |
| `base:gardener` | **Gardener** | **+2** | Không có mô tả chi tiết. | Farming: +1 |
| `base:inventive` | **Inventive** | **+2** | Has lower skill level requirements to research recipes from items or auto learn recipes. | - |
| `base:irongut` | **Iron Gut** | **+2** | Less chance to have food illness. | - |
| `base:lighteater` | **Light Eater** | **+2** | Needs to eat less regularly. | - |
| `base:lowthirst` | **Low Thirst** | **+2** | Needs less water to survive. | - |
| `base:mason` | **Mason** | **+2** | Better at building stone and brick constructions. | Masonry: +2 |
| `base:nutritionist` | **Nutritionist** | **+2** | Can see the nutritional values of any food. | - |
| `base:outdoorsman` | **Outdoorsy** | **+2** | Less affected by harsh weather conditions. | - |
| `base:whittler` | **Whittler** | **+2** | Can carve wood and bone items. | Carving: +2 |
| `base:speeddemon` | **Speed Demon** | **+1** | Drives very fast. | - |
| `base:axeman` | **Ax-pert** | **+0** | Better at chopping trees.<br>Faster axe swing. | - |
| `base:blacksmith2` | **Blacksmith Knowledge** | **+0** | Can use an anvil to create metal items. | - |
| `base:burglar` | **Burglar** | **+0** | Can hotwire vehicles. Less chance of breaking window locks. | - |
| `base:desensitized` | **Desensitized** | **+0** | Far less prone to panic. <br>Ignore discomfort. | - |
| `base:herbalist_prof` | **Herbalist** | **+0** | Can find medicinal herbs and craft medicines and poultices from them. | PlantScavenging: +1 |
| `base:overweight` | **High Weight** | **+0** | Reduced running speed, low endurance and prone to injury. | Fitness: +-1 |
| `base:inventive_prof` | **Inventive** | **+0** | Has lower skill level requirements to research recipes from items or auto learn recipes. | - |
| `base:cook2` | **Keen Cook** | **+0** | Knows cooking recipes. | - |
| `base:underweight` | **Low Weight** | **+0** | Low strength, low endurance and prone to injury. | Fitness: +-1 |
| `base:marksman` | **Marksman** | **+0** | Improved gun accuracy and damage.<br>Quicker reload. | - |
| `base:nightowl` | **Night Owl** | **+0** | Requires little sleep.<br>Stays extra alert even when sleeping. | - |
| `base:nutritionist2` | **Nutritionist** | **+0** | Can see the nutritional values of any food. | - |
| `base:mechanics2` | **Vehicle Knowledge** | **+0** | Has knowledge of common and commercial vehicle models, and repairs. | - |
| `base:obese` | **Very High Weight** | **+0** | Reduced running speed, very low endurance and prone to injury. | Fitness: +-2 |
| `base:sundaydriver` | **Sunday Driver** | **-1** | Drives very slow. | - |
| `base:allthumbs` | **All Thumbs** | **-2** | Transfers inventory items slowly. Can't craft anything while walking.<br>Slower rope climbing speed, and higher chance of falling.<br>Slower weapon shouldering time. | - |
| `base:clumsy` | **Clumsy** | **-2** | Makes more noise when moving. | - |
| `base:cowardly` | **Cowardly** | **-2** | Especially prone to becoming panicked. | - |
| `base:weightloss` | **Fast Metabolism** | **-2** | Permanent tendency to lose weight. Starts with Low Weight trait. | - |
| `base:highthirst` | **High Thirst** | **-2** | Needs more water to survive. | - |
| `base:shortsighted` | **Short Sighted** | **-2** | Smaller view distance.<br>Slower visibility fade.<br>Weapon sights less effective. | - |
| `base:weightgain` | **Slow Metabolism** | **-2** | Permanent tendency to gain weight. Starts with High Weight trait. | - |
| `base:slowreader` | **Slow Reader** | **-2** | Takes longer to read books and other literature. | - |
| `base:weakstomach` | **Weak Stomach** | **-2** | Higher chance to have food illness. | - |
| `base:slowhealer` | **Slow Healer** | **-3** | Recovers slowly from injuries and illness. | - |
| `base:smoker` | **Smoker** | **-3** | Unhappiness rises when tobacco is not smoked.<br>Stress and unhappiness decrease after smoking tobacco. | - |
| `base:agoraphobic` | **Agoraphobic** | **-4** | Gets panicked when outdoors. | - |
| `base:claustrophobic` | **Claustrophobic** | **-4** | Gets panicked when in small indoor rooms. | - |
| `base:conspicuous` | **Conspicuous** | **-4** | More likely to be spotted by zombies. | - |
| `base:hardofhearing` | **Hard of Hearing** | **-4** | Smaller perception radius.<br>Smaller hearing range. | - |
| `base:heartyappetite` | **Hearty Appetite** | **-4** | Needs to eat more regularly. | - |
| `base:pronetoillness` | **Prone to Illness** | **-4** | More prone to disease.<br>Faster rate of zombification. | - |
| `base:needsmoresleep` | **Sleepyhead** | **-4** | Needs more sleep. | - |
| `base:hemophobic` | **Fear of Blood** | **-5** | Panic when performing first aid on self.<br>Cannot perform first aid on others.<br>Gets stressed when bloody. | - |
| `base:pacifist` | **Reluctant Fighter** | **-5** | Decreased combat XP gains. | - |
| `base:asthmatic` | **Short of Breath** | **-5** | Faster endurance loss. | - |
| `base:disorganized` | **Disorganized** | **-6** | Decreased container inventory capacity.<br>Affects world containers and bags, but not your main inventory. | - |
| `base:insomniac` | **Restless Sleeper** | **-6** | Slow loss of tiredness while sleeping. | - |
| `base:slowlearner` | **Slow Learner** | **-6** | Decreased XP gains. | - |
| `base:feeble` | **Weak** | **-6** | Less knockback from melee weapons.<br>Decreased carrying weight. | Strength: +-2 |
| `base:thinskinned` | **Thin-skinned** | **-8** | Higher chance of being scratched or bitten by zombies. | - |
| `base:emaciated` | **Emaciated** | **-10** | Low strength, low endurance and prone to injury. | - |
| `base:illiterate` | **Illiterate** | **-10** | Cannot read any books or in-world text. | - |
| `base:weak` | **Puny** | **-10** | Far less knockback from melee weapons.<br>Extremely small carrying weight. | Strength: +-5 |
| `base:unfit` | **Unfit** | **-10** | Very low endurance and endurance regeneration. | Fitness: +-4 |
| `base:deaf` | **Deaf** | **-12** | Can't hear sound. | - |

---

## 5. CÂY CÔNG NGHỆ & TRẠM CHẾ TÁC CỔ ĐẠI (1.026 RECIPES)

### 5.1. Bảng Phân Bổ Công Thức Theo Trạm Chế Tác Cổ Phong (`recipes`)

| Phân Loại Chế Tác (Category) | Tập Tin Trạm Chế Tác (Workstation File) | Số Lượng Công Thức | Khả Năng Tương Thích Cổ Đại |
| :--- | :--- | :---: | :---: |
| `Tailoring` | `recipes_tailoring_leatherAndHide.txt` | **60 công thức** | ✅ Cổ Đại Tương Thích |
| `Evolvedrecipes` | `evolvedrecipes.txt` | **56 công thức** | ✅ Cổ Đại Tương Thích |
| `Cooking` | `recipes_cooking.txt` | **52 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes_tailoring.txt` | **40 công thức** | ✅ Cổ Đại Tương Thích |
| `Tools` | `recipes_blacksmith_tools.txt` | **37 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmithing` | `recipes_blacksmith_furnaces_i.txt` | **34 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes_tailoring_garbageTapeAndTarp.txt` | **33 công thức** | ✅ Cổ Đại Tương Thích |
| `Carving` | `recipes_carving.txt` | **32 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes_improvised_weapons.txt` | **30 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes_tailoring_armor.txt` | **30 công thức** | ✅ Cổ Đại Tương Thích |
| `Carpentry` | `recipes_carpentry.txt` | **26 công thức** | ✅ Cổ Đại Tương Thích |
| `Packing` | `recipes_packing.txt` | **25 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `entity_Herb_Drying_Rack_craftRecipe.txt` | **25 công thức** | ✅ Cổ Đại Tương Thích |
| `Recipes` | `recipes.txt` | **23 công thức** | ✅ Cổ Đại Tương Thích |
| `Pottery` | `craftrecipe_kiln.txt` | **20 công thức** | ✅ Cổ Đại Tương Thích |
| `Cooking` | `recipes_baking.txt` | **20 công thức** | ✅ Cổ Đại Tương Thích |
| `Blade` | `recipes_blacksmith_blades.txt` | **17 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmithing` | `recipes_blacksmith_bar.txt` | **17 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_blacksmith_other_metals.txt` | **16 công thức** | ✅ Cổ Đại Tương Thích |
| `Armor` | `recipes_blacksmith_armor.txt` | **16 công thức** | ✅ Cổ Đại Tương Thích |
| `Tools` | `recipes_blacksmith_heads.txt` | **14 công thức** | ✅ Cổ Đại Tương Thích |
| `Carving` | `recipes_bone.txt` | **14 công thức** | ✅ Cổ Đại Tương Thích |
| `Pottery` | `craftrecipe_potterywheel.txt` | **13 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_farming.txt` | **13 công thức** | ✅ Cổ Đại Tương Thích |
| `Repair` | `recipes_fixing.txt` | **12 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_gasmasks.txt` | **11 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_blacksmith_bar.txt` | **11 công thức** | ✅ Cổ Đại Tương Thích |
| `Knapping` | `recipes_knapping.txt` | **11 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes_blacksmith_welded_etc_weapons.txt` | **10 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_metalWelding.txt` | **10 công thức** | ✅ Cổ Đại Tương Thích |
| `Cookware` | `recipes_blacksmith_cookware.txt` | **10 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmithing` | `recipes_blacksmith_other_metals.txt` | **10 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes_tailoring_knitting.txt` | **9 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes_bone.txt` | **9 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_metalWelding_Armor.txt` | **9 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_leather_prep.txt` | **9 công thức** | ✅ Cổ Đại Tương Thích |
| `Cooking` | `recipes_cannedFood.txt` | **9 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_blacksmith_standingdrill.txt` | **8 công thức** | ✅ Cổ Đại Tương Thích |
| `Medical` | `recipes_medical.txt` | **8 công thức** | ✅ Cổ Đại Tương Thích |
| `Masonry` | `recipes_stonemasonry_i.txt` | **8 công thức** | ✅ Cổ Đại Tương Thích |
| `Glassmaking` | `recipes_glassmaking.txt` | **8 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `entity_Drying_Rack_craftRecipe.txt` | **8 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_lightsources.txt` | **7 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_bone.txt` | **7 công thức** | ✅ Cổ Đại Tương Thích |
| `Cookware` | `recipes_blacksmith_other_metals.txt` | **7 công thức** | ✅ Cổ Đại Tương Thích |
| `Assembly` | `recipes_assembly.txt` | **7 công thức** | ✅ Cổ Đại Tương Thích |
| `Pottery` | `craftrecipe_handpress.txt` | **6 công thức** | ✅ Cổ Đại Tương Thích |
| `Fishing` | `recipes_fishing.txt` | **6 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes_spears.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes_bone.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes_blacksmith_heads.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Pottery` | `recipes_pottery.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Packing` | `recipes_ammunition.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_trapping.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `entity_stone_mill_craftRecipe.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Assembly` | `recipes_disassembly.txt` | **5 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_sleepingbags_and_tents.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_packing.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Metalworking` | `recipes_blacksmith_bandsaw.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmithing` | `recipes_blacksmith_ii.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmith_woodcharcoal` | `recipes_blacksmith_woodcharcoal.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Assembly` | `recipes_knapping.txt` | **4 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_tobacco.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Masonry` | `recipes_buckets.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_spinning_wheel.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `entity_stone_quern_craftRecipe.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `entity_simple_loom_craftRecipe.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Cooking` | `recipes_fluids.txt` | **3 công thức** | ✅ Cổ Đại Tương Thích |
| `Weaponry` | `recipes.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Tailoring` | `recipes.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Repair` | `recipes.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Packing` | `recipes_sacks.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_survivalist.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_carpentry.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_cardsAndDice.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Kiln` | `craftrecipe_kiln.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Glassmaking` | `recipes_blacksmith_furnace_ii.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_flax.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Cooking` | `recipes_jarring.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Carpentry` | `recipes_buckets.txt` | **2 công thức** | ✅ Cổ Đại Tương Thích |
| `Tools` | `recipes_blacksmith_ii.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_traps.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_medical.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Miscellaneous` | `recipes_camping.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Leather_prep_blair` | `recipes_leather_prep_blair.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Fishing_evolvedrecipe` | `recipes_fishing_evolvedrecipe.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_fiber.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `recipes_butter_churn.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Farming` | `craftrecipe_handpress.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Entity_dome_kiln_craftRecipe` | `entity_dome_kiln_craftRecipe.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Carving` | `recipes_spears.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Blacksmithing` | `recipes_blacksmith_furnace_ii.txt` | **1 công thức** | ✅ Cổ Đại Tương Thích |
| `Electrical` | `recipes_traps.txt` | **16 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |
| `Electrical` | `recipes_electrical.txt` | **9 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |
| `Electrical` | `recipes_radio.txt` | **4 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |
| `Blacksmith_bandsaw` | `recipes_blacksmith_bandsaw.txt` | **3 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |
| `Cooking` | `entity_coffeemachine_craftRecipe.txt` | **2 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |
| `Cooking` | `entity_toaster_craftRecipe.txt` | **1 công thức** | ❌ Hiện Đại (Đã Lọc Bỏ) |

---

## 6. QUAN TƯỢNG ĐÀI CHIÊM TINH & 64 BÍ TỊCH CỔ THƯ

### 6.1. Hệ Thống Quan Tượng Đài & Điềm Báo Thiên Văn (`astronomical_telemetry`)
Trừu tượng hóa từ mạng lưới phát sóng khẩn cấp tự động (AEBS), chuyển thành hệ thống chiêm tinh dự báo thời tiết và điềm báo đại họa cho Session 2:

| Mã Tín Hiệu Điềm Báo | Phân Loại Dự Báo | Nội Dung Điềm Báo Trích Xuất Từ Engine |
| :--- | :--- | :--- |
| `AEBS_Intro` | `dynamicRadio` | *"<bzzt> Fiver Zero Two. <wzzt> Automated Broadcast System. <bzzt>"* |
| `AEBS_Choppah` | `dynamicRadio` | *"Air Activity detected."* |
| `AEBS_Power_1` | `dynamicRadio` | *"Knox Power Grid: Power fluctuations detected."* |
| `AEBS_Power_2` | `dynamicRadio` | *"Knox Power Grid: Systems failing. Network compromised."* |
| `AEBS_Power_3` | `dynamicRadio` | *"Knox Power Grid: Blackout."* |
| `AEBS_Pre_today` | `dynamicRadio` | *"Today,"* |
| `AEBS_Pre_tomorrow` | `dynamicRadio` | *"Tomorrow,"* |
| `AEBS_Pre_dayafter` | `dynamicRadio` | *"Day after tomorrow,"* |
| `AEBS_temperature` | `dynamicRadio` | *"average temperature %1. Minimum: %2. Maximum: %3. Humidity: %4%%..."* |
| `AEBS_wind_0` | `dynamicRadio` | *"%1 wind from the %2. Maximum of %3 expected. %4"* |
| `AEBS_wind_1` | `dynamicRadio` | *"Mild"* |
| `AEBS_wind_2` | `dynamicRadio` | *"Moderate"* |
| `AEBS_wind_3` | `dynamicRadio` | *"Strong"* |
| `AEBS_wind_4` | `dynamicRadio` | *"Storm-strength"* |
| `AEBS_clouds_0` | `dynamicRadio` | *"Clear skies."* |
| `AEBS_clouds_1` | `dynamicRadio` | *"Some clouds."* |
| `AEBS_clouds_2` | `dynamicRadio` | *"Heavy cloud cover."* |
| `AEBS_clouds_3` | `dynamicRadio` | *"Periodical cloudy spells."* |
| `AEBS_clouds_4` | `dynamicRadio` | *"Periods of heavy cloud."* |
| `AEBS_fog_0` | `dynamicRadio` | *"Light fog."* |
| `AEBS_fog_1` | `dynamicRadio` | *"Thick fog."* |
| `AEBS_fog_2` | `dynamicRadio` | *"Very thick fog."* |
| `AEBS_weather_0_a` | `dynamicRadio` | *"WARNING: Period of weather predicted to start in the %1..."* |
| `AEBS_weather_0_b` | `dynamicRadio` | *"Weather continues throughout the day..."* |
| `AEBS_weather_0_c` | `dynamicRadio` | *"Weather period predicted to end in the %1..."* |
| `AEBS_weather_predicted` | `dynamicRadio` | *"expecting "* |
| `AEBS_weather_and_a` | `dynamicRadio` | *"and a "* |
| `AEBS_weather_light_moderate` | `dynamicRadio` | *"expecting light to moderate rain..."* |
| `AEBS_weather_heavy_rain` | `dynamicRadio` | *"showers and or heavy rain"* |
| `AEBS_weather_storm` | `dynamicRadio` | *"thunderstorm"* |
| `AEBS_weather_tropical` | `dynamicRadio` | *"tropical storm"* |
| `AEBS_weather_blizzard` | `dynamicRadio` | *"blizzard"* |
| `AEBS_weather_snowfall` | `dynamicRadio` | *"with a chance of snowfall."* |
| `AEBS_weather_storm_C` | `dynamicRadio` | *"Thunderstorm"* |
| `AEBS_weather_tropical_C` | `dynamicRadio` | *"Tropical storm"* |

### 6.2. Danh Mục Toàn Bộ 64 Bí Tịch Cổ Thư & Thư Tịch Võ Thuật (`skill_tomes`)
Toàn bộ 64 Băng VHS kỹ năng có giá trị thực chiến đã được lọc bỏ yếu tố hiện đại và chuyển hóa thành 64 Pho Bí Tịch truyền thụ kinh nghiệm:

| Mã Bí Tịch | Tên Bí Tịch Cổ Thư | Tác Giả / Xuất Xứ | Phân Loại | Kỹ Năng Truyền Thụ (XP Boosts) | Công Thức Bí Truyền Mở Khóa |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `f33ce778...` | **Basic Gun Handling (Officer Use Only)** | Khuyết Danh | `Home-VHS` | **AIM+2, REL+2, AIM+1, REL+1** | Không có |
| `8c150873...` | **Combat Wound Management** | Khuyết Danh | `Home-VHS` | **DOC+1** | Không có |
| `541b5fff...` | **Grady v King** | Khuyết Danh | `Home-VHS` | **LFT+1** | Không có |
| `00a8b565...` | **Granny Nani** | Khuyết Danh | `Home-VHS` | **TAI+1** | Không có |
| `ba9a65a7...` | **HOLLYWOOD INSIDER SPECIAL** | Khuyết Danh | `Home-VHS` | **STS+0.1** | Không có |
| `68785929...` | **Mathematical Quadratics and Algebraic Configurations** | Khuyết Danh | `Home-VHS` | **BUA+1** | Không có |
| `ffe28381...` | **Muldraugh AV Club** | Khuyết Danh | `Home-VHS` | **ELC+1** | CraftMakeshiftRadio |
| `1559a55a...` | **OSCC '92** | Khuyết Danh | `Home-VHS` | **MEC+1** | Không có |
| `14fca23a...` | **RMFA** | Khuyết Danh | `Home-VHS` | **DOC+1** | Không có |
| `c23539e8...` | **TV repair** | Khuyết Danh | `Home-VHS` | **ELC+1** | Không có |
| `af941537...` | **Tailoring 101** | Khuyết Danh | `Home-VHS` | **TAI+1** | Không có |
| `db7deaf2...` | **Tree Planting Guide** | Khuyết Danh | `Home-VHS` | **FRM+1** | Không có |
| `fe4ede3c...` | **no 9** | Khuyết Danh | `Home-VHS` | **MTL+1** | MetalFenceLvl1, MetalSmallPoleFence, MetalPoleFenceGateSmall, MetalBigMetalFence, MetalPoleFenceGate, MetalSmallWireFence, MetalWireFenceGateSmall, MetalWireFenceGate, MetalBigWireFence, DoubleWireGate |
| `1c6745f6...` | **nof vid** | Khuyết Danh | `Home-VHS` | **MTL+1** | MetalFloorLvl1, MetalWallLvl1 |
| `8904b5c7...` | **stock cars** | Khuyết Danh | `Home-VHS` | **MEC+1** | Không có |
| `fa0cd1c6...` | **A Day on the Farm** | Khuyết Danh | `Retail-VHS` | **HUS+1** | Không có |
| `066ed71a...` | **A Stitch in Time** | Khuyết Danh | `Retail-VHS` | **TAI+1** | Không có |
| `89d33c66...` | **Adefope Fencing Special** | Khuyết Danh | `Retail-VHS` | **MTL+1** | MetalFenceLvl1, MetalSmallPoleFence, MetalPoleFenceGateSmall, MetalBigMetalFence, MetalPoleFenceGate, MetalWireFenceGateSmall, MetalSmallWireFence, MetalWireFenceGate, MetalBigWireFence, DoubleWireGate |
| `c7d905cf...` | **Better Fishing With Jason Master** | Khuyết Danh | `Retail-VHS` | **FIS+1** | MakeFishingRod, FixFishingRod |
| `07b0cd73...` | **Carzone** | Khuyết Danh | `Retail-VHS` | **MEC+1** | Không có |
| `7b192b30...` | **Carzone** | Khuyết Danh | `Retail-VHS` | **MEC+1** | Không có |
| `313fa630...` | **Carzone** | Khuyết Danh | `Retail-VHS` | **MEC+1** | Không có |
| `4653d39e...` | **Combat Wound Management** | Khuyết Danh | `Retail-VHS` | **DOC+1** | Không có |
| `4cd4e3d3...` | **Controlling Nasty Crop Pests and Diseases** | Khuyết Danh | `Retail-VHS` | **FRM+1** | MakeMildewCure, MakeFliesCure, MakeSlugTrap, MakeAphidsCure |
| `31a726b7...` | **Dead Wrong** | Khuyết Danh | `Retail-VHS` | **REL+1, AIM+1** | Không có |
| `955bf539...` | **Emergency First Aid** | Khuyết Danh | `Retail-VHS` | **DOC+1** | Không có |
| `ccfd97d7...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **FIS+1** | MakeFishingRod |
| `6f1be5b5...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **FIS+1** | Không có |
| `7402fb83...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **FRM+1** | Không có |
| `a628a723...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `1123df57...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **CRP+1, FOR+1** | Không có |
| `75928dcd...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **TRA+1** | MakeWoodenBoxTrap |
| `2dcd687b...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **FOR+1** | Không có |
| `f74df7fc...` | **Exposure Survival** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `200e0912...` | **From Ore to Store** | Khuyết Danh | `Retail-VHS` | **BLA+1** | Không có |
| `1a22a4b3...` | **Growing Fruit and Veg at Home 1** | Khuyết Danh | `Retail-VHS` | **FRM+1** | base:cabbage growing season, base:potato growing season, base:radish growing season, base:carrot growing season, base:onion growing season, base:broccoli growing season, base:strawberry growing season, base:tomato growing season |
| `a7cd7482...` | **Growing Fruit and Veg at Home 2** | Khuyết Danh | `Retail-VHS` | **FRM+1** | base:green pea growing season, base:soybean growing season, base:kale growing season, base:garlic growing season, base:sweet potato growing season, base:corn growing season |
| `1c687700...` | **Growing Herbs at Home** | Khuyết Danh | `Retail-VHS` | **FRM+1** | base:basil growing season, base:chives growing season, base:cilantro growing season, base:oregano growing season, base:parsley growing season, base:sage growing season, base:rosemary growing season, base:thyme growing season |
| `c1a3d275...` | **Home Welding Guide** | Khuyết Danh | `Retail-VHS` | **MTL+1** | Không có |
| `9f33c7f2...` | **How Electricity Works** | Khuyết Danh | `Retail-VHS` | **ELC+1** | Không có |
| `c3d6c81d...` | **Knapping: The Ancient Art** | Khuyết Danh | `Retail-VHS` | **FKN+1** | Không có |
| `729fc45b...` | **Knox Gun Owners Club's Guide to Guns** | Khuyết Danh | `Retail-VHS` | **REL+1, AIM+1** | Không có |
| `988e8c5a...` | **Making Sushi at Home** | Khuyết Danh | `Retail-VHS` | **COO+1** | MakeSushi, MakeMaki, MakeOnigiri, MakeFriedShrimp, MakeCabbageRolls |
| `9ec23879...` | **Mother's Boy** | Khuyết Danh | `Retail-VHS` | **SBA+1** | Không có |
| `7e395694...` | **Pottery for Anyone** | Khuyết Danh | `Retail-VHS` | **POT+1** | Không có |
| `0cd1fbcb...` | **Rosewood Medical First Aid** | Khuyết Danh | `Retail-VHS` | **DOC+1** | Không có |
| `e6fd44a9...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | MakeCakeBatter |
| `a754fdaf...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | Không có |
| `7933a371...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | Không có |
| `4cc3b806...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | Không có |
| `2695eb00...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | Không có |
| `01b3d832...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | Không có |
| `2c2a052d...` | **The Cook Show** | Khuyết Danh | `Retail-VHS` | **COO+1** | MakePieDough |
| `21411665...` | **The Dog Goblin** | Khuyết Danh | `Retail-VHS` | **STS+0.2, PAN+25, PAN+100** | Không có |
| `b4830b9a...` | **The Petting Zoo** | Khuyết Danh | `Retail-VHS` | **HUS+1** | Không có |
| `dedd4e83...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `c2d01da8...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `3e9c5cf0...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `0627a89a...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `2bf49f71...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `968591cd...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `01dedfea...` | **Woodcraft** | Khuyết Danh | `Retail-VHS` | **CRP+1** | Không có |
| `dccc28f4...` | **Working Clay at Home** | Khuyết Danh | `Retail-VHS` | **POT+1** | Không có |
| `4f3f30d6...` | **Z-Squad** | Khuyết Danh | `Retail-VHS` | **MTL+1, MEC+1** | Không có |

---

## 7. MẪU VẬT PHẨM ĐẠI DIỆN CỔ PHONG ĐÃ LỌC BỎ HIỆN ĐẠI

Trích xuất các mẫu vật phẩm đại diện tiêu biểu từ 5.092 vật phẩm gốc theo từng danh mục cổ điển:

### 7.1. Nhóm Vật Phẩm Đại Diện: `Food`

| Mã Vật Phẩm (ID) | Tên Vật Phẩm | Trọng Lượng (Weight) | Giá Trị Kim Loại (Metal Value) |
| :--- | :--- | :---: | :---: |
| `SugarPacket` | **Sugar Packet** | 0.01 kg | 0.0 |
| `Ladybug` | **Ladybug** | 0.01 kg | 0.0 |
| `Maggots` | **Maggots** | 0.01 kg | 0.0 |
| `Pillbug` | **Pillbug** | 0.01 kg | 0.0 |
| `Termites` | **Termites** | 0.01 kg | 0.0 |
| `Worm` | **Worm** | 0.01 kg | 0.0 |
| `SoybeansSeed` | **Soybeans (Dried)** | 0.02 kg | 0.0 |
| `SugarCubes` | **Sugar Cubes** | 0.02 kg | 0.0 |
| `SalamiSlice` | **Salami Slices** | 0.02 kg | 0.0 |
| `BaconBits` | **Bacon Bits** | 0.03 kg | 0.0 |
| `DeadMousePups` | **Mouse Pups (Dead)** | 0.03 kg | 0.0 |
| `DeadMousePupsSkinned` | **Skinned Mouse Pups (Dead)** | 0.03 kg | 0.0 |

### 7.2. Nhóm Vật Phẩm Đại Diện: `Container`

| Mã Vật Phẩm (ID) | Tên Vật Phẩm | Trọng Lượng (Weight) | Giá Trị Kim Loại (Metal Value) |
| :--- | :--- | :---: | :---: |
| `KeyRing` | **Key Ring** | 0.05 kg | 0.0 |
| `KeyRing_CarDealer` | **Key Ring** | 0.05 kg | 0.0 |
| `KeyRing_Large` | **Key Ring - Large** | 0.05 kg | 0.0 |
| `KeyRing_SecurityPass` | **Security Pass Key Ring** | 0.05 kg | 0.0 |
| `KeyRing_Forged` | **Key Ring - Forged** | 0.05 kg | 0.0 |
| `KeyRing_Forged_Gold` | **Key Ring - Gold** | 0.05 kg | 0.0 |
| `KeyRing_Forged_Silver` | **Key Ring - Silver** | 0.05 kg | 0.0 |
| `Plasticbag` | **Plastic Bag** | 0.10 kg | 0.0 |
| `Plasticbag_Clothing` | **Plastic Bag** | 0.10 kg | 0.0 |
| `Garbagebag` | **Garbage Bag** | 0.10 kg | 0.0 |
| `Bag_TrashBag` | **Garbage Bag** | 0.10 kg | 0.0 |
| `Plasticbag_Bags` | **Plastic Bag** | 0.10 kg | 0.0 |

### 7.3. Nhóm Vật Phẩm Đại Diện: `Material`

| Mã Vật Phẩm (ID) | Tên Vật Phẩm | Trọng Lượng (Weight) | Giá Trị Kim Loại (Metal Value) |
| :--- | :--- | :---: | :---: |
| `Paperclip` | **Paperclip** | 0.01 kg | 0.0 |
| `GoldScrap` | **Gold Fragments** | 0.05 kg | 0.0 |
| `SilverScrap` | **Silver Fragments** | 0.05 kg | 0.0 |
| `Nails` | **Nails** | 0.05 kg | 0.0 |
| `Screws` | **Screws** | 0.05 kg | 0.0 |
| `SteelSlug` | **Steel Slug** | 0.07 kg | 0.0 |
| `Thread` | **Thread** | 0.10 kg | 0.0 |
| `DentalFloss` | **Dental Floss** | 0.10 kg | 0.0 |
| `SteelPiece` | **Steel Piece** | 0.10 kg | 0.0 |
| `IronPiece` | **Iron Piece** | 0.10 kg | 0.0 |
| `WoolRaw` | **Raw Wool** | 0.10 kg | 0.0 |
| `CheeseCloth` | **Cheese Cloth** | 0.10 kg | 0.0 |

### 7.4. Nhóm Vật Phẩm Đại Diện: `ProtectiveGear`

| Mã Vật Phẩm (ID) | Tên Vật Phẩm | Trọng Lượng (Weight) | Giá Trị Kim Loại (Metal Value) |
| :--- | :--- | :---: | :---: |
| `Hat_BoxingRed` | **Boxing Head Gear** | 0.00 kg | 0.0 |
| `Hat_BoxingBlue` | **Boxing Head Gear** | 0.00 kg | 0.0 |
| `Hat_Army` | **Military Helmet** | 0.00 kg | 0.0 |
| `Hat_SWAT` | **SWAT Helmet** | 0.00 kg | 0.0 |
| `Hat_BaseballHelmet` | **Batting Helmet** | 0.00 kg | 0.0 |
| `Hat_BaseballHelmet_KY` | **Batting Helmet - Kentucky** | 0.00 kg | 0.0 |
| `Hat_BaseballHelmet_Rangers` | **Batting Helmet - Riverside Rangers** | 0.00 kg | 0.0 |
| `Hat_BaseballHelmet_Z` | **Batting Helmet - Z Hurricanes** | 0.00 kg | 0.0 |
| `Hat_BicycleHelmet` | **Bicycle Helmet** | 0.00 kg | 0.0 |
| `Hat_CrashHelmet` | **Crash Helmet** | 0.00 kg | 0.0 |
| `Hat_CrashHelmetFULL` | **Crash Helmet** | 0.00 kg | 0.0 |
| `Hat_CrashHelmetFULL_Black` | **Crash Helmet** | 0.00 kg | 0.0 |

### 7.5. Nhóm Vật Phẩm Đại Diện: `Medical`

| Mã Vật Phẩm (ID) | Tên Vật Phẩm | Trọng Lượng (Weight) | Giá Trị Kim Loại (Metal Value) |
| :--- | :--- | :---: | :---: |

---

**[HOÀN TẤT XUẤT BẢN]** — Tệp Markdown được sinh tự động từ `mechanics_engine.db` với bảng mã UTF-8 tuyệt đối, sẵn sàng để đọc, tra cứu hoặc đồng bộ vào Obsidian Vault.