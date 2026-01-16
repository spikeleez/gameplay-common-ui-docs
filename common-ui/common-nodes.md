---
description: >-
  In this module, you can view all the common nodes that we can use with this
  system.
icon: share-nodes
---

# Common Nodes

#### Push Activatable Widget for Class / Tag

These nodes are used to “push” Widgets to a specific Stack Layer, which activates the Widget and adds it to the Viewport.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

#### Pop Widgets

These three functions are used to remove a widget from a stack.

* **Pop Single Widget**: This function removes a single widget from the stack using its reference.
* **Pop Widgets from Layout**: This function removes all widgets from all stack layers.
* **Pop Widgets from Layer**: This function removes all widgets from a specific Stack Layer.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### Show Confirmations:

Each of these functions is responsible for pushing a Confirmation Dialog to the Modal Layer.

* **Show Confirmation (OK)**: This function pushes a Confirmation Dialog that only has the OK option.
* **Show Confirmation (OK/Cancel)**: This function pushes a Confirmation Dialog that has the OK and Cancel options.
* **Show Confirmation (Yes)**: This function pushes a Confirmation Dialog that only has the Yes option
* **Show Confirmation (Yes/No)**: This function pushes a Confirmation Dialog that has the Yes and No options.
* **Show Confirmation (Yes/No/Cancel)**: This function pushes a Confirmation Dialog that has the Yes, No, and Cancel options.
* **Show Confirmation Custom**: This function pushes a customized Confirmation Dialog that you can register in **Project Settings -> Gameplay Common UI Settings -> Registered Dialog Descriptors**.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

#### Gameplay Common UI Subsystem:

This is the main subsystem of the system. It stores some information and executes specific logic in C++ and Blueprint, providing a way for you to access the current `GameplayCommonUIPolicy` to check/read information from `GameplayPrimaryLayout`.

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

#### Getters & Helpers:

* **Get Local Player from Controller**: A quick way to access the Local Player from a Player Controller.
* **Get Owning Player Input Type**: This returns the user's current Input Type, which can be Mouse and Keyboard, Gamepad, or Touch.
* **Is Owning Player Using Gamepad**: This function checks if the player is using a Gamepad.
* **Is Owning Player Using Touch**: This function checks if the player is using Touch.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
