---
id: "sys_pipeline:doc09"
type: "system_tick_pipeline_spec"
document_source: "Doc 09: Project Zomboid B42 Climate, Seasons & World Erosion Bible (12 Months, Severe Weather, 37.0°C Core Equilibrium & 100-Day Erosion Curve)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[sys_metabolism_calories]]",
  "[[dis_trung_han]]",
  "[[dis_trung_thu]]",
  "[[sys_math_doc09]]"
]
state_machines: [
  "4-Season Climate & Environmental Weather State Machine",
  "17-Part Biological Thermoregulation & Cold Affliction Tick Pipeline",
  "100-Day World Erosion Dynamic Degradation FSM"
]
tick_rate_hz: 60
---

## PHẦN C: SƠ ĐỒ MÁY TRẠNG THÁI & LUỒNG DỮ LIỆU (STATE_MACHINE_AND_TICK_PIPELINE)

### 1. Luồng Khí Hậu 4 Mùa, Cân Bằng Thân Nhiệt $37.0^\circ\text{C}$ & Bào Mòn Thế Giới (`Climate, Thermoregulation & Erosion Tick Pipeline`)

```mermaid
stateDiagram-v2
    [*] --> CLIMATE_MANAGER_TICK : Khởi chạy từ Ngày 09/07/1993 (Vĩ độ 38° Bắc)

    state "1. Vòng lặp Khí quyển 12 Tháng & Thời tiết Cực đoan (Climate & WeatherPeriod)" as ClimatePhase {
        CLIMATE_MANAGER_TICK --> SEASONAL_TRANSITION : Hè (22..38°C) -> Thu (4..28°C, Sương muối T11 -2°C diệt cây)\n-> Đông (-15..17°C, Đêm T12 dài 14h) -> Xuân (1..28°C, Bùn lầy T3)
        SEASONAL_TRANSITION --> SEVERE_WEATHER_EVENT : Kích hoạt Khí tượng Cực đoan:
        SEVERE_WEATHER_EVENT --> TROPICAL_STORM : Bão nhiệt đới -> Tầm nhìn 3-5 ô | Tiếng mưa át bước chân\nSấm sét (Thunder) nổ 100-200 ô khuấy động bầy đàn
        SEVERE_WEATHER_EVENT --> ARCTIC_BLIZZARD : Bão tuyết (T12-T2) -> -15°C đến -25°C + Max Wind Chill\nTuyết dày phạt -50% tốc độ chạy & làm trượt bánh xe
        SEVERE_WEATHER_EVENT --> HEAVY_FOG : Sương mù (T9-T11) -> Giảm -70% tầm nhìn (Zombie cách 2-3 bước mới thấy)
    }

    state "2. Máy Trạng Thái Thân Nhiệt 17 Bộ phận & Cảm Lạnh (Thermoregulator Tick)" as ThermoPhase {
        CLIMATE_MANAGER_TICK --> EVALUATE_17_BODYPARTS : Tính Nhiệt độ Da & Lõi (Chuẩn 37.0°C / An toàn 36.5-37.5°C)\nQua 3 lớp chỉ số quần áo: Insulation, WindResistance, WaterResistance (Poncho = 100%)
        
        EVALUATE_17_BODYPARTS --> COLD_CASCADE : T_core < 36.5°C
        COLD_CASCADE --> SHIVERING_35_365 : 35.0 - 36.5°C: Rét run -> Đốt Calo & Năng lượng x3.0 - x5.0
        COLD_CASCADE --> FROSTBITE_30_35 : 30.0 - 35.0°C: Tê cóng -> Tốc độ chạy -50%, Vung vũ khí -50%, Thao tác -40%
        COLD_CASCADE --> LETHAL_HYPOTHERMIA : < 30.0°C: Rút máu liên tục đến chết (T1 ngoài trời chết sau 1 giờ)

        EVALUATE_17_BODYPARTS --> HEAT_CASCADE : T_core > 37.5°C
        HEAT_CASCADE --> SWEATING_375_39 : 37.5 - 39.0°C: Đổ mồ hôi -> Khát nước x4.0 & Ướt sũng áo lót trong\n(Ngừng vận động gặp gió -> Đảo chiều hạ nhiệt -> Cảm lạnh giữa mùa hè!)
        HEAT_CASCADE --> HYPERTHERMIA_39PLUS : > 39.0°C: Sốc nhiệt -> Choáng váng, Kiệt sức, Tụt máu

        EVALUATE_17_BODYPARTS --> CATCH_COLD_CHECK : Ướt (Wetness) + Lạnh -> CatchAColdDelta >= 100%\n-> Ho & Hắt hơi phát sóng âm 15-20 ô (Dùng Handkerchief/Tissue ở ô phụ để bịt miệng)
    }

    state "3. Tiến trình Bào mòn Tự nhiên 100 Ngày (ErosionMain Engine)" as ErosionPhase {
        CLIMATE_MANAGER_TICK --> EROSION_PROGRESSION : Tiến độ theo ErosionSpeed (20 / 50 / 100 / 200 / 500 ngày)\n- Ngày 0-20: Cỏ non & Rêu\n- Ngày 20-50: Cỏ cao ngang hông che khuất Crawlers (Cắt bằng Machete/Scythe)\n- Ngày 50-80: Dây leo phủ kín tường (Leo được lên tầng 2) & Nứt đường\n- Ngày 80-100+: Cây mọc giữa lòng đường (Xe đâm tốc độ cao -> Vỡ lốc máy, Nát két nước, Nổ lốp)
    }

```

---

---

### Mối Quan Hệ Đồ Thị Vault (Knowledge Graph Links)
- Trung tâm điều hướng: [[00_Master_MOC]]
- Động lực học & Chuyển hóa calo: [[00_System_Engines]], [[sys_metabolism_calories]]
- Bệnh dịch thân nhiệt Đại Việt: [[dis_trung_han]], [[dis_trung_thu]]
- Phương trình toán học đi kèm: [[sys_math_doc09]]
