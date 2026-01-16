---
description: >-
  Nesse módulo você pode visualizar todos os nodes comuns que podemos utilizar
  com esse sistema.
icon: share-nodes
---

# Common Nodes

#### Push Activatable Widget for Class / Tag

Esses nodes são utilizados para fazer um "Push" nas Widgets para uma Stack Layer específica, isso faz com que a Widget seja ativada e adicionada ao Viewport.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

#### Pop Widgets

Essas 3 funções são utilizadas para remover uma Widget de um Stack.

* **Pop Single Widget**: Essa função remove uma única widget do stack utilizando a referência dela.
* **Pop Widgets from Layout**: Essa função remove todas as widgets de todos os stack layers.
* **Pop Widgets from Layer**: Essa função remove todas as widgets de uma Stack Layer específica.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### Show Confirmations:

Cada uma dessas funções tem a responsabilidade de fazer um Push de um Confirmation Dialog para a Layer de Modal.

* **Show Confirmation (OK)**: Essa função faz um Push de um Confirmation Dialog que tem apenas a opção de **Ok**.
* **Show Confirmation (OK/Cancel)**: Essa função faz um Push de un Confirmation Dialog que tem as opções de **Ok e Cancel**.
* **Show Confirmation (Yes)**: Essa função faz um Push de um Confirmation Dialog que tem apenas a opção de **Yes**.
* **Show Confirmation (Yes/No)**: Essa função faz um Push de um Confirmation Dialog que tem as opções de **Yes e No.**
* **Show Confirmation (Yes/No/Cancel)**: Essa função faz um Push de um Confirmation Dialog que tem as opções de **Yes, No e Cancel.**
* **Show Confirmation Custom**: Essa função faz um Push de um Confirmation Dialog customizado que você pode registrar no **Project Settings -> Gameplay Common UI Settings -> Registered Dialog Descriptors.**

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

#### Gameplay Common UI Subsystem:

Esse é o subsystem principal do sistema, ele guarda algumas informações e executa lógicas específicas no C++, e na Blueprint, expõe uma forma de você acessar a `GameplayCommonUIPolicy` atual para verificar/ler informações do `GameplayPrimaryLayout`

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

#### Getters & Helpers:

* **Get Local Player from Controller**: Uma forma rápida de acessar o Local Player de um Player Controller.
* **Get Owning Player Input Type**: Isso retorna o Input Type atual do usuário, podendo ser **Mouse e** **Keyboard**, **Gamepad** e **Touch**.
* **Is Owning Player Using Gamepad**: Essa função verifica se o player está usando Gamepad.
* **Is Owning Player Using Touch**: Essa função verifica se o player está usando Touch.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
