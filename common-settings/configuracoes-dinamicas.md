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

# Configurações Dinâmicas

Neste módulo, aprenderemos a usar as classes **Dynamic** (`ScalarDynamic`, `DiscreteDynamic`) junto com **Macros** poderosas para mapear configurações em segundos.

#### 1. O Segredo das Macros de Configurações

Para que o sistema dinâmico funcione, o Registry precisa saber **onde** buscar e salvar os dados sem que você escreva o código da função manualmente. Usamos macros para apontar para as funções do seu `LocalSettings` ou `SharedSettings`.

**Por que isso é obrigatório?** Porque o `UGameplaySettingValueScalarDynamic` precisa de um ponteiro para função (`UFunction*`) ou propriedade (`FProperty*`) para saber o que alterar.

**Implementando as Macros no seu Registry**

No arquivo header do seu Registry (ex: `MyGameSettingRegistry.h`), você deve definir estas macros helper.

{% hint style="warning" %}
_Adapte os nomes das classes (`UMyLocalPlayer`, `UMySettingsLocal`) para o seu projeto._
{% endhint %}

```c++
// MyGameSettingRegistry.h

// Helper para pegar getters/setters do Settings SHARED (ex: Legendas, Dados de Save)
#define GET_SHARED_SETTINGS_FUNCTION_PATH(FunctionOrPropertyName) \
    GET_SETTINGS_FUNCTION_PATH(UMyLocalPlayer, GetSharedSettings, UMySettingsShared, FunctionOrPropertyName)

// Helper para pegar getters/setters do Settings LOCAL (ex: Gráficos, Resolução)
#define GET_LOCAL_SETTINGS_FUNCTION_PATH(FunctionOrPropertyName) \
    GET_SETTINGS_FUNCTION_PATH(UMyLocalPlayer, GetLocalSettings, UMySettingsLocal, FunctionOrPropertyName)
```

{% hint style="info" %}
**O que essa macro faz?** Ela diz: _"Vá no Player, pegue o objeto de Settings, e encontre a função com ESSE nome."_
{% endhint %}

#### 2. Window Mode (Discrete Dynamic)

Vamos criar uma configuração de **Modo de Janela** (Fullscreen, Windowed, Borderless). Em vez de criar uma classe, usaremos `UGameplaySettingValueDiscreteDynamic` diretamente no `.cpp` do Registry.

**Cenário:** Queremos que essa opção só apareça no PC (Windows/Mac/Linux) e não em Consoles ou Mobile.

```c++
// Dentro de MyGameSettingRegistry.cpp -> InitializeVideoSettings()

UGameplaySettingValueDiscreteDynamic_Enum* Setting = NewObject<UGameplaySettingValueDiscreteDynamic_Enum>();
Setting->SetDevName(TEXT("WindowMode"));
Setting->SetDisplayName(LOCTEXT("WindowMode_Name", "Window Mode"));
Setting->SetDescriptionRichText(LOCTEXT("WindowMode_Description", "In Windowed mode you can interact with other windows more easily, and drag the edges of the window to set the size. In Windowed Fullscreen mode you can easily switch between applications. In Fullscreen mode you cannot interact with other windows as easily, but the game will run slightly faster."));

Setting->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetFullscreenMode));
Setting->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetFullscreenMode));
Setting->AddOption(EWindowMode::Fullscreen, LOCTEXT("WindowModeFullscreen", "Fullscreen"));
Setting->AddOption(EWindowMode::WindowedFullscreen, LOCTEXT("WindowModeWindowedFullscreen", "Windowed Fullscreen"));
Setting->AddOption(EWindowMode::Windowed, LOCTEXT("WindowModeWindowed", "Windowed"));

Setting->AddEditCondition(FGameplaySettingWhenPlatformHasTrait::KillIfMissing(GameplayCommonSettingsTags::Trait_SupportsWindowedMode, TEXT("Platform does not support window mode")));

WindowModeSetting = Setting;

Display->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

#### 3. Brightness (Scalar Dynamic)

Agora um slider simples para o Brightness/Gamma. Usaremos `UGameplaySettingValueScalarDynamic`.

```c++
// Dentro de MyGameSettingRegistry.cpp -> InitializeGameplaySettings()

UGameplaySettingValueScalarDynamic* Setting = NewObject<UGameplaySettingValueScalarDynamic>();
Setting->SetDevName(TEXT("Brightness"));
Setting->SetDisplayName(LOCTEXT("Brightness_Name", "Brightness"));
Setting->SetDescriptionRichText(LOCTEXT("Brightness_Description", "Adjusts the brightness."));

Setting->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetDisplayGamma));
Setting->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetDisplayGamma));
Setting->SetDefaultValue(2.2);
Setting->SetDisplayFormat([](double SourceValue, double NormalizedValue)
{
	return FText::Format(LOCTEXT("BrightnessFormat", "{0}%"), (int32)FMath::GetMappedRangeValueClamped(FVector2D(0, 1), FVector2D(50, 150), NormalizedValue));
});
Setting->SetSourceRangeAndStep(TRange<double>(1.7, 2.7), 0.01);

Setting->AddEditCondition(FGameplaySettingWhenPlayingAsPrimaryPlayer::Get());

Display->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

#### 4. Texture Quality (Discrete Dynamic)

Você pode usar o `UGameplaySettingValueDiscreteDynamic_Number` para criar configurações gráficas de Texture, Post Process Quality, Shadow Quality, dentre outros.

```c++
// Dentro de MyGameSettingRegistry.cpp -> InitializeVideoSettings()

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

// When this setting changes, it can GraphicsQualityPresets to be set to custom, or a particular preset.
GraphicsQualityPresets->AddEditDependency(Setting);

GraphicsQuality->AddSetting(Setting);
```

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

#### 5. Resumo das Classes que utilizamos

| Classe                                     | Uso Ideal                                                                          |
| ------------------------------------------ | ---------------------------------------------------------------------------------- |
| **`UGameplaySettingValueScalarDynamic`**   | Sliders simples (Volume, Brilho, FOV, Sensibilidade).                              |
| **`UGameplaySettingValueDiscreteDynamic`** | Listas de opções (Enum) ou booleanos (On/Off) que mapeiam para funções existentes. |

Você acabou de ver alguns dos exemplos de como podemos utilizar as classes nativas do `GameplayCommonSettings` para criar nossas próprias configurações. No próximo módulo, aprenderemos como podemos registrar essas configurações no `SettingsLocal` ou no `SettingsShared`
