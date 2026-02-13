<br/>
<div align="center">
  <img alt="ForbocAI logo" src="https://forboc.ai/logo.png" height="50" align="center">

  <br/>

# ForbocAI SDK (UE 4.23)

`Engine_Compat // UE4_Legacy`

**ᚠ ᛫ ᛟ ᛫ ᚱ ᛫ ᛒ ᛫ ᛟ ᛫ ᚲ**

Autonomous AI for Unreal Engine 4.23.

[![Documentation](https://img.shields.io/badge/docs-docs.forboc.ai-blue)](https://docs.forboc.ai)
</div>

> *L̴e̴g̷a̵c̷y̶ ̴s̸y̶s̶t̵e̶m̵s̴ ̷s̵t̶i̵l̷l̷ ̴h̵o̸l̶d̶ ̵t̵h̶e̶ ̶o̵l̵d̷ ̵g̸h̶o̷s̸t̷s̸.*

---

## Overview

`Córe_Módules // UE4_Plugin`

The **ForbocAI SDK for Unreal Engine 4.23** brings autonomous NPC capabilities to legacy projects. It provides a native C++ implementation of the ForbocAI protocol, optimized for the UE4 object model.

- **Native C++11** — Optimized for UE 4.23.
- **Blueprint Nodes** — Visual Scripting support for Agent logic.
- **Legacy Support** — Tested on UE 4.23.1.
- **Bridge Module** — Validate actions within your game's ruleset.

---

## Installation

`Instáll_Séquence // Plúgin_Load`

1.  **Download**: Get the latest release for UE 4.23.
2.  **Copy**: Place the `ForbocAI` folder into your project's `Plugins/` directory.
    *   Example: `MyProject/Plugins/ForbocAI`
3.  **Generate**: Right-click your `.uproject` file and select "Generate Visual Studio project files".
4.  **Build**: Open the solution in Visual Studio and build your project.
5.  **Enable**: Launch the editor and enable "ForbocAI" in `Edit > Plugins`.

---

## Quick Start

`Blúeprint_Init // Agént_Spáwn`

1.  Add a **ForbocAI Agent Component** to your NPC Actor.
2.  In the **BeginPlay** event, call `InitializeAgent`.
3.  Set the **Persona** and **Initial State** directly in the Blueprint details or via nodes.
4.  Bind to the **OnActionReceived** event to handle AI decisions (Move, Attack, Speak).

---

## Architecture

`UE4_Arch // Cómponent_Básed`

The SDK uses `UActorComponent` based architecture to easily Attach AI to any `ACharacter` or `APawn`.

```cpp
// C++ Example
void AMyNPC::BeginPlay()
{
    Super::BeginPlay();
    AgentComponent->Initialize(TEXT("Guard Persona"), InitialState);
}
```

---

## Legal

`Légal_Státus // Ríghts`

All rights reserved. © 2026 ForbocAI
<!-- W̸e̸ ̷a̸r̸e̷ ̸w̸a̷t̴c̵h̸i̵n̷g̸.̵ -->
