---
id: "doc:srd_03_toolbox_curses_traps"
package_id: "SRD_03_TOOLBOX_CURSES_TRAPS"
title: "Gameplay Toolbox: Travel Pace, Diseases, Hazards, Poisons, Traps & Magic Item Rules"
original_title: "Gameplay Toolbox: Travel Pace, Diseases, Hazards, Poisons, Traps & Magic Item Rules"
source_reference: "D&D 5E / AD&D / RPG Advanced Design"
system_category: "00_System_Engines"
associated_master_moc: "[[00_Master_MOC]]"
---

# Gameplay Toolbox: Travel Pace, Diseases, Hazards, Poisons, Traps & Magic Item Rules

> **Mã gói (Package ID):** `SRD_03_TOOLBOX_CURSES_TRAPS` | **Nguồn khảo cứu:** *D&D 5E / AD&D / RPG Advanced Design*
> **Phân loại hệ thống:** `[[00_System_Engines]]` | **Bản đồ não bộ:** [[00_Master_MOC]]

## 1. Tóm Lược Nghiệp Vụ & Thiết Kế (Executive Summary)
Dữ liệu luật chơi và cơ chế đã qua chắt lọc.

## 2. Ánh Xạ Cơ Chế Sang Sinh Tồn Thực Tế (Project Zomboid B42 / Hardcore RPG Mapping)
## 3. Travel Pace
- **paces:**
  -
    - **pace:** Fast
    - **miles_per_hour:** 4
    - **miles_per_day:** 30
    - **effect:** -5 penalty to passive Perception checks; cannot use Stealth.
  -
    - **pace:** Normal
    - **miles_per_hour:** 3
    - **miles_per_day:** 24
    - **effect:** Standard travel; normal passive Perception.
  -
    - **pace:** Slow
    - **miles_per_hour:** 2
    - **miles_per_day:** 18
    - **effect:** Able to use Stealth to move undetected; optimal for foraging.
- **environmental_modifiers:**
  - **good_roads:** Increases travel pace by 1/3 (e.g., Fast becomes 40 miles/day).
  - **difficult_terrain:** Halves travel speed (requires 2 miles of movement per 1 mile traveled).

## 4. Curses And Magical Contagions
-
  - **name:** Cackle Fever
  - **incubation:** 1d4 days
  - **symptoms:** High fever, disorientation, uncontrollable bouts of hideous laughter during stress.
  - **mechanics:** DC 13 Constitution save. On failure, takes 1d10 Psychic damage and gains Incapacitated condition while laughing for 1 minute. Each day, save or gain 1 level of Exhaustion.
-
  - **name:** Sewer Plague
  - **incubation:** 1d4 days
  - **symptoms:** Transmitted by bites of filth-dwelling vermin (Giant Rats, Otyughs). Severe fatigue, chills, body cramps.
  - **mechanics:** DC 11 Constitution save. On failure, character regains only half the normal number of hit points from spending Hit Dice, and gains no HP from Long Rests. Hit point max decreases by 1d10 each failed daily save. Dies if max HP reaches 0.
-
  - **name:** Sight Rot
  - **incubation:** 1 day
  - **symptoms:** Contracted from drinking stagnant contaminated water. Eyes turn milky and bleed.
  - **mechanics:** DC 15 Constitution save. On failure, character's vision dims: -2 to attack rolls and Perception checks. Each subsequent day failed save adds another -2 penalty. Reaching -10 causes permanent Blindness.

## 5. Environmental Hazards
- **extreme_cold:** Whenever temperature is below 0 degrees F, a creature exposed without cold-weather gear must make a DC 10 Constitution save at the end of each hour or gain 1 level of Exhaustion.
- **extreme_heat:** Whenever temperature is at or above 100 degrees F, a creature exposed without adequate water must make a Constitution save (DC 5, +1 for each previous hour) or gain 1 level of Exhaustion. Disadvantage if wearing Medium or Heavy armor.
- **heavy_precipitation:** Heavy rain or snowfall causes everything in the area to be lightly obscured. Imposes Disadvantage on Wisdom (Perception) checks relying on sight and hearing. Extinguishes open flames.
- **strong_wind:** Imposes Disadvantage on ranged weapon attack rolls and Perception checks relying on hearing. Halves flying movement speed.
- **high_altitude:** Above 10,000 feet, creatures acclimatized require twice the normal amount of food and water; each hour of travel counts as 2 hours.

## 6. Fear And Mental Stress
- **mechanic:** Severe traumatic shock or otherworldly horrors force a DC 10-15 Wisdom or Charisma saving throw.
- **madness_types:**
  -
    - **type:** Short-Term Madness (1d10 minutes)
    - **effects:**
      - Retreats in terror
      - Paralyzed with fear
      - Screams incoherently
      - Attacks nearest creature
  -
    - **type:** Long-Term Madness (1d10 x 10 hours)
    - **effects:**
      - Severe paranoia
      - Compulsive tremors
      - Amnesia
      - Extreme sensory hallucinations
  -
    - **type:** Indefinite Madness (Permanent until cured)
    - **effects:**
      - Deep personality fracture
      - Phobia of dark/blood
      - Overwhelming obsession

## 7. Poisons Toxicology Registry
-
  - **name:** Assassin's Blood
  - **type:** Ingested
  - **cost_gp:** 150
  - **save_dc:** 10
  - **effect:** Takes 1d12 Poison damage and gains Poisoned condition for 24 hours on failed Con save (half damage, no poisoned on success).
-
  - **name:** Burnt Othur Fumes
  - **type:** Inhaled
  - **cost_gp:** 500
  - **save_dc:** 13
  - **effect:** Takes 3d6 Poison damage on fail. Must repeat save at start of each turn, taking 1d6 Poison damage each round until 3 successful saves are made.
-
  - **name:** Crawler Mucus
  - **type:** Contact
  - **cost_gp:** 200
  - **save_dc:** 13
  - **effect:** Gains the Poisoned condition for 1 minute; while poisoned, is Paralyzed. Repeat save at end of each turn to end.
-
  - **name:** Drow Poison
  - **type:** Injury
  - **cost_gp:** 200
  - **save_dc:** 13
  - **effect:** Gains Poisoned condition for 1 hour. If failed by 5 or more, the creature falls Unconscious for the duration.
-
  - **name:** Essence of Ether
  - **type:** Inhaled
  - **cost_gp:** 300
  - **save_dc:** 15
  - **effect:** Gains Poisoned condition for 8 hours; while poisoned, the creature is Unconscious.
-
  - **name:** Malice
  - **type:** Inhaled
  - **cost_gp:** 250
  - **save_dc:** 15
  - **effect:** Gains Poisoned condition for 1 hour; while poisoned, the creature is Blinded.
-
  - **name:** Midnight Tears
  - **type:** Ingested
  - **cost_gp:** 1500
  - **save_dc:** 17
  - **effect:** Latent venom: has no effect until midnight. At midnight, target takes 9d6 Poison damage on failed save (half on success).
-
  - **name:** Oil of Taggit
  - **type:** Contact
  - **cost_gp:** 400
  - **save_dc:** 13
  - **effect:** Gains Poisoned condition for 24 hours; while poisoned, the creature is Unconscious.
-
  - **name:** Pale Tincture
  - **type:** Ingested
  - **cost_gp:** 250
  - **save_dc:** 16
  - **effect:** Takes 1d6 Poison damage and gains Poisoned condition. Each 24 hours, must repeat save or take another 1d6 Poison damage until cured by 7 successful saves.
-
  - **name:** Purple Worm Poison
  - **type:** Injury
  - **cost_gp:** 2000
  - **save_dc:** 19
  - **effect:** Takes 12d6 Poison damage on failed Con save, or half on success.
-
  - **name:** Serpent Venom
  - **type:** Injury
  - **cost_gp:** 200
  - **save_dc:** 11
  - **effect:** Takes 3d6 Poison damage on failed save, or half on success.
-
  - **name:** Torpor
  - **type:** Ingested
  - **cost_gp:** 600
  - **save_dc:** 15
  - **effect:** Gains Poisoned condition for 4d6 hours; while poisoned, is Incapacitated.
-
  - **name:** Truth Serum
  - **type:** Ingested
  - **cost_gp:** 150
  - **save_dc:** 11
  - **effect:** Gains Poisoned condition for 1 hour; while poisoned, creature cannot deliberately speak a lie.
-
  - **name:** Wyvern Poison
  - **type:** Injury
  - **cost_gp:** 1200
  - **save_dc:** 15
  - **effect:** Takes 7d6 Poison damage on failed Con save, or half on success.

## 8. Traps System
- **trap_severity_matrix:**
  -
    - **danger_tier:** Setback (Level 1-4)
    - **save_dc:** 10-11
    - **attack_bonus:** +3 to +5
    - **damage:** 1d10
  -
    - **danger_tier:** Dangerous (Level 5-10)
    - **save_dc:** 12-15
    - **attack_bonus:** +6 to +8
    - **damage:** 2d10 / 4d10
  -
    - **danger_tier:** Deadly (Level 11-16)
    - **save_dc:** 16-19
    - **attack_bonus:** +9 to +11
    - **damage:** 10d10
  -
    - **danger_tier:** Lethal (Level 17-20)
    - **save_dc:** 20+
    - **attack_bonus:** +12+
    - **damage:** 18d10 / 24d10
- **sample_traps:**
  -
    - **name:** Hidden Pit Trap
    - **trigger:** Stepping onto concealed trapdoor (DC 15 Perception to spot).
    - **effect:** Creature falls 10 to 30 feet, taking 1d6 Bludgeoning damage per 10 feet fallen and landing Prone. Spiked pits add +2d10 Piercing damage.
  -
    - **name:** Poison Needle Trap
    - **trigger:** Tampering with chest lock without key (DC 15 Investigation to find).
    - **effect:** +8 attack roll against picker; deals 1 Piercing damage plus coated poison (e.g. Drow Poison or Pale Tincture).
  -
    - **name:** Rolling Sphere Trap
    - **trigger:** Tripwire across 10-ft hallway.
    - **effect:** Solid 10-ft granite boulder rolls down slope at 60 ft/round. DC 15 Dexterity save or take 4d10 Bludgeoning damage and knocked Prone.
  -
    - **name:** Scythe Blade Trap
    - **trigger:** Pressure plate in corridor floor.
    - **effect:** +8 melee attack roll against all creatures in 10-ft line, dealing 2d8+4 Slashing damage.
  -
    - **name:** Glyph of Warding
    - **trigger:** Stepping within 5 ft of inscribed magical rune.
    - **effect:** Explodes in 20-ft radius sphere. DC 15 Dex save or take 5d8 Fire, Cold, Lightning, or Thunder damage.

## 9. Magic Item Rules
- **cursed_items:** Attuning to a cursed item magically binds it to the wielder. The wielder cannot voluntarily end attunement or discard the item until the curse is broken by a Remove Curse spell.
- **magic_item_resilience:** Most magic items are at least as durable as normal items of their kind. Magic items have Resistance to all damage types. Artifacts can only be destroyed by specific epic quests or legendary means.
- **sentient_items:** Possess Intelligence, Wisdom, Charisma, telepathy or speech, special purposes (Bane, Protector, Defiler), and can initiate a contest of wills if the wielder defies their purpose.
