# 🚀 Nimmake — The Blazing-Fast Build Tool Built for Embedded Development

> **Say goodbye to complex Makefiles. Embrace a modern MCU build experience.**

Nimmake is a lightweight, cross-platform build tool deeply optimized for **ARM** and **RISC-V** architectures. A Python-powered build system for MCU firmware development, it lets you effortlessly compile C/C++ source code for ARM, RISC-V, and a wide range of mainstream microcontrollers.

Built-in extensive MCU and toolchain configurations automatically generate the required compile flags. Add code as third-party libraries without manually adding source files, headers, or macros one by one; you can set dependencies between each library. Especially suitable for rapid development and prototyping.

---

## 🤔 Have You Encountered These Problems?

- ❌ Makefiles growing increasingly complex and hard to maintain?
- ❌ Manually adding source files, header paths, and macros every time you integrate a new library?
- ❌ Full rebuilds taking forever, testing your patience?
- ❌ Inconsistent build behavior across different team members' environments?

**Nimmake was born to solve these pain points.**

---

## ✨ Why Choose Nimmake?

| Pain Point                   | Nimmake's Solution                                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Tedious manual config        | **Modular Party System** — Integrate as third-party libraries, auto-generate compile flags, no manual config needed |
| Slow compilation             | **Incremental Builds + Multi-Process Parallel + Build Cache** — Only compile what changed, doubled speed            |
| Chaotic toolchain management | **Rich built-in MCU and toolchain configs** — Works out of the box, supports armgcc / LLVM / armclang               |
| Lack of IDE support          | **Auto-generate compile_commands.json** — Seamless integration with VS Code, CLion, and other IDEs                  |

---

## 🔥 Core Features

### ⚡ Extreme Compilation Speed

- **Incremental Builds** — Intelligently analyzes which files need rebuilding and compiles only what changed
- **Multi-Process Parallel** — Configurable job count via `-j` parameter, fully utilizing multi-core CPUs
- **Ninja Backend** — Optional Ninja mode (`-n` / `--ninja`) for even faster builds
- **Build Cache** — Cache compiled artifacts (`--cache`), lightning-fast rebuilds

### 🧩 Modular Third-Party Integration

Pre-built modules for **FreeRTOS, LVGL, FatFS, EasyLogger, CherryUSB, FreeModbus**, and more — plug and play, no configuration from scratch.

### 🔧 Multi-Toolchain Support

One codebase, seamlessly switch between compilers:

- **armgcc** — Classic ARM GNU toolchain
- **LLVM/Clang** — Modern compiler with better error diagnostics
- **armclang** — ARM's official compiler, commercial-grade optimization

### 🛠 Developer Friendly

- **Dry-Run Mode** (`--dry-run`) — Preview the build plan before executing
- **Compilation Database** (`--compiledb`) — Generate `compile_commands.json` for IDE IntelliSense and static analysis
- **Concise CLI** — Short command `nmmk` keeps you one step ahead

---

## 📦 Quick Start in One Minute

### 1. Install Nimmake

```bash
pip install nimmake
```

### 2. Verify Installation

```bash
nimmake --version
```

### 3. Clone the Sample Project

```bash
git clone https://github.com/pophu/nimmake.git
cd nimmake/samples/05_arm32
```

### 4. Build with One Command

```bash
nimmake
```

That's it! No Makefile configuration, no manual compile flag management.

---

## 📝 Minimal Nimmake.py Example

A complete STM32F407 build script in just a dozen lines of code:

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

hlp = Helper()

# Chip configuration
CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

# Toolchain configuration
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
})
hlp.Refresh()

# Source modules (Party System)
core = hlp.Parties("CORE", "src/Core")
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL")
core.DependOn([driver])

# Build targets
srcs = ["startup_stm32f407xx.s"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

---

## 🎯 Use Cases

| Scenario                   | Description                                                         |
| -------------------------- | ------------------------------------------------------------------- |
| 🏃 Rapid Prototyping       | Set up a build environment in minutes, no tedious configuration     |
| 🧪 Multi-Platform Firmware | Seamlessly switch between ARM / RISC-V architectures                |
| 👥 Team Collaboration      | Unified build system, eliminating "it works on my machine"          |
| 🔄 CI/CD Integration       | CLI-driven, easily integrates into continuous integration pipelines |

---

## 📚 Documentation Navigation

| Document                        | Contents                                                                            |
| ------------------------------- | ----------------------------------------------------------------------------------- |
| [Basic Usage](usage.md)         | Chip config, toolchain setup, compile flags, Party system, build targets            |
| [Advanced Usage](advanced.md)   | PC programs, ARM32/RISC-V MCU builds, multi-toolchain switching                     |
| [Party System](party.md)        | Third-party library integration details, parameter reference, built-in library list |
| [CLI Parameters](params.md)     | Detailed explanation of all command-line parameters                                 |
| [FAQ](faq.md)                   | Answers to 18 common questions                                                      |
| [Contributing](contributing.md) | How to contribute to Nimmake                                                        |

---

## 🌟 Get Started Now

> **Nimmake makes MCU firmware building as simple as writing Python.**

```bash
pip install nimmake
git clone https://github.com/pophu/nimmake.git
cd nimmake/samples/05_arm32
nimmake
```
