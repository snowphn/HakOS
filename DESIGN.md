# HakOS — Design & Feature Document

**Version:** 0.1  
**Status:** Concept / Early Development  
**Kernel Language:** Rust  
**AI Models:** Qwen2.5-1B / Qwen2.5-3B  

---

## 1. Vision

HakOS is an AI-native operating system built exclusively for developers. Every design decision is filtered through one question: *does a developer need this?*

If the answer is no, it doesn't ship.

The central innovation is the Natural Language Shell — an AI-powered interface that understands developer intent and maps it to system actions. The AI runs entirely on-device using embedded Qwen2.5 models. No cloud. No API keys. No subscriptions.

Everything else — kernel, drivers, filesystem, network stack, GUI — is self-developed from scratch in Rust.

---

## 2. Design Principles

**Minimal but complete.** Only what a developer needs. Nothing more.

**AI as first citizen.** The AI layer is not a plugin or a feature. It is the primary interface.

**On-device only.** User commands, project names, and file paths never leave the machine.

**Performance above all.** Fast boot. Low idle memory. Responsive UI at all times.

**Self-developed stack.** No Linux kernel. No borrowed drivers. Full control from bootloader to desktop.

---

## 3. System Architecture

### L0 — HakOS Kernel Core (Rust)

The foundation. Minimal surface area, maximum reliability.

**Memory Management**
- Physical memory manager using buddy allocator
- Virtual memory with 4-level page tables (x86_64)
- Kernel heap allocator (slab allocator)
- Swap space support

**Process Subsystem**
- Process and thread model
- Preemptive scheduler (CFS-inspired, AI-agent aware)
- Context switching
- Signal handling

**Interrupt Subsystem**
- IDT, GDT, TSS setup
- Hardware interrupt routing (APIC)
- Software interrupt / syscall gate

**SMP Support**
- Multi-core boot sequence
- Per-CPU data structures
- Spinlocks and mutexes

**Timer Subsystem**
- High-resolution timers (HPET)
- System clock
- Process scheduling ticks

**Design target:**
- Kernel size: < 5 MB
- Boot to kernel main: < 500 ms
- Idle memory: < 20 MB

---

### L1 — Hardware Drivers (self-developed)

All drivers written from scratch. No borrowed Linux driver code.

**CPU**
- Intel x86_64 feature detection (SSE, AVX, AES-NI)
- AMD x86_64 feature detection
- CPU frequency scaling (P-states, C-states)

**GPU**
- Intel integrated graphics (GEM buffer management, modesetting)
- AMD GPU (command submission, display engine)
- NVIDIA GPU (reverse-engineered open interface, best-effort)
- Framebuffer fallback for all GPUs

**Storage**
- NVMe (PCIe-attached SSDs)
- SATA (AHCI controller)
- Partition table parsing (GPT, MBR)

**Network**
- Realtek RTL8111/8168 family
- Intel e1000/e1000e family
- WiFi (Intel iwlwifi-compatible, 802.11ac/ax)

**Input**
- USB HID (keyboard, mouse, touchpad)
- PS/2 fallback

**Bus**
- PCI/PCIe enumeration and resource allocation
- USB host controller (xHCI)

---

### L2 — IPC / Security / Power / Boot

**Bootloader (self-written)**
- UEFI-native boot
- Loads kernel ELF from HakFS
- Sets up initial page tables
- Passes hardware info to kernel via boot protocol

**ACPI**
- ACPI table parser (DSDT, SSDT, FADT, MADT)
- Power state management (S0–S5)
- Thermal zone management
- Battery interface (laptops)

**IPC**
- Pipes (anonymous and named)
- Message queues
- Shared memory with capability guards
- Unix-domain socket equivalent

**Security**
- Capability-based permission model
- Per-process syscall filtering
- Sandbox isolation for untrusted code
- User account model with privilege separation

**Power Management**
- CPU frequency scaling based on load
- Display backlight control
- Suspend / resume (S3 sleep)
- Thermal throttling

---

### L3 — HakFS + Network Stack

**HakFS (self-developed filesystem)**
- Journaled for crash safety
- Extent-based allocation (no fragmentation)
- Inline small files
- Copy-on-write snapshots (future)
- VFS abstraction layer (other FSes mountable)
- Read support for ext4, FAT32 (for USB drives, dual-boot)

**Network Stack (TCP/IP from scratch)**

```
Application layer   DNS · TLS · HTTP client
Transport layer     TCP (full RFC 793 implementation) · UDP
Network layer       IPv4 · IPv6 · ICMP · ARP
Link layer          Ethernet framing · WiFi (802.11)
Physical layer      NIC driver interface
```

- TCP: full state machine, congestion control (CUBIC), fast retransmit
- TLS 1.3 for secure connections
- DNS resolver with caching
- DHCP client
- WiFi: WPA2/WPA3 supplicant

---

### L4 — Language Runtime Layer

Each supported language runtime has a native syscall adapter that maps the runtime's OS calls to HakOS kernel interfaces.

**Strategy:** implement the syscall interface each runtime expects, so the runtime itself needs no modification.

**Supported runtimes:**

| Runtime | Syscall adapter strategy |
|---|---|
| CPython | POSIX-subset syscall layer |
| Node.js (V8) | POSIX-subset + libuv shim |
| Go | Direct HakOS syscall (Go has its own runtime) |
| JVM (OpenJDK) | POSIX-subset syscall layer |
| Ruby (MRI) | POSIX-subset syscall layer |
| Rust std | Native HakOS target (upstream Rust target) |
| Swift | POSIX-subset syscall layer |
| R | POSIX-subset syscall layer |
| Julia | POSIX-subset syscall layer |
| Lua | POSIX-subset syscall layer |
| PHP | POSIX-subset syscall layer |
| Dart | POSIX-subset syscall layer |
| Haskell (GHC) | POSIX-subset syscall layer |
| Elixir (BEAM) | POSIX-subset syscall layer |
| Zig | Native HakOS target |

**Version tracking:** each runtime adapter is versioned independently. When Python 3.14 ships, only the Python adapter needs updating.

---

### L5 — Software Compatibility Layer

**Native .hak packages**
- Custom package format (compressed archive + manifest)
- Double-click to install — no confirmation dialogs
- Silent install by default
- Permission prompt only when app requests elevated access
- Package manifest declares: name, version, dependencies, permissions

**Software availability strategy — native .hak packages only**

HakOS does not implement a Linux syscall compatibility layer. All software is distributed as native `.hak` packages compiled specifically for HakOS. This guarantees best performance, no translation overhead, and no compatibility surprises.

Core developer tools are officially maintained as `.hak` packages by the HakOS project:

| Software | Source | Status |
|---|---|---|
| VSCode (Open VSX build) | MIT source | Official |
| Git | Source | Official |
| Docker / Podman | Source | Official |
| Neovim | Source | Official |
| tmux | Source | Official |
| curl / wget | Source | Official |
| Node.js | Source | Official |
| Python | Source | Official |
| Go toolchain | Source | Official |
| Rust toolchain | Source | Official |
| FFmpeg | Source | Official |
| SQLite | Source | Official |

Community-maintained packages fill the long tail. Any developer can compile and submit a `.hak` package via the official package repository. Software not yet packaged can always be built from source — HakOS ships all major language runtimes and build tools pre-installed.

**Archive auto-extract**
- Double-click any archive → extract to current directory
- Preserves directory structure
- `.iso` / `.dmg` → prompt: mount or extract
- Extraction progress shown in file manager

**Supported archive formats:**
`.zip` `.tar` `.tar.gz` `.tgz` `.tar.xz` `.tar.bz2` `.tar.zst`
`.7z` `.rar` `.gz` `.bz2` `.xz` `.zst` `.lz4` `.lzma`
`.iso` `.jar` `.whl` `.apk` `.war` `.ear` `.nupkg` `.cab`

---

### L6 — GUI Compositor + Desktop Environment

**Compositor**
- Wayland-protocol-inspired (self-implemented, not using libwayland)
- GPU-accelerated rendering via direct GPU driver interface
- 60fps target on all supported GPUs
- Hardware overlay support for video playback
- VSync with adaptive sync support

**Animation Engine**
- Spring physics for window animations
- Easing curves for UI transitions
- GPU-side animation (runs on GPU timeline, not CPU)
- Target: macOS-class smoothness

**Window Manager**
- Tiling mode (i3-inspired, keyboard-driven)
- Floating mode (traditional)
- Per-workspace layout
- Window snapping and edge resistance
- Multi-monitor: independent workspaces per monitor

**Font Rendering**
- Subpixel antialiasing
- Hinting with FreeType-equivalent engine (self-implemented)
- Variable font support

**Input System**
- Keyboard event routing with hotkey interception layer
- Multi-touch gesture recognizer (touchpad)
- Pointer acceleration profiles

**Core Desktop Apps**

| App | Description |
|---|---|
| Terminal | GPU-accelerated, tabs, split panes, session restore |
| File Manager | Developer-focused, shows hidden files by default, Git status indicators |
| System Settings | Full settings UI (see Section 5) |
| Task Manager | Process list, CPU/memory/GPU/network per process |
| Notification Center | App notifications, system alerts |
| Clipboard Manager | History of last 50 clipboard entries |
| Screenshot Tool | Region, window, fullscreen; instant copy to clipboard |
| Screen Recorder | MP4 output, region or fullscreen |
| Text Editor | Minimal built-in editor for quick edits |

---

### L7 — Embedded AI Core

**Models**

| Model | Quantization | RAM | Use case |
|---|---|---|---|
| Qwen2.5-1B | INT4/INT8 | ~800 MB | Fast, low-spec devices |
| Qwen2.5-3B | INT4/INT8 | ~2 GB | Default, better accuracy |

Both models are pre-installed on the HakOS image. No download on first boot.

**Inference Engine**
- Custom inference runtime optimized for x86_64 (AVX2/AVX-512)
- GPU offload when NVIDIA/AMD GPU available
- Quantized (INT4/INT8) for minimal memory footprint
- Streaming token output for low perceived latency

**AI Subsystems**
- Intent classification
- Named entity extraction (project names, action verbs, parameters)
- Context window: last 10 commands in session
- Language detection (input language, not code language)

**Privacy**
- All inference local
- No telemetry
- No model update calls home
- Inference logs stored only in RAM, cleared on session end

---

### L8 — Natural Language Shell

The primary user interface of HakOS.

**Input modes**
- Text (keyboard)
- Voice (microphone → whisper-compatible STT, future phase)

**Core Flow**

```
1. User inputs natural language command
2. AI Core parses intent and extracts entities
3. Shell resolves entities against context (project registry, disk index)
4. If single match → execute
5. If multiple matches → list with numbers, wait for selection
6. Generate concrete shell command
7. Show resolved command to user (transparency)
8. Execute command in terminal
9. Stream output back to user
10. Log to session history
```

**Project Detection**

When a project name is mentioned, HakOS:
1. Checks project registry (previously opened projects — instant)
2. Falls back to full disk index (maintained in background)
3. Matches by directory name
4. Returns sorted results (most recently accessed first)

**Language Detection**

HakOS detects the project type by checking for marker files:

| Marker file | Detected type |
|---|---|
| `package.json` | Node.js |
| `requirements.txt` / `pyproject.toml` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` | Java/Maven |
| `build.gradle` | Java/Gradle |
| `Gemfile` | Ruby |
| `Package.swift` | Swift |
| `composer.json` | PHP |
| `Dockerfile` / `docker-compose.yml` | Docker |
| `Makefile` | Make |

**Action Mapping**

| User action word | Node.js | Python | Go | Rust | Java |
|---|---|---|---|---|---|
| run / start / dev | npm run dev | python main.py | go run . | cargo run | mvn spring-boot:run |
| build | npm run build | python setup.py build | go build | cargo build | mvn package |
| test | npm test | pytest | go test ./... | cargo test | mvn test |
| install | npm install | pip install -r requirements.txt | go mod download | cargo fetch | mvn dependency:resolve |
| clean | npm run clean | — | go clean | cargo clean | mvn clean |

**Multi-match Disambiguation**

```
HakOS > open websize and run

Multiple projects found named "websize":
  [1] ~/projects/websize          (Node.js, last opened 2h ago)
  [2] ~/archive/websize-old       (Node.js, last opened 3 months ago)
  [3] ~/work/client-a/websize     (Python, last opened 1 week ago)

Which one? >
```

**NL Scripting**

Users can write `.hak` script files in natural language:

```
# morning.hak
open terminal
open websize and run
open myapi and run
show system performance
```

Run with: `hak run morning.hak`

**Session History**

All NL commands stored in session history with:
- Original natural language input
- Resolved command
- Exit code
- Timestamp

Searchable with: `history search <query>`

---

## 4. System Settings — Full Specification

### Appearance
- Light / Dark / Auto (follows system time) theme
- Accent color (preset palette + custom hex)
- Font size (system-wide scale: Small / Medium / Large)
- Animation speed (Off / Reduced / Normal / Fast)
- Desktop wallpaper (solid color or image)
- Icon size (Small / Medium / Large)
- Window corner radius

### Display
- Resolution per monitor
- Refresh rate per monitor
- Multi-monitor arrangement (drag-and-drop layout)
- Per-monitor scaling (100% / 125% / 150% / 200%)
- Night mode (color temperature, schedule)
- Overscan correction

### Network
- WiFi: scan, connect, forget, password management
- Ethernet: DHCP or static IP/gateway/DNS
- Custom DNS servers (manual entry)
- HTTP/SOCKS5 proxy configuration
- Firewall: on/off, per-app rules

### Bluetooth
- Scan and pair devices
- Connected device list
- Auto-connect toggle per device
- Bluetooth audio codec selection

### Audio
- Output device selection
- Input device selection
- Output volume default
- Input gain default
- Per-app volume (future)

### Keyboard & Touchpad
- Key repeat delay and rate
- Touchpad scroll direction (natural / traditional)
- Touchpad sensitivity
- Tap to click toggle
- Gesture configuration (3-finger, 4-finger swipes)
- Global hotkey editor (add / remove / modify)

### Users & Permissions
- Create / delete user accounts
- Change password
- sudo access toggle per user
- Per-app permissions: camera, microphone, file system access

### Storage
- Disk usage visualization per partition
- Largest files finder
- Cache and temp file cleanup
- HakFS health check (journal status, errors)

### AI Settings
- Model selection: Qwen2.5-1B / Qwen2.5-3B
- NL Shell sensitivity (how confident AI must be before executing without confirmation)
- Project scan directories (which paths disk indexer watches)
- AI features toggle (disable for full manual mode)
- Session history retention (1 day / 1 week / 1 month / never)

### Developer Options
- System log level (Error / Warning / Info / Debug)
- Performance overlay (FPS, CPU, GPU, RAM — floating HUD)
- Kernel parameter editor (sysctl-equivalent)
- GPU mode (Power Save / Balanced / Performance)
- Network packet inspection toggle

### System
- Language and locale
- Timezone (auto-detect or manual)
- NTP time sync toggle and server
- Startup application manager
- System update settings (auto / manual)
- About: HakOS version, kernel version, hardware summary

---

## 5. Development Roadmap

### Phase 1 — NL Shell Prototype (Months 1–3)
Run on top of existing OS to validate the core idea.
- Python-based NL shell
- Qwen2.5 local inference integration
- Project registry and disk indexer
- Language detection and action mapping
- Multi-match disambiguation
- Open source + demo video release

### Phase 2 — Kernel Bootstrap (Months 3–9)
- Self-written UEFI bootloader
- Kernel entry point (assembly → Rust)
- Physical memory management
- Virtual memory (page tables)
- Interrupt handling (IDT, APIC)
- Basic process scheduler
- Milestone: kernel boots and prints "HakOS" to screen

### Phase 3 — Usable Core (Months 9–18)
- HakFS implementation
- TCP/IP network stack
- NVMe and SATA storage drivers
- Realtek and Intel NIC drivers
- Basic display driver (framebuffer)
- Native `.hak` package install path and archive auto-extract (L5)
- Language runtime POSIX-subset adapters for officially shipped runtimes (L4)
- Milestone: boots into a shell, can install and run native `.hak` applications, has network

> **Note:** HakOS does **not** implement a Linux syscall compatibility layer or run unmodified Linux ELF userspace binaries. See L5 and `docs/AI_MEMORY/RUNTIME/LINUX_BINARY_POLICY.md`.

### Phase 4 — Desktop (Months 18–30)
- GPU compositor
- Animation engine
- Window manager (tiling + floating)
- Terminal emulator
- File manager
- System settings UI
- Milestone: usable graphical desktop

### Phase 5 — Full Integration (Months 30–42)
- NL Shell integrated into desktop
- Embedded AI models shipped with OS image
- All language runtime adapters
- `.hak` package format and installer
- Audio subsystem
- WiFi management UI
- Milestone: public alpha release

---

## 6. Non-Goals

The following will never be in HakOS core:

- Printer support
- Scanner support
- Fax support
- Parental controls
- Accessibility features (Phase 1+ only — not in early versions)
- Cloud sync
- App store / marketplace
- Gaming-specific features
- Mobile device sync

---

## 7. Open Questions

1. **NVIDIA GPU support** — NVIDIA's driver situation is complex. Early versions may have limited NVIDIA support (framebuffer only). Full acceleration depends on open driver maturity.

2. **macOS-class animation smoothness** — achieving this requires perfect GPU driver latency. This is the single hardest engineering challenge in the project.

3. **Language runtime maintenance** — as Python, Node, Go etc. release new versions, syscall adapters must be updated. Automated compatibility testing needed.

4. **Security model depth** — capability-based security is the goal, but the exact permission granularity needs refinement during implementation.

---

*This document is a living specification and will evolve as HakOS develops.*

---

## 8. Extended AI Architecture (Addendum)

> These sections extend the existing architecture defined in Sections 1–7. No existing layers are modified or removed. All additions slot in as new subsystems on top of the existing kernel, filesystem, and NL Shell foundations.

---

### 8.1 AI API Layer (hakos-ai-gateway)

A structured syscall interface that allows external AI models (OpenAI, Anthropic, local models) to interact with the HakOS system in a controlled, capability-gated way.

This layer sits above L3 (HakFS + Network) and L6 (GUI Compositor) and is exposed to external processes via a Unix-domain-socket-equivalent IPC channel.

**API Categories (~50 interfaces across 8 classes)**

| Category | Example APIs | Notes |
|---|---|---|
| File System | `file_read`, `file_write`, `dir_list` | Large file chunking, offset reads |
| Process Management | `process_spawn`, `process_kill`, `process_list` | Full lifecycle control |
| Network | `http_request`, `tcp_send`, `dns_lookup` | TLS + proxy support |
| Device & System | `disk_mount`, `sys_memory`, `audio_play` | Hardware query and control |
| Window & Screen | `window_list`, `window_capture`, `screen_capture` | Direct compositor frame access, low latency |
| Clipboard | `clipboard_read`, `clipboard_write` | Text, image, file list |
| Input Simulation | `keyboard_type`, `mouse_click` | High privilege — requires explicit user grant |
| Context Management | `context.store`, `context.retrieve`, `context.summarize` | Long-term memory, semantic search, compression |

**Security model — three-tier capability system:**

All API calls are gated by a three-tier permission model to prevent any AI from becoming a system-level backdoor:

| Tier | APIs | Grant mechanism | Revocation |
|---|---|---|---|
| Tier 1 — Read-only | `file_read`, `dir_list`, `process_list`, `sys_memory`, `clipboard_read` | Auto-granted on app install | Any time in Settings |
| Tier 2 — Write | `file_write`, `process_spawn`, `process_kill`, `clipboard_write`, `http_request` | One-time user confirmation dialog | Any time in Settings |
| Tier 3 — Sensitive | `keyboard_type`, `mouse_click`, `screen_capture`, `window_capture` | Explicit grant per session — never persisted across reboots | Automatic on session end |

**Tier 3 additional protections:**
- Tier 3 capability tokens expire after 60 minutes by default (user-configurable)
- A persistent on-screen indicator shows when any Tier 3 API is active (cannot be hidden by apps)
- All Tier 3 calls are logged to `/Data/hakos-ai-gateway/audit.log` with timestamp, caller, and action
- User can kill all active Tier 3 tokens instantly via a single hotkey (configurable, default: `Ctrl+Shift+Esc`)

**Gateway architecture:**
```
External AI model
      ↓  (Unix socket — no TCP, no remote access by default)
hakos-ai-gateway
      ↓
  ┌── Tier check ── capability token validation
  ├── Rate limiter  ── prevents API flooding
  ├── Audit logger  ── every call logged (Tier 2+)
  └── Router ───────── HakFS · Network · Compositor · Context · Search
```

---

### 8.2 Context Engine (hakos-context-engine)

Moves AI long-term memory out of the model's context window and into the OS layer. Provides unlimited, cross-session, searchable external memory at near-zero cost per query.

**This does not replace or modify the existing NL Shell session history (Section 3.8). It extends it with semantic search and persistence across reboots.**

**Architecture:**
- Vector store: LanceDB or Tantivy-based, local only
- Embedding model: ~100MB GGUF quantized model (e.g. `nomic-embed-text` or `all-minilm` in GGUF format), loaded via the same llama.cpp-compatible inference runtime used by the main 1B/3B models — no separate runtime needed
- Sliding window cache: last 10 conversation turns kept in RAM
- Auto-summarization: uses the already-loaded 1B/3B GGUF model (shared instance, no extra RAM cost) to compress long context into key points before storage

**Memory budget (realistic, GGUF INT4):**

| Component | RAM usage |
|---|---|
| Qwen2.5-1B GGUF INT4 | ~600 MB |
| Qwen2.5-3B GGUF INT4 | ~1.8 GB |
| Embedding model GGUF INT8 | ~80–120 MB |
| Vector store (LanceDB, typical) | ~50–200 MB |
| **Total (1B config)** | **~850 MB** |
| **Total (3B config)** | **~2.1 GB** |

Minimum recommended RAM: 4GB (1B config), 8GB (3B config). Both are realistic for any modern developer machine.

**Interfaces:**
| Interface | Description |
|---|---|
| `context.store(data, tags)` | Store a memory entry with optional tags |
| `context.retrieve(query)` | Semantic nearest-neighbor retrieval |
| `context.search(keyword)` | Full-text keyword search over stored memory |
| `context.summarize(entries)` | Compress multiple entries into a summary |
| `context.snapshot()` | Export current context to a portable file |
| `context.restore(file)` | Restore a previously exported context |

**Auto mode:** when an external AI sets `context_policy: auto`, the context engine automatically injects relevant historical summaries into the AI's next request — only compressed summaries, never raw history. The external AI does not need to call context APIs manually.

**Storage:** all context data stored locally under `/Data/hakos-context/`. Never transmitted externally.

---

### 8.3 Local Search Engine (hakos-search-engine)

Enables millisecond-level full-disk search for files, code, logs, documents, and PDFs — so the AI can retrieve information without reading files blindly.

**This is a new subsystem. It does not modify HakFS internals. It runs as a background service, reading files through the existing VFS layer.**

**Technology:** Rust + Tantivy (full-text search library). Background indexer runs at lowest I/O priority.

**Index storage format:**
- Sharded index: one index shard per logical collection (project, directory, file type)
- Global index manifest: `~/.hakos/index_manifest.txt`
  - Format per line: `short_name|description|tag:kw1,kw2|excluded?`
  - Used for fast query routing before touching individual shards
- Compressed snapshots: Zstd-compressed index snapshots loaded at boot for instant startup
- Target: < 1GB index for typical developer machine, < 2GB for 1TB disk

**Indexing strategy:**
- On first boot: background progressive indexing (user home first, then full disk)
- Ongoing: incremental updates via filesystem event watching (inotify-equivalent on HakOS)
- Exclusions: `/System`, `/Data/Cache`, temp files excluded by default
- User-configurable exclusions via `.hakignore` files (same syntax as `.gitignore`)

**Query routing:**
1. Check index manifest for matching shards (fast path, < 1ms)
2. Query matched shards only
3. If manifest has no match, fall back to parallel search across all shards (guarantees no missed results)

**Performance targets:**

| Query type | Target latency |
|---|---|
| Keyword search | < 20ms |
| Regex search | < 100ms |
| Semantic search (vector) | < 50ms |
| Index rebuild (full disk) | Background, never blocks UI |

---

### 8.4 Speculative Prefetching

Hides retrieval latency by predicting what context and search results the AI will need before it asks.

**Implementation:**
1. As user begins typing, a lightweight local classifier (rule-based or tiny NN) extracts probable keywords in < 10ms
2. Keywords are sent async to the Context Engine and Search Engine for pre-querying
3. Results cached in RAM for 30 seconds
4. When AI subsequently issues the real query, results are returned from cache in < 1ms
5. Pre-fetched results are never injected into AI context automatically — only served on demand

**Performance budget:**
- Keyword extraction: < 10ms
- Pre-query execution: < 20ms per query
- Main thread impact: zero (fully async, separate thread pool)
- Cache TTL: 30 seconds, evicted on session end

---

### 8.5 Dual AI Architecture

HakOS runs two AI models simultaneously with distinct roles.

**External dialogue AI (e.g. GPT-4, Claude, or any API-compatible model)**
- Stateless: receives only current user message + system-injected short context (a few hundred tokens)
- Responsible for: generating final user-facing responses
- Communicates with HakOS via hakos-ai-gateway
- Has no direct file system or process access — everything goes through the API layer

**Local embedded AI (Qwen2.5-1B or 3B, existing from Section 3.7)**
- Always-on, on-device
- Responsible for: intent classification, entity extraction, summary generation, query routing, search result ranking
- Powers the NL Shell (existing behavior, unchanged)
- Also serves as the intelligence layer behind the context engine and speculative prefetcher

**Communication flow:**
```
User message
     ↓
Local AI → intent + entities → NL Shell executes (existing flow)
     ↓ (if external AI is involved)
hakos-ai-gateway → external AI API call
     ↓
External AI → calls context.retrieve, file_read, search.query via gateway
     ↓
Local AI ranks and filters results
     ↓
External AI generates response → displayed to user
```

---

### 8.6 File System Hierarchy

Replaces the traditional Unix directory layout with a developer-friendly structure. **This does not change HakFS internals — only the top-level directory naming convention.**

```
/
├── System/          # Kernel, drivers, services, system libraries
├── Users/           # All user home directories (including root: Users/root/)
├── Programs/        # Globally installed applications
├── Data/            # Logs, caches, temp files, databases, search index
├── Devices/         # Auto-mounted removable media (USB drives, etc.)
└── Volumes/         # Additional disks and partition mount points
```

Rationale: eliminates `/bin`, `/lib`, `/etc`, `/var`, `/tmp` fragmentation that confuses developers coming from macOS or Windows. Everything has one obvious home.

System paths:
- Kernel: `/System/kernel`
- Drivers: `/System/drivers/`
- Services: `/System/services/`
- Search index: `/Data/hakos-search/`
- Context engine data: `/Data/hakos-context/`
- User config: `/Users/<name>/.hakos/`

**No Linux path compatibility layer needed.** Because HakOS uses native `.hak` packages compiled specifically for HakOS (see L5), all software already targets HakOS paths at compile time. There are no Linux binaries that hardcode `/usr/lib` or `/home` to worry about.

---

### 8.7 Updated Roadmap (Revised)

The original 5-phase roadmap (Section 5) remains valid. These phases are appended as extensions:

**Phase 2.5 — Search Engine Foundation** *(between existing Phase 2 and 3)*
- [ ] Tantivy-based indexer running as background service
- [ ] Plain text and source code indexing
- [ ] Keyword query API (`search.query`)
- [ ] Index manifest and shard routing
- [ ] `.hakignore` support

**Phase 3.5 — AI API Skeleton** *(between existing Phase 3 and 4)*
- [ ] hakos-ai-gateway IPC channel (Unix socket)
- [ ] Three-tier capability token system
- [ ] `file_read`, `dir_list`, `process_list`, `search.query` implemented
- [ ] Tier 3 audit log + on-screen indicator
- [ ] External model integration test (connect any OpenAI-compatible API)
- [ ] Official .hak packages: VSCode, Git, Docker, Neovim, tmux, curl

**Phase 4.5 — Context Engine + Prefetch** *(between existing Phase 4 and 5)*
- [ ] GGUF inference runtime (llama.cpp-compatible, shared by all models)
- [ ] Embedding model in GGUF format (~100MB INT8) loaded via shared runtime
- [ ] Vector store integration (LanceDB)
- [ ] `context.store` / `context.retrieve` / `context.summarize`
- [ ] Summarization reuses already-loaded 1B/3B GGUF instance (no extra RAM)
- [ ] Speculative prefetching background service
- [ ] Auto context injection mode
- [ ] Memory usage validation: 1B config < 900MB, 3B config < 2.2GB

**Phase 5.5 — Full Dual AI Integration**
- [ ] External AI model routing via gateway
- [ ] Local AI as ranking/routing layer
- [ ] End-to-end: user message → local AI → gateway → external AI → context → response
- [ ] All 8 API categories implemented (~50 interfaces)

