---
id: "doc:gmagd_04_pattern_library"
package_id: "GMAGD_04_PATTERN_LIBRARY"
title: "Bảng Tra Cứu Machinations & Thư Viện 16 Mẫu Thiết Kế Cơ Chế Chuẩn"
original_title: "Machinations Quick Reference & Design Pattern Library (16 Core Patterns)"
source_reference: "D&D 5E / AD&D / RPG Advanced Design"
system_category: "00_System_Engines"
associated_master_moc: "[[00_Master_MOC]]"
---

# Bảng Tra Cứu Machinations & Thư Viện 16 Mẫu Thiết Kế Cơ Chế Chuẩn

> **Mã gói (Package ID):** `GMAGD_04_PATTERN_LIBRARY` | **Nguồn khảo cứu:** *D&D 5E / AD&D / RPG Advanced Design*
> **Phân loại hệ thống:** `[[00_System_Engines]]` | **Bản đồ não bộ:** [[00_Master_MOC]]

## 1. Tóm Lược Nghiệp Vụ & Thiết Kế (Executive Summary)
Tài liệu tra cứu kỹ thuật chuẩn hóa: Toàn bộ bảng ký hiệu cú pháp Machinations (Nút nguồn, Kho, Xả rác, Chuyển đổi, Cổng xác suất, Liên kết tài nguyên & Liên kết trạng thái); cùng Thư viện đầy đủ 16 Mẫu Thiết Kế Cơ Chế Chuẩn (Static Engine, Dynamic Engine, Converter Engine, Static Friction, Dynamic Friction, Attrition, Stopping Mechanism, Feedback Loop, Mutual Dependency, Shared Resource, Trade, Worker Placement, Escalating Friction, Playing the Odds, Escalating Energy, Arms Race).

## 3. Book Pages
301–336

## 4. Pdf Pages
316–351

## 5. Total Pages
36

## 6. Pdf File
gmagd_04_pattern_library.pdf

## 7. Content Sections
-
  - **pdf_page:** 316
  - **book_page:** 301
  - **header:** AppEnDIx A
  - **section_title:** Machinations Quick reference
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** AppEnDIx A
Appendix A:
Machinations Quick reference
301
Flow of resources
between nodes
Resource Connections
Resource connections dictate how resources flow between nodes.
  - **total_text_lines:** 106
-
  - **pdf_page:** 317
  - **book_page:** 302
  - **header:** 302
  - **section_title:** 302
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 302
Game mechanics: advanced Game desiGn
Random
Skill
Multiplayer
Strategy
Nodes
Nodes represent game elements that take part in the production, distribution and consumption of resources. Nodes can fire. Firing nodes
  - **total_text_lines:** 121
-
  - **pdf_page:** 318
  - **book_page:** 303
  - **header:** AppEnDIx B
  - **section_title:** Design pattern Library
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** AppEnDIx B
Design pattern Library
Static Engine
n Type: Engine
n Intent: Produces a steady flow of resources over time for players to consume or to
collect while playing the game.
n Motivation: A static engine creates a steady flow of resources that never dries up.
applicability
  - **total_text_lines:** 35
-
  - **pdf_page:** 319
  - **book_page:** 304
  - **header:** 304
  - **section_title:** A static engine generally does not inspire long-term strategies: Collecting resources
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 304
Game mechanics: advanced Game desiGn
A static engine generally does not inspire long-term strategies: Collecting resources
from a static engine, if possible at all, will be quite obvious.
implementation
Normally, it is simple to implement a static engine: A single source that produces the
energy will suffice. It is possible to add multiple steps in the energy production, but
in general this will add little to the game.
  - **total_text_lines:** 33
-
  - **pdf_page:** 320
  - **book_page:** 305
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
305
AppEnDIx B
In many turn-based games, the limited number of actions a player can perform in
each turn can be considered a static engine. In this case, the focus of the game is
the choice of actions, and generally players cannot save actions for later turns. The
fantasy board game Descent: Journeys in the Dark uses this mechanism. Players can
choose between one of three actions for their hero every turn: move, attack, or pre-
  - **total_text_lines:** 33
-
  - **pdf_page:** 321
  - **book_page:** 306
  - **header:** 306
  - **section_title:** structure
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 306
Game mechanics: advanced Game desiGn
structure
Participants
n Energy produced by the dynamic
engine
n A source that produces energy
n Upgrades that affect the production
  - **total_text_lines:** 38
-
  - **pdf_page:** 322
  - **book_page:** 307
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
307
AppEnDIx B
dynamic engine will amplify the luck a player has in the beginning of the game,
which might result in too much randomness quickly.
The outcome of random production rates can be, but does not need to be, the same
for every player. By using an unpredictable dynamic engine that generates the same
resources for all players, the luck factor is reduced without affecting the unpredict-
  - **total_text_lines:** 38
-
  - **pdf_page:** 323
  - **book_page:** 308
  - **header:** 308
  - **section_title:** Converter Engine
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 308
Game mechanics: advanced Game desiGn
Converter Engine
n Type: Engine
n Intent: Two converters set up in a loop create a surplus of resources that can be
used elsewhere in the game.
n Motivation: Two resources that can be converted into each other fuel a feedback
loop that produces a surplus of resources. At least one of the converters must output
  - **total_text_lines:** 31
-
  - **pdf_page:** 324
  - **book_page:** 309
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
309
AppEnDIx B
consequences
A converter engine introduces the chance of a deadlock. When both resources dry
up, the engine stops working. Players run the risk of creating deadlocks themselves
if they forget to invest energy to get new fuel. Combine a converter engine with a
weak static engine to prevent this from happening.
  - **total_text_lines:** 41
-
  - **pdf_page:** 325
  - **book_page:** 310
  - **header:** 310
  - **section_title:** effectiveness of the converter engine: The ship’s “hyperspeed” range and its capabili-
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 310
Game mechanics: advanced Game desiGn
effectiveness of the converter engine: The ship’s “hyperspeed” range and its capabili-
ties (or cost) to survive a voyage through hostile territories all affect the profitability
of particular trade routes. Eventually, trade routes become less profitable as the play-
er’s efforts reduce the demand, and thus the price, for certain goods over time (a
mechanism that is omitted from the diagram).
The player’s location on Planet A or Planet B activates the converters that imple-
  - **total_text_lines:** 27
-
  - **pdf_page:** 326
  - **book_page:** 311
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
311
AppEnDIx B
related Patterns
n A converter engine is well suited to be combined with the engine building pattern
because there are many opportunities to change the settings of the engine: the con-
version rate of two converters and possibly the setting of a limiter.
n The positive feedback a converter engine creates is best balanced by introducing
  - **total_text_lines:** 30
-
  - **pdf_page:** 327
  - **book_page:** 312
  - **header:** 312
  - **section_title:** structure
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 312
Game mechanics: advanced Game desiGn
structure
Participants
n The core engine usually is a complex structure combining multiple engine types.
n At least one, but usually multiple, building mechanisms to improve the core
engine.
n Energy is the main resource produced by the core engine.
  - **total_text_lines:** 34
-
  - **pdf_page:** 328
  - **book_page:** 313
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
313
AppEnDIx B
An engine building pattern operates without feedback when energy is not required
to activate building mechanisms. This can be a viable structure when the engine
produces different types of energy that affect the game differently and allows the
players to follow different strategies that favor particular forms of energy above oth-
ers. However, it usually does require that the activation of building mechanisms is
  - **total_text_lines:** 41
-
  - **pdf_page:** 329
  - **book_page:** 314
  - **header:** 314
  - **section_title:** Static Friction
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 314
Game mechanics: advanced Game desiGn
Static Friction
n Type: Friction
n Intent: A drain automatically consumes resources produced by the player.
n Motivation: The static friction pattern counters a production mechanism by
periodically consuming resources. The rate of consumption can be constant or sub-
jected to randomness.
  - **total_text_lines:** 34
-
  - **pdf_page:** 330
  - **book_page:** 315
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
315
AppEnDIx B
be a good alternative to using randomness in the production mechanism. The fre-
quency of the friction is another consideration: When the feedback is applied at
short intervals, the overall system will be more stable than when the feedback is
applied at long or irregular intervals, which might cause periodic behavior in the
system. In general, the effects of a continual loss of energy on the dynamic behavior
  - **total_text_lines:** 34
-
  - **pdf_page:** 331
  - **book_page:** 316
  - **header:** 316
  - **section_title:** Dynamic Friction
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 316
Game mechanics: advanced Game desiGn
Dynamic Friction
n Type: Friction
n Intent: A drain automatically consumes resources produced by the player; the
consumption rate is affected by the state of other elements in the game.
n Motivation: Dynamic friction counteracts production but adapts to the perfor-
mance of the player. Dynamic friction is a classic application of negative feedback in
  - **total_text_lines:** 26
-
  - **pdf_page:** 332
  - **book_page:** 317
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
317
AppEnDIx B
consequences
Dynamic friction is a good way to counter positive feedback created by engine pat-
terns. Dynamic friction adds a negative feedback loop to the game system.
implementation
There are several ways of implementing dynamic feedback. An important consid-
  - **total_text_lines:** 32
-
  - **pdf_page:** 333
  - **book_page:** 318
  - **header:** 318
  - **section_title:** Dynamic friction is used in the city production mechanism in Civilization (Figure B.8).
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 318
Game mechanics: advanced Game desiGn
Dynamic friction is used in the city production mechanism in Civilization (Figure B.8).
In this game, the player builds cities to produce food, shields, and trade. As cities
grow, they need more and more food for their own population. Players have some
control over how much food is produced compared with other resources, but the
players’ options are limited by the surrounding terrain. By choosing to produce a lot
of food early, cities initially produce fewer other resources but grow faster because
  - **total_text_lines:** 34
-
  - **pdf_page:** 334
  - **book_page:** 319
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
319
AppEnDIx B
Stopping Mechanism
n Type: Friction
n Intent: Reduce the effectiveness of a mechanism every time it is activated.
n Also known as: Law of diminishing returns.
n Motivation: To prevent a player from abusing a powerful mechanism, the mech-
  - **total_text_lines:** 26
-
  - **pdf_page:** 335
  - **book_page:** 320
  - **header:** 320
  - **section_title:** Using a stopping mechanism can reduce the effect of a positive feedback loop con-
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 320
Game mechanics: advanced Game desiGn
consequences
Using a stopping mechanism can reduce the effect of a positive feedback loop con-
siderably and even make its return insufficient.
implementation
When implementing a stopping mechanism, it is important to consider whether to
make the effects permanent. When the accumulated output is used to measure the
  - **total_text_lines:** 36
-
  - **pdf_page:** 336
  - **book_page:** 321
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
321
AppEnDIx B
related Patterns
n Stopping mechanisms are often found in systems that implement multiple feedback.
n A stopping mechanism elaborates the dynamic friction pattern.
n A stopping mechanism might be elaborated by a slow cycle pattern.
Attrition
  - **total_text_lines:** 26
-
  - **pdf_page:** 337
  - **book_page:** 322
  - **header:** 322
  - **section_title:** structure
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 322
Game mechanics: advanced Game desiGn
structure
Participants
n Multiple players who have the same (or similar) mechanics and options.
n A strength resource. A player who loses all his strength is eliminated from the
game.
n A special attack action that drains or steals the other player’s strength.
  - **total_text_lines:** 43
-
  - **pdf_page:** 338
  - **book_page:** 323
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
323
AppEnDIx B
It is quite common to implement attrition using two resources, life and energy, instead
of just one, strength. Players use energy to perform actions and lose the game when
they run out of life. When using these two resources, it is important that they be
somehow related. Often, players are allowed to spend energy to gain more life.
Sometimes the relationship between life and energy is implicit. For example, when
  - **total_text_lines:** 31
-
  - **pdf_page:** 339
  - **book_page:** 324
  - **header:** 324
  - **section_title:** 324
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 324
Game mechanics: advanced Game desiGn
In Magic: The Gathering, players can play one card every turn. These cards allow
players to add lands, summon creatures, cast spells to heal, or deal direct damage to
their opponent or their opponent’s creatures. But all actions except playing lands
cost mana (magical energy). The more mana players have, the more they can spend
each turn and the more powerful actions they can play. Creatures will fight other
creatures, and when there are no more enemy creatures, they will damage the oppo-
  - **total_text_lines:** 34
-
  - **pdf_page:** 340
  - **book_page:** 325
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
325
AppEnDIx B
related Patterns
n Attrition works well with any sort of engine pattern. Trade can be used as an alter-
native form of multiplayer feedback that is constructive instead of destructive and is
nearly always negative.
n Attrition elaborates the dynamic friction pattern.
  - **total_text_lines:** 24
-
  - **pdf_page:** 341
  - **book_page:** 326
  - **header:** 326
  - **section_title:** n A feedback mechanism makes the game more difficult as the player progresses
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 326
Game mechanics: advanced Game desiGn
Participants
n Targets represent unresolved tasks.
n Progress represents the player’s progress toward a goal.
n A task either reduces the number of targets or produces progress.
n A feedback mechanism makes the game more difficult as the player progresses
toward the goal or reduces the number of targets.
  - **total_text_lines:** 36
-
  - **pdf_page:** 342
  - **book_page:** 327
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
327
AppEnDIx B
Escalating Complexity
n Type: Escalation
n Intent: Players act against growing complexity, trying to keep the game under
control until positive feedback grows too strong and the accumulated complexity
makes them lose.
  - **total_text_lines:** 33
-
  - **pdf_page:** 343
  - **book_page:** 328
  - **header:** 328
  - **section_title:** The task in a game that implements the escalating complexity pattern is typically
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 328
Game mechanics: advanced Game desiGn
consequences
Given enough skill, players can keep up with the increase in complexity for a long
time, but when players no longer keep up, complexity spins out of control and the
game ends quickly.
implementation
The task in a game that implements the escalating complexity pattern is typically
  - **total_text_lines:** 42
-
  - **pdf_page:** 344
  - **book_page:** 329
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
329
AppEnDIx B
in the game. This represents the possibility to clear more lines at once and enables a
high-risk, high-reward strategy. The chart in Figure B.13 clearly shows that once the
pace grows too great for the player to keep up, the game rapidly spins out of control.
In the independently developed action shooter Super Crate Box, players are required
to pick up crates containing different weapons, while keeping the number of ene-
  - **total_text_lines:** 27
-
  - **pdf_page:** 345
  - **book_page:** 330
  - **header:** 330
  - **section_title:** related Patterns
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 330
Game mechanics: advanced Game desiGn
related Patterns
n Any type of engine pattern can be used to implement the progress mechanism.
n It is common to find the progression mechanism implemented as an escalating
challenge pattern.
Arms Race
n Type: Escalation
  - **total_text_lines:** 21
-
  - **pdf_page:** 346
  - **book_page:** 331
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
331
AppEnDIx B
Participants
n Multiple players that can activate the same (or similar) attack mechanisms.
n A strength resource. A player that loses all his strength is eliminated from the
game.
n An optional energy resource that is consumed by upgrades. In some cases, energy
  - **total_text_lines:** 39
-
  - **pdf_page:** 347
  - **book_page:** 332
  - **header:** 332
  - **section_title:** 332
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 332
Game mechanics: advanced Game desiGn
A good way to prevent an arms race from lengthening the game too much is to
make the resource to activate upgrades heavily contested, either because all players
are trying to harvest the same resources or because upgrades require the player to
invest strength.
An arms race doesn’t have to be symmetrical. It is possible to create an arms race
with two different sides, although this would be more difficult to balance.
  - **total_text_lines:** 31
-
  - **pdf_page:** 348
  - **book_page:** 333
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
333
AppEnDIx B
Playing Style Reinforcement
n Type: Miscellaneous
n Intent: By applying slow, positive, constructive feedback on player actions, the
game encourages specialization and gradually adapts to the player’s preferred play-
ing style.
  - **total_text_lines:** 23
-
  - **pdf_page:** 349
  - **book_page:** 334
  - **header:** 334
  - **section_title:** 334
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 334
Game mechanics: advanced Game desiGn
Participants
n Actions players can perform whose success depends in part on the attributes of
the player’s character or the units involved in the action.
n A resource ability that affects the chance that actions succeed and that can grow
over time.
n An optional resource experience points that can be used to increase an ability.
  - **total_text_lines:** 35
-
  - **pdf_page:** 350
  - **book_page:** 335
  - **header:** desiGn PaTTern LiBrarY
  - **section_title:** desiGn PaTTern LiBrarY
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** desiGn PaTTern LiBrarY
335
AppEnDIx B
implementation
Whether or not to use experience points is an important decision when imple-
menting play style reinforcement. When using experience points, there is no direct
coupling between growth and action, allowing the player to harvest experience
with one strategy to develop the skills to excel in another strategy. On the other
  - **total_text_lines:** 41
-
  - **pdf_page:** 351
  - **book_page:** 336
  - **header:** 336
  - **section_title:** 336
  - **figures_mentioned:**

  - **patterns_mentioned:**

  - **excerpt:** 336
Game mechanics: advanced Game desiGn
In Civilization III, there are different ways in which a player can win the game.
A player reinforces his chosen strategy of military, economic, cultural, or scientific
dominance (or any combination) by building city improvements and wonders of
the world that favor that strategy. In Civilization III, several resources take the role of
experience points; money and production are prominent examples. These resources
are not necessarily tied to one particular strategy in the game. Money generated by
  - **total_text_lines:** 25
