# Advanced Usage

This document covers build examples for various scenarios using Nimmake, including PC programs, ARM32 MCU, RISC-V MCU, and different toolchains.

---

## 1. Building a PC Program

Nimmake can build not only embedded MCU firmware but also regular PC executables.

### Example

Please refer to the `samples\02_win_exe` directory. Run `nimmake` in that directory to generate a `test.exe` file.

**Nimmake.py core example:**

```python
from nimmake.Helper import Helper

hlp = Helper()
hlp.Refresh()

srcs = ["main.c"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

When building PC programs, the default toolchain uses `gcc` from the system PATH. No extra toolchain path or chip configuration is needed.

---

## 2. Building ARM32 MCU Programs with arm-none-eabi-gcc

### 2.1 Download the Toolchain

Download the arm-none-eabi-gcc toolchain from [xpack arm gcc](https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack/).

### 2.2 Sample Directories

| Sample Directory             | Description                                                 |
| ---------------------------- | ----------------------------------------------------------- |
| `samples\03_stm32_led_reg`   | STM32 register-level programming, minimal MCU build example |
| `samples\04_arm32_led_cmsis` | ARM32 build example based on CMSIS                          |
| `samples\05_arm32`           | General ARM32 build with full project structure             |
| `samples\06_arm32_lib`       | ARM32 static library build example                          |

Run `nimmake` in each sample directory to generate a `test.elf` file.

### 2.3 Core Configuration

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

## 3. Building ARM32 MCU Programs with Clang

### 3.1 Download the Toolchain

Download the ARM LLVM embedded toolchain from [arm llvm toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases).

### 3.2 Example

Please refer to the `samples\09_arm_llvm` directory.

### 3.3 Core Configuration

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

The main difference from the GCC toolchain is that `TOOL` is set to `"clang"` and `TOOL_PREFIX` is an empty string.

---

## 4. Building RISC-V MCU Programs with arm-none-eabi-gcc

### 4.1 Download the Toolchain

Download the RISC-V toolchain from [xpack riscv](https://xpack-dev-tools.github.io/riscv-none-elf-gcc-xpack/).

### 4.2 Sample Directories

| Sample Directory     | Description                     |
| -------------------- | ------------------------------- |
| `samples\07_riscv32` | RISC-V 32-bit MCU build example |
| `samples\08_riscv64` | RISC-V 64-bit MCU build example |

### 4.3 Core Configuration

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

For RISC-V 64-bit builds, replace `RV32_CFG` with `RV64_CFG` and change `TOOL_PREFIX` to `"riscv64-unknown-elf-"`.

---

## 5. Quick Multi-Toolchain Switching

In practice, you may need to switch between different toolchains. It is recommended to manage all toolchain configurations centrally:

```python
from nimmake.datasets import CORTEX_M4_CFG, RV32_CFG
from nimmake.Helper import Helper

# Toolchain configuration dictionary
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

# Chip configuration dictionary
CHIPS = {
    "arm32": CORTEX_M4_CFG.clone(),
    "riscv32": RV32_CFG.clone(),
}

hlp = Helper()

# Select chip and toolchain
hlp.Config(CHIPS["arm32"])
hlp.Update(TOOLCHAINS["armgcc"])
hlp.Refresh()
```

With this approach, you can quickly switch chip and toolchain combinations by simply changing the parameters of `hlp.Config()` and `hlp.Update()`.
