---
layout: default
title: SkillSet Documentation
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

Version 1.1 · Unreal Engine 5.8 · Windows (Win64)

Free Windows demo: [SkillSet_Demo_Win64_v1.1.zip](https://github.com/enquel/SkillSet-Demo/releases/tag/v1.1)

---

SkillSet is a skill tree plugin for Unreal Engine 5.8. You describe skills and trees as data assets, give each owner a component, and the plugin keeps track of prerequisites, ranks, costs, unlocking, refunds, effects and saving. The core is C++ with a full Blueprint API. A ready-made screen is built in code, so you can see a working tree on day one, and every part of it can be restyled or replaced.

Platform: Windows (Win64), Unreal Engine 5.8. Full C++ source is included.

## Two-minute demo

1. Copy the `SkillSet` folder into your project's `Plugins` folder and open the project. Enable the plugin if the editor asks.
2. In the Content Browser open the settings (the eye icon) and turn on **Show Plugin Content**.
3. Open `SkillSet Content > Samples > Maps > L_SkillSetDemo` and press Play.
4. The skill screen opens with the **Warrior** tree. You start with 5 points. Sharpened Blade and Tough Skin have ranks (click again to raise them), Whirlwind needs 6 ranks in the other Warrior skills, and Berserker and Sentinel are a choose-one pair.

Keys in the demo:

| Action | Mouse and keyboard | Gamepad |
|---|---|---|
| Buy the focused skill, or its next rank | click, Enter | A |
| Refund one rank | right click, Backspace or Delete | X |
| Respec the whole tree (on in the demo) | T | Y |
| Move between skills | mouse, arrow keys | d-pad, left stick (with the gamepad cursor off) |
| Point and click | mouse | left stick moves the cursor, A clicks (gamepad cursor on, as in the demo) |
| Next or previous tab | Q and E | shoulder buttons |
| Zoom | mouse wheel, plus and minus | triggers |
| Pan | drag empty space or hold the right mouse button | right stick |
| Fit the tree to the screen | Home | click the right stick |
| Close | Esc | B |

Outside the screen the demo also uses: **K** open, **1 2 3** choose one of the three sample trees (Warrior, Tech Lab, Survivor), **P** add points, **L** gain XP (every level gives 2 Points), **R** reset. Small spheres in the level give points when you touch them, and the pillar next to the demo actor opens the Cleave skill of the Warrior tree (it shows Unlock Skill called from the game world).

The demo actor (`Skill Demo Actor`) can be dropped into any level. It owns a skill component and shows how the pieces fit.

## Quick start: Blueprint

1. Create a skill: right-click in the Content Browser, **Miscellaneous > Data Asset > Skill Definition**. Set a **Skill Id** (never change it after release), a title, a description, an icon, the cost and the prerequisites.
2. Create a tree: **Data Asset > Skill Tree**. Set a **Tree Id**, add categories (tabs) if you want them, then add each skill with a category and a grid position.
3. Add a **Skill Component** to your pawn, controller or any actor. Put the tree in its **Trees** list and set **Initial Points**.
4. Call **Open Skill Menu** (player controller, component, tree) from a key press or a button. The returned widget can be closed with **Close Skill Menu**.
5. Give points with **Add Points** (pool name and amount) from pickups, quests or level-ups.

Effects: add entries to the **Effects** list of a skill. Built-in effects are described below; to run your own logic make a Blueprint subclass of **Skill Effect** and override **Apply** and **Remove**.

## Quick start: C++

```cpp
USkillComponent* Skills = NewObject<USkillComponent>(Pawn);
Skills->RegisterComponent();
Skills->AddTree(MyTree);
Skills->AddPoints(TEXT("Points"), 5);

if (Skills->PurchaseSkill(MyTree, TEXT("WarBlade")) == ESkillPurchaseResult::Success)
{
    // The skill's effects are applied; OnSkillPurchased and OnSkillStateChanged have fired.
}
USkillMenuLibrary::OpenSkillMenu(PlayerController, Skills, MyTree, nullptr);
```

Add `SkillSet` to your module's `PublicDependencyModuleNames`.

## Concepts

- **Skill** (`USkillDefinition`): an id, title, description, icon, costs, prerequisites and effects. One skill asset can be used in several trees.
- **Tree** (`USkillTree`): categories (tabs) and the skills with their grid positions. Any number of trees can exist.
- **Component** (`USkillComponent`): one per owner (pawn, controller, AI). It holds the point pools and the state of every skill in its trees. One component can use several trees.
- **Point pools**: named pools such as `Points` or `Research`. A skill can cost several pools. Anything can add points: a function call, or an object that implements **Skill Point Source** and is handed to **Collect From Source**.
- **States**: a skill is *Locked* (prerequisites not met), *Available* (can be bought) or *Purchased*.
- **Prerequisites**: by default every prerequisite must be bought; switch **Any Prerequisite** on to need only one. **Starts Unlocked** makes a skill available from the beginning.
- **Unlock and grant**: **Unlock Skill** makes a skill available without its prerequisites or tree gate and never changes a bought skill. **Grant Skill** gives the next rank for free; given ranks are never refunded.
- **Ranks**: set **Max Rank** above 1 and a skill can be bought again to raise its rank. **Costs** is the price of rank 1; **Higher Rank Costs** holds rank 2, 3 and so on (a rank without an entry costs the same as the rank before). A prerequisite can ask for a minimum rank with **Required Ranks** (for example `WarBlade = 2`). Every effect gets the rank (see Effects).
- **Tree gate**: **Required Tree Ranks** is a tier or row gate: that many ranks must be bought in the other skills of the same tree first ("spend 6 points in Warrior").
- **Exclusive group**: skills of a tree with the same **Exclusive Group** name exclude each other. Once one is bought the others are locked until it is refunded (pick one of several).

## Refunds and respec

| Call | What it does |
|---|---|
| **Refund Skill Rank** | Gives back the highest rank of one skill. |
| **Refund Skill** | Gives back every paid rank of one skill, with one fee. |
| **Respec Tree** / **Respec All** | Gives back every paid rank of every refundable skill of a tree (or all trees). No fee, no dependency check. Works even when refunds are off, so your game decides when to offer it (a trainer, an item). |
| **Reset All** | Forgets everything, including given ranks and unlocks. |
| **Can Refund Skill** | What Refund Skill Rank would return, without changing anything. |

The rule for single refunds is **refuse, never cascade**: a refund is refused (result *Required By Other*) when it would leave another bought skill without what it needs: a prerequisite, a prerequisite rank, or its tree gate. The check follows the whole chain from skills that need nothing, so two skills that only hold each other up (through Any Prerequisite or tree gates) still need their root. The skill view names the skill that stops a refund (**Refund Blocked By**) so the player knows what to refund first. A skill bought through **Any Prerequisite** stays as long as one of its prerequisites is still held up. Skills that were opened directly (Starts Unlocked, Unlock Skill) and skills that hold a given rank do not depend on their prerequisites.

Settings on the component: **Allow Refunds** (on by default; can change at runtime), **Refund Percent** (0 to 100, rounded down per pool) and **Refund Fee** (points taken per refund call; the refunded points can pay it). On a skill: **Refundable** off makes it permanent (Respec keeps it too). A refund gives back what was paid for each rank (newest paid rank first), even when the rank costs in the asset change after the purchase; the paid amounts are part of the save. Ranks given by **Grant Skill** are never refunded.

Events: **On Skill Refunded** (tree, skill, rank left) and **On Skill Rank Changed** (every rank change, including loads and resets).

## Experience and levels (optional)

Add a **Skill Level Component** next to the Skill Component. **Add XP** raises the level; every level gained adds **Points Per Level** to the named pools. **First Level XP** and **Level XP Growth** set the XP curve (or give exact steps in **XP Per Level**), **Max Level** caps it. When the level goes down (**Set Level**, negative XP) the points those levels gave are taken back; if the pools are short, the trees are respecced first so spent points come back before they are removed (**Take Points On Level Down**). **Write Save Data** / **Read Save Data** store the XP; loading never gives or takes points, because the Skill Component's own record holds them. Events: **On Level Changed**, **On XP Changed**.

## Effects

Effects are objects in a skill's **Effects** list. They run when the skill is bought, again after a save is loaded, and are taken back on refund or reset. Effect objects are shared by every owner that uses the skill, so keep them free of state of their own.

Ranks: the effect context carries **Rank** and **Max Rank**. When the rank of a bought skill changes, **Change Rank** runs with the new rank and the old one; by default it is **Remove** at the old rank followed by **Apply** at the new one, so an effect only has to do what one rank number asks for. Override **Change Rank** to update in place (the built-in counter, tag and flag effects do, so their events do not flicker). Each effect can also **Describe** itself for the details panel (the built-in ones do; fill **Display Text** with `{Rank}` and `{MaxRank}` for your own line).

| Effect | What it does |
|---|---|
| Grant Tags | Gives the owner gameplay tags while the skill is bought. Read with **Has Granted Tag**. |
| Grant Counter | Adds an amount to a named counter (extra slots, extra lanes). Read with **Get Counter**. **Scale With Rank** adds the amount per rank. |
| Set Flag | Turns a named flag on. Read with **Has Flag**. |
| Modify Values | Adds to or multiplies named values on the owner. The owner, one of its components, or (for a controller) its pawn implements **Skill Value Target**. The skill's id is sent as the source so each change can be taken back exactly. A skill may change several values. Per modifier: **Scale With Rank** (Add sends Amount x Rank; Multiply sends 1 + (Amount - 1) x Rank, so 1.1 is +10 % per rank), **Amount Per Rank** for exact values, **Display Name** for the details panel. |
| Call Function On Owner | Calls a function without parameters on the owner by name when the skill is bought, and another when it is taken back. |
| Your own | A Blueprint or C++ subclass of **Skill Effect** with **Can Apply**, **Apply** and **Remove**. **Can Apply** can refuse a purchase before any point is spent. |

Nothing in the core depends on an attribute system, inventory, save system or player class. Effects reach your game through the interfaces and the counters above.

## Validation

**Skill Tree** and **Skill Definition** report problems in the editor's data validation and through **Validate** in Blueprint and C++:

- empty or duplicate skill ids, empty tree id
- a prerequisite that is not in the tree, a skill that lists itself, prerequisite cycles
- an unknown category, an empty skill slot, a cost without a pool
- a Max Rank below 1, a required rank of a skill that is not a prerequisite or above its Max Rank
- a tree gate the other skills can never reach, a prerequisite inside the skill's own exclusive group
- warnings for a missing icon or title, and for higher rank costs past the Max Rank

A component refuses a tree that has errors and logs the reasons. Two tree assets with the same Tree Id are refused as well.

## Saving

**Write Save Data** returns a versioned struct (`FSkillSaveData`, format version 2) with the points and the progress of every tree, including the rank of every bought skill and how many of its ranks were given. Version 1 records (from SkillSet 1.0) load: every bought skill gets rank 1. **Read Save Data** restores it, applies the effects of the bought skills once, and returns false (changing nothing) when the version is not one it can read. Skills that no longer exist in a tree are skipped with a warning. A record can be read before the trees are added; each tree takes its part when it arrives.

For a quick start there are **Save To Slot** and **Load From Slot**, which use the engine's save game slots. The struct also fits into any save system you already have. Effect state (tags, counters, modifiers) is not saved; it is rebuilt from the bought skills.

## The skill screen

**Skill Menu Widget** builds the whole screen in code: title, point pools, a tab per category, the tree, a details panel and key hints. Pieces:

- **Skill Tree Widget**: one tab, pannable and zoomable.
- **Skill Node Widget**: one skill (icon, title, cost of the next rank, a rank badge such as `2/5`, state colour; a button buys, a right click refunds one rank).
- **Skill Connection Widget**: one link between two skills.
- **Skill Details Widget**: the panel for the focused skill: state and rank, description, what the effects do now and at the next rank, the next cost, what is still missing (prerequisites with their ranks, the tree gate, the exclusive choice) and the refund line with a **Refund** button, shown only when a refund is allowed. **Describe View** gives the same lines as text for your own tooltips.

Under the tree a feedback line says what the last action did or why it was refused ("Not enough points for Cleave", "Whirlwind needs Cleave: refund Whirlwind first"). **Show Feedback** writes your own message there. **Offer Respec** on the menu adds the respec key (T, gamepad Y) for the shown tree; it is off by default.

### Style

Create a **Skill Core Style** data asset to change colours, corner radius, outline width, hover scale, node and cell sizes, font sizes, zoom limits and the three sounds (purchase, denied, focus). The sample style asset (`DA_SampleStyle`) is an amber variant of the default blue look. Assign it in **Project Settings > Plugins > SkillSet**, or pass it to **Open Skill Menu**. Without one the built-in look is used.

### Replacing parts

Every part is a class property of the menu (**Tree Widget Class**, **Node Widget Class**, **Connection Widget Class**, **Details Widget Class**). Make a Blueprint subclass, build your own layout in the designer, and name the widgets the base class looks for. Any subset is fine; names that are missing are skipped.

| Class | Widget names it uses |
|---|---|
| Skill Node Widget | `Button`, `IconImage`, `TitleText`, `CostText`, `RankText`, `Outline` |
| Skill Details Widget | `IconImage`, `TitleText`, `StateText`, `DescriptionText`, `EffectsText`, `CostText`, `RequirementsText`, `RefundText`, `RefundButton` |
| Skill Tree Widget | `Viewport`, `CanvasBox`, `Canvas` |
| Skill Menu Widget | `TreeHost`, `DetailsHost`, `TabBar`, `PointsText`, `StatusText`, `FeedbackText`, `HintsText` |

Each also has a Blueprint event (for example **On View Updated** on the node) so a skin can react to state changes. The menu has **Set Status Text** for a line of your own text under the header.

### Input

The nodes are buttons, so mouse, keyboard and gamepad all use the same click: the accept key or button of the engine's focus navigation buys the focused skill. Arrow keys, the d-pad and the stick move focus to the nearest card in that direction by grid position (the next column or row first), so no card is skipped on diagonal layouts. Refund one rank: right click, Backspace or Delete, or the gamepad left face button (X), when the owner allows refunds. Tabs change on Q and E or the shoulder buttons; Esc or the gamepad B button closes the screen.

**Gamepad cursor (optional).** Turn on **Gamepad Cursor** in the style asset and the left stick moves the system cursor over the screen while the bottom face button clicks where it points. **Gamepad Cursor Speed** sets how fast it moves. The d-pad still moves focus, and using it hands the face button back to the focused skill. The cursor is switched on only while the screen is open. The demo has it on.

## Limits of version 1.1

- No graph editor for the layout: positions are grid coordinates in the tree asset.
- Single-player behaviour. The state is plain structs, but the component does not replicate and multiplayer is not supported.
- Refunds refuse when another skill depends on the rank; there is no cascading refund.
- No search box on the screen.
- The core has no Gameplay Ability System, Common UI or Enhanced Input dependency. The optional GAS bridge (below) gives abilities and gameplay effects; there is no Common UI or Enhanced Input bridge (the screen uses UMG focus, so open it from your own input action).
- Windows (Win64) and Unreal Engine 5.8 only.
- A skill's effects run on the owner that bought it; effects are not replicated.

## Save with SaveHub (optional)

SkillSet saves through a plain struct and slot helpers, and needs no other plugin. If your project uses the SaveHub plugin, the folder `Extras/SkillSetSaveHub` holds a small bridge plugin:

1. Copy `Extras/SkillSetSaveHub` to `YourProject/Plugins/SkillSetSaveHub` and enable SkillSet, SaveHub and SkillSetSaveHub.
2. Add a **Skill SaveHub Link** component next to the **Skill Component**.

The link registers itself with SaveHub when play begins. The points, bought skills and unlocked skills are then written to every slot, read on every load and reset on a new game. **Save Key** names the record (use a different one for a second skill component). The record is JSON text. Without SaveHub, leave the folder where it is; it is not built unless you copy it out and enable it.

## Gameplay Ability System (optional)

The folder `Extras/SkillSetGAS` holds a small bridge plugin for projects that use the Gameplay Ability System:

1. Copy `Extras/SkillSetGAS` to `YourProject/Plugins/SkillSetGAS` and enable SkillSet, Gameplay Abilities and SkillSetGAS.
2. Add the effects to a skill's **Effects** list:
   - **Grant Gameplay Ability**: gives an ability while the skill is bought. The ability level is the skill rank (or a fixed **Level**).
   - **Apply Gameplay Effect**: applies a gameplay effect (use an infinite one) while the skill is bought, at a level that is the skill rank. Use the level in the effect's scalable magnitudes for "+5 % per rank".

The ability system component is found on the owner, its components, or (for a controller) its pawn. Abilities and effects are given only where the owner has authority. A refund, respec or reset takes them back; a rank change sets the level of the same ability spec and the same active effect (a running ability keeps running). Each tree that holds the skill gives its own grant. Instant gameplay effects are refused with a warning, because they could not be taken back. Without GAS, leave the folder where it is; it is not built unless you copy it out and enable it.

## Tests

The plugin ships automation tests (rules, ranks, refunds and respec, gates and exclusive groups, component, save including version 1 records, levels, validation, widgets and grid navigation, gamepad cursor maths, sample content, sample unlocker; the two bridges have their own). Run them with:

```
UnrealEditor-Cmd.exe YourProject.uproject -ExecCmds="Automation RunTests SkillSet" -TestExit="Automation Test Queue Empty" -unattended -nullrhi -nosplash
```

The sample assets were generated by the console command `SkillSet.BuildSamples` (editor only).

## API list

The [API reference](api.html) lists every public class, function and property with its description. The same text is the Blueprint tooltip.

## Support

Discord: https://discord.gg/VNmNpzVAEB

Online documentation: https://enquel.github.io/SkillSet-docs/
