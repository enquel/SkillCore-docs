---
layout: default
title: SkillCore Documentation
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

Version 1.0 · Unreal Engine 5.8 · Windows (Win64)

---


SkillCore is a skill tree plugin for Unreal Engine 5.8. You describe skills and trees as data assets, give each owner a component, and the plugin keeps track of prerequisites, costs, unlocking, effects and saving. The core is C++ with a full Blueprint API. A ready-made screen is built in code, so you can see a working tree on day one, and every part of it can be restyled or replaced.

Platform: Windows (Win64), Unreal Engine 5.8. Full C++ source is included.

## Two-minute demo

1. Copy the `SkillCore` folder into your project's `Plugins` folder and open the project. Enable the plugin if the editor asks.
2. In the Content Browser open the settings (the eye icon) and turn on **Show Plugin Content**.
3. Open `SkillCore Content > Samples > Maps > L_SkillCoreDemo` and press Play.
4. The skill screen opens with the **Warrior** tree. You start with 5 points.

Keys in the demo:

| Action | Mouse and keyboard | Gamepad |
|---|---|---|
| Buy the focused skill | click, Enter | A |
| Move between skills | mouse, arrow keys | d-pad, left stick (with the gamepad cursor off) |
| Point and click | mouse | left stick moves the cursor, A clicks (gamepad cursor on, as in the demo) |
| Next or previous tab | Q and E | shoulder buttons |
| Zoom | mouse wheel, plus and minus | triggers |
| Pan | drag empty space or hold the right mouse button | right stick |
| Fit the tree to the screen | Home | click the right stick |
| Close | Esc | B |

Outside the screen the demo also uses: **K** open, **1 2 3** choose one of the three sample trees (Warrior, Tech Lab, Survivor), **P** add points, **R** reset. Small spheres in the level give points when you touch them, and the pillar next to the demo actor opens the Cleave skill of the Warrior tree (it shows Unlock Skill called from the game world).

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

Add `SkillCore` to your module's `PublicDependencyModuleNames`.

## Concepts

- **Skill** (`USkillDefinition`): an id, title, description, icon, costs, prerequisites and effects. One skill asset can be used in several trees.
- **Tree** (`USkillTree`): categories (tabs) and the skills with their grid positions. Any number of trees can exist.
- **Component** (`USkillComponent`): one per owner (pawn, controller, AI). It holds the point pools and the state of every skill in its trees. One component can use several trees.
- **Point pools**: named pools such as `Points` or `Research`. A skill can cost several pools. Anything can add points: a function call, or an object that implements **Skill Point Source** and is handed to **Collect From Source**.
- **States**: a skill is *Locked* (prerequisites not met), *Available* (can be bought) or *Purchased*.
- **Prerequisites**: by default every prerequisite must be bought; switch **Any Prerequisite** on to need only one. **Starts Unlocked** makes a skill available from the beginning.
- **Unlock and grant**: **Unlock Skill** makes a skill available without its prerequisites and never changes a bought skill. **Grant Skill** gives a skill for free.

## Effects

Effects are objects in a skill's **Effects** list. They run when the skill is bought, again after a save is loaded, and are taken back on reset. Effect objects are shared by every owner that uses the skill, so keep them free of state of their own.

| Effect | What it does |
|---|---|
| Grant Tags | Gives the owner gameplay tags while the skill is bought. Read with **Has Granted Tag**. |
| Grant Counter | Adds an amount to a named counter (extra slots, extra lanes). Read with **Get Counter**. |
| Set Flag | Turns a named flag on. Read with **Has Flag**. |
| Modify Values | Adds to or multiplies named values on the owner. The owner, one of its components, or (for a controller) its pawn implements **Skill Value Target**. The skill's id is sent as the source so each change can be taken back exactly. A skill may change several values. |
| Call Function On Owner | Calls a function without parameters on the owner by name when the skill is bought, and another when it is taken back. |
| Your own | A Blueprint or C++ subclass of **Skill Effect** with **Can Apply**, **Apply** and **Remove**. **Can Apply** can refuse a purchase before any point is spent. |

Nothing in the core depends on an attribute system, inventory, save system or player class. Effects reach your game through the interfaces and the counters above.

## Validation

**Skill Tree** and **Skill Definition** report problems in the editor's data validation and through **Validate** in Blueprint and C++:

- empty or duplicate skill ids, empty tree id
- a prerequisite that is not in the tree, a skill that lists itself, prerequisite cycles
- an unknown category, an empty skill slot, a cost without a pool
- warnings for a missing icon or title

A component refuses a tree that has errors and logs the reasons. Two tree assets with the same Tree Id are refused as well.

## Saving

**Write Save Data** returns a versioned struct (`FSkillSaveData`) with the points and the progress of every tree. **Read Save Data** restores it, applies the effects of the bought skills once, and returns false (changing nothing) when the version is not one it can read. Skills that no longer exist in a tree are skipped with a warning. A record can be read before the trees are added; each tree takes its part when it arrives.

For a quick start there are **Save To Slot** and **Load From Slot**, which use the engine's save game slots. The struct also fits into any save system you already have. Effect state (tags, counters, modifiers) is not saved; it is rebuilt from the bought skills.

## The skill screen

**Skill Menu Widget** builds the whole screen in code: title, point pools, a tab per category, the tree, a details panel and key hints. Pieces:

- **Skill Tree Widget**: one tab, pannable and zoomable.
- **Skill Node Widget**: one skill (icon, title, cost, state colour; a button buys).
- **Skill Connection Widget**: one link between two skills.
- **Skill Details Widget**: the panel for the focused skill.

### Style

Create a **Skill Core Style** data asset to change colours, corner radius, outline width, hover scale, node and cell sizes, font sizes, zoom limits and the three sounds (purchase, denied, focus). The sample style asset (`DA_SampleStyle`) is an amber variant of the default blue look. Assign it in **Project Settings > Plugins > SkillCore**, or pass it to **Open Skill Menu**. Without one the built-in look is used.

### Replacing parts

Every part is a class property of the menu (**Tree Widget Class**, **Node Widget Class**, **Connection Widget Class**, **Details Widget Class**). Make a Blueprint subclass, build your own layout in the designer, and name the widgets the base class looks for. Any subset is fine; names that are missing are skipped.

| Class | Widget names it uses |
|---|---|
| Skill Node Widget | `Button`, `IconImage`, `TitleText`, `CostText`, `Outline` |
| Skill Details Widget | `IconImage`, `TitleText`, `StateText`, `DescriptionText`, `CostText`, `RequirementsText` |
| Skill Tree Widget | `Viewport`, `CanvasBox`, `Canvas` |
| Skill Menu Widget | `TreeHost`, `DetailsHost`, `TabBar`, `PointsText`, `StatusText` |

Each also has a Blueprint event (for example **On View Updated** on the node) so a skin can react to state changes. The menu has **Set Status Text** for a line of your own text under the header.

### Input

The nodes are buttons, so mouse, keyboard and gamepad all use the same click: the accept key or button of the engine's focus navigation buys the focused skill. Focus moves with the engine's navigation (arrow keys, d-pad, stick). Tabs change on Q and E or the shoulder buttons; Esc or the gamepad B button closes the screen.

**Gamepad cursor (optional).** Turn on **Gamepad Cursor** in the style asset and the left stick moves the system cursor over the screen while the bottom face button clicks where it points. **Gamepad Cursor Speed** sets how fast it moves. The d-pad still moves focus, and using it hands the face button back to the focused skill. The cursor is switched on only while the screen is open. The demo has it on.

## Limits of version 1.0

- No ranks per skill: a skill is bought once.
- No respec or refund.
- No graph editor for the layout: positions are grid coordinates in the tree asset.
- Single-player behaviour. The state is plain structs, but replication and multiplayer are not part of 1.0.
- No Gameplay Ability System, Common UI or Enhanced Input integration. The core does not depend on them.
- Windows (Win64) and Unreal Engine 5.8 only.
- A skill's effects run on the owner that bought it; effects are not replicated.

## Save with SaveHub (optional)

SkillCore saves through a plain struct and slot helpers, and needs no other plugin. If your project uses the SaveHub plugin, the folder `Extras/SkillCoreSaveHub` holds a small bridge plugin:

1. Copy `Extras/SkillCoreSaveHub` to `YourProject/Plugins/SkillCoreSaveHub` and enable SkillCore, SaveHub and SkillCoreSaveHub.
2. Add a **Skill SaveHub Link** component next to the **Skill Component**.

The link registers itself with SaveHub when play begins. The points, bought skills and unlocked skills are then written to every slot, read on every load and reset on a new game. **Save Key** names the record (use a different one for a second skill component). The record is JSON text. Without SaveHub, leave the folder where it is; it is not built unless you copy it out and enable it.

## Tests

The plugin ships automation tests (rules, component, save, validation, widgets, gamepad cursor maths, sample content, sample unlocker). Run them with:

```
UnrealEditor-Cmd.exe YourProject.uproject -ExecCmds="Automation RunTests SkillCore" -TestExit="Automation Test Queue Empty" -unattended -nullrhi -nosplash
```

The sample assets were generated by the console command `SkillCore.BuildSamples` (editor only).

## API list

The [API reference](api.html) lists every public class, function and property with its description. The same text is the Blueprint tooltip.

## Support

Discord: https://discord.gg/VNmNpzVAEB
