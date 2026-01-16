---
description: Welcome to the Gameplay Common UI user guide.
icon: display
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
metaLinks:
  alternates:
    - https://app.gitbook.com/s/yE16Xb3IemPxJWydtPOj/basics/editor
---

# Introduction to Gameplay Common UI

This document is designed for developers familiar with Blueprints in Unreal Engine who want to take the next step into C++ using this plugin.

### 1. Initial Configuration

Before writing code, we must ensure the plugin and the project communicate correctly.

#### Dependencies (Build.cs)

To use the C++ classes from this plugin in your game code, you must add the `GameplayCommonUI` module to your dependency list.

1. Open your project's `.Build.cs` file (usually in `Source/YourProject/YourProject.Build.cs`).
2. Add `"GameplayCommonUI"` and `"CommonUI"` to `PublicDependencyModuleNames`.

```c#
// Example of your .Build.cs file
PublicDependencyModuleNames.AddRange(new string[] { 
    "Core", 
    "CoreUObject", 
    "Engine", 
    "InputCore",
    "CommonUI",           // Required for UI foundation
    "GameplayCommonUI",   // The Plugin
    "GameplayTags"        // Used for Layer management
});
```

#### Viewport Configuration

Common UI requires a specific Viewport class to manage focus (Controller vs. Mouse).

1. In the Editor, go to Project Settings > Engine > General Settings.
2. Look for Game Viewport Client Class.
3. Set it to `CommonGameViewportClient`.

### 2. Understanding Core Classes (Blueprint to C++ Translation)

Here we translate concepts you already know from UMG/Blueprints into this plugin's C++ structure.

* `UGameplayActivatableWidget`: The base class for all main screens (Main Menu, Inventory, Pause, HUD). In Blueprint, this is like a standard `UserWidget` but with input management "superpowers." It knows when it is "Activated" or "Deactivated" and automatically manages cursor visibility and input routing.
* `UGameplayPrimaryLayout`: The master container holding your widgets. In Blueprint, this replaces the manual "HUD" logic where you "Add to Viewport." Instead of manual Z-Order, it uses Gameplay Tags to define layers (e.g., Game Layer, Menu Layer, Modal Layer).
* `UGameplayButtonBase`: A wrapper for `CommonButton`. It automatically handles controller focus, click sounds, and hover/selection states.

### 3. Tutorial: Creating Your First C++ Screen (Pause Menu)

We will create a simple menu. In C++, the logic stays in the code, while the visuals (positioning, colors) stay in the inherited Blueprint.

#### Step A: The Header File (.h)

```c++
// MyPauseMenuWidget.h
#pragma once

#include "CoreMinimal.h"
#include "Widgets/GameplayActivatableWidget.h"
#include "MyPauseMenuWidget.generated.h"

// Forward Declaration to minimize header inclusion
class UGameplayButtonBase;

/**
 * UMyPauseMenuWidget
 * Simple Pause Menu implementation using GameplayCommonUI.
 */
UCLASS()
class SEUPROJETO_API UMyPauseMenuWidget : public UGameplayActivatableWidget
{
    GENERATED_BODY()

protected:
   // --- Lifecycle Events ---
    
    // Called when the widget is created and BindWidget variables are linked
    virtual void NativeOnInitialized() override;

    // Called when the widget is activated and shown on screen
    virtual void NativeOnActivated() override;

    // --- UI Components (Designer Variables) ---

   // meta = (BindWidget) forces the Blueprint to have a button named "ResumeButton"
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UGameplayButtonBase> ResumeButton;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UGameplayButtonBase> QuitButton;

private:
    // --- Logic Functions ---
    
    UFUNCTION()
    void HandleResumeClicked();

    UFUNCTION()
    void HandleQuitClicked();
};
```

#### Step B: The Implementation File (.cpp)

```c++
// MyPauseMenuWidget.cpp
#include "MyPauseMenuWidget.h"
#include "Widgets/GameplayButtonBase.h"

void UMyPauseMenuWidget::NativeOnInitialized()
{
    // Always call the Super function!
    Super::NativeOnInitialized();
    
    // Binding the click events to our functions
    if (ResumeButton)
    {
        ResumeButton->OnClicked().AddUObject(this, &UMyPauseMenuWidget::HandleResumeClicked);
    }

    if (QuitButton)
    {
        QuitButton->OnClicked().AddUObject(this, &UMyPauseMenuWidget::HandleQuitClicked);
    }
}

void UMyPauseMenuWidget::NativeOnActivated()
{
    Super::NativeOnActivated();
    
    // You can trigger Gameplay Cues here or pause game logic.
    // The GameplayActivatableWidget handles Input Mode automatically based on 
    // the "Input Config" set in the Blueprint defaults.
}

void UMyPauseMenuWidget::HandleResumeClicked()
{
    // Deactivates this widget. CommonUI removes it from the stack and returns focus to the game.
    DeactivateWidget();
}

void UMyPauseMenuWidget::HandleQuitClicked()
{
    // Example: Quit the game. In production, you might push a "Confirmation Dialog" here.
    UKismetSystemLibrary::QuitGame(this, GetOwningPlayer(), EQuitPreference::Quit, false);
}
```

#### Step C: Creating the Blueprint

1. In the Unreal Editor, create a new Widget Blueprint.
2. Select `MyPauseMenuWidget` as the Parent Class.
3. In the Designer, add two buttons. Use the `GameplayButtonBase` class.
4. IMPORTANT: Rename the buttons in the Hierarchy panel to `ResumeButton` and `QuitButton` to match the C++ `BindWidget` names.

### 4. Pro Tip: Async Pushing (Avoiding Hard References)

Loading heavy UI classes can cause frame hitches. `GameplayCommonUI` provides async helpers to load and display widgets without blocking the main thread.

```cpp
#include "Async/GameplayPushActivatableWidgetAsync.h"

// Inside a function in your PlayerController or HUD class:
void AMyPlayerController::OpenPauseMenu(TSoftClassPtr<UGameplayActivatableWidget> PauseMenuSoftClass)
{
    // Define which Layer this widget belongs to using Gameplay Tags
    FGameplayTag LayerTag = FGameplayTag::RequestGameplayTag(FName("UI.Layer.Menu"));

    // Starts asynchronous loading and displays the widget once it is ready
    UGameplayPushActivatableWidgetAsync::PushActivatableWidgetClassToLayer(
        this,                  // World Context Object
        PauseMenuSoftClass,    // Soft reference to the Widget Class            
        LayerTag               // Target UI Layer
    );
}
```
