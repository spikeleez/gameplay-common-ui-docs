---
description: Esqueça o AddToViewport e o Z-Order
icon: layer-group
---

# Gerenciamento de Layers e Carregamento Assíncrono

No Common UI, nós não "jogamos" widgets na tela; nós os empilhamos em Camadas (Layers) organizadas semanticamente. Neste módulo, vamos entender a arquitetura que segura sua UI e como navegar entre telas sem travar o jogo.

#### 1. A Arquitetura: O Esqueleto da UI

Para que o `PushActivatableWidget` funcione, ele precisa de uma estrutura existente na tela. Essa estrutura é o **Primary Game Layout**.

**O** `UGameplayPrimaryLayout`

* **O que é:** É o Widget Raiz (Root) do seu jogo. Ele é criado automaticamente pela `GameplayCommonUIPolicy` quando o jogador entra.
* **Função:** Ele é apenas um container. Ele não tem visual próprio (geralmente é transparente), mas contém várias "Pilhas" (Stacks) de widgets.
* **Analogia:** Pense nele como uma estante vazia. As **Layers** são as prateleiras.

**As Layers (Camadas)**

Dentro do `UGameplayPrimaryLayout`, temos widgets do tipo `CommonActivatableWidgetContainerBase` (ou Stacks). Cada Stack representa uma camada e é identificada por uma **Gameplay Tag**.

A ordem visual é definida pela ordem na hierarquia do Blueprint do Layout.

**Estrutura Típica (de trás para frente):**

1. **Layer Game (**`GameplayCommonUI.Layer.Game`**):** HUD, mira, barras de vida. (Fica no fundo).
2. **Layer GameMenu (**`GameplayCommonUI.Layer.GameMenu`**):** Inventário e outras janelas que podem usar o mouse e o teclado ao mesmo tempo.
3. **Layer Menu (**`GameplayCommonUI.Layer.Menu`**):** Pause, Configurações. (Bloqueia o Game).
4. **Layer Modal (**`GameplayCommonUI.Layer.Modal`**):** Popups de confirmação, erros, notificações (Bloqueia tudo).

#### 2. A Ferramenta: `UGameplayPushActivatableWidgetAsync`

Esta é a classe mágica que você usará 90% do tempo para abrir menus.

**O Problema das Referências Rígidas (Hard References)**

Se você tem uma variável `TSubclassOf<UUserWidget>` no seu HUD para o "Menu de Pause", a Unreal carrega o Menu de Pause (e todas as texturas dele) na memória assim que o HUD nasce. Se o menu for pesado, seu jogo trava ao iniciar.

**A Solução Async**

O `GameplayPushActivatableWidgetAsync` usa `TSoftClassPtr`. Ele diz: "Eu sei onde o arquivo do menu está, mas só vou carregá-lo quando você pedir".

**Como funciona o Fluxo (Passo a Passo)**

Quando você chama essa node/função:

1. **Request:** Você diz "Quero abrir a classe `BP_PauseMenu` na camada `UI.Layer.Menu`".
2. **Loading:** A task verifica se a classe já está na memória. Se não estiver, ela inicia um _Streamable Load_ (carregamento em background).
3. **Discovery:** Enquanto carrega, o sistema procura pelo `UGameplayGameLayout` do jogador local.
4. **Layer Lookup:** O Layout procura em seu mapa interno: "Quem é a widget responsável pela tag `UI.Layer.Menu`?".
5. **Push:** Quando o carregamento termina, o widget é instanciado e "empurrado" para dentro dessa Stack específica.
6. **Activation:** O Common UI ativa o widget, dando foco a ele e desativando inputs das camadas inferiores (Game).

#### 3. Implementação Prática

Vamos ver como usar isso no dia a dia.

**Em Blueprints (O jeito mais comum)**

O plugin expõe uma node latente muito útil chamada **"Push Activatable Widget Async"**.

* **World Context:** `Self`.
* **Layer Name:** Selecione a tag (ex: `UI.Layer.Menu`).
* **Activatable Widget Class:** Selecione seu Widget Blueprint (note que a cor do pin é rosa-claro, indicando Soft Reference).
* **State:** (Opcional) Você pode passar um payload de dados iniciais.

**Eventos:**

* **Before Push:** Ótimo para setar variáveis no widget antes dele aparecer na tela (ex: passar o Item que será editado no inventário).
* **After Push:** O widget já está na tela e pronto.

**Em Blueprint**

De qualquer lugar, tendo acesso ao seu `PlayerController` você pode chamar as duas tasks disponíveis para fazer um Push de uma Activatable Widget para o Stack.\
\
**Push Activatable Widget for Class**: Essa task executa um Push de uma widget pela **classe soft** dela, e adiciona ao Stack baseado na `LayerTag` .

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**Push Activatable Widget for Tag:** Essa task executa um Push de uma Widget pela **tag de registro** dela, e adiciona ao Stack baseado na `LayerTag` .

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Para registrar uma Widget com uma tag, você precisa acessar **ProjectSettings > Game > Gameplay Common UI Settings > Registered Activatable Widgets**
{% endhint %}

**Em C++**

Se você estiver em uma classe que tenha acesso ao `PlayerController`, use a biblioteca estática `UGameplayCommonUILibrary` ou a task diretamente.

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

#### 4. Configurando o `GameplayGameLayout` (Setup Único)

Para que tudo acima funcione, você precisa ter seu Layout configurado uma única vez.

1. **Crie o Blueprint:** Crie um Widget Blueprint herdando de `GameplayPrimaryLayout`. Nomeie como `WBP_PrimaryLayout`.
2. **Adicione as Stacks:** No Designer, adicione 3 widgets do tipo **Common Activatable Widget Stack**
3. **Nomeie e Registre:**
   * Stack 1: Nomeie **GameStack**. No Graph, no `Construct`, chame `RegisterLayer` passando a tag `GameplayCommonUI.Layer.Game` e a referência ao **GameStack**.
   * Stack 2: Nomeie **MenuStack**. Registre com `GameplayCommonUI.Layer.Menu`.
   * Stack 3: Nomeie **ModalStack**. Registre com `GameplayCommonUI.Layer.Modal`.
4. **Defina na Policy:** Vá nas configurações do `GameplayCommonUI` ou crie uma classe `GameplayCommonUIPolicy` e defina este `WBP_PrimaryLayout` como o layout padrão.

Agora, o sistema sabe exatamente onde colocar cada widget quando você pedir!

{% hint style="info" %}
No plugin já vem criado uma `GameplayPrimaryLayout` chamada na Blueprint de `PrimaryLayout` , por padrão a `GameplayCommonUIPolicy` inicia essa widget para você automaticamente, caso queira mudar para uma customizada sua, você antes, deve criar uma `GameplayCommonUIPolicy` customizada e definir ela em **Project Settings > Game > Gameplay Common UI Settings > UIPolicyClass.**
{% endhint %}
