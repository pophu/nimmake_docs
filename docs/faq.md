# Nimmake Frequently Asked Questions (FAQ)

**Nimmake** is a Python-based cross-platform build tool optimized for ARM and RISC-V architecture MCU firmware development. It supports multiple toolchains such as armgcc, LLVM/Clang, and armclang, with built-in configurations for many MCUs and toolchains, and can automatically generate compilation flags.

---

## Getting Started

### 1. Which platforms does Nimmake support?

The build scripts are based on pure Python syntax, supporting **Windows**, **Linux**, and **macOS**.

It also adapts to different terminal types (PowerShell, CMD, Linux Shell, etc.), avoiding compatibility issues with different terminal command formats.

### 2. Does the toolchain directory need to be added to the system PATH?

**No.** Configure the toolchain directory directly in `Nimmake.py`, and the system will automatically find the toolchain without modifying the system PATH:

```python
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
})
```

When these three parameters are not set, the defaults are used:

| Parameter     | Default                         | Description                 |
| ------------- | ------------------------------- | --------------------------- |
| `TOOLPATH`    | System PATH toolchain directory | Toolchain installation path |
| `TOOL`        | `"gcc"`                         | Toolchain type              |
| `TOOL_PREFIX` | `""` (empty string)             | Toolchain prefix            |

### 3. How to set the chip type?

Import built-in chip configurations from the `nimmake.datasets` module and apply them using `hlp.Config()`:

```python
from nimmake.datasets import CORTEX_M4_CFG

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
hlp.set_cfg("linkscript", "STM32F407XX_FLASH.ld")
```

Built-in chip configurations include various Cortex-M series (M0/M3/M4/M7, etc.) and RISC-V configurations. `hlp.Config()` sets chip-related parameters such as CPU, FPU, ABI, and Arch. `hlp.Refresh()` automatically generates the corresponding compilation flags.

You can also match by vendor model:

```python
from nimmake.datasets import VENDOR_MODEL_OF, Vendor

CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")
hlp.Config(CFG)
```

Or manually set low-level parameters:

```python
hlp.Update({
    "TARGET_CPU": "cortex-m4",
    "TARGET_FPU": "fpv4-sp-d16",
    "TARGET_ABI": "hard",
    "TARGET_THUMB": "1",
    "TARGET_ARCH": "arm",
})
```

### 4. How to set the toolchain?

Set toolchain-related parameters using `hlp.Update()`, then call `hlp.Refresh()` to apply the configuration:

```python
# Define multiple toolchain configurations
toolchain_gcc = {
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
}

toolchain_clang = {
    "TOOLPATH": r"D:\LLVM\arm-llvm\bin",
    "TOOL": "clang",
    "TOOL_PREFIX": "",
}

toolchain_riscv = {
    "TOOLPATH": r"D:\LLVM\riscv-gcc\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "riscv32-unknown-elf-",
}

# Select a toolchain
hlp.Update(toolchain_gcc)
hlp.Refresh()  # Must call Refresh() to apply the toolchain configuration
```

---

## Source Files and Build Targets

### 5. How to add source files to compile?

There are two ways to add source files:

**Method 1: Manually list source files**

Define a program target and specify the source file list using `hlp.Program()`:

```python
srcs = [
    "startup_stm32f407xx.s",
    "main.c",
    "src/main.c",
    "src/utils.c",
]
t = hlp.Program("my_app", sources=srcs)
hlp.DefaultTarget(t)
```

**Method 2: Use the Party system (recommended)**

Register source directories as Parties, and Nimmake will automatically scan them for source files. New files added later require no changes to `Nimmake.py`:

```python
core = hlp.Parties("CORE", "src/Core")
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL")
```

### 6. How to set build targets?

Nimmake supports four target types:

| Method                              | Output                        | Description                           |
| ----------------------------------- | ----------------------------- | ------------------------------------- |
| `hlp.Program(name, sources)`        | Executable (`.elf`/`.exe`)    | Executable program target             |
| `hlp.Library_STATIC(name, sources)` | Static library (`.a`)         | Merged into the program at link time  |
| `hlp.Library_SHARED(name, sources)` | Shared library (`.so`/`.dll`) | Loaded at runtime                     |
| `hlp.Phony(name, targets)`          | No actual file                | Virtual target for combining commands |

```python
# Program target
app = hlp.Program("firmware", sources=["main.c"])
hlp.DefaultTarget(app)

# Static library target
mylib = hlp.Library_STATIC("mylib", sources=["lib.c"])

# Shared library target
myso = hlp.Library_SHARED("myso", sources=["so.c"])
```

When multiple targets are defined, you must specify the default build target using `hlp.DefaultTarget()`.

### 7. How to generate library files?

**Generate static library (`.a`):**

```python
# Method 1: Set the entire build target as a static library
mylib = hlp.Library_STATIC("mylib", sources=["lib.c", "utils.c"])
hlp.DefaultTarget(mylib)

# Method 2: Build a Party as a static library, then link it
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type="STATIC",  # Build as static library
)
```

**Generate shared library (`.so`/`.dll`):**

```python
myso = hlp.Library_SHARED("myso", sources=["so.c"])
hlp.DefaultTarget(myso)
```

---

## Party System

### 8. How to add third-party libraries (Parties)?

Use the `hlp.Parties()` method to add third-party libraries. Nimmake automatically scans the directory for source files and includes them in the build:

```python
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

# User code (GENERIC type, default)
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)

# HAL driver (built-in type)
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type="STATIC",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    params=PARTY_PARAM,
)

# FreeRTOS
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")

# LVGL
lvgl = hlp.Parties("LVGL", "Middlewares/LVGL", third_party="LVGL")
```

Main parameters of `hlp.Parties()`:

| Parameter          | Type   | Default                                                     | Description                                                                              |
| ------------------ | ------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `name`             | `str`  | Required                                                    | Party name                                                                               |
| `root`             | `str`  | Required                                                    | Source root directory                                                                    |
| `third_party`      | `str`  | `"GENERIC"`                                                 | Third-party type (`"HAL"`, `"FREERTOS"`, `"LVGL"`, `"FATFS"`, etc.)                      |
| `build_type`       | `str`  | `"OBJECT"`                                                  | Build type (`OBJECT`, `STATIC`, `SHARED`)                                                |
| `source_exts`      | `set`  | `{".c", ".cpp", ".cxx", ".cc", ".c++", ".s", ".S", ".asm"}` | Source file extensions                                                                   |
| `header_exts`      | `set`  | `{".h", ".hpp", ".hxx", ".hh"}`                             | Header file extensions                                                                   |
| `recursive`        | `bool` | `True`                                                      | Whether to recursively scan subdirectories                                               |
| `depends`          | `list` | `[]`                                                        | List of dependent Parties                                                                |
| `exclude_dirs`     | `set`  | `{}`                                                        | Excluded subdirectories (additionally excludes `{"doc", "docs", "examples"}` by default) |
| `exclude_prefixes` | `set`  | `{}`                                                        | Excluded filename prefixes                                                               |
| `exclude_suffixes` | `set`  | `{"copy", "template"}`                                      | Excluded filename suffixes                                                               |
| `defines`          | `dict` | `{}`                                                        | Macro definitions, e.g. `{"DEBUG": "1"}` is equivalent to `-DDEBUG=1`                    |
| `include_macros`   | `dict` | `{}`                                                        | Internal macros within the Party                                                         |
| `params`           | `dict` | `{}`                                                        | Additional parameters passed to the Party                                                |

### 9. How to set dependencies between Parties?

Use the `DependOn()` method of the Party object to set dependencies. The dependent Party's `defines` and header file paths are automatically passed to the depending Party:

```python
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)
hal = hlp.Parties("HAL_Driver", "src/Drivers", third_party="HAL")
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")

# Set up dependency chain: Core -> FreeRTOS -> HAL
core.DependOn([freertos])
freertos.DependOn([hal])
```

Dependencies affect:

- **Build order** — Dependent modules are compiled first
- **Header paths** — The dependent Party's header directories are automatically added to the search path
- **Macro definitions** — The dependent Party's `defines` are automatically passed to the depending Party

---

## Compilation Flags and Debugging

### 10. How to view the current compilation flags?

In the `Nimmake.py` script, call `hlp.Refresh()` and then use `print(hlp.Flags)` to view all current compilation flags:

```python
from nimmake.Helper import Helper

hlp = Helper()
# ... configure toolchain and chip, etc. ...
hlp.Refresh()
print(hlp.Flags)
```

`hlp.Flags` outputs a formatted string containing:

| Flag Category | Description                   |
| ------------- | ----------------------------- |
| `CFLAGS`      | C compiler flags              |
| `CXXFLAGS`    | C++ compiler flags            |
| `ASFLAGS`     | Assembler flags               |
| `ARFLAGS`     | Static library archiver flags |
| `LINKFLAGS`   | Linker flags                  |
| `DEFINES`     | Macro definitions             |

### 11. How to modify compilation flags?

Nimmake provides several ways to modify compilation flags:

**`set_cfg` — Modify key parameters:**

```python
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")
```

**`Append` / `Prepend` — Append flags:**

```python
hlp.Append(LIBS="m")                          # Append link library at the end
hlp.Prepend(LIBS="c")                         # Insert link library at the beginning
hlp.Append(DEFINES={"XXXX": "123"})           # Append macro definition
hlp.Prepend(DEFINES={"YYYY": "456"})          # Insert macro definition at the beginning
hlp.Append(TOOLPATH=toolpath2)                # Append toolchain path
```

**`Update` — Batch update:**

```python
hlp.Update(toolchain)
hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": "gcc", "TOOL_PREFIX": "arm-none-eabi-"})
```

**Direct assignment:**

```python
hlp["TOOLPATH"] = [toolpath2]
```

### 12. How to view build errors and debugging methods?

Nimmake provides several command-line debugging options:

| Command                             | Description                                              |
| ----------------------------------- | -------------------------------------------------------- |
| `nimmake -f Nimmake.py --verbose`   | Display the full command for each executed step          |
| `nimmake -f Nimmake.py --dry-run`   | Preview operations without compiling                     |
| `nimmake -f Nimmake.py --compiledb` | Generate `compile_commands.json` for IDE static analysis |
| `nimmake -f Nimmake.py --ninja`     | Build with the Ninja backend for faster builds           |
| `nimmake -f Nimmake.py -j4`         | Parallel compilation with 4 threads                      |
| `nimmake -f Nimmake.py --cache`     | Enable compilation cache to speed up rebuilds            |
| `nimmake -f Nimmake.py -c`          | Clean build artifacts                                    |

In the `Nimmake.py` script, you can also set these via code:

```python
hlp.DRY_RUN = True    # Equivalent to --dry-run
hlp.VERBOSE = True    # Equivalent to --verbose
```

---

## Phony Targets and Advanced Usage

### 13. How to set Phony targets?

Use the `hlp.Phony()` method to define virtual targets (no actual file output), for combining multiple build steps or executing specific commands:

```python
# Define command targets
bin_cmd = hlp.Command("BIN", [
    f'{hlp["OBJCOPY"]} -O binary build/test.elf build/test.bin',
    f'{hlp["OBJCOPY"]} -O ihex build/test.elf build/test.hex',
])

flash_cmd = hlp.Command("FLASH", [
    "openocd -f interface/cmsis-dap.cfg -f target/stm32f4x.cfg -c 'program build/test.elf verify reset exit'"
])

# Define Phony targets
hlp.Phony("my", ["test", "BIN", "FLASH"])    # Combined target: build + generate bin + flash
hlp.Phony("flash", ["BIN", "FLASH"])          # Flashing target
hlp.Phony("bin", ["BIN"])                     # Generate bin only
```

Run `nimmake flash` to compile and flash in one step.

---

## Platform and Toolchain

### 14. How to compile ARM32 programs with Clang?

1. Download the toolchain: [arm llvm toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases)
2. Set `TOOL` to `"clang"` and `TOOL_PREFIX` to an empty string:

```python
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-llvm\bin",
    "TOOL": "clang",
    "TOOL_PREFIX": "",
})
hlp.Refresh()
```

3. Reference example: <https://github.com/pophu/nimmake/tree/main/samples/09_arm_llvm>

### 15. How to compile STM32CubeMX generated code with Nimmake?

Reference example project: <https://github.com/pophu/nimmake/tree/main/samples/05_arm32>

Basic steps:

1. Register the CubeMX-generated `Core/` and `Drivers/` directories as Parties
2. Set the chip type to the corresponding STM32 model
3. Configure `linkscript` to point to the CubeMX-generated `.ld` file
4. Set the corresponding macro definitions (e.g., `STM32F407xx`, `USE_HAL_DRIVER`)

### 16. Can picolibc and newlib be compiled for MCU development?

This feature is currently under development and will be supported in a future version.

---

## Framework Building and Contribution

### 17. How can MCU chip manufacturers build their own development framework with Nimmake?

Chip manufacturers can easily build their own development framework using Nimmake modules, quickly setting up a development environment without reinventing the wheel.

**Recommended project structure:**

- Build targets go in the `Targets` directory, with one directory per board and one directory per demo
- CMSIS, Driver libraries, and shared third-party code go in the `Studio` directory
- Chip manufacturers place their low-level drivers in the `Drivers` directory

```txt
Studio
├── Drivers          # Chip low-level drivers
├── CMSIS            # CMSIS standard interface
├── FreeRTOS         # Third-party libraries
└── Targets          # Demo programs
    ├── PCB_BOARD_01
    │   ├── 01_DEMO
    │   │   ├── src
    │   │   │   ├── Core
    │   │   │   └── main.c
    │   │   └── Nimmake.py
    │   └── 02_DEMO
    └── PCB_BOARD_02
        └── 01_DEMO
```

`Nimmake.py` builds the development framework using built-in third-party library types like `HAL` and `FreeRTOS`. Different chip demo programs are placed as PCB subdirectories under `Targets`, making it easy to quickly demonstrate chip API usage and project compilation.

### 18. How to contribute to Nimmake?

Nimmake is an open-source project hosted on GitHub:

- Repository: <https://github.com/pophu/nimmake>
- Documentation: <https://nimmake-docs.readthedocs.io/en/latest/>
- License: MIT License

**Contribution workflow:**

1. **Fork the repository** — Fork <https://github.com/pophu/nimmake> to your account on GitHub
2. **Clone the code** — `git clone https://github.com/<your-username>/nimmake.git`
3. **Create a branch** — `git checkout -b feature/your-feature`
4. **Modify code** — Edit the source code in the `src/nimmake/` directory
5. **Add tests** — Add corresponding test cases in the `tests/` directory
6. **Submit a PR** — Push to your forked repository and submit a Pull Request on GitHub

**Project structure:**

| Directory                   | Description                                                                  |
| --------------------------- | ---------------------------------------------------------------------------- |
| `src/nimmake/`              | Core source code (`Helper.py`, `Backends.py`, `executor.py`, `cli.py`, etc.) |
| `src/nimmake/builders/`     | Builder modules                                                              |
| `src/nimmake/datasets/`     | Chip configurations (`chips.py`, `presets.py`, `tools.py`, `vendor.py`)      |
| `src/nimmake/flags/`        | Flag configurations                                                          |
| `src/nimmake/thirdParties/` | Third-party library integrations                                             |
| `samples/`                  | Sample projects (`basic`, `stm32`, `arm32`, `riscv32`, etc.)                 |
| `tests/`                    | Test code                                                                    |
