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

The **ForbocAI SDK for Unreal Engine 4.23** brings autonomous NPC capabilities to legacy projects. A native C++ plugin implementing the ForbocAI neuro-symbolic AI protocol, written in strict **Functional C++11**.

- **Strict Functional C++11** — No classes, factory functions only, free functions for all operations.
- **Blueprint Nodes** — Visual Scripting support via Agent/Bridge/Memory components *(planned)*.
- **Legacy Support** — Tested on UE 4.23.1.
- **Bridge Module** — Validate actions within your game's ruleset.

### Modules

| Module | Purpose |
|--------|---------|
| **AgentModule** | Agent creation (factories), state management, AI processing |
| **BridgeModule** | Neuro-symbolic action validation against game rules |
| **MemoryModule** | Immutable memory store with embedding-based recall |
| **SoulModule** | Portable identity serialization (JSON import/export) |
| **CLIModule** | HTTP-based API operations for CLI commands |
| **Commandlet** | UE command-line interface for verification and admin |
| **functional_core.hpp** | Pure C++11 FP library: Maybe, Either, Currying, Lazy, Pipeline, Composition |

---

## Installation

`Instáll_Séquence // Plúgin_Load`

1.  **Download**: Get the latest release for UE 4.23.
2.  **Copy**: Place the `ForbocAI_SDK` folder into your project's `Plugins/` directory.
    *   Example: `MyProject/Plugins/ForbocAI_SDK/`
3.  **Generate**: Right-click your `.uproject` file and select "Generate Visual Studio project files".
4.  **Build**: Open the solution in Visual Studio and build your project.
5.  **Enable**: Launch the editor and enable "ForbocAI SDK" in `Edit > Plugins`.

---

## Quick Start

`Fáctory_Init // Agent_Créate`

**C++ — Using Factories and Free Functions (Strict FP):**

```cpp
#include "AgentModule.h"

// 1. Create an agent via factory function (no constructors, no classes)
FAgentConfig Config;
Config.Persona = TEXT("Nomad Trader");
Config.ApiUrl = TEXT("https://api.forboc.ai");

const FAgent Trader = AgentFactory::Create(Config);

// 2. Process input via free function — returns a new response value
const FAgentResponse Response = AgentOps::Process(
    Trader, TEXT("Tell me about the wastelands"), {});

// 3. Functional update — returns a NEW agent (original unchanged)
const FAgentState NewState = TypeFactory::AgentState(TEXT("Wary"));
const FAgent UpdatedTrader = AgentOps::WithState(Trader, NewState);

// 4. Export to portable soul
const FSoul Soul = AgentOps::Export(UpdatedTrader);
```

**Blueprint (planned — see [TODO.md](./TODO.md), Phase 13):**

1.  Add a **ForbocAI Agent Component** to your NPC Actor.
2.  In **BeginPlay**, call `InitializeAgent`.
3.  Set the **Persona** and **Initial State** in the Blueprint details panel.
4.  Bind to the **OnActionReceived** event to handle AI decisions.

---

## Command Line Interface

`CLI // Verification`

The SDK includes a built-in Commandlet for verification and administration.

```bash
# Windows
UE4Editor-Cmd.exe "C:\Path\To\Your.uproject" -run=ForbocAI_SDK -Command=doctor

# macOS
"/Path/To/UE4Editor-Cmd" "/Path/To/Your.uproject" -run=ForbocAI_SDK -Command=doctor
```

**Commands:**
*   `doctor`: Check API connection status.
*   `agent_list`: List active agents.
*   `agent_create -Persona="..."`: Create a new agent.
*   `agent_process -Id="..." -Input="..."`: Interact with an agent.
*   `soul_export -Id="..."`: Export an agent's soul.

---

## Architecture

`UE4_Arch // Fúnctional_C++11`

ForbocAI enforces **strict Functional Programming** in C++11. The entire SDK contains **no classes** (except UE-required `UCLASS` for the Commandlet). All code follows these rules:

| Rule | Pattern |
|------|---------|
| **Data** | `struct` only — public, immutable where possible, no member functions |
| **Construction** | Factory functions in namespaces (`AgentFactory::Create`, `TypeFactory::Soul`) |
| **Operations** | Free functions in namespaces (`AgentOps::Process`, `MemoryOps::Recall`) |
| **Updates** | Copy-on-write — always return a new value (`AgentOps::WithState`) |
| **Error handling** | `Maybe<T>` / `Either<E,T>` monads via free functions (`func::fmap`, `func::mbind`) |
| **Chaining** | `func::pipe(x) \| f \| g` or `func::compose(f, g)` |

**Key references:**
- **[Functional C++11 Guide](./C++11-FP-GUIDE.md)** — Patterns, rules, and examples
- **[`functional_core.hpp`](./Plugins/ForbocAI_SDK/Source/ForbocAI_SDK/Public/Core/functional_core.hpp)** — Production FP library
- **[Port TODO](./TODO.md)** — Detailed porting plan from UE 5.7 to UE 4.23
- **[`style-guide.md`](./style-guide.md)** — Aesthetic protocols

---

## License

`Légal_Státus // Ríghts`

All rights reserved. © 2026 ForbocAI. See [LICENSE](./LICENSE) for full details.
<!-- W̸e̸ ̷a̸r̸e̷ ̸w̸a̷t̴c̵h̸i̵n̷g̸.̵ -->
