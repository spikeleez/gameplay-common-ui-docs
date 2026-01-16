---
description: >-
  In this module, we will explore the interfaces that come with the plugin and
  how to create your own to make the code cleaner.
icon: tower-cell
---

# Interfaces e Comunicação Desacoplada

### 1. IGameplaySettingActionInterface

Location: `Interfaces/GameplaySettingActionInterface.h`

Not every setting is a simple value like a Slider or a Combobox. Some settings are Actions—for example, "Calibrate HDR," "View Credits," or "Reset to Default."

If you create a custom widget for a specific setting (e.g., a specialized button for calibration), it should implement this interface to communicate seamlessly with the Settings system.

Core Functions:

* `ExecuteAction()`: Triggered when the player confirms the action (e.g., clicks the button).
* `IsActionEnabled()`: Allows you to disable the button dynamically (e.g., graying out the "Reset" button if settings are already at their default values).

### 2. IGameplayCommonSettingsInterface

Location: `Interfaces/GameplayCommonSettingsInterface.h`

This interface is the "glue" that allows the Registry to remain decoupled from your specific game classes. The Registry doesn't need to know the exact name of your Local Player or Settings class; it only needs to know that "someone" can provide the required data.

As we saw in the Dynamic Settings module, the macros rely on this interface to find the correct data paths.

Core Functions:

* `GetLocalSettings()`: Returns a reference to the `UGameplaySettingsLocal` (or its child class).
* `GetSharedSettings()`: Returns a reference to the `UGameplaySettingsShared` (or its child class).

#### Why this is Essential for Production

By implementing `IGameplayCommonSettingsInterface` in your `ULocalPlayer` (as shown in the previous module), you ensure that the system is modular. If you ever decide to change how settings are stored or accessed, you only need to update the interface implementation, and the rest of your UI and Registry logic will remain untouched.

***

#### Summary of the Settings Architecture

With these pieces in place, your C++ settings architecture follows a clean, professional flow:

1. Interface: Provides a standard way to access settings via the `LocalPlayer`.
2. Shared/Local Classes: Store and apply the actual values.
3. Registry: Lists the settings and connects them to the classes using [Dynamic Macros](dynamic-settings.md).
4. UI: Automatically generates the entries and handles player input.
