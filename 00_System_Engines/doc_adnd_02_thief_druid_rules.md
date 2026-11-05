---
id: "doc:adnd_02_thief_druid_rules"
package_id: "ADND_02_THIEF_DRUID_RULES"
title: "Quy Tắc Lớp Druid & Rogue: Toàn Bộ Kỹ Năng Đạo Tặc, Đâm Lén & Nhạc Công Bard"
original_title: "Druid & Rogue Class Mechanics (Thief Skills, Backstab & Bard)"
source_reference: "Advanced Dungeons & Dragons 2nd Edition Player's Handbook (Revised)"
system_category: "00_System_Engines"
associated_master_moc: "[[00_Master_MOC]]"
---

# Quy Tắc Lớp Druid & Rogue: Toàn Bộ Kỹ Năng Đạo Tặc, Đâm Lén & Nhạc Công Bard

> **Mã gói (Package ID):** `ADND_02_THIEF_DRUID_RULES` | **Nguồn khảo cứu:** *Advanced Dungeons & Dragons 2nd Edition Player's Handbook (Revised)*
> **Phân loại hệ thống:** `[[00_System_Engines]]` | **Bản đồ não bộ:** [[00_Master_MOC]]

## 1. Tóm Lược Nghiệp Vụ & Thiết Kế (Executive Summary)
Toàn bộ cơ chế vận hành, tỷ lệ phần trăm (d100) và quy tắc hành động của Đạo tặc (Thief), đâm lén (Backstab), năng lực tự nhiên của Druid và tài năng của Bard.

## 2. Ánh Xạ Cơ Chế Sang Sinh Tồn Thực Tế (Project Zomboid B42 / Hardcore RPG Mapping)
**Ý nghĩa cốt lõi:** Lấy chi tiết cơ chế giải thích kỹ năng Đạo tặc (Pick Pockets, Find/Remove Traps, Open Locks, Move Silently, Hide in Shadows, Detect Noise) và đặc quyền Tuần lâm/Druid để tích hợp vào hệ thống lén lút và sinh tồn rừng sâu Zomboid B42.

### Bảng Đối Chiếu Cơ Chế Tương Đương:
- **adnd_mechanic:** Move Silently & Hide in Shadows
- **b42_equivalent:** Lightfoot & Sneak: Giảm bán kính phát ra tiếng bước chân và giảm tầm nhìn phát hiện của zombie.

- **adnd_mechanic:** Detect Noise
- **b42_equivalent:** Keen Hearing trait: Phát hiện âm thanh zombie đằng sau cửa hoặc góc khuất.

- **adnd_mechanic:** Open Locks & Find/Remove Traps
- **b42_equivalent:** Burglar Profession + Lockpicking mod / B42 trap mechanics.

- **adnd_mechanic:** Backstab (+4 Hit & x2-x5 Damage)
- **b42_equivalent:** Jaw Stab / Stealth Kill: Đâm lén sau lưng kết liễu 1 hit bằng dao săn hoặc vũ khí ngắn.

- **adnd_mechanic:** Druid Wild Shape & Woodland Passage
- **b42_equivalent:** Foraging & Outdoor Survival: Di chuyển qua bụi rậm không bị cào xước, nhận diện nấm độc và thảo dược tự nhiên.


## 3. Sections
- **druid_mechanics:**
  - **title:** Druid Class Rules & Wilderness Abilities
  - **pdf_pages:** 36-37
  - **book_pages:** 35-36
  - **allowed_equipment:**
    - **armor:** Nonmetallic armor only (Padded, Leather, or Hide) and wooden shields.
    - **weapons:** Club, sickle, dart, spear, dagger, scythe, sling, staff.
  - **special_powers:**
    -
      - **level:** Level 1
      - **power:** Secret Language
      - **description:** Druidic secret language known only to initiated druids.
    -
      - **level:** Level 3
      - **power:** Identify Plant and Animal
      - **description:** Identify plant and animal species, and detect whether water is pure and potable.
    -
      - **level:** Level 3
      - **power:** Woodland Stride
      - **description:** Pass through overgrown thickets, thorns, and briars at normal movement speed without suffering damage or leaving a trail.
    -
      - **level:** Level 7
      - **power:** Immunity to Woodland Charm
      - **description:** Immunity to charm and mind-affecting abilities of fey and woodland creatures (dryads, nymphs, pixies).
    -
      - **level:** Level 7
      - **power:** Shapechange (Wild Shape)
      - **description:** Can change form into a reptile, bird, or mammal up to 3 times per day (once per day for each animal kingdom category). Regains 10-60% of lost Hit Points upon transforming.
  - **druid_hierarchy:** Druids advance up to 11th level (Druid). Beyond 11th level, only a fixed number exist in the world (Archdruid, Great Druid, Grand Druid) requiring ritual challenges to advance.
- **thief_mechanics:**
  - **title:** Thief Class Rules & Detailed Skills
  - **pdf_pages:** 37-41
  - **book_pages:** 36-40
  - **allowed_equipment:**
    - **armor:** Leather, padded, studded leather, or elven chain. Wearing studded leather or elven chain imposes penalties on thieving percentages.
    - **weapons:** Club, dagger, dart, hand crossbow, knife, lasso, shortbow, sling, broadsword, longsword, shortsword, staff.
  - **backstab_detailed_rules:**
    - **requirements:** The thief must strike from behind and remain completely undetected by the target prior to the strike.
    - **to_hit_bonus:** +4 bonus on the attack roll.
    - **target_armor:** Target's shield bonus is ignored, and target cannot apply Dexterity AC bonuses against the backstab.
    - **damage_multiplier:** Roll weapon damage and add Strength bonus normally, then multiply the entire total by the backstab multiplier (x2 at lv 1-4, x3 at lv 5-8, x4 at lv 9-12, x5 at lv 13+).
    - **restrictions:** Applicable only against creatures with discernible anatomy and backs (cannot backstab an ochre jelly or fire elemental).
  - **thief_skills_descriptions:**
    -
      - **skill:** Pick Pockets
      - **description:** Stealing small items from belts, pockets, or pouches without being noticed. Failure by 20% or more means the target instantly catches the thief's hand.
    -
      - **skill:** Open Locks
      - **description:** Picking standard padlocks, door locks, or iron chest clasps using thieves' tools. Can only be attempted once per lock per level of experience.
    -
      - **skill:** Find/Remove Traps
      - **description:** Examining a door, chest, or floor mechanism for trip wires, needles, poison sprays, or pit releases, and carefully disabling them. Failure does not necessarily trigger the trap unless rolled 90-00%.
    -
      - **skill:** Move Silently
      - **description:** Walking across creaky floorboards, gravel, or dry leaves without making audible footfalls. If successful, observers cannot hear the thief unless using magical acoustics.
    -
      - **skill:** Hide in Shadows
      - **description:** Melting into shadowy alcoves, dark corners, or drapery. The thief must remain stationary; moving while hidden requires a separate Move Silently check.
    -
      - **skill:** Detect Noise
      - **description:** Pressing an ear against a wooden door, stone wall, or grate to overhear whispering, shifting claws, or chanting on the other side.
    -
      - **skill:** Climb Walls
      - **description:** Scaling vertical rough stone walls, brick masonry, or sheer cliffs without ropes or climbing gear. Base success rate is high (60%+).
    -
      - **skill:** Read Languages
      - **unlock_level:** Level 4
      - **description:** 80% chance to read obscure ancient tongues, treasure maps, secret letters, and thieves' cant.
    -
      - **skill:** Use Scrolls
      - **unlock_level:** Level 10
      - **description:** Can read and cast from Wizard or Priest spell scrolls with a 25% chance of magical misfire/backfire.
  - **guild_headquarters:** At 10th level, a thief can establish a hideout or thieves' guild branch, attracting 4d6 novice cutpurses, pickpockets, and burglars as loyal followers.
- **bard_mechanics:**
  - **title:** Bard Class Rules & Abilities
  - **pdf_pages:** 41-43
  - **book_pages:** 40-42
  - **allowed_equipment:** Any weapon, light armor (leather/padded/studded leather), but cannot perform thieving skills or spellcasting while wearing metal armor or holding a shield.
  - **abilities:**
    -
      - **name:** Inspire Allies
      - **description:** Performing for 3 full rounds before battle grants allies within 30 ft a +1 bonus to attack rolls, +1 bonus to saving throws, or +2 bonus to morale checks.
    -
      - **name:** Counter Hostile Sound
      - **description:** Singing or playing loud music counters song-based or sound-based magical attacks (such as Harpy songs) for allies within 30 feet.
    -
      - **name:** Legend Lore
      - **description:** 5% chance per level to identify the history, background, purpose, and curse of any magical or legendary artifact encountered.
    -
      - **name:** Influence Crowds
      - **description:** Fascinating an audience of NPCs (forcing saving throws vs paralyzation to adjust crowd attitudes by one tier).
    -
      - **name:** Wizard Spellcasting
      - **description:** Beginning at 2nd level, bards learn and cast Wizard spells from spellbooks (up to 6th-level spells at high level).
