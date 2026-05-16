# HakOS

[![Status](https://img.shields.io/badge/status-concept%20%2F%20early%20dev-orange)]()
[![Build Status](https://img.shields.io/badge/build-not%20configured-lightgrey)]()
[![Test Coverage](https://img.shields.io/badge/coverage-N%2FA-lightgrey)]()
[![Latest Version](https://img.shields.io/badge/version-0.0.1--dev-blue)]()
[![Downloads](https://img.shields.io/badge/downloads-0-lightgrey)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()
[![Kernel](https://img.shields.io/badge/kernel-Rust-orange)]()
[![AI](https://img.shields.io/badge/AI-Qwen2.5%201B%20%2F%203B-blue)]()
[![Platform](https://img.shields.io/badge/platform-x86__64-lightgrey)]()

> An AI-native operating system built for developers. Natural language is the shell.  
> The AI runs on your machine. No API keys. No subscription. No cloud.

---

## What is HakOS?

**HakOS** is an operating system designed from the ground up for developers – with AI as a first-class citizen at every layer of the stack.

You don't type cryptic commands. You say what you want:

```bash
> open websize and run
```

HakOS searches your entire disk, finds the project named `websize`, detects it's a Node.js project, and runs `npm run dev` – automatically. If multiple matches exist, it lists them and lets you pick.

The AI that powers this runs fully **on-device** using embedded Qwen2.5 models. No internet connection. No API key. No subscription.

---

## Core Philosophy

- **For developers, by a developer** – No printer support, no parental controls, no bloat.
- **AI is the shell** – Natural language replaces the command line as the primary interface.
- **On-device AI only** – Your commands and project names never leave your machine.
- **Minimal but complete** – Everything a developer needs, nothing they don't.
- **Self‑developed stack** – Custom bootloader, kernel, drivers, filesystem, network stack – all written in Rust from scratch.

---

## Status

> ⚠️ **HakOS is currently in early concept / design phase.**  
> The repository contains the full design documentation and a working Rust scaffolding (crates, data structures, trait definitions).  
> The goal is to evolve this into a bootable, usable developer OS step by step, with heavy AI assistance.

See [`DESIGN.md`](DESIGN.md) for the complete architecture and roadmap.

---

## Architecture (planned)

```
┌──────────────────────────────────────────────┐
│         L8  Natural Language Shell           │
│  Project search · Lang detect · Auto command │
├──────────────────────────────────────────────┤
│     L7  Embedded AI Core (Qwen2.5 1B/3B)     │
│  On-device · 100+ languages · No API key     │
├──────────────────────────────────────────────┤
│       L6  GUI Compositor + Desktop           │
│  GPU-accelerated · 60fps · Developer-first   │
├──────────────────────────────────────────────┤
│      L5  Software Compatibility Layer        │
│  Native .hak packages · No Linux binary mess │
├──────────────────────────────────────────────┤
│        L4  Language Runtime Layer            │
│  Python · Node · Go · JVM · Ruby · Swift ... │
├──────────────────────────────────────────────┤
│         L3  HakFS + Network Stack            │
│  Self-developed FS · Full TCP/IP from scratch│
├──────────────────────────────────────────────┤
│      L2  IPC / Security / Power / Boot       │
│  Self-written bootloader · ACPI · Sandbox    │
├──────────────────────────────────────────────┤
│       L1  Hardware Drivers (self-developed)  │
│  Intel/AMD/NVIDIA GPU · NVMe · NIC · USB     │
├──────────────────────────────────────────────┤
│          L0  HakOS Kernel Core (Rust)        │
│  Memory · Scheduler · IRQ · SMP · Timers     │
└──────────────────────────────────────────────┘
```

> 📄 Full architecture details: [`DESIGN.md`](DESIGN.md)

---

## Features (design – in progress)

| Area | Planned capabilities |
|------|----------------------|
| **Natural Language Shell** | Intent parsing, project detection, multi-match disambiguation, NL scripting |
| **Desktop Environment** | GPU compositor, tiling+floating WM, multi‑monitor, gestures, hotkeys |
| **Terminal** | GPU‑accelerated, tabs, split panes, session restore |
| **Filesystem** | HakFS – journaled, extent‑based, inline small files, snapshots (future) |
| **Network Stack** | Full TCP/IP from scratch, TLS 1.3, DNS, DHCP, WiFi supplicant |
| **Language Runtimes** | Python, Node, Go, Rust, JVM, Ruby, Swift, R, Julia, Lua, PHP, Dart, Haskell, Elixir, Zig … |
| **Package Management** | `.hak` native packages, silent install, permission prompts only on escalation |
| **AI on device** | Qwen2.5‑1B / 3B, local inference, context engine, search engine, speculative prefetch |
| **Settings UI** | Full developer‑focused control panel (see DESIGN.md §4) |

---

## Natural Language Shell – How It Works

```text
User:  "open websize and run"
                  ↓
    AI parses intent + entity
    intent=open_and_run  project=websize
                  ↓
    Search entire disk for "websize"
                  ↓
    Single match found
    ├── Detect language → Node.js (package.json found)
    ├── Map "run" → npm run dev
    └── Execute in terminal
                  ↓
    Multiple matches found
    ├── [1] ~/projects/websize (Node.js)
    ├── [2] ~/archive/websize-old (Node.js)
    └── Which one? >
```

### Action mapping examples

| You say | Project type | Command executed |
|---|---|---|
| `open websize and run` | Node.js | `npm run dev` |
| `open server and run` | Python | `python main.py` |
| `open app and build` | Rust | `cargo build` |
| `open service and test` | Go | `go test ./...` |
| `open api and run` | Java/Maven | `mvn spring-boot:run` |
| `open project and deploy` | Docker | `docker compose up` |

---

## Embedded AI Models (on‑device)

Both models are pre‑installed on the HakOS image. Switch instantly in Settings → AI Settings. No download required.

| Model | RAM usage | Speed | Recommended for |
|---|---|---|---|
| Qwen2.5‑1B (INT4/INT8) | ~800 MB | Very fast | Low‑spec devices, simple commands |
| Qwen2.5‑3B (INT4/INT8) | ~2 GB | Fast | Default – better accuracy |

---

## Getting Started (for contributors)

This repository currently contains the **design documentation and Rust scaffolding**. To explore the codebase:

```bash
git clone https://github.com/snowphn/HakOS.git
cd HakOS
cargo build --workspace
```

To see the full design and roadmap: open [`DESIGN.md`](DESIGN.md).

> **Note:** The codebase is under active construction. Most modules are still skeletons or `todo!()` stubs – they define the API shape but are not yet executable.  
> The next phase is to replace stubs with real implementations, one small function at a time, using AI assistance.

---

## Roadmap (high‑level)

| Phase | Focus | Target |
|-------|-------|--------|
| **1** | NL Shell prototype (on existing OS) | Validating the concept |
| **2** | Kernel bootstrap (bootloader, memory, interrupts) | Boot to `HakOS` message |
| **3** | Usable core (HakFS, network stack, drivers) | Boot to shell, run native `.hak` apps |
| **4** | Desktop (compositor, WM, apps) | Graphical desktop |
| **5** | Full integration (AI, all runtimes, public alpha) | First public release |

Detailed timeline: [`DESIGN.md` §5](DESIGN.md).

---

## Contributing

HakOS is open source and welcomes contributors.  
If you're interested in OS development, Rust, or AI, feel free to:

- Open an issue for discussion or bug reports
- Submit pull requests (code, docs, tests)
- Star the repository to show support

See [`CONTRIBUTING.md`](CONTRIBUTING.md) (coming soon) for guidelines.

---

## License

MIT License – built in public, for everyone.

---

**Built by one developer + AI** – a middle school student exploring what's possible when you combine modern tools with a bold vision.

---

*HakOS is in early concept / development phase. Stars, issues, and constructive feedback are welcome.*
