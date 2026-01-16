---
description: No mundo moderno dos jogos, nem todas as configurações são iguais.
icon: folder-gear
---

# Local & Shared Settings

Se eu mudo a **Resolução** no meu PC Gamer 4K, essa configuração **NÃO** deve ir para o meu Steam Deck (720p). Isso é **LOCAL**.

Se eu inverto o **Eixo Y** ou ligo **Legendas**, eu quero que isso funcione em qualquer lugar que eu logar. Isso é **SHARED** (Compartilhado/Conta).

O plugin `GameplayCommonSettings` força essa separação arquitetural para manter seu jogo profissional.

#### 1. GameplaySettingsLocal (O "Gerente de Hardware")

Esta classe cuida de tudo que é específico da máquina onde o jogo está rodando.

* **Base Class:** `UGameplaySettingsLocal`
* **Onde Salva:** Geralmente no arquivo `.ini` local (`GameUserSettings.ini`).
* **Responsabilidade:** Gráficos, Janela, Desempenho.

**Criando sua classe (MySettingsLocal.h)**

Você deve herdar de `UGameplaySettingsLocal`. É aqui que você cria as funções que suas **Macros Dinâmicas** (vistas no [configuracoes-dinamicas.md](configuracoes-dinamicas.md "mention")) vão chamar.

```c++
// MySettingsLocal.h
#pragma once

#include "Framework/GameplaySettingsLocal.h"
#include "MySettingsLocal.generated.h"

UCLASS()
class UMySettingsLocal : public UGameplaySettingsLocal
{
    GENERATED_BODY()

public:
    UMySettingsLocal();

    // Cria um Getter/Setter para o Registry "ver" via macro
    UFUNCTION()
    bool GetFullscreenMode() const;

    UFUNCTION()
    void SetFullscreenMode(bool bIsFullscreen);

    // Função padrão do GameUserSettings para aplicar tudo de uma vez
    virtual void ApplyNonResolutionSettings() override;
};
```

**Implementando a Lógica (MySettingsLocal.cpp)**

Aqui nós conversamos com o `GEngine->GameUserSettings` real.

```c++
// MySettingsLocal.cpp
#include "MySettingsLocal.h"
#include "GameFramework/GameUserSettings.h"

bool UMySettingsLocal::GetFullscreenMode() const
{
    // Lê do sistema real da engine
    return GetGameUserSettings()->GetFullscreenMode() == EWindowMode::Fullscreen;
}

void UMySettingsLocal::SetFullscreenMode(bool bIsFullscreen)
{
    // Aplica no sistema real
    EWindowMode::Type NewMode = bIsFullscreen ? EWindowMode::Fullscreen : EWindowMode::Windowed;
    GetGameUserSettings()->SetFullscreenMode(NewMode);
    
    // Opcional: Aplicar imediatamente para feedback visual
    GetGameUserSettings()->ApplyResolutionSettings(false);
}
```

#### 2. GameplaySettingsShared (O "Perfil do Jogador")

Esta classe cuida das preferências pessoais. Ela funciona quase como um SaveGame.

* **Base Class:** `UGameplaySettingsShared`
* **Onde Salva:** Em um arquivo `.sav` (Slot de Save) ou na Nuvem.
* **Responsabilidade:** Gameplay, Áudio, Acessibilidade, Keybindings.

**Criando sua classe (MySettingsShared.h)**

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
    // Exemplo: O jogador gosta de legendas?
    UPROPERTY(Config)
    bool bSubtitlesEnabled = true;

    // Getter/Setter para o Registry Dinâmico
    UFUNCTION()
    bool GetSubtitlesEnabled() const { return bSubtitlesEnabled; }

    UFUNCTION()
    void SetSubtitlesEnabled(bool bNewValue);
    
    // Chamado quando o Save é carregado. É aqui que você aplica as coisas!
    virtual void ApplySettings() override;
};
```

**Implementando a Lógica (MySettingsShared.cpp)**

Diferente do Local (que aplica na hora), o Shared precisa saber se "aplicar" ao carregar o jogo.

```c++
// MySettingsShared.cpp
#include "MySettingsShared.h"
#include "MySubtitlesSystem.h" // Exemplo fictício

void UMySettingsShared::SetSubtitlesEnabled(bool bNewValue)
{
    if (bSubtitlesEnabled != bNewValue)
    {
        bSubtitlesEnabled = bNewValue;
        ApplySettings(); // Aplica a mudança
        SaveSettings();  // Salva no disco/nuvem
    }
}

void UMySettingsShared::ApplySettings()
{
    Super::ApplySettings();

    // Aplica a lógica real no jogo
    if (UMySubtitlesSystem* Subs = GetWorld()->GetSubsystem<UMySubtitlesSystem>())
    {
        Subs->SetEnabled(bSubtitlesEnabled);
    }
}
```

#### 3. Conectando tudo no Local Player

Agora vem a peça final. Como o `Registry` encontra essas classes? Ele pergunta ao `LocalPlayer`.

Você precisa criar uma classe `LocalPlayer` customizada no seu projeto e inicializar esses objetos nela.

**MyLocalPlayer.h:**

```c++
UCLASS()
class UMyLocalPlayer : public ULocalPlayer, public IGameplayCommonSettingsInterface
{
    GENERATED_BODY()

public:
    // Getters que as Macros do Registry usam
    UFUNCTION()
    UMySettingsLocal* GetLocalSettings() const override;

    UFUNCTION()
    UMySettingsShared* GetSharedSettings() const override;

protected:
    // O Local Player nasce muito cedo. Vamos criar os settings aqui.
    virtual void PostInitProperties() override;

private:
    UPROPERTY(Transient)
    TObjectPtr<UMySettingsLocal> LocalSettings;

    UPROPERTY(Transient)
    TObjectPtr<UMySettingsShared> SharedSettings;
};
```

{% hint style="warning" %}
**Configuração no Editor:** Não esqueça de ir em **Project Settings > Engine > General Settings** e mudar a **Local Player Class** para `MyLocalPlayer` e também mudar a **Game User Settings Class** para a `MySettingsLocal`
{% endhint %}

#### 4. Resumo do Fluxo Completo

Agora você tem o ciclo completo do GameplaySettings:

1. **UI (Widget):** O usuário move um Slider.
2. **Registry (Dynamic):** O `GameplaySettingValueScalarDynamic` recebe o evento.
3. **Macro:** Ele usa a macro `GET_LOCAL_SETTINGS_FUNCTION_PATH` para achar o alvo.
4. **LocalPlayer:** A macro pega o `MyLocalPlayer` e chama `GetLocalSettings()`.
5. **SettingsLocal:** A função `SetFullscreenMode` é executada.
6. **Engine:** O `GameUserSettings` da Unreal altera a resolução.

Parece complexo na primeira configuração, mas depois disso, adicionar uma nova setting é trivial: Adiciona a variável no `Shared/Local`, cria o Getter/Setter, e adiciona elas no `Registry`.
