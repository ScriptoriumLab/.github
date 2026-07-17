# Scriptorium IME

Scriptorium is a modern, decoupled, cross-platform Chinese Input Method Editor (IME) framework.

It explores how system-level software such as IMEs can be designed using Clean Architecture and Multi-Process Design, addressing common issues in traditional IME implementations:
- Tight coupling between OS APIs and core logic
- Poor cross-platform reuse
- Crash propagation to host applications
- Hard-to-extend engine architecture

Inspired by the “Four Treasures of the Study” (文房四宝), Scriptorium separates responsibilities into independent components while keeping the core logic portable.

👉 See [Architecture Evolution](ARCHITECTURE_EVOLUTION.md).

# Architecture Overview

![Modian Architecture](assets/Modian%20Architecture%20V1.1.0.png)
*(Scriptorium V1.0 Clean Architecture)*

Scriptorium decouples the IME into four independent cooperating components:

## Scriptorium-Brush (The Pen)

**OS Driver / Adapter**

Role
Handles OS-specific events, intercepts keystrokes, and commits text to the host application.

Design
OS-specific thin client designed for minimal logic and crash isolation.

Stack
C++23, Windows TSF (planned macOS IMK)

## Scriptorium-Inkstone (The Inkstone)

**Core Logic Server**

Role
Central process responsible for state management, candidate generation, and engine coordination.

Design
Single source of truth. Runs as background service to isolate failures from host apps.

Stack
C++23, IPC (Named Pipes)

## Scriptorium-Inkstick (The Inkstick)

**Engine Plugins (Planned)**

Role
Defines input conversion logic (Pinyin, Wubi, etc.)

Design
Pluggable architecture enabling independent engine development.

Stack
C ABI plugin interface (planned)

## Scriptorium-Ink (The Ink)

**Stateless UI Renderer**

Role
Displays candidate window and interaction UI.

Design
Stateless UI process communicating with core via IPC.

Stack
Rust, Tauri, Svelte

# Design Goals

- Multi-process architecture for crash isolation
- Clean architecture layering for testability
- IPC-based communication instead of shared memory
- Pluggable engine architecture
- Cross-platform core logic
- Modular CMake project structure
- Incremental architecture evolution

# Tech Stack

Core
C++23

UI
Rust + Tauri + Svelte

Build
CMake

IPC
Named Pipes (Windows)

Testing
Unit tests + Integration tests + Benchmarks

# Architecture Highlights

- Multi-process IME architecture
- TSF thin client + C++ core server
- IPC-based state synchronization
- Clean architecture layering
- Modular build system
- Iterative architecture evolution
- Designed for cross-platform support
