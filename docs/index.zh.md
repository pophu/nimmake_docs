# 🚀 Nimmake — 为嵌入式开发而生的极速构建工具

> **告别繁琐的 Makefile，拥抱现代化的 MCU 构建体验。**

Nimmake 是一款专为 **ARM** 和 **RISC-V** 架构深度优化的轻量级跨平台构建工具。基于 Python 打造的 MCU 固件构建系统，让你轻松编译 ARM、RISC-V 及各类主流微控制器的 C/C++ 源码。

内置丰富的 MCU 和工具链配置，自动生成所需的编译标志。以第三方库的方式添加代码，无需逐个手动添加源文件、头文件或宏定义；可设置各库之间的依赖关系。特别适合快速开发和原型验证。

---

## 🤔 你是否也遇到过这些问题？

- ❌ Makefile 越写越复杂，维护成本居高不下？
- ❌ 每次集成新库都要手动添加源文件、头文件路径和宏定义？
- ❌ 全量编译耗时太久，等得心焦？
- ❌ 团队成员的开发环境不一致，构建行为千奇百怪？

**Nimmake 正是为解决这些痛点而生。**

---

## ✨ 为什么选择 Nimmake？

| 痛点           | Nimmake 的解决方案                                                         |
| -------------- | -------------------------------------------------------------------------- |
| 繁琐的手动配置 | **模块化 Party 系统** — 以第三方库方式集成，自动生成编译标志，无需手动配置 |
| 编译速度慢     | **增量构建 + 多进程并行 + 构建缓存** — 只编译变更部分，速度翻倍            |
| 工具链管理混乱 | **内置丰富的 MCU 和工具链配置** — 开箱即用，支持 armgcc / LLVM / armclang  |
| 缺少 IDE 支持  | **自动生成 compile_commands.json** — 无缝对接 VS Code、CLion 等 IDE        |

---

## 🔥 核心特性

### ⚡ 极致编译速度

- **增量构建** — 智能分析需要重新编译的文件，仅编译发生变更的部分
- **多进程并行** — 支持 `-j` 参数配置并发任务数，充分利用多核 CPU
- **Ninja 后端** — 可选 Ninja 模式（`-n` / `--ninja`），获得更极致的构建速度
- **构建缓存** — 缓存编译产物（`--cache`），二次构建快到飞起

### 🧩 模块化第三方集成

预置 **FreeRTOS、LVGL、FatFS、EasyLogger、CherryUSB、FreeModbus** 等常用模块，即插即用，无需从零配置。

### 🔧 多工具链支持

一套代码，多种编译器随意切换：

- **armgcc** — 经典的 ARM GNU 工具链
- **LLVM/Clang** — 现代化编译器，更好的错误提示
- **armclang** — ARM 官方编译器，商业级优化

### 🛠 开发者友好

- **预演模式**（`--dry-run`）— 预览构建计划，心中有数再动手
- **编译数据库**（`--compiledb`）— 生成 `compile_commands.json`，为 IDE 智能提示和静态分析提供支持
- **简洁命令行** — 短命令 `nmmk` 让你快人一步

---

## 📦 一分钟快速上手

### 1. 安装 Nimmake

```bash
pip install nimmake
```

### 2. 验证安装

```bash
nimmake --version
```

### 3. 克隆示例项目

```bash
git clone https://github.com/pophu/nimmake.git
cd nimmake/samples/05_arm32
```

### 4. 一键编译

```bash
nimmake
```

就是这么简单！无需配置 Makefile，无需手动管理编译标志。

---

## 📝 Nimmake.py 最小示例

一个完整的 STM32F407 构建脚本只需十几行代码：

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

hlp = Helper()

# 芯片配置
CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

# 工具链配置
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
})
hlp.Refresh()

# 源码模块（Party 系统）
core = hlp.Parties("CORE", "src/Core")
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL")
core.DependOn([driver])

# 构建目标
srcs = ["startup_stm32f407xx.s"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

---

## 🎯 适用场景

| 场景              | 说明                                   |
| ----------------- | -------------------------------------- |
| 🏃 快速原型验证   | 无需繁琐配置，几分钟即可搭建编译环境   |
| 🧪 多平台固件开发 | ARM / RISC-V 多架构无缝切换            |
| 👥 团队协作       | 统一的构建系统，消除"我这儿能跑"的尴尬 |
| 🔄 CI/CD 集成     | 命令行驱动，轻松接入持续集成流水线     |

---

## 📚 文档导航

| 文档                        | 内容                                                 |
| --------------------------- | ---------------------------------------------------- |
| [基本使用](usage.md)        | 芯片配置、工具链设置、编译标志、Party 系统、构建目标 |
| [高级用法](advanced.md)     | PC 程序、ARM32/RISC-V MCU 构建、多工具链切换         |
| [Party 系统](party.md)      | 第三方库集成详解，参数说明，内置库列表               |
| [命令行参数](params.md)     | 所有命令行参数的详细说明                             |
| [常见问题](faq.md)          | 18 个常见问题解答                                    |
| [贡献指南](contributing.md) | 如何参与 Nimmake 开发                                |

---

## 🌟 立即开始

> **Nimmake 让 MCU 固件构建像写 Python 一样简单。**

```bash
pip install nimmake
git clone https://github.com/pophu/nimmake.git
cd nimmake/samples/05_arm32
nimmake
```
