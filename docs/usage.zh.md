# Nimmake 基本用法

## 概述

Nimmake 是一款专为 ARM 和 RISC-V 架构优化的轻量级跨平台构建工具，基于 Python 语法编写构建脚本。它内置了大量 MCU 和工具链配置，能够自动生成编译标志，让开发者无需手动拼凑复杂的编译参数。

本文档将详细介绍 Nimmake 的核心用法，包括芯片配置、工具链设置、编译标志管理、源码组织（Party 系统）以及构建目标的定义。

---

## 1. 芯片类型配置

编译依赖芯片的架构（arch）、浮点单元（fpu）、浮点 ABI（float abi）等参数。Nimmake 内置了大量芯片类型，你只需指定芯片类型即可，无需手动设置这些底层参数。

### 1.1 使用内置 CPU 类型

Nimmake 提供了各系列 Cortex-M 内核的预置配置，直接导入即可使用：

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

hlp = Helper()

# 克隆一份配置，避免修改原始配置
CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
```

内置的芯片配置包括：

- `CORTEX_M0_CFG` — Cortex-M0 系列
- `CORTEX_M3_CFG` — Cortex-M3 系列
- `CORTEX_M4_CFG` — Cortex-M4 系列
- `CORTEX_M7_CFG` — Cortex-M7 系列
- RISC-V 各系列配置（RV32、RV64 等）

`hlp.Config()` 会将 CPU、FPU、ABI、Arch 等参数写入配置，后续 `hlp.Refresh()` 会根据这些配置自动生成对应的编译标志。

### 1.2 使用厂商型号定义

如果你知道具体的 MCU 型号，可以直接使用 `VENDOR_MODEL_OF` 函数，Nimmake 会自动匹配对应的芯片参数：

```python
from nimmake.datasets import VENDOR_MODEL_OF, Vendor
from nimmake.Helper import Helper

hlp = Helper()

# 直接指定厂商和型号
CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")
hlp.Config(CFG)
```

支持的厂商通过 `Vendor` 枚举指定，包括 `Vendor.ST`、`Vendor.GD` 等。

### 1.3 使用外部 TOML 文件

你也可以将芯片配置写入 `NIMMAKE_CFG.toml` 文件，然后通过 `hlp.TOML()` 加载：

```python
from nimmake.Helper import Helper

hlp = Helper()
hlp.TOML()  # 自动加载当前目录下的 NIMMAKE_CFG.toml
```

`NIMMAKE_CFG.toml` 示例：

```toml
[cpu]
name = "cortex-m4"
fpu = "fpv4-sp-d16"
abi = "hard"
arch = "arm"
```

### 1.4 手动设置芯片参数

如果内置配置不满足需求，你也可以通过 `hlp.Update()` 手动设置底层参数：

```python
hlp.Update({
    "TARGET_CPU": "cortex-m4",
    "TARGET_FPU": "fpv4-sp-d16",
    "TARGET_ABI": "hard",
    "TARGET_THUMB": "1",
    "TARGET_ARCH": "arm",
})
```

---

## 2. 工具链配置

### 2.1 默认行为

如果工具链已经全局安装（即在系统 PATH 中），你无需设置工具链目录。系统默认使用 `gcc` 作为工具链，如果你使用的正是 gcc，也无需额外设置。

**默认值：**

- `TOOLPATH`：系统 PATH 中的工具链目录
- `TOOL`：`"gcc"`
- `TOOL_PREFIX`：`""`（空字符串）

### 2.2 自定义工具链路径

如果你的工具链安装在自定义路径（如 ARM 交叉编译器），需要显式指定：

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

hlp = Helper()

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({
    "TOOLPATH": toolpath_armgcc,
    "TOOL": tool,
    "TOOL_PREFIX": prefix,
})

hlp.Refresh()  # 必须调用 Refresh() 使工具链配置生效
```

### 2.3 多工具链切换

Nimmake 支持 gcc、clang、armclang 等多种工具链。你可以预先定义多种工具链配置，按需切换：

```python
# GCC 工具链（ARM 交叉编译）
toolchain_gcc = {
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
}

# Clang 工具链（ARM LLVM）
toolchain_clang = {
    "TOOLPATH": r"D:\LLVM\arm-llvm\bin",
    "TOOL": "clang",
    "TOOL_PREFIX": "",
}

# RISC-V 工具链
toolchain_riscv = {
    "TOOLPATH": r"D:\LLVM\riscv-gcc\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "riscv32-unknown-elf-",
}

# 选择使用 GCC 工具链
hlp.Update(toolchain_gcc)
hlp.Refresh()
```

---

## 3. 查看编译标志

当芯片类型和工具链配置完成后，你可以通过 `hlp.Flags` 属性查看当前所有编译参数。`hlp.Flags` 会输出一个格式化的字符串，包含以下内容：

| 标志类别    | 说明           |
| ----------- | -------------- |
| `CFLAGS`    | C 编译器标志   |
| `CXXFLAGS`  | C++ 编译器标志 |
| `ASFLAGS`   | 汇编器标志     |
| `ARFLAGS`   | 静态库归档标志 |
| `LINKFLAGS` | 链接器标志     |
| `DEFINES`   | 宏定义         |

**完整示例：**

```python
from nimmake.datasets import CORTEX_M4_CFG, TOOL_OF
from nimmake.Helper import Helper

toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

print("== Welcome to Nimmake! ==")
hlp = Helper()

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
hlp.Refresh()

# 打印工具链信息
print(TOOL_OF(tool, prefix))

# 打印当前芯片配置
print(hlp["CFG"])

# 打印所有编译标志
print("====== 编译标志 ==========")
print(hlp.Flags)
print("====== 编译标志 ==========")
```

---

## 4. 修改编译标志

Nimmake 提供了多种方式来修改编译标志，以适应不同的项目需求。

### 4.1 通过 `set_cfg` 修改

`set_cfg` 用于修改与芯片/工具链直接相关的关键参数：

```python
# 设置链接脚本
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")

# 设置 CPU 型号
hlp.set_cfg("cpu", "cortex-m4")
```

### 4.2 通过 `Append` / `Prepend` 追加标志

- `Append` — 在现有标志**末尾**追加
- `Prepend` — 在现有标志**开头**插入

```python
# 追加链接库
hlp.Append(LIBS="m")       # 追加 libm
hlp.Prepend(LIBS="c")      # 在前面插入 libc

# 追加宏定义
hlp.Append(DEFINES={"XXXX": "123"})    # 追加 -DXXXX=123
hlp.Prepend(DEFINES={"YYYY": "456"})   # 在前面插入 -DYYYY=456

# 追加工具链路径
hlp.Append(TOOLPATH=toolpath2)
```

### 4.3 通过 `Update` 批量更新

`Update` 可以一次性更新多个参数，支持字典或另一个配置对象：

```python
# 字典方式
hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": "gcc", "TOOL_PREFIX": "arm-none-eabi-"})

# 传入配置对象
toolchain = {"TOOLPATH": toolpath_armgcc, "TOOL": "gcc", "TOOL_PREFIX": "arm-none-eabi-"}
hlp.Update(toolchain)
```

### 4.4 直接赋值

你可以像操作字典一样直接给 `hlp` 赋值：

```python
hlp["TOOLPATH"] = [toolpath2]
```

---

## 5. 添加源码 — Party 系统

Party 是 Nimmake 中最核心的概念之一，来源于 "thirdparty"（第三方库）。通过 Party 系统，你可以将外部目录或自己编写的源码目录作为一个整体模块来管理，Nimmake 会自动扫描该目录下的源文件并参与编译。

**Party 系统的优势：**

1. 自动搜索指定扩展名的文件作为源码，无需逐个手动列出
2. 支持排除指定子目录、文件名前缀或后缀
3. 支持为整个 Party 设置统一的宏定义
4. 后续在该目录下新增文件，无需修改 Nimmake.py

### 5.1 基本用法

```python
driver = hlp.Parties(
    name="Driver",
    root="src_stm/Drivers",
    source_exts=["*.c"],
    header_exts=["*.h"],
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
)
```

### 5.2 Party 参数详解

`hlp.Parties()` 支持的完整参数列表：

| 参数               | 类型   | 默认值                                                      | 说明                                                                   |
| ------------------ | ------ | ----------------------------------------------------------- | ---------------------------------------------------------------------- |
| `name`             | `str`  | **必填**                                                    | Party 名称，用于标识和依赖引用                                         |
| `root`             | `str`  | **必填**                                                    | 源码根目录路径                                                         |
| `third_party`      | `str`  | `"GENERIC"`                                                 | 第三方类型，内置包括 `"HAL"`、`"FREERTOS"`、`"LVGL"`、`"FATFS"` 等     |
| `build_type`       | `str`  | `"OBJECT"`                                                  | 构建类型：`OBJECT`（目标文件）、`STATIC`（静态库）、`SHARED`（动态库） |
| `source_exts`      | `set`  | `{".c", ".cpp", ".cxx", ".cc", ".c++", ".s", ".S", ".asm"}` | 源文件扩展名集合                                                       |
| `header_exts`      | `set`  | `{".h", ".hpp", ".hxx", ".hh"}`                             | 头文件扩展名集合                                                       |
| `recursive`        | `bool` | `True`                                                      | 是否递归扫描子目录                                                     |
| `depends`          | `list` | `[]`                                                        | 依赖的 Party 列表，也可通过 `DependOn()` 方法设置                      |
| `exclude_dirs`     | `list` | `[]`（额外默认排除 `{"doc", "docs", "examples"}`）          | 排除的子目录列表                                                       |
| `exclude_prefixes` | `list` | `[]`                                                        | 排除的文件名前缀列表                                                   |
| `exclude_suffixes` | `set`  | `{"copy", "template"}`                                      | 排除的文件名后缀列表（注：不受扩展名影响，如 `aa_copy.c` 不会被编译）  |
| `defines`          | `dict` | `{}`                                                        | 宏定义，如 `{"DEBUG": "1"}` 等价于 `-DDEBUG=1`                         |
| `include_macros`   | `dict` | `{}`                                                        | Party 内部包含宏，如 `{"XXXX": "1"}` 等价于 `-DXXXX=1`                 |
| `params`           | `dict` | `{}`                                                        | 传递给 Party 的额外参数字典                                            |

### 5.3 内置 Party 类型

Nimmake 预置了以下第三方库模块，设置 `third_party` 参数即可自动集成：

```python
PARTIES_DCT = {
    "GENERIC":  GenericParty,    # 通用类型（默认）
    "_DEFAULT": DefaultParty,    # 默认 Party
    "COMMON":   CommonDir,       # 公共目录
    "HAL":      HALDrivers,      # HAL 驱动库
    "FATFS":    FatFS,           # FAT 文件系统
    "FREERTOS": FreeRTOS,        # FreeRTOS 实时操作系统
    "LVGL":     LVGL,            # LVGL 图形库
}
```

使用示例：

```python
hal_driver = hlp.Parties(
    "HAL_Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
)
```

### 5.4 Party 依赖

通过 `DependOn()` 方法可以设置 Party 之间的依赖关系，依赖方的宏定义和头文件路径会自动传递给被依赖方：

```python
# 定义两个 Party
core = hlp.Parties("CORE", "Core")
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
)

# 设置依赖关系
core.DependOn([driver])   # Core 依赖 Driver
driver.DependOn([core])   # Driver 依赖 Core（双向依赖）
```

依赖关系会影响编译顺序，以及头文件路径和宏定义的传播。

### 5.5 `_DEFAULT` Party

`_DEFAULT` 是一个特殊的 Party，用于承载全局默认的编译参数和头文件路径。一般情况下不需要手动操作。

---

## 6. 设置编译目标

Nimmake 支持多种构建目标类型，可以同时定义多个目标，但需要指定一个默认目标。

### 6.1 可执行程序目标 (`Program`)

```python
srcs = [
    "src_stm/startup_stm32f407xx.s",
    "src_stm/main.c",
]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

### 6.2 静态库目标 (`Library_STATIC`)

生成 `.a` 静态库文件：

```python
mylib = hlp.Library_STATIC("mylib", sources=["lib.c", "utils.c"])
hlp.DefaultTarget(mylib)
```

### 6.3 动态库目标 (`Library_SHARED`)

生成 `.so` / `.dll` 动态库文件：

```python
myso = hlp.Library_SHARED("myso", sources=["so.c"])
hlp.DefaultTarget(myso)
```

### 6.4 伪目标 (`Phony`)

Phony 目标不产生实际文件，用于组合多个构建步骤或执行特定命令：

```python
# 定义命令目标
bin_cmd = hlp.Command("BIN", [
    f'{hlp["OBJCOPY"]} -O binary build/test.elf build/test.bin',
    f'{hlp["OBJCOPY"]} -O ihex build/test.elf build/test.hex',
])

flash_cmd = hlp.Command("FLASH", [
    "openocd -f interface/cmsis-dap.cfg -f target/stm32f4x.cfg -c 'program build/test.elf verify reset exit'"
])

# 定义 Phony 目标
hlp.Phony("my", ["test", "BIN", "FLASH"])    # 组合目标：编译 + 生成 bin + 烧录
hlp.Phony("flash", ["BIN", "FLASH"])          # 烧录目标
hlp.Phony("bin", ["BIN"])                     # 仅生成 bin 文件
```

### 6.5 设置默认目标

当定义了多个目标时，必须通过 `hlp.DefaultTarget()` 指定默认构建目标（即执行 `nimmake` 不加参数时构建的目标）：

```python
hlp.DefaultTarget(t)
```

---

## 7. 命令行参数

Nimmake 提供了丰富的命令行选项，用于调试、加速构建和生成辅助文件：

| 参数            | 简写 | 说明                                                    |
| --------------- | ---- | ------------------------------------------------------- |
| `-f Nimmake.py` | `-f` | 指定构建脚本文件                                        |
| `--verbose`     |      | 显示每条执行的完整命令，用于调试                        |
| `--dry-run`     |      | 预览将要执行的操作，不实际编译                          |
| `--compiledb`   | `-c` | 生成 `compile_commands.json`，供 IDE 和静态分析工具使用 |
| `--ninja`       | `-n` | 使用 Ninja 后端构建，速度更快                           |
| `-j N`          | `-j` | 并行编译线程数，如 `-j4` 表示 4 线程并行                |
| `--cache`       |      | 启用编译缓存，加速重新构建                              |
| `--version`     |      | 显示 Nimmake 版本号                                     |
| `-c` (clean)    |      | 清理构建产物                                            |

使用示例：

```bash
# 基本构建
nimmake -f Nimmake.py

# 详细输出
nimmake -f Nimmake.py --verbose

# 预览模式
nimmake -f Nimmake.py --dry-run

# 4 线程并行编译
nimmake -f Nimmake.py -j4

# 生成 compile_commands.json
nimmake -f Nimmake.py --compiledb

# 使用 Ninja 后端加速
nimmake -f Nimmake.py --ninja

# 启用缓存
nimmake -f Nimmake.py --cache

# 清理构建产物
nimmake -f Nimmake.py -c

# 也可使用短命令 nmmk
nmmk -f Nimmake.py
```

---

## 8. 完整示例

下面是一个完整的 `Nimmake.py` 构建脚本示例，涵盖了芯片配置、工具链设置、Party 定义、依赖关系和目标定义：

```python
from nimmake.datasets import CORTEX_M4_CFG, VENDOR_MODEL_OF, Vendor, TOOL_OF
from nimmake.Helper import Helper, BuildType

# ============================================================
# 1. 初始化 Helper
# ============================================================
hlp = Helper()

# ============================================================
# 2. 芯片配置（三选一）
# ============================================================

# 方式一：使用内置 CPU 类型
CFG = CORTEX_M4_CFG.clone()

# 方式二：使用厂商型号（注释掉上面，启用下面）
# CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")

# 方式三：使用外部 TOML 文件（注释掉上面，启用下面）
# hlp.TOML()

hlp.Config(CFG)

# ============================================================
# 3. 工具链配置
# ============================================================
toolchain = {
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
}
hlp.Update(toolchain)
hlp.Refresh()

# ============================================================
# 4. 额外编译参数
# ============================================================
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.Append(LIBS="m")
hlp.Append(DEFINES={"DEBUG": "1"})

# ============================================================
# 5. 定义 Party（源码模块）
# ============================================================
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

# Core 模块（用户代码）
core = hlp.Parties(
    "CORE",
    "src/Core",
    params=PARTY_PARAM,
)

# HAL 驱动模块
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type=BuildType.STATIC.name,
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    params=PARTY_PARAM,
)

# 设置依赖关系
core.DependOn([driver])

# ============================================================
# 6. 定义构建目标
# ============================================================
srcs = [
    "src_stm/startup_stm32f407xx.s",
]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)

# ============================================================
# 7. 可选：定义 Phony 目标
# ============================================================
bin_cmd = hlp.Command("BIN", [
    f'{hlp["OBJCOPY"]} -O binary build/test.elf build/test.bin',
    f'{hlp["OBJCOPY"]} -O ihex build/test.elf build/test.hex',
])
hlp.Phony("bin", ["BIN"])

# ============================================================
# 8. 查看编译标志（调试用）
# ============================================================
print("====== 当前编译标志 ==========")
print(hlp.Flags)
print("====== 编译标志结束 ==========")
```

---

## 9. 调试技巧

### 9.1 查看编译标志

在 `hlp.Refresh()` 之后，打印 `hlp.Flags` 即可查看当前所有编译参数：

```python
print(hlp.Flags)
```

### 9.2 查看详细编译命令

使用 `--verbose` 参数运行构建，可以看到每条编译命令的完整细节：

```bash
nimmake -f Nimmake.py --verbose
```

### 9.3 预览模式

使用 `--dry-run` 参数预览将要执行的操作，不会实际编译：

```bash
nimmake -f Nimmake.py --dry-run
```

### 9.4 生成 IDE 配置文件

使用 `--compiledb` 参数生成 `compile_commands.json`，供 VSCode、CLion 等 IDE 的静态分析和智能提示使用：

```bash
nimmake -f Nimmake.py --compiledb
```

---

## 10. 总结

Nimmake 的核心工作流程可以概括为以下几步：

1. **初始化** — 创建 `Helper` 实例
2. **配置芯片** — 通过 `Config()` 设置 MCU 类型
3. **配置工具链** — 通过 `Update()` 设置编译器路径和前缀
4. **刷新配置** — 调用 `Refresh()` 使配置生效
5. **调整标志** — 通过 `set_cfg()`、`Append()`、`Prepend()` 微调编译参数
6. **组织源码** — 通过 `Parties()` 将源码目录作为模块添加
7. **设置依赖** — 通过 `DependOn()` 设置模块间依赖关系
8. **定义目标** — 通过 `Program()`、`Library_STATIC()` 等定义构建产物
9. **执行构建** — 运行 `nimmake -f Nimmake.py`

这种基于 Python 的构建方式，让你可以用熟悉的编程语言来描述构建过程，避免了 Makefile 繁琐的语法，同时保持了高度的灵活性和可维护性。
