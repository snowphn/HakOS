HakOS

An operating system built around a simple idea: you shouldn't need a terminal to talk to your computer. Natural language is the shell. The AI runs on your own hardware. No cloud, no keys, no subscriptions.

This is an early-stage project. The kernel is being written in Rust. Most of the stack is built from scratch. Target audience: developers who want their machine to understand what they mean, not just what they type.

---

What it does

You open a launcher or a terminal, type something like:

```
open websize and run
```

HakOS scans every disk, finds the folder called websize, checks what kind of project it is (Node, Python, whatever), and runs the right dev command. If it finds more than one match, it asks which one you meant. No manual cd, no ls, no package.json inspection.

All of that happens on-device. The language model is embedded in the OS (around 1B parameters by default; a 3B variant is available if you have more RAM). Nothing is sent to the internet.

---

Stack overview (bottom to top)

```
[ Natural language shell ]
[ Embedded AI (1B/3B model) ]
[ Compositor & desktop ]
[ Linux ELF compat layer ]
[ Language runtimes ]
[ HakFS + TCP/IP stack ]
[ IPC, security, boot ]
[ Hardware drivers ]
[ Kernel (Rust) ]
```

Each of these is being built as part of the OS. There's no Linux kernel underneath, no GNU userland. The filesystem, network stack, drivers, bootloader—all custom.

---

What's inside

- The shell is a natural language interface, but you can also drop into a regular terminal when you want.
- A GPU-accelerated compositor (target is 60fps). Window management is a mix of tiling and floating.
- HakFS — a journaled file system designed from scratch for this OS. Not ext4, not something borrowed.
- TCP/IP stack written in-house. No lwIP, no existing net stack.
- Linux ELF compatibility — so you can still run VSCode, Docker CLI, Git, and other tools without waiting for native ports.
- .hak packages — double-click an archive and it installs. No wizards, no "Are you sure?" prompts.
- Archive support for all common formats (zip, tar.*, 7z, rar, iso, etc.) — double-click extracts in place.
- System settings with the usual suspects: display, network, audio, users, keyboard. AI model can be toggled between 1B and 3B in a dropdown.

---

Hardware goals

Aiming for x86_64 initially. Intel and AMD CPUs. Graphics support for Intel iGPU, AMD, and NVIDIA (yeah, that one's going to hurt). NVMe and SATA. A handful of common NICs and USB HID devices. Not trying to support everything. If it runs on a typical dev laptop or desktop, it should work.

---

Build status & license

Concept / early development. Nothing production-ready. If you want to help, issues and PRs are open.

License: MIT. Use it, fork it, break it, fix it.

---

Why this exists

Most operating systems treat natural language as a layer on top of something older. HakOS inverts that: the AI is part of the boot image, and the shell is an LLM. The goal isn't to replace terminals, it's to stop treating memorized commands as a prerequisite for development workflows.

One developer. Some help from local models during design and code generation. No company, no cloud dependency, no data collection.
