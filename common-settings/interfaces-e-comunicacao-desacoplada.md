---
description: >-
  Neste módulo, vamos explorar as Interfaces que vêm com o plugin e como criar a
  sua própria para tornar o código mais limpo.
icon: tower-cell
---

# Interfaces e Comunicação Desacoplada

#### 1. IGameplaySettingActionInterface

Localização: `Interfaces/GameplaySettingActionInterface.h`

Algumas configurações não são apenas valores (Slider/Combobox). Algumas são **Ações**, como "Calibrar HDR", "Ver Créditos" ou "Resetar para Padrão".

Se você criar um Widget customizado para uma configuração específica (ex: um botão bonito para Calibração), ele deve implementar essa interface para se comunicar com o sistema de Settings.

**Funções Principais:**

* `ExecuteAction()`: Chamado quando o jogador confirma a ação.
* `IsActionEnabled()`: Permite desabilitar o botão (ex: não pode resetar se já estiver no padrão).

#### 2. IGameplayCommonSettings

Localização: `Interfaces/GameplayCommonSettingsInterface.h`&#x20;

Graças a essa interface, o Registry não precisa mais saber qual é a classe responsável por manter as configurações locais e compartilhadas. Ele só quer "alguém que forneça elas".

**Funções Principais:**

* `GetLocalSettings()`: Pega a referência para o `GameplaySettingsLocal` ou classes filhas.
* `GetSharedSettings()`: Pega a referência para o `GameplaySettingsShared` ou classes filhas.

Isso torna seu sistema de configurações extremamente robusto e modular, nós esperamos chamar essas funções na macro vista em [configuracoes-dinamicas.md](configuracoes-dinamicas.md "mention")
