---
description: >-
  Before we start modifying the GameplayCommonUI code, we need to ensure that
  Visual Studio has all the necessary tools to compile Unreal Engine.
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

# Setting Up the C++ Development Environment

{% hint style="warning" %}
This step is optional if you are only using the **GameplayCommonUI Module**, but if you want to use the **GameplayCommonSettings Module**, this step becomes mandatory.
{% endhint %}

#### Step 1: Downloading Visual Studio

If you don't already have it, download Visual Studio 2022 or 2026 (the Community version is free and sufficient).

1. Go to: [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
2. Download the Community 2022 or 2026 installer.
3. Run `VisualStudioSetup.exe`.

#### Step 2: The Workload (The Critical Step)

When you open the installer, you will see a screen with several boxes to check. **Do not just install the basic editor.**

1. Scroll down until you find the **Gaming category.**
2. Check the box: **Game development with C++.**

When you check this, look at the right side panel (“Installation details”). Make sure the following optional items are checked:

* \[x] **C++ profiling tools**
* \[x] **C++ AddressSanitizer** (Useful for debugging memory crashes)
* \[x] **Windows 10 SDK** (or 11, depending on your OS)
* \[x] **Unreal Engine installer** (Optional, but recommended as it installs integration tools)
* \[x] **IDE support for Unreal Engine** (Essential for viewing Blueprints within VS)

Click Install/Modify and wait (it's a large download, so grab a coffee ☕).

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

#### Step 3: Installing the “UnrealVS” Extension (Epic's Secret)

Epic Games includes an official extension for Visual Studio that makes life much easier (it allows you to run the game directly from VS with arguments, view build shortcuts, etc.), **but it is not installed automatically.**

1. Go to the folder where your Unreal Engine is installed.
   * Usually: `C:\Program Files\Epic Games\UE_5.X\Engine\Extras\UnrealVS\VS2022\`
2. Double-click the **`UnrealVS.vsix`** file.
3. Follow the installer to add it to your Visual Studio 2022.

#### Step 4: Configuring Unreal Engine to use VS

Now that VS is ready, we need to tell Unreal to use it.

1. Open your Unreal Project.
2. Go to **Edit** > **Editor Preferences**.
3. In the search bar, type: **Source Code**.
4. In **Source Code Editor**, change from “Null” or “**Visual Studio Code**” to **Visual Studio 2022** or **2026**.

#### Step 5: Generating Project Files

Whenever you add a new C++ class or a Plugin (such as `GameplayCommonUI`), you need to update the Visual Studio solution.

1. Close Visual Studio.
2. Go to your project folder (using Windows Explorer).
3. Right-click on the .uproject file. `.uproject`.
4. Select **Generate Visual Studio project files**.
5. A small black window will open and close quickly.
6. Now open the `.sln` (Solution) file that appeared (or was updated).

#### Step 6: The First Build

1. With the `.sln` open in Visual Studio.
2. Look at the top bar. Make sure the configuration is set to “**Development Editor**” and the platform is set to “**Win64.”**
3. In the “**Solution Explorer**” (right pane), right-click on Your Project Name (not the Engine folder) and select Build.

#### Golden Tips for Survival

* **Slow IntelliSense:** VS may take a while to “read” all of Unreal's code the first time. Be patient if the code colors take a while to appear.
* **Include Error**: If VS underlines `#include “Something.h”` in red, but the Build works, it's just IntelliSense lagging. Trust the Build Output, not the red lines in the editor.
* **Live Coding**: Unreal 5 uses Live Coding (Ctrl+Alt+F11 with the editor open). For small changes in `.cpp`, use this. To change `.h` headers or add new classes, close the editor and compile through VS (**Ctrl+Shift+B**).

For more information on installing Visual Studio, feel free to visit the official Epic Games website on the subject: [Support Link](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine)
