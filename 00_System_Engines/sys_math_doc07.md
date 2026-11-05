---
id: "sys_math:doc07"
type: "system_mathematics_spec"
document_source: "Doc 07: Project Zomboid B42 Media Broadcasts, Knox Lore & Skill VHS Bible (AEBS Oracle, 5-Tier Skill Books, 64 Instructional VHS & Idempotent XP)"
associated_master_moc: "[[00_Master_MOC]]"
related_systems: [
  "[[00_Master_MOC]]",
  "[[00_System_Engines]]",
  "[[01_World_And_Factions]]",
  "[[doc_prp_01_goals_milestones]]",
  "[[sys_pipeline_doc07]]"
]
key_equations: [
  "Skill Book 5-Tier XP Multiplication Equation (x3, x5, x8, x12, x16)",
  "Idempotent Media XP Stacking and Multi-Source Exclusion Rule",
  "AEBS Random Frequency Selection (88.0-108.0 MHz, 0.2 MHz Step)",
  "3-Stage Power Grid Collapse and Aerial Activity Warning Schedule"
]
---

## PHẦN A: BẢNG HẰNG SỐ & PHƯƠNG TRÌNH TOÁN HỌC CỐT LÕI (MATHEMATICAL_FORMULAS_AND_CONSTANTS)

### 1. Phương Trình Toán Học & Mã Giả Thuật Toán (LaTeX & Deterministic Pseudocode)

#### 1.1. Phương trình Cộng hưởng Kinh nghiệm Bản ghi Tri thức & Hệ số Sách Kỹ năng 5 Bậc (Instructional Record & Skill Book XP Multiplier)



Khi nhân vật lĩnh hội một dòng tri thức (Line Entry) có gắn cờ mã kỹ năng (như `COO+1`, `CRP+1`, `FIS+1`, `HUS+1`, `FKN+1`, `BLA+1`, `POT+1`) từ Tín hiệu Phát sóng Trực tiếp hoặc Bản ghi Lưu trữ (trừu tượng hóa từ TV / Retail-VHS / Home-VHS), lượng điểm kinh nghiệm thực nhận $\Delta \text{XP}_{\text{media}}$ cho kỹ năng $k$ ở cấp độ hiện tại $L(k) \in [0, 10]$ phụ thuộc vào bậc hệ số Sách Kỹ năng $M_{\text{book}}(L(k))$ đã đọc, hệ số cấp khởi đầu $M_{\text{init}}(L_0(k))$ (từ Doc 04) và đặc điểm `Fast Learner` ($+30\%$):

$$M_{\text{book}}(L(k)) = \begin{cases} 3.0 & \text{nếu } L(k) \in \{1, 2\} \text{ và đã đọc Sách Tập 1 (Vol 1)} \\ 5.0 & \text{nếu } L(k) \in \{3, 4\} \text{ và đã đọc Sách Tập 2 (Vol 2)} \\ 8.0 & \text{nếu } L(k) \in \{5, 6\} \text{ và đã đọc Sách Tập 3 (Vol 3)} \\ 12.0 & \text{nếu } L(k) \in \{7, 8\} \text{ và đã đọc Sách Tập 4 (Vol 4)} \\ 16.0 & \text{nếu } L(k) \in \{9, 10\} \text{ và đã đọc Sách Tập 5 (Vol 5)} \\ 1.0 & \text{nếu chưa đọc Sách Kỹ năng tương ứng với cấp hiện tại} \end{cases}$$

$$\Delta \text{XP}_{\text{media}}(k) = \Delta \text{XP}_{\text{line\_base}}(k) \times M_{\text{book}}(L(k)) \times \left(1 + 0.30 \cdot \mathbb{I}_{\text{FastLearner}}\right) \times M_{\text{init}}(L_0(k))$$

(Ví dụ kiểm chứng từ tài liệu: Khi kết hợp Sách Tập 1 ($\times 3$) với `Fast Learner` ($+30\%$), hệ số nhân gộp đạt $3.0 \times 1.30 = 3.9\times$, biến một chương trình Mộc cơ bản $75\text{ XP}$ thành $225\text{--}300\text{ XP}$).

#### 1.2. Phương trình Quy tắc Chồng XP Độc lập & Loại trừ Trùng lặp (`Live Broadcast` vs `Standard Archive` vs `Unique Field Record`)



Gọi $E_{\text{id}}$ là định danh của một phân đoạn nội dung tri thức. Quy tắc kiểm tra chống nhận trùng điểm kinh nghiệm (Idempotency & Stacking Rule) giữa 3 nguồn phát $\text{Source} \in \{\text{LIVE\_BROADCAST}, \text{RETAIL\_ARCHIVE}, \text{HOME\_FIELD\_RECORD}\}$ được xác định bởi hàm chỉ thị $\mathbb{I}_{\text{grant}}(p, E_{\text{id}}, \text{Source})$ cho người chơi $p$:

$$\mathbb{I}_{\text{grant}}(p, E_{\text{id}}, \text{Source}) = \begin{cases} 1 & \text{nếu } \text{Source} = \text{LIVE\_BROADCAST} \land E_{\text{id}} \notin \mathcal{H}_{\text{watched}}(p) \\ 1 & \text{nếu } \text{Source} = \text{RETAIL\_ARCHIVE} \land E_{\text{id}} \notin \mathcal{H}_{\text{watched}}(p) \\ 0 & \text{nếu } \text{Source} = \text{RETAIL\_ARCHIVE} \land E_{\text{id}} \in \mathcal{H}_{\text{watched}}(p) \ (\text{đã xem trên Live Broadcast}) \\ 1 & \text{nếu } \text{Source} = \text{HOME\_FIELD\_RECORD} \land E_{\text{id}} \notin \mathcal{H}_{\text{home\_watched}}(p) \ (\text{Độc lập 100\% với Live Broadcast}) \end{cases}$$

```text
FUNCTION Process_Instructional_Media_Line(player, media_item, line_entry):
    // 1. Kiểm tra quy tắc chồng XP (Stacking Rule: Live vs Retail vs Home)
    IF media_item.Category IN ["LIVE_BROADCAST", "RETAIL_ARCHIVE"] THEN
        // Retail VHS trùng nội dung với TV: nếu đã xem tập đó trên TV thì KHÔNG nhận thêm XP
        IF player.HasConsumedBroadcastLine(line_entry.SharedEpisodeLineID) == TRUE THEN
            RETURN XP_GRANT_BLOCKED_ALREADY_WATCHED
        END IF
        player.MarkBroadcastLineConsumed(line_entry.SharedEpisodeLineID)
    ELSE IF media_item.Category == "HOME_FIELD_RECORD" THEN
        // Home VHS luôn trao XP độc lập với TV, chỉ kiểm tra xem chính cuộn băng đó đã xem chưa
        IF player.HasConsumedHomeRecordLine(line_entry.UniqueHomeLineID) == TRUE THEN
            RETURN XP_GRANT_BLOCKED_ALREADY_WATCHED
        END IF
        player.MarkHomeRecordLineConsumed(line_entry.UniqueHomeLineID)
    END IF

    // 2. Mở khóa Công thức Chế tác Đặc biệt (nếu dòng thoại gắn Recipe Unlock)
    FOR EACH recipe_id IN line_entry.UnlockedRecipes DO
        player.UnlockRecipe(recipe_id)
    END FOR

    // 3. Tính toán XP với hệ số Sách Kỹ năng 5 bậc (x3, x5, x8, x12, x16)
    skill_id = line_entry.TargetSkill
    current_level = player.GetSkillLevel(skill_id)
    book_mult = player.GetActiveSkillBookMultiplier(skill_id, current_level) // 1.0, 3.0, 5.0, 8.0, 12.0, 16.0
    
    fast_learner_mult = 1.0
    IF player.HasTrait("base:fastlearner") THEN
        fast_learner_mult = 1.30
    END IF

    total_xp = line_entry.BaseXP * book_mult * fast_learner_mult
    player.AddSkillXP(skill_id, total_xp)
    RETURN XP_GRANT_SUCCESS
END FUNCTION

```

#### 1.3. Thuật toán Khởi tạo Tần số Điềm báo Khẩn cấp & Dự báo Siêu sự kiện (`ISDynamicRadio` / AEBS Oracle Engine)



Khi khởi tạo một thế giới mới, tần số của Hệ thống Tín hiệu Dự báo Khẩn cấp (`AEBS` $\rightarrow$ trừu tượng hóa thành `World_Event_Telemetry_Oracle`) được chọn ngẫu nhiên rời rạc trong dải $[88000, 108000]\text{ Hz}$ (tương đương $88.0\text{ MHz}$ đến $108.0\text{ MHz}$) với bước nhảy $\Delta f = 200\text{ Hz}$ ($0.2\text{ MHz}$), tổng cộng có $101$ tần số khả dĩ:

$$f_{\text{AEBS}} = 88000 + 200 \times k, \quad k \sim \mathcal{U}_{\text{int}}(0, 100) \quad (\text{Đơn vị hiển thị: } \frac{f_{\text{AEBS}}}{1000} \in [88.0, 108.0]\text{ MHz})$$

```text
FUNCTION Initialize_World_Telemetry_Oracle_Frequency():
    // Tương đương hàm radio:getRandomFrequency(88000, 108000) với bước nhảy 0.2 MHz (200 đơn vị)
    step_index = RandomInt(0, 100)
    aebs_frequency_khz = 88000 + (step_index * 200)
    RETURN aebs_frequency_khz
END FUNCTION

FUNCTION Broadcast_Hourly_Telemetry_Oracle(world_time_hour, world_day):
    // Phát định kỳ vào đầu mỗi giờ (XX:00)
    IF world_time_hour.Minutes != 00 THEN RETURN END IF

    EmitSignalHeader("<bzzt> Fiver Zero Two. <wzzt> Automated Broadcast System. <bzzt>")
    EmitWeatherForecast(window_hours_min = 24, window_hours_max = 48) // Gió, sương mù, bão trước 24-48h

    // Cảnh báo Siêu sự kiện Trên không (Helicopter / Aerial Acoustic Lure)
    IF IsAerialLureScheduledToday(world_day) AND world_time_hour.Hour >= 9 THEN
        EmitAlert("Air Activity detected.") // Sự kiện sẽ kích hoạt trong khung 12:00 - 19:00 cùng ngày
    END IF

    // Cảnh báo 3 giai đoạn sụp đổ lưới năng lượng (Power Grid Blackout)
    IF HoursUntilPowerShutoff() >= 48 AND HoursUntilPowerShutoff() <= 72 THEN
        EmitAlert("Knox Power Grid: Power fluctuations detected.")
    ELSE IF HoursUntilPowerShutoff() > 0 AND HoursUntilPowerShutoff() < 48 THEN
        EmitAlert("Knox Power Grid: Systems failing. Network compromised.")
    ELSE IF HoursUntilPowerShutoff() <= 0 THEN
        EmitAlert("Knox Power Grid: Blackout.")
    END IF
END FUNCTION

```

---

### 2. Bảng Hằng Số Truyền Thông, Hệ Số Sách Kỹ Năng & Lịch Trình Sự Kiện (System Constants Table)

| Nhóm Hệ Thống | Tên Hằng Số / Biến Số Gốc | Giá Trị Chính Xác | Đơn Vị Đo | Ý Nghĩa Vật Lý & Cơ Học Hệ Thống |
| --- | --- | --- | --- | --- |
| **Media Database** | `RadioData_Broadcast_Entries` | `> 1120` | `entries` | Tổng số phân đoạn phát sóng trong `RadioData.xml` / `RadioData.json`.

 |
| **Media Database** | `RadioData_Dialogue_Lines` | `13166` | `lines` | Tổng số câu thoại phát sóng trên toàn hệ thống.

 |
| **Media Database** | `Total_Broadcast_Channels` | `18` | `channels` | Tổng số kênh truyền hình, phát thanh thương mại, dân sự và quân sự.

 |
| **Media Database** | `Total_Recorded_Media_Items` | `352` | `items` | Tổng số bản ghi lưu trữ trong `recorded_media.lua` ($155 + 128 + 69$).

 |
| **Media Database** | `Retail_VHS_Count` | `155` | `tapes` | Số lượng bản ghi thương mại sản xuất hàng loạt (Retail-VHS).

 |
| **Media Database** | `Home_VHS_Count` | `128` | `tapes` | Số lượng bản ghi thực địa/gia đình độc bản (Home-VHS).

 |
| **Media Database** | `Audio_CD_Count` | `69` | `discs` | Số lượng đĩa âm thanh giải trí (CDs).

 |
| **Media Database** | `Skill_And_Recipe_VHS_Count` | `64` | `tapes` | Số lượng bản ghi VHS trao điểm XP kỹ năng và mở khóa công thức chế tạo.

 |
| **Skill Book Mult** | `SkillBook_Vol1_Multiplier` | `3.0` (`3.9` w/ Fast Learner) | `multiplier (Lv 1-2)` | Hệ số nhân XP cho Cấp 1–2 khi đọc xong Sách Tập 1.

 |
| **Skill Book Mult** | `SkillBook_Vol2_Multiplier` | `5.0` | `multiplier (Lv 3-4)` | Hệ số nhân XP cho Cấp 3–4 khi đọc xong Sách Tập 2.

 |
| **Skill Book Mult** | `SkillBook_Vol3_Multiplier` | `8.0` | `multiplier (Lv 5-6)` | Hệ số nhân XP cho Cấp 5–6 khi đọc xong Sách Tập 3.

 |
| **Skill Book Mult** | `SkillBook_Vol4_Multiplier` | `12.0` | `multiplier (Lv 7-8)` | Hệ số nhân XP cho Cấp 7–8 khi đọc xong Sách Tập 4.

 |
| **Skill Book Mult** | `SkillBook_Vol5_Multiplier` | `16.0` | `multiplier (Lv 9-10)` | Hệ số nhân XP cho Cấp 9–10 khi đọc xong Sách Tập 5.

 |
| **Broadcast Schedule** | `LifeAndLiving_Channel_ID` | `203` | `channel_id` | Kênh đào tạo kỹ năng sinh tồn chủ lực (phát từ Ngày 0 đến Ngày 8).

 |
| **Broadcast Schedule** | `LifeAndLiving_Daily_Slots` | `06:00 / 12:00 / 18:00` | `24h clock` | 3 khung giờ cố định mỗi ngày: `06:00` (Cooking), `12:00` (Carpentry), `18:00` (Exposure Survival).

 |
| **Broadcast Schedule** | `TV_Signal_Termination_Day` | `Day 8 (after 06:00)` | `world_day` | Thời điểm tín hiệu truyền hình trực tiếp tắt vĩnh viễn, chuyển sang nhiễu trắng (Static).

 |
| **Broadcast Schedule** | `Woodcraft_Single_Ep_XP` | `75` (`225 - 300` w/ Vol 1) | `XP` | Lượng XP gốc của 1 tập chương trình Mộc và lượng XP sau khi nhân Sách Tập 1.

 |
| **Broadcast Channels** | `Other_TV_Channel_IDs` | `200, 201, 204..210` | `channel_id` | Các kênh phụ trợ: `200` (Triple-N), `201` (WBLN), `204` (TURBO: Boredom `-1`), `205` (PawsTV), `206` (KPATV), `207` (Music), `208` (Sports), `209` (Brennan Movie), `210` (GBC).

 |
| **Broadcast Channels** | `Civilian_Radio_Frequency` | `91.2` | `MHz` | Tần số đài nghiệp dư dân sự phát tín hiệu cầu cứu sau khi sụp đổ.

 |
| **Special VHS Stats** | `TheDogGoblin_Horror_Effect` | `Stress: +0.2 (STS+0.2) | +Panic` | `stat_delta` | Biến đổi tâm lý khi xem bản ghi kinh dị *The Dog Goblin*.

 |
| **Special VHS Stats** | `BasicGunHandling_Skill_Grant` | `Aiming +3, Reloading +3` | `skill_levels` | Bản ghi thực địa quý nhất (`AIM+2, REL+2, AIM+1, REL+1`), nâng thẳng Bắn súng & Nạp đạn lên Cấp 3–4.

 |
| **AEBS Oracle** | `AEBS_Frequency_Range` | `88000 - 108000` (`88.0 - 108.0`) | `Hz / MHz` | Dải tần số ngẫu nhiên của Đài Khẩn cấp Tự động trong mỗi thế giới mới.

 |
| **AEBS Oracle** | `AEBS_Frequency_Step` | `0.2` (`200`) | `MHz (Hz)` | Bước nhảy chia lưới tần số của hàm `getRandomFrequency(88000, 108000)`.

 |
| **AEBS Oracle** | `AEBS_Broadcast_Interval` | `XX:00` (Mỗi giờ) | `hourly_tick` | Chu kỳ phát bản tin dự báo tự động vào đúng phút thứ 00 mỗi giờ.

 |
| **AEBS Oracle** | `AirActivity_Warning_Window` | `Alert >= 09:00 | Event: 12:00-19:00` | `24h clock` | Khung giờ phát cảnh báo `"Air Activity detected."` và khung giờ sự kiện bay qua thực tế.

 |
| **AEBS Oracle** | `Power_Fluctuation_Lead_Time` | `48 - 72` | `hours (giờ)` | Thời gian báo trước của cảnh báo giai đoạn 1 trước khi mất điện toàn cục.

 |
| **AEBS Oracle** | `Weather_Forecast_Lead_Time` | `24 - 48` | `hours (giờ)` | Thời gian dự báo trước về gió, sương mù và bão tuyết/bão nhiệt đới.

 |
| **World Collapse** | `World_Collapse_Timeline_Days` | `14` (`Day 0 = July 9, 1993`) | `days` | Chu kỳ 14 ngày sụp đổ hoàn toàn của hệ thống hạ tầng và phát sóng thế giới (Ngày 7 bùng phát chủng lây qua không khí).

 |
| **Stealth Tactic** | `Safe_TV_Volume_Level` | `1 - 2` | `volume_bars` | Mức âm lượng an toàn khi nghe/xem thiết bị phát để không thu hút thực thể thù địch bên ngoài.

 |

---
