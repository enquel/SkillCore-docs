---
layout: default
title: SkillCore API reference
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

---


Generated from the doc comments of the public headers (`Source/SkillCore/Public`). The same text is the tooltip of the Blueprint node.

## SkillComponent.h


### USkillComponent

Holds one owner's skill progress: named point pools, the state of every skill in its trees, and the effect state skills produce (tags, counters). Put it on a pawn, a controller or an AI actor. Single-player correct; the state is plain structs, so your own replication code can carry it.

- **Property** `TArray<TObjectPtr<USkillTree>> Trees`: Trees this owner can use from the start. More can be added at runtime with Add Tree.
- **Property** `TMap<FName, int32> InitialPoints`: Points given once at begin play, per named pool.
- **Property** `FSkillPurchasedSignature OnSkillPurchased`: Broadcast after a skill was bought or granted and its effects applied. Not broadcast when a save is loaded; use On Skill State Changed for that.
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
- **Function** `ESkillPurchaseResult CanPurchaseSkill(const USkillTree* Tree, FName SkillId) const`: What Purchase Skill would return right now, without changing anything.
- **Function** `ESkillPurchaseResult PurchaseSkill(USkillTree* Tree, FName SkillId)`: Buys a skill: checks prerequisites and points, spends the cost, applies the effects.
- **Function** `bool UnlockSkill(USkillTree* Tree, FName SkillId)`: Makes a skill available without its prerequisites. A bought skill is never changed. Returns true when the state changed.
- **Function** `bool GrantSkill(USkillTree* Tree, FName SkillId)`: Gives a skill for free (no cost, no prerequisites) and applies its effects. Returns false when unknown or already bought.
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
- **Function** `void SetLink(FVector2D InFrom, FVector2D InTo, bool bInActive, const USkillCoreStyle* InStyle)`: Sets the two end points (pixels inside the canvas) and whether the link is active.
- **Function** `void SetActive(bool bInActive)`: Changes only the active state (the first skill of the link was bought).
- **Function** `void GetLinkPoints(FVector2D& OutFrom, FVector2D& OutTo) const`: The two end points of the link, in pixels inside the canvas.
- **Function** `bool IsActive() const`: True when the first skill of the link is bought.

## SkillCoreSettings.h


### USkillCoreSettings

Project settings for SkillCore (Project Settings > Plugins > SkillCore).

- **Property** `TSoftObjectPtr<USkillCoreStyle> DefaultStyle`: Style used by the ready-made widgets when a widget has none of its own. Empty uses the built-in look.
- **Property** `FName DefaultPoolId`: The point pool the sample menu and the helper functions use when none is named.

## SkillCoreStyle.h


### USkillCoreStyle

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
- **Property** `TSoftObjectPtr<USoundBase> FocusSound`: Played when focus moves to another node.
- **Function** `FLinearColor GetNodeColor(ESkillState State, bool bCanAfford) const`: Tint for a skill state. Used by the node widget and available to your own skins.

## SkillCoreTypes.h


### ESkillState

State of one skill for one owner.


### ESkillPurchaseResult

Result of asking for, or attempting, a purchase.


### ESkillModifierOp

How a value modifier combines with the value it targets.


### FSkillCost

A price: an amount taken from one named point pool.

- **Property** `FName PoolId`: The named pool the points come from, for example "Points" or "Talent".
- **Property** `int32 Amount`: Points taken from the pool when the skill is bought.

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
- **Property** `TArray<FSkillCost> Costs`: Points the skill costs.
- **Property** `TArray<FName> Prerequisites`: Skills that must be bought first.
- **Property** `TArray<FText> PrerequisiteTitles`: The titles of the prerequisites, in the same order, for display.
- **Property** `bool bCanAfford`: True when every cost pool holds enough points right now.
- **Property** `bool bCanPurchase`: True when a purchase would succeed right now.

### FSkillTreeSaveData

Saved progress of one tree.

- **Property** `FName TreeId`: Id of the tree.
- **Property** `TArray<FName> Purchased`: Bought skills, in purchase order.
- **Property** `TArray<FName> Unlocked`: Skills unlocked directly (shown as available without prerequisites).

### FSkillSaveData

Versioned save record of one skill component. Store it in any save system: it is a plain struct. Effect state (tags, counters, modifiers) is not saved; it is rebuilt by applying the bought skills on load.

- **Property** `int32 Version`: Format version of this record. Written by Write Save Data; checked and migrated by Read Save Data.
- **Property** `TMap<FName, int32> Points`: Points held per named pool.
- **Property** `TArray<FSkillTreeSaveData> Trees`: Saved progress of each tree.

## SkillDefinition.h


### USkillDefinition

One skill: what it is called, what it costs, what it needs and what it does. Reusable across trees.

- **Property** `FName SkillId`: Stable key of the skill inside a tree. Saves, purchases and prerequisites use it. Never change it after release.
- **Property** `FText Title`: Display name.
- **Property** `FText Description`: Text shown in the details panel.
- **Property** `TSoftObjectPtr<UTexture2D> Icon`: Icon texture.
- **Property** `TArray<FSkillCost> Costs`: Points taken from named pools when the skill is bought. Empty means free.
- **Property** `TArray<FName> Prerequisites`: Skills of the same tree that must be bought first.
- **Property** `bool bAnyPrerequisite`: Off: all prerequisites are needed. On: one of them is enough.
- **Property** `bool bStartsUnlocked`: The skill can be bought from the start even when its prerequisites are missing.
- **Property** `TArray<TObjectPtr<USkillEffect>> Effects`: What the skill does while bought.
- **Property** `TMap<FName, double> Parameters`: Free numbers and tags for your own game (damage, range, a gameplay tag). The plugin does not read them.
- **Property** `FGameplayTagContainer Tags`: Gameplay tags. For the Grant Tags effect these are given while the skill is bought; on a skill they are free data for your game and the plugin does not read them.

## SkillDetailsWidget.h


### USkillDetailsWidget

The side panel that describes the skill under the cursor or focus. To skin it, make a Blueprint subclass with widgets named IconImage, TitleText, StateText, DescriptionText, CostText and RequirementsText (any subset) and override On View Set if you want full control.

- **Function** `void SetView(const FSkillView& InView, const USkillCoreStyle* InStyle)`: Shows a skill. Pass the resolved style so colours match the rest of the menu.
- **Function** `void ClearView()`: Clears the panel (nothing selected).
- **Function** `void OnViewSet(const FSkillView& NewView)`: For Blueprint skins: called after the panel was filled.
- **Property** `TObjectPtr<UImage> IconImage`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> TitleText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> StateText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> DescriptionText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> CostText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> RequirementsText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.

## SkillEffect.h


### FSkillEffectContext

Who asked, and for which skill, when an effect runs.

- **Property** `TObjectPtr<USkillComponent> Component`: The component that owns the progress.
- **Property** `TObjectPtr<AActor> Owner`: The actor that owns the component (a pawn, a controller, an AI).
- **Property** `TObjectPtr<const USkillDefinition> Skill`: The skill this refers to.
- **Property** `FName TreeId`: Id of the tree.
- **Property** `FName SkillId`: Id of the skill.
- **Property** `FName SourceId`: Unique per skill and tree ("TreeId.SkillId"). Use it as the source id when you add something that must be taken back.

### USkillEffect

What a skill does. Create a Blueprint subclass and override Apply and Remove, or use the built-in effects. Effect objects are shared by every owner that uses the skill, so keep them stateless: put state on the owner.

- **Function** `bool CanApply(const FSkillEffectContext& Context) const`: Return false to refuse the purchase before any point is spent. Default: true. This is also asked while the screen refreshes, so it must only look, never change anything.
- **Function** `void Apply(const FSkillEffectContext& Context) const`: Called once when the skill is bought, and again after a save is loaded.
- **Function** `void Remove(const FSkillEffectContext& Context) const`: Called when the progress is reset or replaced. Undo exactly what Apply did.

### USkillEffect_GrantTags

Gives the owner gameplay tags while the skill is bought. Read them with Has Granted Tag on the component.

- **Property** `FGameplayTagContainer Tags`: Gameplay tags. For the Grant Tags effect these are given while the skill is bought; on a skill they are free data for your game and the plugin does not read them.

### USkillEffect_GrantCounter

Adds an amount to a named counter on the component while the skill is bought (extra slots, extra lanes, ...).

- **Property** `FName CounterName`: Name of the counter to change.
- **Property** `int32 Amount`: Amount added to the counter while the skill is bought.

### USkillEffect_SetFlag

Turns a named flag on while the skill is bought. Read it with Has Flag on the component.

- **Property** `FName FlagName`: Name of the flag to turn on.

### FSkillValueModifier

One change of a named value.

- **Property** `FName ValueName`: The name of the value on the target (for example "MaxHealth"). The target decides what names mean.
- **Property** `ESkillModifierOp Operation`: Add to the value or multiply it.
- **Property** `double Amount`: Amount to add, or the factor to multiply by.

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

## SkillMenuLibrary.h


### USkillMenuLibrary

One-call helpers to open and close the skill screen from any Blueprint, level script or controller.

- **Function** `static USkillMenuWidget* OpenSkillMenu(APlayerController* PlayerController, USkillComponent* Component, USkillTree* Tree, TSubclassOf<USkillMenuWidget> MenuClass, USkillCoreStyle* Style = nullptr)`: Creates the skill screen for a tree, shows it to the player and moves input to it. Menu Class can be your own Blueprint subclass of Skill Menu Widget; leave it empty for the built-in screen. Style is optional: leave it empty to use the project setting or the built-in look. Returns the widget so you can change its properties or bind its events.
- **Function** `void CloseSkillMenu(USkillMenuWidget* Menu)`: Closes a menu opened with Open Skill Menu and gives input back to the game.

## SkillMenuWidget.h


### USkillTabButton

A tab button that remembers which category it opens. Used by the menu; usable in your own skins.

- **Property** `FName CategoryId`: Id of the category (tab).
- **Property** `FSkillTabClickedSignature OnTabClicked`: Fired when the tab is clicked.

### USkillMenuWidget

The complete skill screen: title, point pools, one tab per category, the tree, the details panel and key hints. Everything is built in code from the tree data and the style asset. The widget classes of the parts are properties, so each part can be replaced with your own Blueprint subclass. Input: mouse click, or keyboard and gamepad through native focus (arrows / stick / d-pad move, Enter / accept buys). Previous and next tab: Q and E, or the gamepad shoulder buttons. Close: Escape or the gamepad back button.

- **Property** `TObjectPtr<USkillCoreStyle> StyleAsset`: Look and sounds. Empty uses the project setting, then the built-in look.
- **Property** `FText MenuTitle`: Title at the top of the screen.
- **Property** `TArray<FName> PoolsToShow`: Pools shown in the header. Empty shows every pool that a skill of the tree costs.
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
- **Function** `void ShowFor(APlayerController* PlayerController)`: Adds the menu to the screen of a player, switches input to the menu and focuses the first skill.
- **Function** `void CloseMenu()`: Takes the menu off the screen and gives input back to the game.

## SkillNodeWidget.h


### USkillNodeWidget

One skill on screen: icon, title, cost and state colour, with a button that buys the skill. Built in code. To skin it, make a Blueprint subclass: if your widget tree has widgets named Button, IconImage, TitleText, CostText and Outline (any subset), they are used instead of the default layout. Mouse, keyboard and gamepad all use the button's native focus and click, so the gamepad accept button buys.

- **Function** `void Setup(USkillComponent* InComponent, USkillTree* InTree, FName InSkillId, const USkillCoreStyle* InStyle)`: Connects the node to a skill and shows its current state. Call once after creating the widget.
- **Function** `void Refresh()`: Reads the skill state again and updates colours and texts.
- **Function** `ESkillPurchaseResult TryPurchase()`: Tries to buy the skill, plays the matching sound and reports the result. This is what a click or the accept button runs.
- **Function** `FName GetSkillId() const`: The id of the skill this node shows.
- **Function** `FSkillView GetView() const`: The last state snapshot shown.
- **Function** `UButton* GetButton() const`: The button that takes mouse clicks and the keyboard or gamepad accept key. Null in a skin that has none.
- **Function** `void FocusNode()`: Puts keyboard and gamepad focus on this node.
- **Property** `FSkillNodeFocusedSignature OnNodeFocused`: Fired when the mouse hovers the node or focus moves onto it.
- **Property** `FSkillNodeAttemptSignature OnPurchaseAttempted`: Fired after every purchase attempt with its result.
- **Function** `void OnViewUpdated(const FSkillView& NewView)`: For Blueprint skins: called after every refresh with the new snapshot.
- **Property** `TObjectPtr<UButton> Button`: Optional button with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UImage> IconImage`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> TitleText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UTextBlock> CostText`: Optional widget with this name. The default layout creates it; a Blueprint skin can provide its own.
- **Property** `TObjectPtr<UBorder> Outline`: Optional border with this name, used as the focus ring. The default layout creates it.

## SkillSaveGame.h


### USkillSaveGame

A save game object that holds one skill record. Used by the slot functions of the skill component.

- **Property** `FSkillSaveData Data`: The saved points and progress.

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

- **Function** `void Setup(USkillComponent* InComponent, USkillTree* InTree, FName InCategoryId, const USkillCoreStyle* InStyle, TSubclassOf<USkillNodeWidget> NodeClass, TSubclassOf<USkillConnectionWidget> ConnectionClass)`: Builds the nodes and links of one category. Pass an empty Category Id for a tree without categories. Node Class and Connection Class replace the default widgets; leave them empty for the built-in ones.
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
- **Property** `TObjectPtr<UBorder> Viewport`: The clipped area the tree is shown in. A Blueprint skin can supply its own with this name.
- **Property** `TObjectPtr<USizeBox> CanvasBox`: The box that is moved and scaled.
- **Property** `TObjectPtr<UCanvasPanel> Canvas`: The canvas the nodes are placed on. A Blueprint skin can supply its own panel with this name.

