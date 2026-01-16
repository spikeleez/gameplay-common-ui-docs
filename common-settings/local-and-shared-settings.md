---
description: In the modern world of gaming, not all configurations are created equal.
icon: folder-gear
---

# Local & Shared Settings

This module explains the architectural separation between machine-specific configurations and player-specific preferences. Understanding this distinction is crucial for delivering a professional experience across different hardware (like a 4K PC vs. a Steam Deck).

### 1. GameplaySettingsLocal (The Hardware Manager)

This class handles everything tied to the specific machine where the game is running.

* Base Class: `UGameplaySettingsLocal`
* Storage: Usually saved in a local `.ini` file (`GameUserSettings.ini`).
* Responsibility: Graphics, Resolution, Window Mode, Performance.

#### Creating the Class (`MySettingsLocal.h`)

You must inherit from `UGameplaySettingsLocal`. This is where you implement the functions that your [Dynamic Macros](dynamic-settings.md) will call.

```c++
// MySettingsLocal.h
#pragma once

#include "Framework/GameplaySettingsLocal.h"
#include "MySettingsLocal.generated.h"

/**
 * UMySettingsLocal
 * Manages machine-specific settings like graphics and performance.
 */
UCLASS()
class UMySettingsLocal : public UGameplaySettingsLocal
{
    GENERATED_BODY()

public:
    UMySettingsLocal();

    // Getter and Setter for the Registry to see via macros
    UFUNCTION()
    bool GetFullscreenMode() const;

    UFUNCTION()
    void SetFullscreenMode(bool bIsFullscreen);

    // Standard engine function to apply non-resolution changes
    virtual void ApplyNonResolutionSettings() override;
};
```

**Implementando a Lógica (MySettingsLocal.cpp)**

Aqui nós conversamos com o `GEngine->GameUserSettings` real.

```c++
// MySettingsLocal.cpp
#include "MySettingsLocal.h"
#include "GameFramework/GameUserSettings.h"

UMySettingsLocal::UMySettingsLocal()
{
    // Constructor logic if needed
}

bool UMySettingsLocal::GetFullscreenMode() const
{
    // Reading directly from the engine's core system
    return GetGameUserSettings()->GetFullscreenMode() == EWindowMode::Fullscreen;
}

void UMySettingsLocal::SetFullscreenMode(bool bIsFullscreen)
{
    EWindowMode::Type NewMode = bIsFullscreen ? EWindowMode::Fullscreen : EWindowMode::Windowed;
    GetGameUserSettings()->SetFullscreenMode(NewMode);
    
    // Optional: Apply immediately for instant visual feedback
    GetGameUserSettings()->ApplyResolutionSettings(false);
}
```

### 2. GameplaySettingsShared (The Player Profile)

This class manages personal preferences. It behaves similarly to a standard `SaveGame`.

* Base Class: `UGameplaySettingsShared`
* Storage: Saved in a `.sav` file (Save Slot) or the Cloud.
* Responsibility: Gameplay, Audio, Accessibility, Keybindings, Invert Y-Axis.

#### Creating the Class (`MySettingsShared.h`)

```c++
// MySettingsShared.h
#pragma once

#include "Framework/GameplaySettingsShared.h"
#include "MySettingsShared.generated.h"

UCLASS()
class UMySettingsShared : public UGameplaySettingsShared
{
    GENERATED_BODY()

public:
    // Example: Does the player want subtitles?
    UPROPERTY(Config)
    bool bSubtitlesEnabled = true;

    // Dynamic Registry accessors
    UFUNCTION()
    bool GetSubtitlesEnabled() const { return bSubtitlesEnabled; }

    UFUNCTION()
    void SetSubtitlesEnabled(bool bNewValue);
    
    // Called when settings are loaded or changed. This is where logic is triggered!
    virtual void ApplySettings() override;
};
```

#### Implementing the Logic (`MySettingsShared.cpp`)

```c++
// MySettingsShared.cpp
#include "MySettingsShared.h"
#include "MySubtitlesSystem.h" // Exemplo fictício

void UMySettingsShared::SetSubtitlesEnabled(bool bNewValue)
{
    if (bSubtitlesEnabled != bNewValue)
    {
        bSubtitlesEnabled = bNewValue;
        ApplySettings(); // Apply settings
        SaveSettings();  // Persists to disk/cloud
    }
}

void UMySettingsShared::ApplySettings()
{
    Super::ApplySettings();

    if (UMySubtitlesSystem* Subs = GetWorld()->GetSubsystem<UMySubtitlesSystem>())
    {
        Subs->SetEnabled(bSubtitlesEnabled);
    }
}
```

### 3. Connecting Everything in the Local Player

The Registry finds these classes by asking the `LocalPlayer`. You must create a custom `LocalPlayer` class and initialize these objects there.

#### `MyLocalPlayer.h`

```c++
UCLASS()
class UMyLocalPlayer : public ULocalPlayer, public IGameplayCommonSettingsInterface
{
    GENERATED_BODY()

public:
    // Interface overrides used by the Registry Macros
    UFUNCTION()
    UMySettingsLocal* GetLocalSettings() const override;

    UFUNCTION()
    UMySettingsShared* GetSharedSettings() const override;

protected:
    virtual void PostInitProperties() override;

private:
    UPROPERTY(Transient)
    TObjectPtr<UMySettingsLocal> LocalSettings;

    UPROPERTY(Transient)
    TObjectPtr<UMySettingsShared> SharedSettings;
};
```

{% hint style="warning" %}
Editor Configuration: 1. Go to **Project Settings > Engine > General Settings**. 2. Change Local Player Class to `UMyLocalPlayer`. 3. Change Game User Settings Class to `UMySettingsLocal`.
{% endhint %}

### 4. Full Flow Summary

1. UI (Widget): User moves a Slider.
2. **Registry (Dynamic)**: The `GameplaySettingValueScalarDynamic` receives the event.
3. **Macro: Uses**: `GET_LOCAL_SETTINGS_FUNCTION_PATH` to find the target.
4. **LocalPlayer**: The macro accesses `UMyLocalPlayer` and calls `GetLocalSettings()`.
5. **SettingsLocal**: The function `SetIsFullscreenEnabled` is executed.
6. **Engine**: Unreal's `GameUserSettings` updates the resolution or window mode.

While the initial setup involves several steps, adding a new setting later is trivial: add the variable to Shared/Local, create the Getter/Setter, and register them in the Registry.
