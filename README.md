---
description: >-
  Antes de começarmos a modificar o código do GameplayCommonUI, precisamos
  garantir que o Visual Studio tenha todas as ferramentas necessárias para
  compilar a Unreal Engine.
icon: bolt
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
    - https://app.gitbook.com/s/yE16Xb3IemPxJWydtPOj/getting-started/quickstart
---

# Configurando o Ambiente de Desenvolvimento C++

{% hint style="warning" %}
Esta etapa é opcional se for usar apenas o **GameplayCommonUI Module**, se você deseja usar o **GameplayCommonSettings Module**, esta etapa passa a ser obrigatória.
{% endhint %}

#### Passo 1: Baixando o Visual Studio

Se você ainda não tem, baixe o **Visual Studio 2022 ou 2026** (A versão **Community** é gratuita e suficiente).

1. Acesse: [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
2. Baixe o instalador do **Community 2022 ou 2026**.
3. Execute o `VisualStudioSetup.exe`.

#### Passo 2: A Carga de Trabalho (O Passo Crítico)

Ao abrir o instalador, você verá uma tela com várias caixas para marcar. **Não instale apenas o editor básico.**

1. Role para baixo até encontrar a categoria **Jogos (Gaming)**.
2. Marque a caixa: **Desenvolvimento de jogos com C++ (Game development with C++)**.

Ao marcar isso, observe o painel lateral direito ("Detalhes da instalação"). Certifique-se de que os seguintes itens opcionais estejam marcados:

* \[x] **C++ profiling tools**
* \[x] **C++ AddressSanitizer** (Útil para debugar crash de memória)
* \[x] **Windows 10 SDK** (ou 11, dependendo do seu OS)
* \[x] **Unreal Engine installer** (Opcional, mas recomendado pois instala ferramentas de integração)
* \[x] **IDE support for Unreal Engine** (Fundamental para ver Blueprints dentro do VS)

Clique em **Instalar/Modificar** e aguarde (é um download grande, pegue um café ☕).

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

#### Passo 3: Instalando a Extensão "UnrealVS" (O Segredo da Epic)

A Epic Games inclui uma extensão oficial para o Visual Studio que facilita muito a vida (permite rodar o jogo direto do VS com argumentos, ver atalhos de build, etc), mas ela **não é instalada automaticamente**.

1. Vá até a pasta onde sua Unreal Engine está instalada.
   * Geralmente: `C:\Program Files\Epic Games\UE_5.X\Engine\Extras\UnrealVS\VS2022\`
2. Dê um clique duplo no arquivo **`UnrealVS.vsix`**.
3. Siga o instalador para adicionar ao seu Visual Studio 2022.

#### Passo 4: Configurando a Unreal Engine para usar o VS

Agora que o VS está pronto, precisamos dizer à Unreal para usá-lo.

1. Abra seu Projeto Unreal.
2. Vá no menu **Edit** > **Editor Preferences**.
3. Na barra de busca, digite: **Source Code**.
4. Em **Source Code Editor**, mude de "Null" ou "Visual Studio Code" para **Visual Studio 2022 ou 2026**.

#### Passo 5: Gerando os Arquivos de Projeto

Sempre que você adicionar uma nova classe C++ ou um Plugin (como o `GameplayCommonUI`), você precisa atualizar a solução do Visual Studio.

1. Feche o Visual Studio.
2. Vá na pasta do seu projeto (pelo Windows Explorer).
3. Clique com o botão direito no arquivo **`.uproject`**.
4. Selecione **Generate Visual Studio project files**.
5. Uma janelinha preta vai abrir e fechar rapidamente.
6. Agora abra o arquivo **`.sln`** (Solution) que apareceu (ou foi atualizado).

#### Passo 6: O Primeiro Build

1. Com o `.sln` aberto no Visual Studio.
2. Olhe na barra superior. Certifique-se de que a configuração está como **"Development Editor"** e a plataforma **"Win64"**.
3. No "Solution Explorer" (painel direito), clique com o botão direito no **Nome do Seu Projeto** (não na pasta Engine) e selecione **Build**.

#### Dicas de Ouro para Sobrevivência

* **IntelliSense Lento:** O VS pode demorar um pouco para "ler" todo o código da Unreal na primeira vez. Tenha paciência se as cores do código demorarem a aparecer.
* **Erro de Include:** Se o VS sublinhar `#include "AlgumaCoisa.h"` em vermelho, mas o Build funcionar, é apenas o IntelliSense atrasado. Confie no Output do Build, não nas linhas vermelhas do editor.
* **Live Coding:** A Unreal 5 usa Live Coding (Ctrl+Alt+F11 com o editor aberto). Para mudanças pequenas em `.cpp`, use isso. Para mudar cabeçalhos `.h` ou adicionar novas classes, feche o editor e compile pelo VS (Ctrl+Shift+B).

Para mais informações sobre a instalação do Visual Studio, fique a vontade para visitar o site oficial da Epic Games sobre o assunto: [Link de Suporte](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine)
