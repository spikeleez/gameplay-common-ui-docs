---
description: Welcome to Gameplay Common Settings.
icon: gear-complex
---

# Introduction to Gameplay Common Settings

If you have ever tried to create a "Graphics Options" menu by manually connecting Sliders to `GameUserSettings` functions in Blueprint, you know how quickly it turns into "spaghetti code."

The `GameplayCommonSettings` module proposes a radical shift: You define the settings in C++, and the UI builds itself automatically.

### 1. The Concept: What is it and Why Use it?

#### The Old Way (Traditional Blueprint/UMG)

* You drag a Slider onto the screen.
* In Event Construct, you read the current value and adjust the slider position.
* In OnValueChanged, you apply the change to the engine.
* You repeat this 50 times for Brightness, Volume, Sensitivity, Texture Quality, etc.
* If you change the slider design, you have to edit 50 individual widgets.

#### The Gameplay Common Settings Way

* You create a C++ class called a Registry.
* In it, you list the options: "There is a setting called 'Master Volume', it ranges from 0 to 100."
* The UI Widget reads this list and automatically generates a row for each setting.
* Result: Logic is separated from Visuals. Zero UI maintenance.

### 2. The Architecture: Pieces of the Puzzle

To understand the code, we need to understand the hierarchy of the classes found in the GameplayCommonSettings folder.

#### A. The Manager: `UGameplaySettingsLocal`

* File: `GameplaySettingsLocal.h`
* Function: It is a glorified "SaveGame." It lives within the `GameInstance` (or a Subsystem) and stores the values the player chose. It knows how to save to disk and load when the game opens.

#### B. The Menu: `UGameplaySettingRegistry`

* File: `GameplaySettingRegistry.h`
* Function: This is where the magic happens. It is the class where you "register" all available options in your game.
* Analogy: Think of it as a Restaurant Menu. It isn't the food itself; it is the list of everything available to order.

#### C. The Categories: `UGameplaySettingCollection`

* File: `GameplaySettingCollection.h`
* Function: Groups settings together (e.g., "Video," "Audio," "Controls").

#### D. The Individual Item: `UGameplaySetting`

* File: `GameplaySetting.h`
* Function: A single option. It can be:
  * `UGameplaySettingValueScalar`: A number (Slider/Bar). Ex: Volume, Sensitivity.
  * `UGameplaySettingValueDiscrete`: A list of options (Combobox/Rotator). Ex: Resolution (1080p, 4K), Quality (Low, Medium, High).

### 3. How does the Data Flow?

When the player opens the Settings screen:

1. Initialization: The `GameplaySettingScreen` widget asks the Registry: "Give me the list of settings."
2. Regeneration: The Registry checks the current environment (e.g., "Am I on PC or PS5?") and returns only valid options (e.g., hiding "Graphics Quality" on consoles if desired).
3. Visualization: The Widget uses a ListView to create a visual row for each `UGameplaySetting`.
4. Interaction: When the player moves a slider, the Widget notifies the `UGameplaySetting`.
5. Application: The `UGameplaySetting` applies the change immediately (e.g., changing the `AudioMixer` volume) and tells `UGameplaySettingsLocal` to mark the data as "dirty" (needing to be saved).

### 4. First Steps in Code

To begin, you must create your own Registry class inheriting from `UGameplaySettingRegistry`.

#### Header Example (`MyGameSettingRegistry.h`):

```c++
#include "Framework/GameplaySettingRegistry.h"
#include "MyGameSettingRegistry.generated.h"

/**
 * UMyGameSettingRegistry
 * Manages the registration of all in-game settings.
 */
UCLASS()
class UMyGameSettingRegistry : public UGameplaySettingRegistry
{
    GENERATED_BODY()

protected:
    /** Main initialization function to override */
    virtual void OnInitialize(UGameplaySettingsLocal* InLocalSettings) override;
};
```

#### Implementation Example (`MyGameSettingRegistry.cpp`):

```c++
void UMyGameSettingRegistry::OnInitialize(UGameplaySettingsLocal* InLocalSettings)
{
    // 1. Create a Category (Collection)
    UGameplaySettingCollection* AudioPageCollection = NewObject<UGameplaySettingCollection>(this);
    AudioPageCollection->SetDevName(TEXT("AudioCollection"));
    AudioPageCollection->SetDisplayName(NSLOCTEXT("MyGame", "Audio", "Audio"));
    
    // 2. Register the collection into the registry
    RegisterSetting(AudioPageCollection);
}
```

### 5. The Role of Special Widgets

The files `GameplaySettingListView.cpp` and `GameplaySettingListEntry.cpp` handle the visual heavy lifting.

* ListView: An optimized list. Even if you have 1,000 settings, it only creates widgets for those currently visible on the screen.
* ListEntry: The "row" widget. It contains the logic: "If the setting is a Scalar, show a Slider. If it is Discrete, show a Rotator/Button."

This means that visually, you only need to style one Slider and one Rotator in UMG, and the system will use those templates for every single option in your game.

Would you like me to show you how to register a specific Graphics Setting using the engine's built-in `GameUserSettings` within this C++ Registry?
