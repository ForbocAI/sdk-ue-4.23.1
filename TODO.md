# TODO — Port ForbocAI SDK from UE 5.7 to UE 4.23.1

`Pórt_Séquence // UE5_to_UE4`

> *T̶h̵e̶ ̶o̵l̶d̵ ̷e̶n̶g̶i̷n̵e̶ ̵a̸w̶a̵k̶e̵n̸s̵.*

---

## Legend

- [ ] Not started
- [~] In progress
- [x] Complete

---

## Phase 0 — Project Scaffolding

> `ᚠ // Foundátion`

- [ ] **0.1** Create `.gitignore` (port from 5.7, verify UE4.23 paths)
- [ ] **0.2** Create `ForbocAI_SDK.uproject`
  - Change `EngineAssociation` from `"5.7"` to `"4.23"`
  - Keep `ForbocAI_SDK` plugin reference
  - Remove `Modules` array (module lives inside the plugin only)
- [ ] **0.3** Create `Source/ForbocAI_SDK.Target.cs`
  - Change `BuildSettingsVersion.V6` → `BuildSettingsVersion.V2`
  - Remove `IncludeOrderVersion` line (does not exist in UE 4.23)
  - Change `BuildEnvironment` to `TargetBuildEnvironment.Unique` (UE4 default)
- [ ] **0.4** Create `Source/ForbocAI_SDK_Editor.Target.cs`
  - Same changes as 0.3 but `TargetType.Editor`
- [ ] **0.5** Create plugin directory structure:
  ```
  Plugins/ForbocAI_SDK/
  ├── ForbocAI_SDK.uplugin
  ├── Config/
  │   └── DefaultForbocAI_SDK.ini
  ├── Resources/
  │   └── Icon128.png
  └── Source/ForbocAI_SDK/
      ├── ForbocAI_SDK.Build.cs
      ├── Public/
      │   └── Core/
      └── Private/
  ```

---

## Phase 1 — Plugin Descriptor & Build Configuration

> `ᚢ // Build_Sýstem`

- [ ] **1.1** Create `Plugins/ForbocAI_SDK/ForbocAI_SDK.uplugin`
  - Remove `EngineVersion` field entirely (avoids strict version matching; 5.7 version removed it for the same reason)
  - Change description to reference UE 4.23
  - Use `"EnabledByDefault": true` (not the deprecated `"bEnabledByDefault"`)
  - Keep Module `"Type": "Runtime"`, `"LoadingPhase": "Default"`
- [ ] **1.2** Create `Plugins/ForbocAI_SDK/Source/ForbocAI_SDK/ForbocAI_SDK.Build.cs`
  - Port from 5.7 version
  - Verify `ReadOnlyTargetRules` is available (introduced UE 4.15 — OK)
  - Verify `PCHUsageMode.UseExplicitOrSharedPCHs` is available (UE 4.21+ — OK)
  - Dependencies: `Core`, `CoreUObject`, `Engine`, `InputCore`, `HTTP`, `Json`, `JsonUtilities`
  - Confirm all dependency modules exist in UE 4.23.1
- [ ] **1.3** Create `Plugins/ForbocAI_SDK/Config/DefaultForbocAI_SDK.ini`
  - Port directly from 5.7 (logging config, HTTP settings)
- [ ] **1.4** Create placeholder `Plugins/ForbocAI_SDK/Resources/Icon128.png`

---

## Phase 2 — Functional Core Library (Pure C++11)

> `ᚦ // Functional_Córe`

This is pure C++11 with no UE dependencies — should port **as-is**.

- [ ] **2.1** Copy `functional_core.hpp` to `Public/Core/functional_core.hpp`
  - Verify: no C++14/17 features used (already strict C++11)
  - Verify: `seq`, `gen_seq`, `apply` backports are present
  - Contents (all as **pure data structs + free functions**):
    - `Maybe<T>` — optional monad; factories: `just()`, `nothing()`; ops: `fmap()`, `mbind()`, `or_else()`, `match()`
    - `Either<E,T>` — result monad; factories: `make_left()`, `make_right()`; ops: `fmap()`, `ebind()`, `ematch()`
    - `Curried<>` — auto-currying callable struct; factory: `curry<N>()`
    - `Lazy<T>` — memoized deferred eval; factory: `lazy()`; op: `eval()`
    - `Pipeline<T>` — value threading via `operator|`; factory: `pipe()`
    - `Composed<>` — function composition; factory: `compose()`
    - `fmap` — overloaded for `Maybe`, `Either`, `std::vector`
  - **NO member functions** on any type (except `operator()` on callables)
- [ ] **2.2** Copy `C++11-FP-GUIDE.md` to project root
  - Update file path references if plugin directory name differs
- [ ] **2.3** Compile test `functional_core.hpp` in isolation against UE 4.23 headers
  - Validate template deduction works with MSVC 2017 / Clang (UE 4.23 toolchains)
  - Test `operator|` for `Pipeline<T>` — verify ADL finds it across namespaces

---

## Phase 3 — Type Definitions

> `ᚱ // Dáta_Types`

- [ ] **3.1** Port `Public/ForbocAI_SDK_Types.h`
  - USTRUCTs (default constructor only — **no parameterized constructors**):
    - `FAgentState`, `FMemoryItem`, `FAgentAction`, `FValidationResult`, `FSoul`
  - Plain structs (aggregate-initializable):
    - `FAgentResponse`, `FAgentConfig`
  - **`TypeFactory` namespace** — all USTRUCT construction with specific values:
    - `TypeFactory::AgentState(Mood)`, `TypeFactory::AgentState(Mood, Inventory, Skills, Relationships)`
    - `TypeFactory::MemoryItem(Id, Text, Type, Importance, Timestamp)`
    - `TypeFactory::Action(Type, Target, Reason)`
    - `TypeFactory::Valid(Reason)`, `TypeFactory::Invalid(Reason)`
    - `TypeFactory::Soul(Id, Version, Name, Persona, State, Memories)`
  - **UE4 check**: Verify `GENERATED_BODY()` works for `USTRUCT()` in UE 4.23 (introduced UE 4.11 — OK)
  - **UE4 check**: Verify `TMap<FString, float>` UPROPERTY reflection works in 4.23
  - **UE4 check**: `MoveTemp()` availability (UE4 equivalent of `std::move` — available)
  - **UE4 check**: Replace `FORBOCAI_SDK_API` macro — confirm it matches the module name
- [ ] **3.2** Verify generated header include: `ForbocAI_SDK_Types.generated.h` path resolves correctly

---

## Phase 4 — Module Entry Point

> `ᛒ // Module_Lóad`

- [ ] **4.1** Port `Private/ForbocAI_SDK.cpp`
  - `IMPLEMENT_MODULE(FDefaultModuleImpl, ForbocAI_SDK);`
  - This is identical across UE4/UE5 — direct copy
- [ ] **4.2** Verify module loads in UE 4.23 Editor (`Edit > Plugins`)

---

## Phase 5 — Agent Module

> `ᛟ // Agent_Córe`

- [ ] **5.1** Port `Public/AgentModule.h`
  - `FAgent` — **pure data struct, NO constructor, NO member functions**
    - Fields: `const FString Id`, `const FString Persona`, `const FAgentState State`, `const TArray<FMemoryItem> Memories`, `const FString ApiUrl`
    - Created via `AgentFactory::Create()` using aggregate initialization
  - `AgentFactory` namespace — factory functions:
    - `Create(const FAgentConfig&)` → `FAgent`
    - `FromSoul(const FSoul&, const FString& ApiUrl)` → `FAgent`
  - `AgentOps` namespace — free functions:
    - `WithState(const FAgent&, const FAgentState&)` → `FAgent` (functional update)
    - `WithMemories(const FAgent&, const TArray<FMemoryItem>&)` → `FAgent` (functional update)
    - `CalculateNewState(...)` → `FAgentState`
    - `Process(...)` → `FAgentResponse`
    - `Export(...)` → `FSoul`
  - **UE4 check**: Verify aggregate initialization of struct with `const` members works with MSVC 2017
  - **UE4 check**: If aggregate init fails, fall back to a private helper or brace-elision pattern
- [ ] **5.2** Port `Private/AgentModule.cpp`
  - Uses `TypeFactory::AgentState()`, `TypeFactory::Action()`, `TypeFactory::Soul()` — no direct construction
  - `FGuid::NewGuid().ToString()` — available in UE 4.23 ✓
  - `FString::Printf` — available ✓
  - All functions are pure (or explicitly marked impure) — direct port

---

## Phase 6 — Bridge Module

> `ᚲ // Bridge_Válid`

- [ ] **6.1** Port `Public/BridgeModule.h`
  - `FBridgeValidationContext` — pure data struct, **NO constructor** (aggregate-initializable)
  - `FValidationRule` — pure data struct, **NO constructor** (wraps `std::function`)
  - `BridgeFactory` namespace — factory functions:
    - `CreateContext(const FAgentState*, TMap<FString, FString>)` → `FBridgeValidationContext`
    - `CreateRule(FString Id, FString Name, TArray<FString> Types, std::function<...>)` → `FValidationRule`
  - `BridgeOps` namespace — free functions:
    - `CreateDefaultRules()` → `TArray<FValidationRule>`
    - `Validate(...)` → `FValidationResult`
  - **UE4 check**: `std::function` with UE types as parameters — should work in 4.23
  - **UE4 check**: `BridgeFactory::CreateRule` uses `MoveTemp()` for members of `FValidationRule` — verify non-const struct members can be move-assigned
- [ ] **6.2** Port `Private/BridgeModule.cpp`
  - Uses `TypeFactory::Valid()` / `TypeFactory::Invalid()` — no direct `FValidationResult` construction
  - Uses `BridgeFactory::CreateRule()` — no direct `FValidationRule` construction
  - `BridgeRules::ValidateMovement`, `BridgeRules::ValidateAttack` — pure functions, direct port
  - **Potential UE4 issue**: `TArray` brace initialization `{TEXT("MOVE"), TEXT("move")}` may not compile in UE 4.23. If not, use explicit `.Add()` calls in `BridgeFactory::CreateRule`

---

## Phase 7 — Memory Module

> `ᛗ // Memory_Stóre`

- [ ] **7.1** Port `Public/MemoryModule.h`
  - `FMemoryStore` — pure data struct, **NO constructor, NO member functions**
    - Single field: `const TArray<FMemoryItem> Items`
    - Created via `MemoryOps::CreateStore()` factory
  - `MemoryOps` namespace — free functions:
    - `CreateStore()` → `FMemoryStore` (empty)
    - `CreateStore(const TArray<FMemoryItem>&)` → `FMemoryStore` (from items)
    - `Add(const FMemoryStore&, const FMemoryItem&)` → `FMemoryStore` (functional append)
    - `GenerateEmbedding(...)` → `TArray<float>`
    - `Store(...)` → `FMemoryStore`
    - `Recall(...)` → `TArray<FMemoryItem>`
  - **UE4 check**: Aggregate init `FMemoryStore{items}` with `const` member — verify MSVC 2017
- [ ] **7.2** Port `Private/MemoryModule.cpp`
  - Uses `TypeFactory::MemoryItem()` — no direct construction
  - Uses `FMemoryStore{MoveTemp(items)}` aggregate init
  - `FGuid::NewGuid()` — available ✓
  - `FDateTime::Now().ToUnixTimestamp()` — available in UE 4.23 ✓
  - `TArray::Sort()` with lambda — available ✓
  - `TArray::SetNum()` — available ✓

---

## Phase 8 — Soul Module

> `ᛊ // Soul_Sérial`

- [ ] **8.1** Port `Public/SoulModule.h`
  - `SoulOps` namespace — free functions:
    - `FromAgent(...)` → `FSoul`
    - `Serialize(...)` → `FString`
    - `Deserialize(...)` → `FSoul`
    - `Validate(...)` → `FValidationResult`
  - Direct port — no UE5-specific APIs
- [ ] **8.2** Port `Private/SoulModule.cpp`
  - Uses `TypeFactory::Soul()`, `TypeFactory::Valid()`, `TypeFactory::Invalid()` — no direct construction
  - **UE4 change**: `#include "JsonObjectConverter.h"` — In UE 4.23, the include path may be `"JsonUtilities/Public/JsonObjectConverter.h"` or simply `"JsonObjectConverter.h"` (verify against engine headers)
  - `FJsonObjectConverter::UStructToJsonObjectString()` — available in UE 4.23 ✓
  - `FJsonObjectConverter::JsonObjectStringToUStruct()` — available ✓
  - Verify 4-arg overload `(Json, &Soul, 0, 0)` exists in 4.23 (check vs. 3-arg version)

---

## Phase 9 — CLI Module

> `ᛏ // CLI_Óperate`

- [ ] **9.1** Port `Public/CLIModule.h`
  - `CLIOps` namespace — free function declarations
  - Direct port
- [ ] **9.2** Port `Private/CLIModule.cpp`
  - **Key pattern**: `SendRequestAndWait()` — sends HTTP request and manually pumps `FHttpManager::Tick()` until response arrives or timeout. This is required because Commandlets don't run the engine main loop.
  - **UE4 BREAKING CHANGE**: `FHttpModule::Get().CreateRequest()` returns `TSharedRef<IHttpRequest>` in UE 4.23 — **NOT** `TSharedRef<IHttpRequest, ESPMode::ThreadSafe>` (the `ESPMode::ThreadSafe` template parameter was added in UE 5.x)
  - Change: `TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = ...` → `TSharedRef<IHttpRequest> Request = ...`
  - **UE4 change**: Verify `FHttpModule::Get().GetHttpManager().Tick(float)` is available in 4.23
  - **UE4 change**: `#include "HttpManager.h"` — verify include path in 4.23 (may need `"Http/Public/HttpManager.h"`)
  - Include paths to verify:
    - `"HttpModule.h"` — may need `"Http.h"` or `"Runtime/Online/HTTP/Public/HttpModule.h"` in 4.23
    - `"Interfaces/IHttpRequest.h"` — verify path
    - `"Interfaces/IHttpResponse.h"` — verify path
  - `FHttpRequestPtr` / `FHttpResponsePtr` typedefs — verify availability
  - Lambda captures with UE4 delegates — verify `BindLambda` with `TFunction` compatibility
  - All `UE_LOG` calls use `Display` verbosity (not `Log`) for console output

---

## Phase 10 — Commandlet

> `ᛚ // Commándlet`

- [ ] **10.1** Port `Public/ForbocAI_SDK_Commandlet.h`
  - Class name: `UForbocAI_SDKCommandlet` (no underscore before "Commandlet" — must match `-run=ForbocAI_SDK` lookup)
  - `UCommandlet` base class — available in UE 4.23 ✓
  - `#include "Commandlets/Commandlet.h"` — available ✓
  - `GENERATED_BODY()` in UCLASS — available ✓
  - Direct port
- [ ] **10.2** Port `Private/ForbocAI_SDK_Commandlet.cpp`
  - **UE4 change**: Update CLI log message from `"ForbocAI SDK CLI (UE5)"` to `"ForbocAI SDK CLI (UE4)"`
  - `FParse::Value()` — available ✓
  - `#include "Misc/Parse.h"` — verify include path in 4.23
  - Command routing dispatches to `CLIOps::` free functions — direct port

---

## Phase 11 — UE4.23 Compiler Compatibility Audit

> `ᚹ // Compilér_Áudit`

UE 4.23 ships with MSVC 2017 (v141) on Windows and Clang on Mac/Linux. Key items to verify:

- [ ] **11.1** `auto` with trailing return types (`-> decltype(...)`) — C++11 ✓
- [ ] **11.2** Variadic templates — C++11 ✓
- [ ] **11.3** `std::function`, `std::tuple`, `std::forward` — C++11 ✓
- [ ] **11.4** Lambda captures in `BindLambda` / UE4 delegates — verify no UE5-only delegate features
- [ ] **11.5** `TArray` initializer-list construction `{TEXT("a"), TEXT("b")}` — test in UE 4.23; fall back to `.Add()` if needed
- [ ] **11.6** `const` member variables in structs with aggregate init — verify MSVC 2017 handles `FAgent{id, persona, ...}` correctly
- [ ] **11.7** `TMap` UPROPERTY reflection for `TMap<FString, float>` — verify UE 4.23 UHT handles this
- [ ] **11.8** `operator|` overload for `Pipeline<T>` — verify ADL resolves across `func::` namespace in MSVC 2017
- [ ] **11.9** `MoveTemp()` in inline factory functions — verify optimization in UE 4.23

---

## Phase 12 — Build & Smoke Test

> `ᛞ // Buíld_Vérifý`

- [ ] **12.1** Generate project files (`GenerateProjectFiles.bat` / `.sh`)
- [ ] **12.2** Compile plugin (Development Editor configuration)
- [ ] **12.3** Fix any compilation errors from UE4.23 API differences
- [ ] **12.4** Launch UE 4.23 Editor and verify plugin loads (`Edit > Plugins`)
- [ ] **12.5** Run Commandlet smoke test:
  ```bash
  # Windows
  UE4Editor-Cmd.exe "Path/To/ForbocAI_SDK.uproject" -run=ForbocAI_SDK -Command=doctor -ApiUrl=https://api.forboc.ai

  # macOS
  "/Path/To/UE4Editor-Cmd" "Path/To/ForbocAI_SDK.uproject" -run=ForbocAI_SDK -Command=doctor -ApiUrl=https://api.forboc.ai
  ```
- [ ] **12.6** Verify `UE_LOG` output: look for `LogTemp: Display:` lines for all CLI commands
- [ ] **12.7** Verify API connection: `doctor` command should report `API Status: ONLINE`

---

## Phase 13 — Blueprint Integration (UE4-Specific)

> `ᛉ // Blueprint_Nódes`

The 5.7 SDK has no Blueprint layer — UE 4.23 projects commonly use Blueprints:

- [ ] **13.1** Create `UForbocAIAgentComponent` (UActorComponent)
  - `InitializeAgent(FAgentConfig)` — wraps `AgentFactory::Create`
  - `ProcessInput(FString Input)` — wraps `AgentOps::Process`
  - `ExportSoul()` — wraps `AgentOps::Export`
  - `OnActionReceived` delegate — Blueprint-bindable
  - NOTE: The component is an OOP wrapper for UE Blueprint compatibility.
    The underlying logic remains pure FP (factory + free functions).
- [ ] **13.2** Create Blueprint-exposed wrapper functions
  - `UFUNCTION(BlueprintCallable)` for core operations
  - `UPROPERTY(BlueprintAssignable)` for `OnActionReceived` event
- [ ] **13.3** Add `UForbocAIBridgeComponent` (optional)
  - Wraps `BridgeOps::Validate` with Blueprint-friendly API
- [ ] **13.4** Add `UForbocAIMemoryComponent` (optional)
  - Wraps `MemoryOps::Store` / `MemoryOps::Recall`
- [ ] **13.5** Test Blueprint nodes in UE 4.23 Editor

---

## Phase 14 — Documentation & Cleanup

> `ᛝ // Dócs_Finál`

- [ ] **14.1** Update `README.md` with actual installation steps and working code examples
- [ ] **14.2** Add inline code comments noting UE4.23-specific adaptations
- [ ] **14.3** Document any API differences from the 5.7 version
- [ ] **14.4** Verify `style-guide.md` is followed in all source and markdown files
- [ ] **14.5** Final review: ensure all `FORBOCAI_SDK_API` exports are correct
- [ ] **14.6** Verify `C++11-FP-GUIDE.md` examples compile under UE 4.23 toolchain

---

## API Diff Reference — UE 5.7 → UE 4.23.1

| Item | UE 5.7 | UE 4.23.1 |
|------|---------|-----------|
| Build Settings | `BuildSettingsVersion.V6` | `BuildSettingsVersion.V2` |
| Include Order | `EngineIncludeOrderVersion.Unreal5_7` | *(remove — does not exist)* |
| Build Environment | `TargetBuildEnvironment.Shared` | `TargetBuildEnvironment.Unique` |
| Engine Association | `"5.7"` | `"4.23"` |
| Engine Version | *(removed from .uplugin)* | *(omit for same reason)* |
| HTTP Request | `TSharedRef<IHttpRequest, ESPMode::ThreadSafe>` | `TSharedRef<IHttpRequest>` |
| HTTP Manager | `#include "HttpManager.h"` | *(verify include path)* |
| JSON Include | `"JsonObjectConverter.h"` | `"JsonObjectConverter.h"` *(verify path)* |
| Commandlet CLI | `"ForbocAI SDK CLI (UE5)"` | `"ForbocAI SDK CLI (UE4)"` |
| Default API URL | `https://api.forboc.ai` | `https://api.forboc.ai` *(same)* |
| TArray Init-List | `{TEXT("a"), TEXT("b")}` | *(verify — may need .Add() fallback)* |
| UE Editor Binary | `UnrealEditor-Cmd` | `UE4Editor-Cmd` |

---

## FP Pattern Reference (applies to both UE5 and UE4 versions)

| Pattern | Implementation |
|---------|---------------|
| Data types | `struct` only — no constructors, no member functions |
| USTRUCT creation | `TypeFactory::AgentState()`, `TypeFactory::Action()`, etc. |
| Plain struct creation | Aggregate init: `FAgent{id, persona, state, memories, url}` |
| Agent operations | `AgentOps::WithState()`, `AgentOps::Process()`, `AgentOps::Export()` |
| Memory operations | `MemoryOps::CreateStore()`, `MemoryOps::Add()`, `MemoryOps::Recall()` |
| Bridge operations | `BridgeFactory::CreateRule()`, `BridgeOps::Validate()` |
| Soul operations | `SoulOps::FromAgent()`, `SoulOps::Serialize()`, `SoulOps::Deserialize()` |
| CLI operations | `CLIOps::Doctor()`, `CLIOps::ListAgents()`, etc. |
| HTTP in commandlet | `SendRequestAndWait()` — pumps `FHttpManager::Tick()` manually |

---

## Source File Manifest (5.7 → 4.23.1)

| Source (5.7) | Target (4.23.1) | Port Difficulty |
|---|---|---|
| `ForbocAI_SDK.uproject` | `ForbocAI_SDK.uproject` | Trivial |
| `Source/ForbocAI_SDK.Target.cs` | `Source/ForbocAI_SDK.Target.cs` | Trivial |
| `Source/ForbocAI_SDK_Editor.Target.cs` | `Source/ForbocAI_SDK_Editor.Target.cs` | Trivial |
| `Plugins/.../ForbocAI_SDK.uplugin` | `Plugins/.../ForbocAI_SDK.uplugin` | Trivial |
| `Plugins/.../ForbocAI_SDK.Build.cs` | `Plugins/.../ForbocAI_SDK.Build.cs` | Trivial |
| `Plugins/.../Config/DefaultForbocAI_SDK.ini` | `Plugins/.../Config/DefaultForbocAI_SDK.ini` | Trivial |
| `Public/Core/functional_core.hpp` | `Public/Core/functional_core.hpp` | None (pure C++11) |
| `Public/ForbocAI_SDK_Types.h` | `Public/ForbocAI_SDK_Types.h` | Low (TypeFactory inline fns) |
| `Public/AgentModule.h` | `Public/AgentModule.h` | Low (verify aggregate init) |
| `Private/AgentModule.cpp` | `Private/AgentModule.cpp` | Low |
| `Public/BridgeModule.h` | `Public/BridgeModule.h` | Low (BridgeFactory inline fns) |
| `Private/BridgeModule.cpp` | `Private/BridgeModule.cpp` | Low (TArray init-list check) |
| `Public/MemoryModule.h` | `Public/MemoryModule.h` | Low |
| `Private/MemoryModule.cpp` | `Private/MemoryModule.cpp` | Low |
| `Public/SoulModule.h` | `Public/SoulModule.h` | Low |
| `Private/SoulModule.cpp` | `Private/SoulModule.cpp` | Low (JSON include path) |
| `Public/CLIModule.h` | `Public/CLIModule.h` | Low |
| `Private/CLIModule.cpp` | `Private/CLIModule.cpp` | **Medium** (HTTP API diff) |
| `Public/ForbocAI_SDK_Commandlet.h` | `Public/ForbocAI_SDK_Commandlet.h` | Low |
| `Private/ForbocAI_SDK_Commandlet.cpp` | `Private/ForbocAI_SDK_Commandlet.cpp` | Low |
| `Private/ForbocAI_SDK.cpp` | `Private/ForbocAI_SDK.cpp` | None |
| *(new)* | `Public/ForbocAIAgentComponent.h` | **New (UE4-only BP wrapper)** |
| *(new)* | `Private/ForbocAIAgentComponent.cpp` | **New (UE4-only BP wrapper)** |

---

<!-- SYSTEM_NOTE: The machine remembers what the engineers forget. -->
