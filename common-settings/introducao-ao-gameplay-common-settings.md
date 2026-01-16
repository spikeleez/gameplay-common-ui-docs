---
description: Bem-vindo ao sistema de configurações.
icon: gear-complex
---

# Introdução ao Gameplay Common Settings

Se você já tentou criar um menu de "Opções Gráficas" conectando manualmente Sliders a funções do GameUserSettings em Blueprint, você sabe como isso vira um espaguete rapidamente.

O módulo `GameplayCommonSettings` propõe uma mudança radical: **Você define as configurações em C++, e a UI se constrói sozinha.**

#### 1. A Proposta: O Que é e Por Que Usar?

**O Jeito Antigo (Blueprint/UMG Tradicional)**

1. Você arrasta um Slider para a tela.
2. No `Event Construct`, você lê o valor atual e ajusta o slider.
3. No `OnValueChanged`, você aplica a mudança.
4. Você repete isso 50 vezes para Brilho, Volume, Sensibilidade, Qualidade de Textura, etc.
5. Se mudar o design do slider, você tem que editar 50 widgets.

**O Jeito Gameplay Common Settings**

1. Você cria uma classe C++ chamada `GameplayRegistry` (Registro).
2. Nela, você lista: "Existe uma configuração chamada 'Volume Mestre', ela vai de 0 a 100".
3. O Widget de UI lê essa lista e gera automaticamente uma linha para cada configuração.
4. **Resultado:** Lógica separada do Visual. Manutenção zero na UI.

#### 2. A Arquitetura: As Peças do Quebra-Cabeça

Para entender o código, precisamos entender a hierarquia das classes que você encontrou na pasta `GameplayCommonSettings`.

**A. O Gerente:** `UGameplaySettingsLocal`

* **Arquivo:** `GameplaySettingsLocal.h`
* **Função:** É o "SaveGame" glorificado. Ele vive no `GameInstance` (ou Subsystem) e guarda os valores que o jogador escolheu. Ele sabe salvar no disco e carregar quando o jogo abre.

**B. O Cardápio:** `UGameplaySettingRegistry`

* **Arquivo:** `GameplaySettingRegistry.h`
* **Função:** É aqui que a mágica acontece. É uma classe onde você "registra" todas as opções disponíveis no seu jogo.
* **Analogia:** Pense nele como o **Menu do Restaurante**. Ele não é a comida, é a lista do que está disponível para pedir.

**C. As Categorias:** `UGameplaySettingCollection`

* **Arquivo:** `GameplaySettingCollection.h`
* **Função:** Agrupa configurações. Ex: "Vídeo", "Áudio", "Controles".

**D. O Item Individual:** `UGameplaySetting`

* **Arquivo:** `GameplaySetting.h`
* **Função:** Uma única opção. Pode ser:
  * `UGameplaySettingValueScalar`: Um número (Slider/Barra). Ex: Volume, Sensibilidade.
  * `UGameplaySettingValueDiscrete`: Uma lista de opções (Combobox/Rotator). Ex: Resolução (1080p, 4K), Qualidade (Baixa, Média, Alta).

#### 3. Como Funciona o Fluxo de Dados?

Quando o jogador abre a tela de Settings:

1. **Inicialização:** O Widget `GameplaySettingScreen` pede ao `Registry`: "Me dê a lista de configurações".
2. **Regeneração:** O `Registry` verifica o jogo atual (ex: "Estou no PC ou no PS5?") e devolve apenas as opções válidas (ex: não mostra "Qualidade Gráfica" em consoles se você não quiser).
3. **Visualização:** O Widget usa um `ListView` para criar uma linha visual para cada `GameplaySetting`.
4. **Alteração:** Quando o jogador mexe no slider, o Widget avisa o `GameplaySetting`.
5. **Aplicação:** O `GameplaySetting` aplica a mudança imediatamente (ex: muda o volume do AudioMixer) e avisa o `GameplaySettingsLocal` para marcar como "sujo" (precisa salvar).

#### 4. Primeiros Passos no Código

Para começar a usar, você precisará criar sua própria classe de Registro herdando de `UGameplaySettingRegistry`.

**Exemplo Teórico (MyGameSettingRegistry.h):**

```c++
#include "Framework/GameplaySettingRegistry.h"
#include "MyGameSettingRegistry.generated.h"

UCLASS()
class UMyGameSettingRegistry : public UGameplaySettingRegistry
{
    GENERATED_BODY()

protected:
    // Esta é a função principal que você vai sobrescrever
    virtual void OnInitialize(UGameplaySettingsLocal* InLocalSettings) override;
};
```

**Exemplo Teórico (MyGameSettingRegistry.cpp):**

```c++
void UMyGameSettingRegistry::OnInitialize(UGameplaySettingsLocal* InLocalSettings)
{
    // 1. Criar uma Categoria (Collection)
    UGameplaySettingCollection* AudioPage = NewObject<UGameplaySettingCollection>(this);
    AudioPage->SetDevName(TEXT("AudioCollection"));
    AudioPage->SetDisplayName(NSLOCTEXT("MyGame", "Audio", "Áudio"));
    RegisterSetting(AudioPage); // Adiciona ao registro
}
```

#### 5. O Papel dos Widgets Especiais

Você viu arquivos como `GameplaySettingListView.cpp` e `GameplaySettingListEntry.cpp`.

* **ListView:** É uma lista otimizada. Mesmo que você tenha 1000 configurações, ele só cria widgets para as que cabem na tela.
* **ListEntry:** É o widget da "linha". Ele tem a lógica de: "Se o Setting for um Slider, mostre um Slider. Se for um Botão, mostre um Botão".

Isso significa que, visualmente, você só precisa estilizar **um** Slider e **um** Botão Rotator no UMG, e o sistema usa eles para todas as opções do jogo.
