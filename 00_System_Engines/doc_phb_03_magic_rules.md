---
id: "doc:phb_03_magic_rules"
package_id: "PHB_03_MAGIC_RULES"
title: "Quy Tắc Ma Thuật Cốt Lõi: Niệm Chú, Nguyên Liệu, Tập Trung (Concentration) & Vùng Ảnh Hưởng"
original_title: "Rules of Magic: Casting Spells, Components, Concentration & Areas of Effect"
source_reference: "Dungeons & Dragons 2024 Player's Handbook (5.5e)"
system_category: "00_System_Engines"
associated_master_moc: "[[00_Master_MOC]]"
---

# Quy Tắc Ma Thuật Cốt Lõi: Niệm Chú, Nguyên Liệu, Tập Trung (Concentration) & Vùng Ảnh Hưởng

> **Mã gói (Package ID):** `PHB_03_MAGIC_RULES` | **Nguồn khảo cứu:** *Dungeons & Dragons 2024 Player's Handbook (5.5e)*
> **Phân loại hệ thống:** `[[00_System_Engines]]` | **Bản đồ não bộ:** [[00_Master_MOC]]

## 1. Tóm Lược Nghiệp Vụ & Thiết Kế (Executive Summary)
Dữ liệu luật chơi và cơ chế đã qua chắt lọc.

## 3. Sections
- **spell_preparation_and_slots:**
  - **title:** Spell Preparation & Spell Slots
  - **pdf_pages:** 238
  - **book_pages:** 235
  - **rules:**
    - **known_vs_prepared:** In the 2024 revision, all spellcasters prepare spells from their class list after finishing a Long Rest (or as specified by class features).
    - **spell_slots:** Expending a spell slot fuels the casting of a 1st-level or higher spell. Expended slots are regained upon finishing a Long Rest.
    - **cantrips:** 0-level spells that can be cast at will without expending spell slots.
    - **upcasting:** Casting a spell using a slot of higher level than the spell's base level enhances the spell if its description specifies 'At Higher Levels'.
- **casting_time:**
  - **title:** Casting Times & The Bonus Action Rule
  - **pdf_pages:** 238-239
  - **book_pages:** 235-236
  - **casting_time_types:**
    -
      - **type:** Action
      - **description:** Standard combat casting time taking the Magic action on your turn.
    -
      - **type:** Bonus Action
      - **description:** Especially swift spells cast with a Bonus Action on your turn.
      - **one_spell_per_turn_rule_2024:** If you cast a spell with a casting time of a Bonus Action, you cannot cast any other spell on that same turn, except for a cantrip with a casting time of 1 action.
    -
      - **type:** Reaction
      - **description:** Cast instantly in response to a specific trigger (e.g., Shield in response to being hit, Counterspell in response to a creature casting a spell).
    -
      - **type:** Longer Casting Times & Rituals
      - **description:** Spells requiring minutes or hours of continuous concentration. If the spell has the Ritual tag and you have the Ritual Caster capability, you can add 10 minutes to the casting time to cast the spell without expending a spell slot.
- **range_and_targets:**
  - **title:** Range & Targeting
  - **pdf_pages:** 239
  - **book_pages:** 236
  - **rules:**
    - **clear_path_to_target:** To target a creature or point, you must have a clear path to it. A target behind Total Cover cannot be targeted directly.
    - **range_categories:**
      -
        - **category:** Self
        - **description:** The spell affects only the caster or originates from the caster's body.
      -
        - **category:** Touch
        - **description:** Requires the caster to physically touch the target.
      -
        - **category:** Distance in Feet
        - **description:** Must be within the specified linear distance.
      -
        - **category:** Sight
        - **description:** Any visible point or creature within line of sight on the same plane.
- **spell_components:**
  - **title:** Components: Verbal, Somatic, Material
  - **pdf_pages:** 239-240
  - **book_pages:** 236-237
  - **component_types:**
    -
      - **component:** Verbal (V)
      - **nature:** Chanted mystical words with specific pitch, resonance, and cadence.
      - **restrictions:** A creature that is Gagged, under the Silence spell, or unable to speak cannot cast a spell with a Verbal component.
    -
      - **component:** Somatic (S)
      - **nature:** Precise, forceful somatic gestures made with hand and fingers.
      - **restrictions:** The caster MUST have at least one free hand to perform somatic components. A hand holding a spellcasting focus used for the spell's material component can also be used to perform somatic components.
    -
      - **component:** Material (M)
      - **nature:** Physical substances or objects required to manifest the spell.
      - **rules:**
        - **component_pouch_or_focus:** A caster can use a Component Pouch or a Spellcasting Focus (Arcane Focus, Holy Symbol, Druidic Focus) in place of mundane material components that lack a specified gold cost and are not consumed.
        - **costed_materials:** If a material component specifies a gold cost (e.g., a diamond worth at least 300 GP for Revivify), the caster MUST possess that exact component.
        - **consumed_materials:** If the spell description specifies that the component is consumed, that item is permanently destroyed upon casting.
- **duration_and_concentration:**
  - **title:** Duration & Concentration Mechanics
  - **pdf_pages:** 240
  - **book_pages:** 237
  - **durations:**
    -
      - **type:** Instantaneous
      - **description:** The magic harms, heals, or alters reality instantly and cannot be dispelled afterward.
    -
      - **type:** Concentration (up to X duration)
      - **description:** Requires continuous mental focus. The spell lasts up to the listed time or until concentration breaks.
  - **concentration_break_conditions:**
    -
      - **cause:** Taking Damage
      - **mechanics:** Whenever you take damage while concentrating, you must succeed on a Constitution saving throw to maintain focus. The DC equals 10 or half the damage taken (rounded down), whichever is higher. If damage comes from multiple sources, roll separate saves for each source.
    -
      - **cause:** Casting Another Concentration Spell
      - **mechanics:** You lose concentration on a spell the instant you cast another spell that requires Concentration. You can never concentrate on two spells simultaneously.
    -
      - **cause:** Being Incapacitated or Killed
      - **mechanics:** Suffering the Incapacitated, Unconscious, Paralyzed, Petrified, or Stunned condition automatically breaks concentration immediately.
    -
      - **cause:** Environmental Phenomena
      - **mechanics:** Sudden violent events (e.g., a massive wave crashing over a ship) may prompt a DC 10 Constitution saving throw to maintain concentration at the DM's discretion.
- **areas_of_effect:**
  - **title:** 5 Geometric Areas of Effect
  - **pdf_pages:** 240-241
  - **book_pages:** 237-238
  - **geometry_rules:**
    -
      - **shape:** Cone
      - **origin:** Originates from a point on the caster's space and extends forward.
      - **dimensions:** A cone's width at any given point along its length is equal to that point's distance from the point of origin.
    -
      - **shape:** Cube
      - **origin:** You select the cube's point of origin, which lies on any face of the cubic effect.
      - **dimensions:** Extends out from the chosen origin face in a volume measured along its edges.
    -
      - **shape:** Cylinder
      - **origin:** Point of origin is the center of a circular base at the top or bottom.
      - **dimensions:** Extends straight up or down to define a cylinder of specified radius and height.
    -
      - **shape:** Line
      - **origin:** Originates from a point on the caster's space and extends in a straight path.
      - **dimensions:** Extends for a specified length and width (typically 5 or 10 feet wide).
    -
      - **shape:** Sphere
      - **origin:** You select the sphere's point of origin, and the sphere radiates outward in all directions.
      - **dimensions:** Defined by its radius extending in three dimensions.
- **spell_calculations:**
  - **title:** Spell DC & Attack Calculations
  - **pdf_pages:** 241
  - **book_pages:** 238
  - **formulas:**
    - **spell_save_dc:** 8 + your Proficiency Bonus + your Spellcasting Ability Modifier
    - **spell_attack_modifier:** your Proficiency Bonus + your Spellcasting Ability Modifier
