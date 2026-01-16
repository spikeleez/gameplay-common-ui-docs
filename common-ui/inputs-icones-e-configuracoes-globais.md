---
description: >-
  Neste módulo, vamos sair do código C++ puro e focar na Configuração do
  Projeto.
icon: keyboard
---

# Inputs, Ícones e Configurações Globais

O Common UI depende pesadamente de Data Assets e configurações no Project Settings para funcionar. Se você já se perguntou _"Por que meu botão não funciona com o controle?"_ ou _"Por que vejo teclas de teclado mesmo jogando no controle?"_, este módulo é a resposta.

#### 1. Habilitando o Enhanced Input no Common UI

O Common UI precisa saber quais **Input Actions** (do Enhanced Input) correspondem às ações "universais" de UI, como Confirmar (A/X) e Voltar (B/Bola).

**Passo A: Criando o Input Data**

Você precisa de um Data Asset que faça a ponte entre o Enhanced Input e o Common UI.

1. No Content Browser, clique com o botão direito.
2. Vá em **BlueprintClass** > Pesquise por **`CommonUIInputData`** e crie.
3. Nomeie como `BP_CommonInputData`.

{% hint style="info" %}
Você pode usar o que já vem disponível no plugin, nomeado como `GameplayCommonInputData`
{% endhint %}

**O que configurar dentro dele:**

* **Default Click Action:** Selecione sua Input Action de "Confirmar/Atirar".
* **Default Back Action:** Selecione sua Input Action de "Voltar/Cancelar".
* _Nota: Essas ações devem ser as mesmas que você usa no gameplay (IA\_Confirm, IA\_Cancel)._

**Passo B: Configurando o Projeto**

Agora dizemos ao projeto para usar esse arquivo.

1. Vá em **Project Settings** > **Game** > **Common Input Settings**.
2. Em **Input Data**, selecione o `BP_CommonInputData` que você criou. (ou o que já vem no plugin)
3. Certifique-se de que **Enable Enhanced Input Support** esteja marcado. (Isso ainda está em experimental pela Epic Games, mas nas versões mais atuais 5.5+ não tive problemas ao utiliza-lo).

#### 2. O Truque do "Generic" (Ícones de Gamepad no PC)

Por padrão, a Unreal assume que se você está no Windows, você está usando Mouse/Teclado. O Common UI muda os ícones dinamicamente baseado no último input detectado, mas precisamos configurar os dados do controle corretamente.

O Common UI usa classes chamadas `CommonInputBaseControllerData` para saber qual ícone (Brush) mostrar para cada botão.

Por sorte, ja deixei preparado 5 Common Input Base Controllers Data para você usar, sem precisar criar um do zero, lembrando que você pode sim e deve, em um projeto final, criar os seus próprios `CommonInputBaseControllerData` .

1. **Force o Windows a aceitar Gamepad:**
   * Vá em **Project Settings** > **Game** > **Common Input Settings**.
   * Procure a seção **Platform Input**.
   * Encontre **Windows**.
   * **Supports Gamepad:** Marque como `True`.
   * **Default Gamepad Name:** Mude para `Generic` (ou o nome que corresponda ao seu Controller Data).
   * **Controller Data:** Adicione o seu ControllerData aqui, ou apenas, crie 5 indices e preencha com os que disponibilizei para você.

**Por que "Generic"?** Ao definir o Controller Data do Windows para usar um set genérico (ex: estilo Xbox), você garante que, assim que o jogador tocar em _qualquer_ botão do controle no PC, o sistema de UI troque todos os prompts para os ícones definidos nesse Data Asset.

No final de tudo, você deve ter algo parecido com isso:<br>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### 3. Common UI Framework

O plugin `Common UI Framework` adiciona uma camada extra de controle sobre o Common UI, especificamente para gerenciar **Traits** (Características) da plataforma via C++.

Arquivo de referência: `CommonUISettings.h`

**O que são Platform Traits?**

São Gameplay Tags que definem capacidades da plataforma atual.

* Exemplos: `Platform.Trait.Mobile`, `Platform.Trait.SupportsTouch`, `Platform.Trait.WeakHardware`.

**Como Configurar:**

1. Vá em **Project Settings** > **Plugins** > **Common UI Framework**.
2. Você verá uma lista de **Platform Traits**.
3. Você pode adicionar overrides por plataforma (ex: adicionar a tag `Mobile` apenas na plataforma Android).

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**Como usar isso no C++?**

Isso é extremamente útil para carregar UIs diferentes (ex: botões virtuais grandes no Mobile, UI compacta no PC).

```c++
// Exemplo de uso em qualquer Widget ou Actor C++
#include "GameplayTagsManager.h"
#include "CommonUISettings.h"

void UMyWidget::CheckPlatform()
{
    // Tag que queremos verificar
    FGameplayTag MobileTag = FGameplayTag::RequestGameplayTag(FName("Platform.Trait.Mobile"));

    // Verifica se a plataforma atual tem essa trait
    if (UCommonUISettings::GetPlatformTraits().HasTag(MobileTag))
    {
        // Ativar interface touch
        ActivateTouchControls();
    }
}
```

#### 4. Resumo da Integração C++ (Common Bound Action Button)

Agora que os Inputs estão configurados no Project Settings, seus botões C++ (`GameplayBoundActionButton`) funcionarão magicamente.

No **Módulo 1**, falamos sobre `UGameplayButtonBase`. Existe uma versão mais avançada no plugin chamada `UGameplayBoundActionButton` (veja `GameplayBoundActionButton.h`).

* **O que ele faz:** Ele se "liga" a uma configuração de Input Action automaticamente e mostra o ícone correto ao lado do texto.
* **Como usar:**
  1. No seu Widget Blueprint, use o `GameplayBoundActionButton`.
  2. Ele lerá automaticamente o `BP_CommonInputData` ou `GameplayCommonInputData` que configuramos no Passo 1 para saber qual ícone mostrar.

{% hint style="info" %}
Para que isso realmente funcione com a opção `EnableEnhancedInputSupport`  você precisa na tela em que você quer que o GameplayBoundActionButton ou GameplayButton mostre seus ícones, adicionar o Input Mapping em que os Input Actions com as teclas/botões que devem aparecer no botão, esteja injetado naquela tela (Geralmente uma `GameplayActivatableWidget`). Segue o exemplo abaixo.
{% endhint %}

Exemplo da tela de Settings, que vamos abordar nos próximos módulos, se eu quiser que mostre os ícones corretamente nos botões, você precisa adicionar o Input Mapping Context nessa tela.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Caso você não queira marcar `EnableEnhancedInputSupport`  como verdadeiro, pode usar a forma padrão do Common UI, que é utilizando as UniversalActions diretamente, e ignorar essa etapa.
{% endhint %}
