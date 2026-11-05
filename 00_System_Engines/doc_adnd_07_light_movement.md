---
id: "doc:adnd_07_light_movement"
package_id: "ADND_07_LIGHT_MOVEMENT"
title: "Chapter 13 & 14: Vision, Light Sources, Infravision, Movement & Climbing"
original_title: "Chapter 13 & 14: Vision, Light Sources, Infravision, Movement & Climbing"
source_reference: "D&D 5E / AD&D / RPG Advanced Design"
system_category: "00_System_Engines"
associated_master_moc: "[[00_Master_MOC]]"
---

# Chapter 13 & 14: Vision, Light Sources, Infravision, Movement & Climbing

> **Mã gói (Package ID):** `ADND_07_LIGHT_MOVEMENT` | **Nguồn khảo cứu:** *D&D 5E / AD&D / RPG Advanced Design*
> **Phân loại hệ thống:** `[[00_System_Engines]]` | **Bản đồ não bộ:** [[00_Master_MOC]]

## 1. Tóm Lược Nghiệp Vụ & Thiết Kế (Executive Summary)
Dữ liệu luật chơi và cơ chế đã qua chắt lọc.

## 2. Ánh Xạ Cơ Chế Sang Sinh Tồn Thực Tế (Project Zomboid B42 / Hardcore RPG Mapping)
## 3. Visibility And Vision
- **table_62_visibility_ranges:**
  -
    - **condition:** Clear daylight / Bright sun
    - **maximum_range:** 3 miles (level terrain)
    - **detail_recognition:** 500 yards
    - **combat_targeting:** Standard weapon ranges
  -
    - **condition:** Twilight / Dusk / Dawn
    - **maximum_range:** 1 mile
    - **detail_recognition:** 100 yards
    - **combat_targeting:** Standard ranges
  -
    - **condition:** Full moon (clear night)
    - **maximum_range:** 300 yards
    - **detail_recognition:** 50 yards
    - **combat_targeting:** Medium range maximum
  -
    - **condition:** Starlight / Crescent moon
    - **maximum_range:** 100 yards
    - **detail_recognition:** 20 yards
    - **combat_targeting:** Short range only (-2 to hit beyond 30 ft)
  -
    - **condition:** Dense fog / Blinding snowstorm
    - **maximum_range:** 10 to 30 feet
    - **detail_recognition:** 5 feet
    - **combat_targeting:** Melee only; missiles impossible beyond 15 ft
  -
    - **condition:** Heavy mist / Moderate rain
    - **maximum_range:** 100 feet
    - **detail_recognition:** 30 feet
    - **combat_targeting:** Missile attacks suffer -2 penalty
  -
    - **condition:** Dense smoke
    - **maximum_range:** 10 to 20 feet
    - **detail_recognition:** 5 feet
    - **combat_targeting:** Severe penalty (-4 to hit); choking risk
  -
    - **condition:** Total subterranean pitch blackness
    - **maximum_range:** 0 feet
    - **detail_recognition:** Touch only
    - **combat_targeting:** -4 to hit, lose Dex and shield bonus
- **infravision_mechanics:**
  - **range:** 60 feet for Dwarves, Elves, Gnomes, and Half-Elves (90 feet for certain underground monsters).
  - **spectrum:** Detects infrared heat radiation; warm living creatures radiate red and yellow, cold dungeon walls and undead appear blue and violet.
  - **limitations:** Cannot read text, distinguish fine physical features, or discern colors. Any exposure to open flame, torchlight, or magical light instantly blinds infravision for 1 round (eyes must readjust).

## 4. Light Sources
- **table_63_light_sources:**
  -
    - **source:** Bonfire / Campfire
    - **bright_light_radius_ft:** 40
    - **dim_light_radius_ft:** 80
    - **burn_duration:** Requires 1 bundle of dry wood per hour
    - **wind_extinguish_chance:** 10% in high winds
  -
    - **source:** Bullseye lantern
    - **bright_light_radius_ft:** 80
    - **dim_light_radius_ft:** 100
    - **burn_duration:** 6 hours per 1 pint flask of oil
    - **wind_extinguish_chance:** Protected; 0% in normal gale
    - **beam_angle:** Narrow 30-degree directional cone
  -
    - **source:** Candle
    - **bright_light_radius_ft:** 5
    - **dim_light_radius_ft:** 10
    - **burn_duration:** 1 hour per candle
    - **wind_extinguish_chance:** 50% in gentle breeze, 100% running
  -
    - **source:** Continuous Light spell
    - **bright_light_radius_ft:** 60
    - **dim_light_radius_ft:** 120
    - **burn_duration:** Permanent until dispelled
    - **wind_extinguish_chance:** Immune to wind and water
  -
    - **source:** Hooded lantern
    - **bright_light_radius_ft:** 30
    - **dim_light_radius_ft:** 50
    - **burn_duration:** 6 hours per 1 pint flask of oil
    - **wind_extinguish_chance:** Protected shutter; can be dimmed to 5 ft
  -
    - **source:** Torch
    - **bright_light_radius_ft:** 30
    - **dim_light_radius_ft:** 40
    - **burn_duration:** 30 minutes per torch
    - **wind_extinguish_chance:** 25% in heavy rain or rushing air

## 5. Movement And Travel
- **table_64_base_movement_rates:**
  -
    - **species_or_type:** Human, Elf, Half-Elf
    - **base_move:** 12
    - **combat_yards_per_round:** 12
    - **dungeon_feet_per_turn:** 120
    - **overland_miles_per_day:** 24
  -
    - **species_or_type:** Dwarf, Gnome, Halfling
    - **base_move:** 9
    - **combat_yards_per_round:** 9
    - **dungeon_feet_per_turn:** 90
    - **overland_miles_per_day:** 18
  -
    - **species_or_type:** Draft Horse / Heavy Warhorse
    - **base_move:** 12
    - **combat_yards_per_round:** 12
    - **dungeon_feet_per_turn:** 120
    - **overland_miles_per_day:** 24
  -
    - **species_or_type:** Medium Warhorse
    - **base_move:** 15
    - **combat_yards_per_round:** 15
    - **dungeon_feet_per_turn:** 150
    - **overland_miles_per_day:** 30
  -
    - **species_or_type:** Light Warhorse / Riding Horse
    - **base_move:** 24
    - **combat_yards_per_round:** 24
    - **dungeon_feet_per_turn:** 240
    - **overland_miles_per_day:** 48
- **movement_tactics:**
  - **dungeon_exploration_pace:** Normal dungeon pace (10 times base move in feet per 10-minute turn) assumes careful mapping, checking for traps, and stealth.
  - **combat_running:** A character can sprint at 5 times base combat speed in yards per round for a number of rounds equal to their Constitution score before collapsing from exhaustion.
  - **forced_march:** Traveling beyond 8 hours a day adds 1 mile per extra hour up to +8 miles. Every character must roll a daily Constitution check; failure inflicts 1d6 damage and reduces movement rate by half.

## 6. Climbing System
- **table_65_base_climbing_rates:**
  - **non_thief_characters:** Base chance to climb ordinary surfaces (rough stone, cliff face, trees) is 40% with appropriate handholds, 0% on sheer walls without specialized gear.
  - **thief_characters:** Uses the Climb Walls percentage skill (Table 27 / Chapter 3), modified by armor and environment.
- **table_66_environmental_climbing_modifiers:**
  -
    - **condition:** Climbing with knotted rope and wall surface to brace
    - **modifier:** +40%
  -
    - **condition:** Climbing with plain rope and wall surface to brace
    - **modifier:** +25%
  -
    - **condition:** Climbing rope without wall (hanging free in air)
    - **modifier:** -10%
  -
    - **condition:** Surface is slightly wet or damp
    - **modifier:** -10%
  -
    - **condition:** Surface is covered in ice or slime
    - **modifier:** -25%
  -
    - **condition:** Narrow vertical chimney with back brace
    - **modifier:** +20%
  -
    - **condition:** Overhang or negative incline ceiling
    - **modifier:** -35%
  -
    - **condition:** Using pitons, hammer, and climbing spikes
    - **modifier:** +15% and eliminates fall lethality on safety tether
- **table_67_armor_climbing_penalties:**
  -
    - **armor_worn:** No armor / Normal clothing
    - **penalty:** 0%
    - **speed_rate:** Full climbing rate (10 ft / round)
  -
    - **armor_worn:** Padded armor / Leather armor
    - **penalty:** -5%
    - **speed_rate:** Full climbing rate (10 ft / round)
  -
    - **armor_worn:** Studded leather / Ring mail
    - **penalty:** -15%
    - **speed_rate:** 3/4 climbing rate (7.5 ft / round)
  -
    - **armor_worn:** Scale mail / Chain mail / Splint mail
    - **penalty:** -30%
    - **speed_rate:** 1/2 climbing rate (5 ft / round)
  -
    - **armor_worn:** Banded mail / Plate mail
    - **penalty:** -40%
    - **speed_rate:** 1/4 climbing rate (2.5 ft / round)
  -
    - **armor_worn:** Field plate / Full plate
    - **penalty:** -60%
    - **speed_rate:** 1/4 climbing rate; fall triggers automatic drowning/crush injury
