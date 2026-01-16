---
icon: sliders
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
---

# Dynamic Settings

In this module, you will learn how to leverage Dynamic classes (`ScalarDynamic`, `DiscreteDynamic`) alongside powerful Macros to map your game settings in seconds.

### 1. The Secret of Setting Macros

For the dynamic system to function, the Registry needs to know exactly where to fetch and save data without you writing manual glue code for every function. We use macros to point directly to the functions in your `LocalSettings` or `SharedSettings`.

This is mandatory because `UGameplaySettingValueScalarDynamic` requires a function pointer (`UFunction*`) or property (`FProperty*`) to know what to modify.

#### Implementing Macros in Your Registry

In your Registry header file (e.g., `MyGameSettingRegistry.h`), define these helper macros. Ensure you adapt the class names (`UMyLocalPlayer`, `UMySettingsLocal`) to match your project.

{% hint style="warning" %}
_Adapte os nomes das classes (`UMyLocalPlayer`, `UMySettingsLocal`) para o seu projeto._
{% endhint %}

```c++
// MyGameSettingRegistry.h

// Helper to retrieve getters/setters from SHARED Settings (e.g., Subtitles, Save Data)
#define GET_SHARED_SETTINGS_FUNCTION_PATH(FunctionOrPropertyName) \
    GET_SETTINGS_FUNCTION_PATH(UMyLocalPlayer, GetSharedSettings, UMySettingsShared, FunctionOrPropertyName)

// Helper to retrieve getters/setters from LOCAL Settings (e.g., Graphics, Resolution)
#define GET_LOCAL_SETTINGS_FUNCTION_PATH(FunctionOrPropertyName) \
    GET_SETTINGS_FUNCTION_PATH(UMyLocalPlayer, GetLocalSettings, UMySettingsLocal, FunctionOrPropertyName)
```

{% hint style="info" %}
What does this macro do? It instructs the system: "Go to the Player, get the Settings object, and find the function with THIS name."
{% endhint %}

### 2. Window Mode (Discrete Dynamic)

We will create a Window Mode configuration (Fullscreen, Windowed, Borderless). Instead of creating a new class, we use `UGameplaySettingValueDiscreteDynamic_Enum` directly in the Registry's implementation file.

Scenario: We want this option to appear only on PC (Windows/Mac/Linux) and hide it on Consoles or Mobile.

```c++
// Inside MyGameSettingRegistry.cpp -> InitializeVideoSettings()

UGameplaySettingValueDiscreteDynamic_Enum* Setting = NewObject<UGameplaySettingValueDiscreteDynamic_Enum>();
Setting->SetDevName(TEXT("WindowMode"));
Setting->SetDisplayName(LOCTEXT("WindowMode_Name", "Window Mode"));
Setting->SetDescriptionRichText(LOCTEXT("WindowMode_Description", "In Windowed mode you can interact with other windows more easily, and drag the edges of the window to set the size. In Windowed Fullscreen mode you can easily switch between applications. In Fullscreen mode you cannot interact with other windows as easily, but the game will run slightly faster."));

// Mapping to existing functions in LocalSettings
Setting->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetFullscreenMode));
Setting->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetFullscreenMode));

// Adding the options
Setting->AddOption(EWindowMode::Fullscreen, LOCTEXT("WindowModeFullscreen", "Fullscreen"));
Setting->AddOption(EWindowMode::WindowedFullscreen, LOCTEXT("WindowModeWindowedFullscreen", "Windowed Fullscreen"));
Setting->AddOption(EWindowMode::Windowed, LOCTEXT("WindowModeWindowed", "Windowed"));

// Platform check: Kill the setting if the platform doesn't support windowed modes
Setting->AddEditCondition(FGameplaySettingWhenPlatformHasTrait::KillIfMissing(GameplayCommonSettingsTags::Trait_SupportsWindowedMode, TEXT("Platform does not support window mode")));

WindowModeSetting = Setting;

Display->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### 3. Brightness (Scalar Dynamic)

Now, let's implement a simple slider for Brightness/Gamma using `UGameplaySettingValueScalarDynamic`.

```c++
// Inside MyGameSettingRegistry.cpp -> InitializeGameplaySettings()

UGameplaySettingValueScalarDynamic* Setting = NewObject<UGameplaySettingValueScalarDynamic>();
Setting->SetDevName(TEXT("Brightness"));
Setting->SetDisplayName(LOCTEXT("Brightness_Name", "Brightness"));
Setting->SetDescriptionRichText(LOCTEXT("Brightness_Description", "Adjusts the brightness."));

Setting->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetDisplayGamma));
Setting->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetDisplayGamma));
Setting->SetDefaultValue(2.2);

// Formatting the displayed text (e.g., mapping 1.7-2.7 range to 50%-150%)
Setting->SetDisplayFormat([](double SourceValue, double NormalizedValue)
{
	return FText::Format(LOCTEXT("BrightnessFormat", "{0}%"), (int32)FMath::GetMappedRangeValueClamped(FVector2D(0, 1), FVector2D(50, 150), NormalizedValue));
});
Setting->SetSourceRangeAndStep(TRange<double>(1.7, 2.7), 0.01);

// Only allow the primary player to change this
Setting->AddEditCondition(FGameplaySettingWhenPlayingAsPrimaryPlayer::Get());

Display->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

### 4. Texture Quality (Discrete Dynamic)

You can use `UGameplaySettingValueDiscreteDynamic_Number` to create graphical settings for Textures, Post-Processing, Shadows, and more.

```c++
// Inside MyGameSettingRegistry.cpp -> InitializeVideoSettings()

UGameplaySettingValueDiscreteDynamic_Number* Setting = NewObject<UGameplaySettingValueDiscreteDynamic_Number>();

Setting->SetDevName(TEXT("TextureQuality"));
Setting->SetDisplayName(LOCTEXT("TextureQuality_Name", "Textures"));

Setting->SetDescriptionRichText(LOCTEXT("TextureQuality_Description", "Texture quality determines the resolution of textures in game. Increasing this setting will make objects more detailed, but can reduce performance."));

Setting->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetTextureQuality));
Setting->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetTextureQuality));
Setting->AddOption(0, LOCTEXT("TextureQuality_Low", "Low"));
Setting->AddOption(1, LOCTEXT("TextureQuality_Medium", "Medium"));
Setting->AddOption(2, LOCTEXT("TextureQuality_High", "High"));
Setting->AddOption(3, LOCTEXT("TextureQuality_Epic", "Epic"));

Setting->AddEditDependency(AutoSetQuality);
Setting->AddEditDependency(GraphicsQualityPresets);
Setting->AddEditCondition(MakeShared<FGameplaySettingEditCondition_VideoQuality>(TEXT("Platform does not support Texture quality")));

// Linking dependencies: If presets change, this updates; if this changes, presets set to "Custom"
GraphicsQualityPresets->AddEditDependency(Setting);

GraphicsQuality->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

### 5. Summary of Utilized Classes

| Class                                      | Ideal Use Case                                                               |
| ------------------------------------------ | ---------------------------------------------------------------------------- |
| **`UGameplaySettingValueScalarDynamic`**   | Simple sliders (Volume, Brightness, FOV, Sensitivity).                       |
| **`UGameplaySettingValueDiscreteDynamic`** | Lists of options (Enums) or Booleans (On/Off) mapping to existing functions. |

You have now seen how to use the native GameplayCommonSettings classes to create custom configurations efficiently.

Would you like me to show you how to implement the `GetLocalSettings` and `GetSharedSettings` functions in your Local Player class to support these macros?
