# Nimmake Frequently Asked Questions (FAQ)

**Nimmake** is a Python-based cross-platform build tool optimized for ARM and RISC-V architecture MCU firmware development. It supports multiple toolchains such as armgcc, LLVM/Clang, and armclang, with built-in configurations for many MCUs and toolchains, and can automatically generate compilation flags.

---

## 1. How to View the Compilation Flags of the Current Nimmake.py

In the `Nimmake.py` script, call `hlp.Refresh()` and then use `print(hlp.Flags)` to view all current compilation flags. `hlp.Flags` is a property that outputs a formatted string containing the following:

- **CFLAGS**: C compiler flags
- **CXXFLAGS**: C++ compiler flags
- **ASFLAGS**: Assembler flags
- **ARFLAGS**: Static library archiver flags
- **LINKFLAGS**: Linker flags
- **DEFINES**: Macro definitions

```python
from nimmake.Helper import Helper
hlp = Helper()
# ... configure toolchain and chip, etc. ...
hlp.Refresh()
print(hlp.Flags)  # View all compilation flags
```

## 2. How to Add Source Files to Compile

Define a program target and specify the source file list using the `hlp.Program()` method.

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

## 3. How to Modify Compilation Flags

Modify toolchain flags using the `set_cfg` function.

```python
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")
```

Modify flags using the `APPEND` function or `UPDATE` function.

```python
   hlp.Append(LIBS="m")
   hlp.Prepend(LIBS="c")
   hlp.Append(DEFINES={"XXXX": "123"})
   hlp.Prepend(DEFINES={"YYYY": "456"})
   hlp["TOOLPATH"] = [toolpath2]
   hlp.Append(TOOLPATH=toolpath2)
   hlp.Update(toolchain)
   hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
   print(TOOL_OF(tool, prefix))
```

## 4. How to Set Build Targets

Nimmake supports three target types:

- `hlp.Program(name, sources)` — Executable program target
- `hlp.Library_STATIC(name, sources)` — Static library target (`.a`)
- `hlp.Library_SHARED(name, sources)` — Shared library target (`.so`)

```python
# Program target
app = hlp.Program("firmware", sources=["main.c"])
hlp.DefaultTarget(app)

# Static library target
mylib = hlp.Library_STATIC("mylib", sources=["lib.c"])
hlp.DefaultTarget(mylib)

# Shared library target
myso = hlp.Library_SHARED("myso", sources=["so.c"])
hlp.DefaultTarget(myso)
```

## 5. How to Add Third-Party Libraries

Use the `hlp.Parties()` method to add third-party libraries (called "Party"), supporting configuration of root directory, third-party type, build type, dependencies, macro definitions, etc.:

```python
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

core = hlp.Parties(
    "CORE",
    "src/Core",
    params=PARTY_PARAM,
)

driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type=BuildType.STATIC.name,
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    params=PARTY_PARAM,
)
```

Main parameters of `hlp.Parties()`:

- `name` — Party name
- `root` — Source root directory
- `third_party` — Third-party type (e.g., built-in modules like "HAL", "GENERIC")
- `build_type` — Build type (OBJECT, STATIC, SHARED)
- `source_exts` — List of source file extensions
- `header_exts` — List of header file extensions
- `recursive` — Whether to recursively scan subdirectories
- `depends` — List of dependent Parties
- `exclude_dirs` / `exclude_prefixes` / `exclude_suffixes` — Exclusion rules
- `defines` — Macro definitions
- `params` — Dictionary of parameters passed to the Party

## 6. How to Set the Toolchain

Set toolchain-related parameters using `hlp.Update()`, then call `hlp.Refresh()` to apply the configuration:

```python
toolpath_armgcc = r"/path/to/arm-none-eabi-gcc/bin"
toolpath_clang = r"/path/to/clang/bin"
toolpath_riscv = r"/path/to/riscv/bin"

toolchain_gcc = {
    "TOOLPATH": toolpath_armgcc,
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
}

toolchain_clang = {
    "TOOLPATH": toolpath_clang,
    "TOOL": "clang",
    "TOOL_PREFIX": "",
}

toolchain_riscv = {
    "TOOLPATH": toolpath_riscv,
    "TOOL": "gcc",
    "TOOL_PREFIX": "riscv32-unknown-elf",
}

# Select a toolchain
hlp.Update(toolchain_gcc)
hlp.Refresh()  # Must call Refresh() to apply the toolchain configuration
```

## 7. How to Set the Chip Type

Import built-in chip configurations from the `nimmake.datasets` module and apply them using `hlp.Config()`:

```python
from nimmake.datasets import CORTEX_M4_CFG

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
hlp.set_cfg("linkscript", "STM32F407XX_FLASH.ld")
```

Built-in chip configurations include various Cortex-M series (M0/M3/M4/M7, etc.) and RISC-V configurations. `hlp.Config()` sets chip-related parameters such as CPU, FPU, ABI, and Arch. `hlp.Refresh()` automatically generates the corresponding compilation flags based on the chip configuration.

You can also manually set low-level parameters using `hlp.Update()`:

```python
hlp.Update({
    "TARGET_CPU": "cortex-m4",
    "TARGET_FPU": "fpv4-sp-d16",
    "TARGET_ABI": "hard",
    "TARGET_THUMB": "1",
    "TARGET_ARCH": "arm",
})
```

## 8. How to Set Dependencies Between Parties

Use the `DependOn()` method of the Party object to set dependencies between Parties:

```python
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL", params=PARTY_PARAM)

# Set dependency: CORE depends on Driver
core.DependOn(driver)

# Bidirectional dependency
driver.DependOn(core)
```

Dependencies affect the compilation order and the propagation of header files and macro definitions.

## 9. How to Set Phony Targets

Define phony targets using the `hlp.Phony()` method. Phony targets do not produce actual files; they only execute commands:

```python
# Define command targets
bin_cmd = hlp.Command("BIN", [
    f'{hlp["OBJCOPY"]} -O binary build/test.elf build/test.bin',
    f'{hlp["OBJCOPY"]} -O ihex build/test.elf build/test.hex',
])
flash_cmd = hlp.Command("FLASH", [
    "openocd -f interface/cmsis-dap.cfg -f target/stm32f4x.cfg ..."
])
# Define phony targets
hlp.Phony("my", ["test", "BIN", "FLASH"])       # Combined target
hlp.Phony("flash", [bin_cmd.name, flash_cmd.name])  # Flashing target
hlp.Phony("bin", [bin_cmd.name])                 # Generate bin target
```

## 10. How to View Build Errors and Debugging Methods

Nimmake provides several command-line debugging options:

| Option                              | Description                                              |
| ----------------------------------- | -------------------------------------------------------- |
| `nimmake -f Nimmake.py --verbose`   | Display the full command for each executed step          |
| `nimmake -f Nimmake.py --dry-run`   | Preview the operations to be performed without compiling |
| `nimmake -f Nimmake.py --compiledb` | Generate `compile_commands.json` for IDE static analysis |
| `nimmake -f Nimmake.py --ninja`     | Build with the Ninja backend for faster builds           |
| `nimmake -f Nimmake.py -j4`         | Parallel compilation with 4 threads                      |
| `nimmake -f Nimmake.py --cache`     | Enable compilation cache to speed up rebuilds            |
| `nimmake -f Nimmake.py -c`          | Clean build artifacts                                    |

In the `Nimmake.py` script, you can also set `hlp.DRY_RUN = True` or `hlp.VERBOSE = True`.

## 11. How to Contribute to Nimmake

Nimmake is an open-source project hosted on GitHub:

- Repository: https://github.com/pophu/nimmake
- Documentation: https://nimmake-docs.readthedocs.io/en/latest/
- License: MIT License

General contribution workflow:

1. **Fork the repository** — Fork https://github.com/pophu/nimmake to your account on GitHub
2. **Clone the code** — `git clone https://github.com/<your-username>/nimmake.git`
3. **Create a branch** — `git checkout -b feature/your-feature`
4. **Modify code** — Edit the source code in the `src/nimmake/` directory
5. **Add tests** — Add corresponding test cases in the `tests/` directory
6. **Submit a PR** — Push to your forked repository and submit a Pull Request on GitHub

Project structure:

- `src/nimmake/` — Core source code (Helper.py, Backends.py, executor.py, cli.py, etc.)
- `src/nimmake/builders/` — Builder modules
- `src/nimmake/datasets/` — Chip configurations (chips.py, presets.py, tools.py, vendor.py)
- `src/nimmake/flags/` — Flag configurations
- `src/nimmake/thirdParties/` — Third-party library integrations
- `samples/` — Sample projects (basic, stm32, arm32, riscv32, etc.)
- `tests/` — Test code
