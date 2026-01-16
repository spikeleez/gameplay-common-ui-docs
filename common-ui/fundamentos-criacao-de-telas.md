---
description: Bem-vindo ao guia de utilização do Gameplay Common UI.
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

# Introdução ao Gameplay Common UI

Este documento foi desenhado para desenvolvedores familiarizados com Blueprints na Unreal Engine que desejam dar o próximo passo para o C++ utilizando este plugin.

#### 1. Configuração Inicial

Antes de escrevermos código, precisamos garantir que o plugin e o projeto "se conversem".

**Dependências (Build.cs)**

Para usar as classes do C++ deste plugin no seu código do jogo, você precisa adicionar o módulo `GameplayCommonUI` à sua lista de dependências.

1. Abra o arquivo `.Build.cs` do seu projeto (geralmente em `Source/SeuProjeto/SeuProjeto.Build.cs`).
2. Adicione `"GameplayCommonUI"` e `"CommonUI"` em `PublicDependencyModuleNames`.

```c#
// Exemplo do seu arquivo .Build.cs
PublicDependencyModuleNames.AddRange(new string[] { 
    "Core", 
    "CoreUObject", 
    "Engine", 
    "InputCore",
    "CommonUI",           // Necessário
    "GameplayCommonUI",   // O Plugin
    "GameplayTags"        // Usado para gerenciar Layers
});
```

**Configuração do Viewport**

O Common UI exige uma classe específica de Viewport para gerenciar o foco (controle vs mouse).

1. No Editor, vá em **Project Settings** > **Engine** > **General Settings**.
2. Procure por **Game Viewport Client Class**.
3. Defina como `CommonGameViewportClient`.

#### 2. Entendendo as Classes Principais (A Tradução Blueprint -> C++)

Aqui vamos traduzir os conceitos que você já conhece de UMG/Blueprints para a estrutura deste plugin.

**A Base da Tela: `UGameplayActivatableWidget`**

* **O que é:** É a classe pai de todas as telas principais do seu jogo (Menu Principal, Inventário, Pause, HUD).
* **No Blueprint seria:** Um `UserWidget` comum, mas com superpoderes de Input.
* **Por que usar:** Diferente de um Widget comum, este widget sabe quando foi "Ativado" (apareceu na tela) ou "Desativado". Ele gerencia automaticamente se o mouse deve aparecer ou se o input deve ir para o jogo.

**O Layout Mestre: `UGameplayGameLayout`**

* **O que é:** É o container que segura todas as suas widgets.
* **No Blueprint seria:** O "HUD Principal" onde você adiciona as coisas com "Add to Viewport".
* **A Diferença:** Em vez de usar Z-Order manual, o `GameplayGameLayout` usa **Gameplay Tags** para definir camadas (Layers). Ex: Camada de Jogo, Camada de Menu, Camada de Modal.

**Botões Padronizados: `UGameplayButtonBase`**

* **O que é:** Um botão wrapper do CommonButton.
* **No Blueprint seria:** Um Widget customizado com um botão dentro.
* **Por que usar:** Ele já trata foco de controle, som de clique e estados de hover/selecionado automaticamente.

#### 3. Tutorial: Criando sua Primeira Tela C++ (Menu de Pause)

Vamos criar um menu simples. Em C++, a lógica fica no código, e o visual (posicionamento, cores) fica no Blueprint herdado.

**Passo A: O Arquivo de Cabeçalho (.h)**

Crie uma nova classe C++ herdando de `GameplayActivatableWidget`. Vamos chamá-la de `MyPauseMenuWidget`.

```c++
// MyPauseMenuWidget.h
#pragma once

#include "CoreMinimal.h"
#include "Widgets/GameplayActivatableWidget.h" // Importando a base do plugin
#include "MyPauseMenuWidget.generated.h"

// Forward Declaration (Dizemos que essas classes existem sem incluir o header pesado aqui)
class UGameplayButtonBase;

/**
 * Menu de Pause simples.
 * Analogia: Este é o "Event Graph" do seu Widget Blueprint.
 */
UCLASS()
class SEUPROJETO_API UMyPauseMenuWidget : public UGameplayActivatableWidget
{
    GENERATED_BODY()

protected:
    // --- 1. Eventos de Ciclo de Vida (Como Event Construct) ---
    
    // Executado quando o widget é criado e suas variáveis (BindWidget) são conectadas.
    virtual void NativeOnInitialized() override;

    // Executado quando o widget ganha foco (aparece na tela).
    virtual void NativeOnActivated() override;

    // --- 2. Componentes da UI (Como as variáveis no Designer do UMG) ---

    // meta = (BindWidget) obriga você a ter um botão chamado "ResumeButton" no Blueprint.
    // Se não tiver, o jogo não compila/da erro, evitando crashes.
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UGameplayButtonBase> ResumeButton;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UGameplayButtonBase> QuitButton;

private:
    // --- 3. Funções de Lógica (Seus Custom Events) ---
    
    UFUNCTION()
    void HandleResumeClicked();

    UFUNCTION()
    void HandleQuitClicked();
};
```

**Passo B: O Arquivo de Implementação (.cpp)**

Aqui definimos o comportamento.

```c++
// MyPauseMenuWidget.cpp
#include "MyPauseMenuWidget.h"
#include "Widgets/GameplayButtonBase.h" // Agora incluímos o header real do botão

void UMyPauseMenuWidget::NativeOnInitialized()
{
    Super::NativeOnInitialized(); // Sempre chame o Super!

    // Analogia: Isso é como conectar o evento "OnClicked" no Event Graph.
    
    if (ResumeButton)
    {
        // Quando ResumeButton for clicado, execute a função HandleResumeClicked
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
    
    // Aqui você pode pausar o jogo, tocar um som, etc.
    // O GameplayActivatableWidget já pode lidar com Input Mode automaticamente
    // dependendo da configuração "Input Config" no Blueprint.
}

void UMyPauseMenuWidget::HandleResumeClicked()
{
    // Desativa este widget. O CommonUI remove ele da pilha e devolve o foco para o jogo.
    DeactivateWidget();
}

void UMyPauseMenuWidget::HandleQuitClicked()
{
    // Exemplo: Sair do jogo
    // No mundo real, aqui você chamaria um "Confirmation Dialog" (veremos em breve!)
    // GEngine->Exec(GetWorld(), TEXT("Quit")); // Exemplo simples
}
```

**Passo C: Criando o Blueprint**

1. Vá no Unreal Editor.
2. Crie um novo **Widget Blueprint**.
3. Quando pedir a classe pai, procure por `MyPauseMenuWidget` (a classe C++ que acabamos de criar).
4. No Designer, adicione dois botões (Use a classe `GameplayButtonBase` ou uma BP herdada dela).
5. **IMPORTANTE:** Renomeie os botões no painel Hierarchy para `ResumeButton` e `QuitButton` (exatamente como no C++). O ícone deve mudar, indicando que estão vinculados.
6. Estilize como quiser.

#### 4. Dica Pro: Push Async (Evitando Hard References)

Uma dor comum em C++ é ter que carregar classes. O `GameplayCommonUI` tem helpers maravilhosos para isso.

Em vez de carregar a classe manualmente, você pode usar o `UGameplayPushActivatableWidgetAsync` (vimos esse arquivo no código fonte!).

**Exemplo de uso (No seu PlayerController ou HUD C++):**

```cpp
#include "Async/GameplayPushActivatableWidgetAsync.h"

// ... dentro de alguma função que abre o menu ...

// Define qual camada (Layer) esse widget vai entrar. 
// As Tags são configuradas no Project Settings > Gameplay Common UI
FGameplayTag LayerTag = FGameplayTag::RequestGameplayTag("UI.Layer.Menu");

// Inicia o carregamento assíncrono e exibe quando pronto
UGameplayPushActivatableWidgetAsync::PushActivatableWidget(
    this,              // World Context
    LayerTag,          // Em qual layer vai aparecer
    PauseMenuSoftClass // TSoftClassPtr<UGameplayActivatableWidget> configurada no editor
);
```

Isso garante que seu jogo não trave (hitch) ao tentar abrir um menu pesado pela primeira vez
