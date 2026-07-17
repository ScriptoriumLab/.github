# 🏛️ Architecture Evolution

> “Architecture is about the important stuff. Whatever that is.” — Ralph Johnson
>
> Scriptorium’s architecture was not built in a day. It evolved organically—driven by a deeper understanding of Windows TSF, an obsession with stability, and the need for cross-platform reusability.

## Phase 1: The Monolithic Era

**Version: V0.1 – V0.1.1**

In the early stage of the project, the primary goal was to validate TSF (Text Services Framework) feasibility — simply **“getting text input working in Notepad.”** The architecture at this point was a classic in-process monolithic DLL.

![Architecture V0.1](assets/Modian%20Architecture%20V0.1.png)
*(Scriptorium V0.1)*

![Architecture V0.1.1](assets/Modian%20Architecture%20V0.1.1.png)
*(Scriptorium V0.1.1)*

### Key Characteristics

- **Single DLL**: All logic — COM registration, TSF handling, and input engine — lived in one module.
- **Direct Calls**: The infrastructure layer directly invoked core logic, resulting in tight coupling and blurred boundaries.

### Pain Points

1. **Fragility**: A crash in the input engine would terminate the host application (Word, Notepad, etc.).
2. **System Leakage**: Windows TSF COM logic leaked into core logic, making cross-platform reuse difficult.

---

## Phase 2: Layered Architecture

**Version: V0.1.2 – V0.1.3**

To improve maintainability, a layered architecture was introduced. The goal was to separate OS-specific complexity from core business logic.

![Architecture V0.1.2](assets/Modian%20Architecture%20V0.1.2.png)
*(Scriptorium V0.1.2)*

![Architecture V0.1.3](assets/Modian%20Architecture%20V0.1.3.png)
*(Scriptorium V0.1.3)*

### Key Changes

- **Manager Layer Introduced** -- A new layer between Infra and Core:
    - Infra handles TSF/COM interaction
    - Manager coordinates business logic
    - Core becomes platform-agnostic
- **Early Dependency Inversion** -- Interfaces introduced to reduce direct coupling.

---

## Phase 3: Multi-Process Architecture

**Version: V1.0 (Initial IPC Design)**

As development progressed, layering alone could not solve stability issues. Process isolation became necessary.

The architecture evolved from a monolithic DLL into a **Client–Server model**.

![Architecture V1.0 IPC](assets/Modian%20Architecture%20V1.0%20Draft.png)
*(Scriptorium V1.0 Draft)*

### Key Changes

- **Scriptorium-Brush (Client)**: Lightweight DLL running inside host process
- **Scriptorium-Inkstone (Server)**: Core logic moved to standalone background process
- **IPC Communication**: Named Pipes introduced for inter-process communication

### Benefits

#### Crash Isolation

A crash in IME core no longer terminates host applications.

#### UI Responsiveness

Heavy dictionary loading and processing moved off UI thread.

---

## Phase 4: Clean Architecture

**Version: v1.0 (Refined)**

After establishing the multi-process model, dependencies were reorganized to follow the **Dependency Rule**.

![Clean Architecture V1.0](assets/Modian%20Architecture%20V1.0.png)
*(Scriptorium V1.0)*

### Final Structure

#### Inward Dependencies

Outer layers depend on inner layers only.

#### DTO-Based Communication

Protocols defined as pure data structures, decoupling transport from business logic.

#### Onion Architecture

- **Core**: Domain logic
- **Manager**: Application logic and use cases
- **Infra**: Implementation details (IPC, TSF, logging)

---

## Phase 5: Stateless UI Process

After introducing the multi-process architecture, the IME core logic was moved out of the TSF DLL into a dedicated server process. However, the candidate window UI was still coupled with the core logic, introducing new issues.

### Pain Points

1. UI logic tightly coupled with business logic
2. UI crashes may affect core stability
3. Difficult to adopt modern UI frameworks
4. Hard to reuse UI across platforms
5. Risk of UI and core state inconsistency

![Clean Architecture V1.1.0](assets/Modian%20Architecture%20V1.1.0.png)
*(Scriptorium V1.1.0)*

### Key Changes

We further split the UI into an independent process: Scriptorium-Ink
- UI runs as a separate process
- Core server becomes single source of truth
- UI becomes stateless renderer
- State synchronized via IPC
- User actions sent back to core

Architecture evolves into:

TSF Client → Core Server → UI Renderer

### Design Principles

**Stateless UI**

UI does not own business state. It renders based on RenderState.

**Unidirectional Data Flow**

Core → UI : RenderState
UI → Core : UserAction

**Crash Isolation**

UI crashes do not affect IME core.

**Cross-platform UI**

Allows using modern web-based UI frameworks.

### Benefits

- Full decoupling between UI and core
- UI crash isolation
- Supports modern UI frameworks (Tauri / Web) and allows switching rendering implementations depending on performance or platform requirements (e.g. WinUI, ImGui, native UI).
- Cross-platform UI reuse
- Clear data flow and state management
- Foundation for plugin-based UI extensions

---

## Summary

From **V0.1** to **V1.0**, Scriptorium demonstrates a key principle:

> Good architecture evolves; it is not designed upfront.

The IPC architecture was introduced only when complexity and stability requirements justified it. This allowed rapid early development while enabling long-term robustness.
