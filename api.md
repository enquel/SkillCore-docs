---
layout: default
title: SkillSet API reference
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

---

Generated from the doc comments of the public headers (`Source/SkillSet/Public`). The same text is the tooltip of the Blueprint node.

## SkillComponent.h

### USkillComponent

Holds one owner's skill progress: named point pools, the state of every skill in its trees, and the effect state skills produce (tags, counters). Put it on a pawn, a controller or an AI actor. Single-player correct; the state is plain structs, so your own replication code can carry it.

- **Property** `TArray<TObjectPtr<USkillTree>> Trees`: Trees this owner can use from the start. More can be added at runtime with Add Tree.
- **Property** `TMap<FName, int32> InitialPoints`: Points given once at begin play, per named pool.
- **Property** `bool bAllowRefunds`: Players may refund skills (Refund Skill Rank, Refund Skill). Respec Tree and Reset All work either way. Can be changed at runtime, for example only near a trainer.
- **Property** `int32 RefundPercent`: Share of the cost given back by a refund or respec, 0 to 100. Rounded down per pool.
- **Property** `TArray<FSkillCost> RefundFee`: Points taken for each refund call (Refund Skill Rank or Refund Skill), for example 1 "Gold". Respec Tree takes no fee. Empty for none.
- **Property** `FSkillPurchasedSignature OnSkillPurchased`: Broadcast after a skill rank was bought or granted and its effects applied (once per rank). Not broadcast when a save is loaded; use On Skill State Changed for that.
- **Property** `FSkillRankChangedSignature OnSkillRefunded`: Broadcast after a refund (one rank, a whole skill or a respec) with the rank that is left.
- **Property** `FSkillRankChangedSignature OnSkillRankChanged`: Broadcast whenever the rank of a skill changes: purchase, grant, refund, respec, reset or load.
- **Property** `FSkillStateChangedSignature OnSkillStateChanged`: Broadcast whenever a skill changes state (locked, available, purchased), including after a load or reset.
- **Property** `FSkillPointsChangedSignature OnPointsChanged`: Broadcast when a pool amount changes.
- **Property** `FSkillTagChangedSignature OnGrantedTagChanged`: Broadcast when a skill effect grants or takes back a gameplay tag.
- **Property** `FSkillCounterChangedSignature OnCounterChanged`: Broadcast when a skill effect changes a named counter.
- **Function** `bool AddTree(USkillTree* Tree)`: Adds a tree to this owner. Returns false when the tree is invalid, has no id, or a tree with the same id is already added.
- **Function** `bool HasTree(const USkillTree* Tree) const`: True when the tree is added to this owner.
- **Function** `void AddPoints(FName PoolId, int32 Amount)`: Adds points to a named pool. A negative amount takes points away (the pool never goes below zero).
- **Function** `void SetPoints(FName PoolId, int32 Amount)`: Sets a pool to an exact amount.
- **Function** `int32 GetPoints(FName PoolId) const`: Points currently in a pool.
- **Function** `int32 CollectFromSource(UObject* Source)`: Takes the points from an object that implements Skill Point Source. Returns the amount added.
- **Function** `ESkillState GetSkillState(const USkillTree* Tree, FName SkillId) const`: State of a skill. Locked for unknown trees or skills.
- **Function** `bool IsSkillPurchased(const USkillTree* Tree, FName SkillId) const`: True when the skill is bought.
- **Function** `int32 GetSkillRank(const USkillTree* Tree, FName SkillId) const`: Ranks bought of a skill (0 while not bought).
- **Function** `int32 GetSkillMaxRank(const USkillTree* Tree, FName SkillId) const`: Highest rank of a skill (1 for a skill bought once, and for unknown skills).
- **Function** `int32 GetTreeRanks(const USkillTree* Tree) const`: Ranks bought in all skills of a tree together. A tree gate counts the same ranks minus those of the gated skill itself.
- **Function** `ESkillPurchaseResult CanPurchaseSkill(const USkillTree* Tree, FName SkillId) const`: What Purchase Skill would return right now, without changing anything.
- **Function** `ESkillPurchaseResult PurchaseSkill(USkillTree* Tree, FName SkillId)`: Buys a skill, or its next rank when it is already bought: checks the rules and points, spends the cost, applies the effects with the new rank.
- **Function** `bool UnlockSkill(USkillTree* Tree, FName SkillId)`: Makes a skill available without its prerequisites or tree gate (an exclusive group still applies). A bought skill is never changed. Returns true when the state changed.
- **Function** `bool GrantSkill(USkillTree* Tree, FName SkillId)`: Gives the next rank of a skill for free (no cost, no rules) and applies its effects. Given ranks are never refunded. Returns false when unknown or at the highest rank.
- **Function** `ESkillRefundResult CanRefundSkill(const USkillTree* Tree, FName SkillId) const`: What Refund Skill Rank would return right now, without changing anything.
- **Function** `ESkillRefundResult RefundSkillRank(USkillTree* Tree, FName SkillId)`: Gives back the highest rank of a skill: returns its cost (times Refund Percent), takes the Refund Fee and the effects of that rank. Refused when another bought skill needs the rank (a prerequisite, a required rank or a tree gate): refund that skill first. Ranks given by Grant Skill are never refunded.
- **Function** `ESkillRefundResult RefundSkill(USkillTree* Tree, FName SkillId)`: Gives back every paid rank of a skill at once, with the same rules as Refund Skill Rank and one fee.
- **Function** `bool RespecTree(USkillTree* Tree)`: Full respec of one tree: every paid rank of every refundable skill is given back (times Refund Percent), without fee and without the dependency rule. Given ranks, skills marked not refundable and unlocks stay. Works even when Allow Refunds is off, so the game decides when to offer it. Returns true when anything changed.
- **Function** `bool RespecAll()`: Respec Tree for every tree of this owner. Returns true when anything changed.
- **Function** `void ResetAll(bool bResetPoints = true)`: Forgets every purchase and unlock of all trees and takes back all effects. Optionally empties the point pools too.
- **Function** `FSkillView GetSkillView(const USkillTree* Tree, FName SkillId) const`: Snapshot of one skill for UI code. Skill Id is empty when the skill is unknown.
- **Function** `TArray<FSkillView> GetSkillViews(const USkillTree* Tree, FName CategoryId) const`: Snapshots of the skills of a tree. Pass an empty Category Id for all of them.
- **Function** `bool HasGrantedTag(FGameplayTag Tag) const`: True when a bought skill currently grants this tag.
- **Function** `FGameplayTagContainer GetGrantedTags() const`: Every tag currently granted by bought skills.
- **Function** `int32 GetCounter(FName CounterName) const`: Value of a named counter (0 when no skill touched it).
- **Function** `bool HasFlag(FName FlagName) const`: True when the counter is above zero.
- **Function** `void AddGrantedTag(FGameplayTag Tag)`: Adds one grant of a tag. Counted, so two skills granting the same tag do not cancel each other.
- **Function** `void RemoveGrantedTag(FGameplayTag Tag)`: Takes back one grant of a tag. The tag disappears when no grant is left.
- **Function** `void AddToCounter(FName CounterName, int32 Delta)`: Adds to (or, with a negative delta, takes from) a named counter. Reset All and Read Save Data clear all counters and tags.
- **Function** `UObject* FindInterfaceImplementer(TSubclassOf<UInterface> InterfaceClass) const`: Finds an object implementing an interface: the owner, then its components, then (for a controller) its pawn and the pawn's components, or (for a pawn) its controller. Returns null when nothing implements it.
- **Function** `FSkillSaveData WriteSaveData() const`: A versioned record of points and progress. Store it in any save system.
- **Function** `bool ReadSaveData(const FSkillSaveData& Data)`: Restores a record: replaces points and progress, then applies the effects of the bought skills once. Returns false (and changes nothing) when the version is unknown. Skills that no longer exist are skipped.
- **Function** `bool SaveToSlot(const FString& SlotName, int32 UserIndex = 0) const`: Writes the record to a save slot (a file in the project's save folder). The name may hold letters, digits, space, underscore and dash (up to 64). Returns true when the file was written.
- **Function** `bool LoadFromSlot(const FString& SlotName, int32 UserIndex = 0)`: Reads a record from a save slot and applies it as Read Save Data does. Returns false when the slot is missing or unreadable.

## SkillConnectionWidget.h

### USkillConnectionWidget

A line between two points of the tree canvas. It fills the whole canvas and draws one link. To change how links look, make a Blueprint subclass, turn off Draw Default Link, draw in its On Paint event (the end points come from Get Link Points) and pick it as the connection class of the menu.

- **Property** `bool bDrawDefaultLink`: Off: the built-in curve is not drawn, so a Blueprint skin can draw its own in On Paint.
- **Function** `void SetLink(FVector2D InFrom, FVector2D InTo, bool bInActive, const USkillSetStyle* InStyle)`: Sets the two end points (pixels inside the canvas) and whether the link is active.
- **Function** `void SetActive(bool bInActive)`: Changes only the active state (the first skill of the link was bought).
- **Function** `void GetLinkPoints(FVector2D& OutFrom, FVector2D& OutTo) const`: The two end points of the link, in pixels inside the canvas.
- **Function** `bool IsActive() const`: True when the first skill of the link is bought.

## SkillDefinition.h

### USkillDefinition

One skill: what it is called, what it costs, what it needs and what it does. Reusable across trees.

- **Property** `FName SkillId`: Stable key of the skill inside a tree. Saves, purchases and prerequisites use it. Never change it after release.
- **Property** `FText Title`: Display name.
- **Property** `FText Description`: Text shown in the details panel.
- **Property** `TSoftObjectPtr<UTexture2D> Icon`: Icon texture.
- **Property** `TArray<FSkillCost> Costs`: Points taken from named pools when the skill is bought (its first rank). Empty means free.
- **Property** `int32 MaxRank`: Highest rank. 1: the skill is bought once. More: it can be bought again to raise its rank, and its effects get the rank.
- **Property** `TArray<FSkillRankCost> HigherRankCosts`: Cost of rank 2, 3 and so on (the first entry is rank 2; Costs is rank 1). A rank without an entry costs the same as the rank before.
- **Property** `TArray<FName> Prerequisites`: Skills of the same tree that must be bought first.
- **Property** `bool bAnyPrerequisite`: Off: all prerequisites are needed. On: one of them is enough.
- **Property** `TMap<FName, int32> RequiredRanks`: Minimum rank of a prerequisite, by skill id (for example "Blade" = 3). Prerequisites not listed here need rank 1.
- **Property** `bool bStartsUnlocked`: The skill can be bought from the start even when its prerequisites or tree gate are not met.
- **Property** `int32 RequiredTreeRanks`: Tier or row gate: ranks that must be bought in the other skills of the same tree before this skill can be bought ("spend 5 points in this tree first"). 0 for none.
- **Property** `FName ExclusiveGroup`: Pick one of several: skills of a tree with the same group name exclude each other. Once one is bought the others are locked until it is refunded. Leave empty for none.
- **Property** `bool bRefundable`: Off: the skill is never refunded, by Refund or by Respec (Reset All still removes it).
- **Property** `TArray<TObjectPtr<USkillEffect>> Effects`: What the skill does while bought.
- **Property** `TMap<FName, double> Parameters`: Free numbers and tags for your own game (damage, range, a gameplay tag). The plugin does not read them.
- **Property** `FGameplayTagContainer Tags`: Gameplay tags. For the Grant Tags effect these are given while the skill is bought; on a skill they are free data for your game and the plugin does not read them.
- **Function** `TArray<FSkillCost> GetRankCost(int32 Rank) const`: Cost of one rank (1 is the first). Empty for a rank above Max Rank.

## SkillDetailsWidget.h

### USkillMouseButton

A button that takes mouse clicks but never keyboard or gamepad focus (the Refund button of the details panel).

### USkillDetailsWidget

The side panel that describes the skill under the cursor or focus: state and rank, description, what the effects do now and at the next rank, cost, what is still missing, and the refund action when a refund is allowed. To skin it, make a Blueprint subclass with widgets named IconImage, TitleText, StateText, DescriptionText, EffectsText, CostText, RequirementsText, RefundText and RefundButton (any subset) and override On View Set if you want full control.

- **Function** `void SetView(const FSkillView& InView, const USkillSetStyle* InStyle)`: Shows a skill. Pass the resolved style so colours match the rest of the menu.
- **Function** `void ClearView()`: Clears the panel (nothing selected).
- **Function** `void OnViewSet(const FSkillView& NewView)`: For Blueprint skins: called after the panel was filled.
- **Property** `FText RefundKeysHint`: Keys shown under the refund line while a refund is allowed. Change it when you use the panel with other controls; empty hides it.
- **Property** `FSkillDetailsRefundSignature OnRefundClicked`: Fired when the Refund button of the panel is clicked. The menu refunds one rank of the shown skill.
- **Function** `void DescribeView(const FSkillView& View, FText& OutState, FText& OutEffects, FText& OutCost, FText& OutRequirements, FText& OutRefund)`: The text of the panel lines for a view, without a widget: state, effects, cost, requirements and refund. Useful for your own tooltips.
- **Property** `TObjectPtr<UImage> IconImage`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> TitleText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> StateText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> DescriptionText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> CostText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> RequirementsText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> EffectsText`: Optional text with this name: what the effects do now and at the next rank.
- **Property** `TObjectPtr<UTextBlock> RefundText`: Optional text with this name: what a refund gives back, or why it is refused.
- **Property** `TObjectPtr<UButton> RefundButton`: Optional button with this name; shown only while a refund is allowed, and fires On Refund Clicked. The built-in one never takes keyboard or gamepad focus; in a skin, use Skill Mouse Button (or turn Is Focusable off) for the same.

## SkillEffect.h

### FSkillEffectContext

Who asked, and for which skill, when an effect runs.

- **Property** `TObjectPtr<USkillComponent> Component`: The component that owns the progress.
- **Property** `TObjectPtr<AActor> Owner`: The actor that owns the component (a pawn, a controller, an AI).
- **Property** `TObjectPtr<const USkillDefinition> Skill`: The skill this refers to.
- **Property** `FName TreeId`: Id of the tree.
- **Property** `FName SkillId`: Id of the skill.
- **Property** `FName SourceId`: Unique per skill and tree ("TreeId.SkillId"). Use it as the source id when you add something that must be taken back.
- **Property** `int32 Rank`: Rank this call is about: the new rank in Apply, the rank being taken back in Remove, the rank asked about in Can Apply and Describe.
- **Property** `int32 MaxRank`: Highest rank of the skill.
### USkillEffect

What a skill does. Create a Blueprint subclass and override Apply and Remove, or use the built-in effects. Effect objects are shared by every owner that uses the skill, so keep them stateless: put state on the owner. Ranks: when the rank of a bought skill changes, Change Rank runs with the new rank (Context Rank) and the old one. By default it is Remove at the old rank followed by Apply at the new one, so an effect only has to do what one rank number asks for; override it to update in place.

- **Function** `bool CanApply(const FSkillEffectContext& Context) const`: Return false to refuse the purchase before any point is spent. Default: true. This is also asked while the screen refreshes, so it must only look, never change anything.
- **Function** `void Apply(const FSkillEffectContext& Context) const`: Called once when the skill is bought, and again after a save is loaded.
- **Function** `void Remove(const FSkillEffectContext& Context) const`: Called when the skill is refunded, its rank changes, or the progress is reset or replaced. Undo exactly what Apply did for Context Rank.
- **Function** `void ChangeRank(const FSkillEffectContext& Context, int32 OldRank) const`: Called when the rank of a bought skill changes (buy, grant or refund of a rank, respec, load). Context Rank is the new rank. Default: Remove at Old Rank, then Apply. Override to update in place.
- **Function** `FText Describe(const FSkillEffectContext& Context) const`: One line for the details panel at Context Rank (for example "+20 Max Health"). Default: Display Text, else what the built-in effect says, else nothing.
- **Property** `FText DisplayText`: Optional line for the details panel; {Rank} and {MaxRank} are replaced. Leave empty to let the effect describe itself.
### USkillEffect_GrantTags

Gives the owner gameplay tags while the skill is bought. Read them with Has Granted Tag on the component.

- **Property** `FGameplayTagContainer Tags`: Gameplay tags. For the Grant Tags effect these are given while the skill is bought; on a skill they are free data for your game and the plugin does not read them.
### USkillEffect_GrantCounter

Adds an amount to a named counter on the component while the skill is bought (extra slots, extra lanes, ...).

- **Property** `FName CounterName`: Name of the counter to change.
- **Property** `int32 Amount`: Amount added to the counter while the skill is bought.
- **Property** `bool bScaleWithRank`: On: the counter gets Amount for every rank (rank 3 adds three times Amount).
### USkillEffect_SetFlag

Turns a named flag on while the skill is bought. Read it with Has Flag on the component.

- **Property** `FName FlagName`: Name of the flag to turn on.
### FSkillValueModifier

One change of a named value.

- **Property** `FName ValueName`: The name of the value on the target (for example "MaxHealth"). The target decides what names mean.
- **Property** `ESkillModifierOp Operation`: Add to the value or multiply it.
- **Property** `double Amount`: Amount to add, or the factor to multiply by.
- **Property** `bool bScaleWithRank`: On: the change grows with the rank. Add sends Amount x Rank; Multiply sends 1 + (Amount - 1) x Rank, so 1.1 is +10 % per rank.
- **Property** `TArray<double> AmountPerRank`: Optional exact amount per rank (first entry = rank 1). When set it is used instead of Amount; ranks past the end use the last entry.
- **Property** `FText DisplayName`: Name shown in the details panel (for example "Max Health"). Empty shows Value Name.
### USkillEffect_ModifyValues

Changes named numeric values on the owner through the Skill Value Target interface. The interface is looked up on the owner, its components and, for a controller, its pawn (and the reverse).

- **Property** `TArray<FSkillValueModifier> Modifiers`: The changes to make.
### USkillEffect_CallFunction

Calls a function without parameters on the owner by name when the skill is bought, and another when it is removed.

- **Property** `FName ApplyFunction`: Function called when the skill is bought.
- **Property** `FName RemoveFunction`: Function called when the skill is taken back.

## SkillInterfaces.h

### ISkillValueTarget

Implement this on a pawn, controller or component to receive numeric modifiers from skills (the Modify Value effect). The skill id is the source, so a modifier can be taken back exactly.

- **Function** `void ApplySkillModifier(FName ValueName, ESkillModifierOp Operation, double Amount, FName SourceId)`: Add or multiply the named value. SourceId identifies the skill that asked, so it can be removed exactly.
- **Function** `void RemoveSkillModifiers(FName SourceId)`: Take back every modifier that came from SourceId.
### ISkillPointSource

Implement this on anything that hands out skill points (a pickup, a quest, an NPC). Use Collect From Source on the component.

- **Function** `FName GetSkillPointPool() const`: The named pool the points go to.
- **Function** `int32 GetSkillPointAmount() const`: How many points this source gives.

## SkillLevelComponent.h

### FSkillLevelSaveData

Saved experience of one level component. The level follows from the XP.

- **Property** `int32 Version`: Format version of this record.
- **Property** `int32 XP`: Total experience.
### USkillLevelComponent

Optional experience and levels. Put it next to a Skill Component: every level gained adds points to named pools (Points Per Level), and a level lost takes them back. When the pools hold fewer points than a level down takes, every tree of the Skill Component is respecced first (at its Refund Percent), so spent points come back before they are removed; what is still short leaves the pool at zero. Points are only given while a Skill Component is found. The XP needed per level comes from First Level XP and Level XP Growth, or from an exact list.

- **Property** `int32 MaxLevel`: Highest level.
- **Property** `int32 FirstLevelXP`: XP from level 1 to level 2.
- **Property** `float LevelXPGrowth`: Each next level needs this many times the XP of the step before (1 = every level costs the same).
- **Property** `TArray<int32> XPPerLevel`: Optional exact XP per level step (first entry = level 1 to 2). Steps past the end use the formula above.
- **Property** `TMap<FName, int32> PointsPerLevel`: Points added to the Skill Component for every level gained, per pool.
- **Property** `bool bTakePointsOnLevelDown`: On: a level lost takes back the points it gave (respeccing the trees first when the pools are short).
- **Property** `FSkillLevelChangedSignature OnLevelChanged`: Broadcast when the level changes, with the old level.
- **Property** `FSkillXPChangedSignature OnXPChanged`: Broadcast when the XP changes.
- **Function** `void AddXP(int32 Amount)`: Adds experience (a negative amount takes it away). Levels gained give points; levels lost take them back.
- **Function** `void SetXP(int32 NewXP)`: Sets the total experience.
- **Function** `void SetLevel(int32 NewLevel)`: Jumps to a level (XP becomes the start of that level).
- **Function** `int32 GetLevel() const`: The current level (1 at the start).
- **Function** `int32 GetXP() const`: Total experience.
- **Function** `int32 GetXPForLevel(int32 InLevel) const`: Total XP needed to reach a level (0 for level 1).
- **Function** `int32 GetLevelForXP(int32 InXP) const`: The level a total XP amount gives.
- **Function** `float GetLevelProgress() const`: Progress from the current level to the next, 0 to 1 (1 at the highest level).
- **Function** `int32 GetXPToNextLevel() const`: XP still needed for the next level (0 at the highest level).
- **Function** `void SetSkillComponent(USkillComponent* InSkillComponent)`: The Skill Component that gets the points: the one set here, else the one on the same actor.
- **Function** `USkillComponent* GetSkillComponent() const`: The Skill Component that gets the points (null when there is none).
- **Function** `FSkillLevelSaveData WriteSaveData() const`: A versioned record of the XP. Store it next to the skill record; the points are saved by the Skill Component.
- **Function** `bool ReadSaveData(const FSkillLevelSaveData& Data)`: Restores the XP and level without giving or taking points (the Skill Component's own record holds those). Returns false for an unknown version.

## SkillMenuLibrary.h

### USkillMenuLibrary

One-call helpers to open and close the skill screen from any Blueprint, level script or controller.

- **Function** `USkillMenuWidget* OpenSkillMenu(APlayerController* PlayerController, USkillComponent* Component, USkillTree* Tree, TSubclassOf<USkillMenuWidget> MenuClass, USkillSetStyle* Style = nullptr)`: Creates the skill screen for a tree, shows it to the player and moves input to it. Menu Class can be your own Blueprint subclass of Skill Menu Widget; leave it empty for the built-in screen. Style is optional: leave it empty to use the project setting or the built-in look. Returns the widget so you can change its properties or bind its events.
- **Function** `void CloseSkillMenu(USkillMenuWidget* Menu)`: Closes a menu opened with Open Skill Menu and gives input back to the game.

## SkillMenuWidget.h

### USkillTabButton

A tab button that remembers which category it opens. Used by the menu; usable in your own skins.

- **Property** `FName CategoryId`: Id of the category (tab).
- **Property** `FSkillTabClickedSignature OnTabClicked`: Fired when the tab is clicked.
### USkillMenuWidget

The complete skill screen: title, point pools, one tab per category, the tree, the details panel and key hints. Everything is built in code from the tree data and the style asset. The widget classes of the parts are properties, so each part can be replaced with your own Blueprint subclass. Input: mouse click, or keyboard and gamepad through native focus (arrows / stick / d-pad move, Enter / accept buys). Refund one rank (when the owner allows refunds): right click a card, Backspace or Delete, or the gamepad left face button. Respec the shown tree (when Offer Respec is on): T or the gamepad top face button. Previous and next tab: Q and E, or the gamepad shoulder buttons. Close: Escape or the gamepad back button. A feedback line under the tree says what the last action did, or why it was refused.

- **Property** `TObjectPtr<USkillSetStyle> StyleAsset`: Look and sounds. Empty uses the project setting, then the built-in look.
- **Property** `FText MenuTitle`: Title at the top of the screen.
- **Property** `TArray<FName> PoolsToShow`: Pools shown in the header. Empty shows every pool that a skill of the tree costs.
- **Property** `bool bOfferRespec`: Offers the respec action on this screen (T or the gamepad top face button): every paid rank of the shown tree is given back. Off by default; the game decides when a respec is allowed.
- **Property** `float FeedbackSeconds`: Seconds the feedback line stays after an action.
- **Property** `TSubclassOf<USkillTreeWidget> TreeWidgetClass`: Replaces the widget that draws one tab of the tree.
- **Property** `TSubclassOf<USkillNodeWidget> NodeWidgetClass`: Replaces the widget that draws one skill.
- **Property** `TSubclassOf<USkillConnectionWidget> ConnectionWidgetClass`: Replaces the widget that draws a link between two skills.
- **Property** `TSubclassOf<USkillDetailsWidget> DetailsWidgetClass`: Replaces the side panel that describes a skill.
- **Property** `FSkillMenuClosedSignature OnCloseRequested`: Fired when the player asks to close (Escape, back button) or Close Menu is called.
- **Function** `void SetupMenu(USkillComponent* InComponent, USkillTree* InTree)`: Connects the menu to an owner and a tree and builds the first tab. Call once after creating the widget.
- **Function** `void SelectCategory(FName CategoryId)`: Shows another tab.
- **Function** `FName GetSelectedCategory() const`: The tab being shown (None for a tree without categories).
- **Function** `USkillTreeWidget* GetActiveTreeWidget() const`: The widget showing the current tab, or null before Setup Menu.
- **Function** `void SetStatusText(const FText& Text)`: Shows a line of text under the header (for example current stats). Empty hides it.
- **Function** `void ShowFeedback(const FText& Text)`: Shows a message on the feedback line for Feedback Seconds. The screen uses it for purchases, refunds and refusals.
- **Function** `FText GetFeedback() const`: The text on the feedback line right now (empty when it has faded).
- **Function** `ESkillRefundResult RefundFocusedSkill()`: Refunds one rank of the skill under the cursor or focus, as Backspace does.
- **Function** `bool RespecShownTree()`: Respecs the shown tree, as the respec key does (also when Offer Respec is off). Returns true when anything changed.
- **Function** `void ShowFor(APlayerController* PlayerController)`: Adds the menu to the screen of a player, switches input to the menu and focuses the first skill.
- **Function** `void CloseMenu()`: Takes the menu off the screen and gives input back to the game.

## SkillNodeWidget.h

### USkillNodeWidget

One skill on screen: icon, title, cost, rank badge and state colour, with a button that buys the skill (or its next rank). Built in code. To skin it, make a Blueprint subclass: if your widget tree has widgets named Button, IconImage, TitleText, CostText, RankText and Outline (any subset), they are used instead of the default layout. Mouse, keyboard and gamepad all use the button's native focus and click, so the gamepad accept button buys. A right click refunds one rank when the owner allows refunds. Arrow keys and the d-pad move to the nearest card in that direction by grid position.

- **Function** `void Setup(USkillComponent* InComponent, USkillTree* InTree, FName InSkillId, const USkillSetStyle* InStyle)`: Connects the node to a skill and shows its current state. Call once after creating the widget.
- **Function** `void Refresh()`: Reads the skill state again and updates colours and texts.
- **Function** `ESkillPurchaseResult TryPurchase()`: Tries to buy the skill, plays the matching sound and reports the result. This is what a click or the accept button runs.
- **Function** `ESkillRefundResult TryRefund()`: Tries to refund one rank, plays the matching sound and reports the result. This is what a right click runs.
- **Function** `FName GetSkillId() const`: The id of the skill this node shows.
- **Function** `FSkillView GetView() const`: The last state snapshot shown.
- **Function** `UButton* GetButton() const`: The button that takes mouse clicks and the keyboard or gamepad accept key. Null in a skin that has none.
- **Function** `void FocusNode()`: Puts keyboard and gamepad focus on this node.
- **Property** `FSkillNodeFocusedSignature OnNodeFocused`: Fired when the mouse hovers the node or focus moves onto it.
- **Property** `FSkillNodeAttemptSignature OnPurchaseAttempted`: Fired after every purchase attempt with its result.
- **Property** `FSkillNodeRefundSignature OnRefundAttempted`: Fired after every refund attempt with its result.
- **Function** `void OnViewUpdated(const FSkillView& NewView)`: For Blueprint skins: called after every refresh with the new snapshot.
- **Property** `TObjectPtr<UButton> Button`: Optional button with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UImage> IconImage`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> TitleText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> CostText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> RankText`: Optional text with this name: the rank badge ("2/5"), hidden for skills with one rank. The default layout creates it.
- **Property** `TObjectPtr<UBorder> Outline`: Optional border with this name, used as the focus ring. The default layout creates it.

## SkillSaveGame.h

### USkillSaveGame

A save game object that holds one skill record. Used by the slot functions of the skill component.

- **Property** `FSkillSaveData Data`: The saved points and progress.

## SkillSetSettings.h

### USkillSetSettings

Project settings for SkillSet (Project Settings > Plugins > SkillSet).

- **Property** `TSoftObjectPtr<USkillSetStyle> DefaultStyle`: Style used by the ready-made widgets when a widget has none of its own. Empty uses the built-in look.
- **Property** `FName DefaultPoolId`: The point pool the sample menu and the helper functions use when none is named.

## SkillSetStyle.h

### USkillSetStyle

Look and sound of the ready-made skill widgets: colours, sizes and sounds in one asset. Create one in the content browser (Miscellaneous > Data Asset > Skill Core Style), edit it, and assign it to the menu widget or to the project settings. Without an asset the built-in defaults are used.

- **Property** `FLinearColor BackgroundColor`: Screen backdrop.
- **Property** `FLinearColor PanelColor`: Side panel and tab strip.
- **Property** `FLinearColor NodeFillColor`: Inside colour of a skill card (state colours tint it).
- **Property** `FLinearColor PanelOutlineColor`: Thin line around panels.
- **Property** `FLinearColor TextColor`: Main text colour.
- **Property** `FLinearColor SubtleTextColor`: Secondary text colour (costs, hints).
- **Property** `FLinearColor LockedColor`: Node tint while the skill is locked.
- **Property** `FLinearColor AvailableColor`: Node tint while the skill can be bought.
- **Property** `FLinearColor UnaffordableColor`: Node tint for an available skill the owner cannot afford.
- **Property** `FLinearColor PurchasedColor`: Tint and outline of a bought skill.
- **Property** `FLinearColor FocusColor`: Outline of the node that has focus (mouse hover, keyboard or gamepad).
- **Property** `FLinearColor LineInactiveColor`: Link line between two skills when the first one is not bought.
- **Property** `FLinearColor LineActiveColor`: Link line when the first skill is bought.
- **Property** `float LineThickness`: Thickness of the link lines, in pixels.
- **Property** `float CornerRadius`: Corner radius of cards, tabs and panels, in pixels.
- **Property** `float OutlineWidth`: Width of the coloured line around a skill card.
- **Property** `float HighlightScale`: Scale of a skill card under the mouse or focus. 1 turns the effect off.
- **Property** `FVector2D CellSize`: Pixels per grid cell of the tree.
- **Property** `FVector2D NodeSize`: Size of one skill node.
- **Property** `FVector2D IconSize`: Size of the icon on a card and in the details panel.
- **Property** `FVector2D TreePadding`: Free space around the tree inside its scroll area.
- **Property** `float DetailsPanelWidth`: Width of the side panel, in pixels.
- **Property** `float MinZoom`: Smallest zoom of the tree view (1 is full size).
- **Property** `float MaxZoom`: Largest zoom of the tree view.
- **Property** `float ZoomStep`: Zoom change per mouse wheel notch or key press.
- **Property** `float StickPanSpeed`: Pixels per second the gamepad right stick pans at full tilt.
- **Property** `bool bGamepadCursor`: Gamepad cursor: the left stick moves the system cursor and the bottom face button clicks where it points. The d-pad keeps moving focus. Off by default.
- **Property** `float GamepadCursorSpeed`: Pixels per second the gamepad cursor moves at full tilt.
- **Property** `float TriggerZoomSpeed`: Zoom per second the gamepad triggers change at full pull.
- **Property** `float FitMaxZoom`: Largest zoom Fit To View uses, so a small tree is enlarged but not blown up.
- **Property** `bool bFitOnOpen`: Fit the whole tree into view when a tab opens.
- **Property** `int32 TitleFontSize`: Font size of titles.
- **Property** `int32 BodyFontSize`: Font size of body text.
- **Property** `int32 NodeFontSize`: Font size of the text on skill cards.
- **Property** `TSoftObjectPtr<USoundBase> PurchaseSound`: Played after a successful purchase. Leave empty for silence.
- **Property** `TSoftObjectPtr<USoundBase> DeniedSound`: Played when a purchase is refused (locked, too expensive).
- **Property** `TSoftObjectPtr<USoundBase> RefundSound`: Played after a successful refund. Leave empty for silence.
- **Property** `TSoftObjectPtr<USoundBase> FocusSound`: Played when focus moves to another node.
- **Function** `FLinearColor GetNodeColor(ESkillState State, bool bCanAfford) const`: Tint for a skill state. Used by the node widget and available to your own skins.

## SkillSetTypes.h

### ESkillState

State of one skill for one owner.

### ESkillPurchaseResult

Result of asking for, or attempting, a purchase.

### ESkillRefundResult

Result of asking for, or attempting, a refund.

### ESkillModifierOp

How a value modifier combines with the value it targets.

### FSkillCost

A price: an amount taken from one named point pool.

- **Property** `FName PoolId`: The named pool the points come from, for example "Points" or "Talent".
- **Property** `int32 Amount`: Points taken from the pool when the skill is bought.
### FSkillRankCost

The price of one rank of a skill.

- **Property** `TArray<FSkillCost> Costs`: Points taken from named pools for this rank. Empty means the rank is free.
### FSkillCategory

A tab of a tree. Categories are data, not an enum.

- **Property** `FName CategoryId`: Stable key referenced by the skills of the tree.
- **Property** `FText Title`: Name shown on the tab.
- **Property** `TSoftObjectPtr<UTexture2D> Icon`: Optional tab icon.
### FSkillView

Read-only snapshot of one skill for UI code.

- **Property** `FName TreeId`: Id of the tree.
- **Property** `FName SkillId`: Id of the skill.
- **Property** `FName CategoryId`: Id of the category (tab).
- **Property** `FText Title`: Display name.
- **Property** `FText Description`: Text shown in the details panel.
- **Property** `TSoftObjectPtr<UTexture2D> Icon`: Icon texture.
- **Property** `FVector2D Position`: Position in grid cells inside the category.
- **Property** `ESkillState State`: State of the skill.
- **Property** `TArray<FSkillCost> Costs`: Points the next rank costs (the first rank while the skill is not bought). Empty when the skill is free or at its highest rank.
- **Property** `int32 Rank`: Ranks bought (0 while not bought).
- **Property** `int32 MaxRank`: Highest rank of the skill (1 for a skill that is bought once).
- **Property** `int32 FreeRanks`: Ranks given by Grant Skill. They are never refunded.
- **Property** `TArray<FName> Prerequisites`: Skills that must be bought first.
- **Property** `TArray<FText> PrerequisiteTitles`: The titles of the prerequisites, in the same order, for display.
- **Property** `TArray<int32> PrerequisiteRanks`: The rank each prerequisite needs, in the same order (1 unless the skill asks for more).
- **Property** `bool bAnyPrerequisite`: True when one prerequisite is enough, false when all are needed.
- **Property** `int32 RequiredTreeRanks`: Ranks that must be bought in the other skills of the tree before this one (0 for none).
- **Property** `int32 TreeRanks`: Ranks currently bought in the other skills of the tree, to show next to Required Tree Ranks.
- **Property** `FName ExclusiveGroup`: Name of the exclusive group (None when the skill has none).
- **Property** `TArray<FText> ExclusiveWith`: Titles of the other skills of the exclusive group.
- **Property** `FText ExcludedBy`: Title of the bought skill that excludes this one; empty when none does.
- **Property** `bool bCanAfford`: True when every cost pool holds enough points for the next rank right now.
- **Property** `bool bCanPurchase`: True when a purchase (of the next rank) would succeed right now.
- **Property** `ESkillPurchaseResult PurchaseResult`: What Purchase Skill would return right now.
- **Property** `bool bCanRefund`: True when refunding one rank would succeed right now.
- **Property** `ESkillRefundResult RefundResult`: What Refund Skill Rank would return right now.
- **Property** `TArray<FSkillCost> RefundValue`: Points one refunded rank gives back (after the refund percent).
- **Property** `bool bRefundable`: False when the skill is marked not refundable (permanent).
- **Property** `FText RefundBlockedBy`: Title of a bought skill that stops the refund (Refund Result is Required By Other).
- **Property** `TArray<FText> CurrentEffects`: What the effects do at the current rank, one line per effect that describes itself. Empty while not bought.
- **Property** `TArray<FText> NextEffects`: What the effects will do at the next rank. Empty at the highest rank.
### FSkillRankSaveData

Saved rank of one skill.

- **Property** `FName SkillId`: Id of the skill.
- **Property** `int32 Rank`: Ranks bought, including given ranks.
- **Property** `int32 FreeRanks`: Ranks given by Grant Skill (never refunded).
- **Property** `TArray<FSkillRankCost> PaidCosts`: What each paid rank cost, oldest first; a refund gives this back. When it is missing, the current rank costs are used.
### FSkillTreeSaveData

Saved progress of one tree.

- **Property** `FName TreeId`: Id of the tree.
- **Property** `TArray<FName> Purchased`: Bought skills, in purchase order (of their first rank).
- **Property** `TArray<FSkillRankSaveData> Ranks`: Rank of each bought skill (format version 2). A bought skill missing here has rank 1, as in version 1 records.
- **Property** `TArray<FName> Unlocked`: Skills unlocked directly (shown as available without prerequisites).
### FSkillSaveData

Versioned save record of one skill component. Store it in any save system: it is a plain struct. Effect state (tags, counters, modifiers) is not saved; it is rebuilt by applying the bought skills on load.

- **Property** `int32 Version`: Format version of this record. Written by Write Save Data; checked and migrated by Read Save Data.
- **Property** `TMap<FName, int32> Points`: Points held per named pool.
- **Property** `TArray<FSkillTreeSaveData> Trees`: Saved progress of each tree.

## SkillTree.h

### FSkillTreeEntry

Where one skill sits in a tree.

- **Property** `TObjectPtr<USkillDefinition> Skill`: The skill this refers to.
- **Property** `FName CategoryId`: Tab the skill appears on. Leave empty when the tree has no categories.
- **Property** `FVector2D Position`: Grid cell inside the category: X to the right, Y downwards. The widgets turn cells into pixels.
### USkillTree

A skill tree: tabs and skills with positions. Any number of trees can exist; a component can use several.

- **Property** `FName TreeId`: Stable key of the tree inside saves. Never change it after release.
- **Property** `FText Title`: Display name.
- **Property** `TArray<FSkillCategory> Categories`: Tabs. Leave empty for a single flat tree.
- **Property** `TArray<FSkillTreeEntry> Entries`: The skills of the tree and where each one sits.
- **Function** `TArray<FName> GetSkillIds() const`: Skill ids in entry order.
- **Function** `bool Validate(TArray<FString>& OutErrors, TArray<FString>& OutWarnings) const`: Checks the tree: empty or duplicate ids, missing prerequisites, cycles, unknown categories, missing icons. Returns true when there are no errors. Warnings (for example a missing icon) do not make it false.

## SkillTreeWidget.h

### USkillTreeWidget

One tab of a tree on screen: a pannable, zoomable canvas with a node per skill and a line per prerequisite link. The layout comes from the grid positions in the tree asset and the cell size in the style. View controls: mouse wheel zooms around the cursor; drag empty space (or hold the right or middle button) to pan; the gamepad right stick pans, the triggers zoom and a click of the right stick fits the whole tree; Home fits the tree and plus and minus zoom. Moving focus to a skill pans it into view.

- **Function** `void Setup(USkillComponent* InComponent, USkillTree* InTree, FName InCategoryId, const USkillSetStyle* InStyle, TSubclassOf<USkillNodeWidget> NodeClass, TSubclassOf<USkillConnectionWidget> ConnectionClass)`: Builds the nodes and links of one category. Pass an empty Category Id for a tree without categories. Node Class and Connection Class replace the default widgets; leave them empty for the built-in ones.
- **Function** `void Refresh()`: Updates every node and link from the current state.
- **Function** `void FocusFirstNode()`: Puts focus on the first skill that can be bought, else on the first skill.
- **Function** `TArray<USkillNodeWidget*> GetNodes() const`: Every node widget of this tab.
- **Function** `float GetZoom() const`: Current zoom (1 is full size).
- **Function** `FVector2D GetPan() const`: Current offset of the tree inside the view, in pixels.
- **Function** `void ZoomBy(float Steps, FVector2D Pivot)`: Zooms by a number of steps (positive zooms in) keeping the point under Pivot (view pixels) where it is.
- **Function** `void PanBy(FVector2D Delta)`: Moves the tree by a number of pixels.
- **Function** `void FitToView()`: Fits the whole tree into the view and centres it. The style limits the zoom (Min Zoom, Max Zoom, Fit Max Zoom).
- **Function** `void EnsureVisible(FName SkillId)`: Pans the least amount needed to bring a skill fully into view.
- **Property** `FSkillNodeFocusedSignature OnNodeFocused`: Fired when any node of this tab is hovered or focused.
- **Property** `FSkillNodeAttemptSignature OnPurchaseAttempted`: Fired after a purchase attempt on any node of this tab.
- **Property** `FSkillNodeRefundSignature OnRefundAttempted`: Fired after a refund attempt on any node of this tab.
- **Function** `USkillNodeWidget* FindNeighbour(const USkillNodeWidget* From, EUINavigation Direction) const`: The node an arrow key or d-pad press should move to from From, by grid position; null when there is none in that direction.
- **Function** `void HandleNodeFocused(USkillNodeWidget* Node)`: The pure rule behind Find Neighbour: index of the candidate to move to from From in a direction, or INDEX_NONE. Only candidates ahead in that direction and within about 63 degrees of it count; the nearest one wins, with sideways distance weighted half, so the next column or row is reached before one further away.
- **Property** `TObjectPtr<UBorder> Viewport`: The clipped area the tree is shown in. A Blueprint skin can supply its own with this name.
- **Property** `TObjectPtr<USizeBox> CanvasBox`: The box that is moved and scaled.
- **Property** `TObjectPtr<UCanvasPanel> Canvas`: The canvas the nodes are placed on. A Blueprint skin can supply its own panel with this name.

