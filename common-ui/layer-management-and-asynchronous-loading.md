---
description: Forget AddToViewport and Z-Order
icon: layer-group
---

# Layer Management and Asynchronous Loading

In Common UI, we don't just "throw" widgets onto the screen; we stack them in semantically organized Layers. In this module, we will explore the architecture behind your UI and how to navigate between screens without causing frame hitches.

### 1. The Architecture: The UI Skeleton

For `PushActivatableWidget` to work, there must be a structure already present on the screen. This structure is the Primary Game Layout.

#### The `UGameplayPrimaryLayout`

* What it is: The Root Widget of your game. It is automatically created by the `GameplayCommonUIPolicy` when the player logs in.
* Function: It acts as a container. It has no visual elements of its own (usually transparent) but contains several Stacks of widgets.
* Analogy: Think of it as an empty bookshelf. The Layers are the shelves.

#### The Layers (Stacks)

Inside the `UGameplayPrimaryLayout`, we use widgets of type `CommonActivatableWidgetContainerBase` (or Stacks). Each Stack represents a layer identified by a Gameplay Tag.

#### Typical Structure (Back to Front):

1. Game Layer (`UI.Layer.Game`): HUD, crosshairs, health bars.
2. GameMenu Layer (`UI.Layer.GameMenu`): Inventories or windows that allow simultaneous mouse and keyboard/gamepad use.
3. Menu Layer (`UI.Layer.Menu`): Pause menu, Settings. (Blocks Game input).
4. Modal Layer (`UI.Layer.Modal`): Confirmation pop-ups, errors, notifications. (Blocks all layers below).

### 2. The Tool: `UGameplayPushActivatableWidgetAsync`

This is the "magic" class you will use 90% of the time to open menus.

#### The Problem: Hard References

If your HUD contains a `TSubclassOf<UUserWidget>` variable for a "Pause Menu," Unreal loads that menu (and all its textures) into memory as soon as the HUD is created. If the menu is heavy, your game will hitch/freeze upon starting.

#### The Solution: Async Loading

`UGameplayPushActivatableWidgetAsync` uses `TSoftClassPtr`. It essentially says: _"I know where the menu file is, but I will only load it when you explicitly ask for it."_

The Execution Flow:

1. Request: You call: "Open `WBP_PauseMenu` on layer `UI.Layer.Menu`."
2. Loading: The task checks if the class is in memory. If not, it starts a Streamable Load in the background.
3. Discovery: While loading, the system finds the Local Player's `UGameplayGameLayout`.
4. Layer Lookup: The Layout checks its internal map: _"Which widget stack is responsible for the tag `UI.Layer.Menu`?"_
5. Push: Once loaded, the widget is instantiated and "pushed" into that specific Stack.
6. Activation: Common UI activates the widget, giving it focus and disabling inputs for lower layers.

### 3. Practical Implementation

#### In Blueprints

The plugin exposes a powerful latent node called Push Activatable Widget Async.

* Activatable Widget Class: Select your Widget Blueprint (notice the pin color is light pink, indicating a Soft Reference).
* Layer Tag: Select the destination (e.g., `UI.Layer.Menu`).

Events:

* Before Push: Perfect for setting variables on the widget instance before it appears (e.g., passing item data to an inventory slot).
* After Push: The widget is now on screen and active.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**Push Activatable Widget for Tag:** This task performs a Push of a Widget by its registration tag, and adds it to the Stack based on the `LayerTag`.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
To register a widget with a tag, you need to go to **Project Settings > Game > Gameplay Common UI Settings > Registered Activatable Widgets**.
{% endhint %}

#### In C++

* If you are in a class with access to the `PlayerController`, use the async task directly for an optimized, production-ready flow.
* **Push Activatable Widget for Class:** This task performs a Push of a widget by its soft class, and adds it to the Stack based on the `LayerTag`.

If you are in a class that has access to PlayerController, use the static library UGameplayCommonUILibrary or the task directly.

```c++
#include "Async/GameplayPushActivatableWidgetAsync.h"
#include "GameplayTagsManager.h"

void AMyPlayerController::OpenInventory()
{
    TSoftClassPtr<UCommonActivatableWidget> InventoryWidgetClass = nullptr;
		
		UGameplayPushActivatableWidgetAsync* Task = UGameplayPushActivatableWidgetAsync::PushActivatableWidgetClassToLayer(
		   this, 
		   InventoryWidgetClass,
		   GameplayCommonTags::Layer_GameMenu, 
		   true
		 );
}
```

### 4. Setting up the `GameplayPrimaryLayout` (One-Time Setup)

For the system to function, your Layout must be configured once:

1. Create the Blueprint: Create a Widget Blueprint inheriting from `GameplayPrimaryLayout` (e.g., `WBP_PrimaryLayout`).
2. Add Stacks: In the Designer, add widgets of type `Common Activatable Widget Stack`.
3. Register Layers: \* In the Construct event of your `WBP_PrimaryLayout`, call Register Layer for each stack.
   * Map `GameStack` -> `UI.Layer.Game`.
   * Map `MenuStack` -> `UI.Layer.Menu`.
4. Set the Policy: \* The plugin includes a default `PrimaryLayout`.
   * If you want a custom one, go to Project Settings > Game > Gameplay Common UI Settings.
   * Set your `UIPolicyClass` and ensure it points to your custom `WBP_PrimaryLayout`.

{% hint style="info" %}
The plugin already has a `GameplayPrimaryLayout` created and called in the PrimaryLayout Blueprint. By default, `GameplayCommonUIPolicy` automatically starts this widget for you. If you want to change it to a custom one, you must first create a custom `GameplayCommonUIPolicy` and define it in **Project Settings > Game > Gameplay Common UI Settings > UIPolicyClass**.
{% endhint %}
