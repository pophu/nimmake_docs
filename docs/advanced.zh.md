# 高级用法

本文档介绍 Nimmake 在各种场景下的构建示例，涵盖 PC 程序、ARM32 MCU、RISC-V MCU 等不同平台和工具链。

---

## 1. 构建 PC 程序

Nimmake 不仅可以用于嵌入式 MCU 开发，也可以构建普通的 PC 端可执行程序。

### 示例

请参考 `samples\02_win_exe` 目录。在该目录下执行 `nimmake`，将生成 `test.exe` 文件。

**Nimmake.py 核心示例：**

```python
from nimmake.Helper import Helper

hlp = Helper()
hlp.Refresh()

srcs = ["main.c"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

PC 程序构建时，工具链默认使用系统 PATH 中的 `gcc`，无需额外配置工具链路径和芯片参数。

---

## 2. 使用 arm-none-eabi-gcc 构建 ARM32 MCU 程序

### 2.1 下载工具链

从 [xpack arm gcc](https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack/) 下载 arm-none-eabi-gcc 工具链。

### 2.2 示例目录

| 示例目录                     | 说明                                  |
| ---------------------------- | ------------------------------------- |
| `samples\03_stm32_led_reg`   | STM32 寄存器级编程，最简 MCU 构建示例 |
| `samples\04_arm32_led_cmsis` | 基于 CMSIS 的 ARM32 构建示例          |
| `samples\05_arm32`           | ARM32 通用构建，包含完整工程结构      |
| `samples\06_arm32_lib`       | ARM32 静态库构建示例                  |

在每个示例目录下执行 `nimmake`，将生成 `test.elf` 文件。

### 2.3 核心配置

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

hlp = Helper()

toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({
    "TOOLPATH": toolpath_armgcc,
    "TOOL": tool,
    "TOOL_PREFIX": prefix,
})

hlp.Refresh()

srcs = ["main.c"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

---

## 3. 使用 clang 构建 ARM32 MCU 程序

### 3.1 下载工具链

从 [arm llvm toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases) 下载 ARM LLVM 嵌入式工具链。

### 3.2 示例

请参考 `samples\09_arm_llvm` 目录。

### 3.3 核心配置

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper

hlp = Helper()

toolpath_clang = r"D:\LLVM\arm-llvm\bin"
tool = "clang"
prefix = ""

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({
    "TOOLPATH": toolpath_clang,
    "TOOL": tool,
    "TOOL_PREFIX": prefix,
})

hlp.Refresh()

srcs = ["main.c"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

与 GCC 工具链的主要区别在于 `TOOL` 设置为 `"clang"`，且 `TOOL_PREFIX` 为空字符串。

---

## 4. 使用 arm-none-eabi-gcc 构建 RISC-V MCU 程序

### 4.1 下载工具链

从 [xpack riscv](https://xpack-dev-tools.github.io/riscv-none-elf-gcc-xpack/) 下载 RISC-V 工具链。

### 4.2 示例目录

| 示例目录             | 说明                      |
| -------------------- | ------------------------- |
| `samples\07_riscv32` | RISC-V 32 位 MCU 构建示例 |
| `samples\08_riscv64` | RISC-V 64 位 MCU 构建示例 |

### 4.3 核心配置

```python
from nimmake.datasets import RV32_CFG
from nimmake.Helper import Helper

hlp = Helper()

toolpath_riscv = r"D:\LLVM\riscv-gcc\bin"
tool = "gcc"
prefix = "riscv32-unknown-elf-"

CFG = RV32_CFG.clone()
hlp.Config(CFG)

hlp.Update({
    "TOOLPATH": toolpath_riscv,
    "TOOL": tool,
    "TOOL_PREFIX": prefix,
})

hlp.Refresh()

srcs = ["main.c"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

RISC-V 64 位构建时，将 `RV32_CFG` 替换为 `RV64_CFG`，并将 `TOOL_PREFIX` 改为 `"riscv64-unknown-elf-"` 即可。

---

## 5. 多工具链快速切换

在实际开发中，可能需要在不同工具链之间切换。建议将所有工具链配置集中管理：

```python
from nimmake.datasets import CORTEX_M4_CFG, RV32_CFG
from nimmake.Helper import Helper

# 工具链配置字典
TOOLCHAINS = {
    "armgcc": {
        "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
        "TOOL": "gcc",
        "TOOL_PREFIX": "arm-none-eabi-",
    },
    "clang": {
        "TOOLPATH": r"D:\LLVM\arm-llvm\bin",
        "TOOL": "clang",
        "TOOL_PREFIX": "",
    },
    "riscv32": {
        "TOOLPATH": r"D:\LLVM\riscv-gcc\bin",
        "TOOL": "gcc",
        "TOOL_PREFIX": "riscv32-unknown-elf-",
    },
}

# 芯片配置字典
CHIPS = {
    "arm32": CORTEX_M4_CFG.clone(),
    "riscv32": RV32_CFG.clone(),
}

hlp = Helper()

# 选择芯片和工具链
hlp.Config(CHIPS["arm32"])
hlp.Update(TOOLCHAINS["armgcc"])
hlp.Refresh()
```

通过这种方式，只需修改 `hlp.Config()` 和 `hlp.Update()` 的参数即可快速切换芯片和工具链组合。
