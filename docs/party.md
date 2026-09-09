# Party Third-Party Library Integration

Party is one of the core features of Nimmake.

Party is derived from "thirdparty", meaning it manages third-party code or your own source directories as a unified module. Once a directory is defined as a Party, Nimmake automatically scans its source files and includes them in the build. When you add new files to that directory later, **there is no need to modify the Nimmake.py file**.

---

## 1. Basic Party Usage

### 1.1 Simplest Party

```python
from nimmake.Helper import Helper

hlp = Helper()

# Register src/Core as a Party, automatically scanning .c/.cpp files
core = hlp.Parties("CORE", "src/Core")
```

Just specify the name and root directory. Nimmake will recursively scan all supported source files and include them in the build.

### 1.2 Party with Configuration

```python
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
    source_exts={".c"},
    header_exts={".h"},
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    recursive=True,
    exclude_dirs={"test", "backup"},
    exclude_suffixes={"copy", "template"},
)
```

---

## 2. Party Parameters

The full list of parameters supported by `hlp.Parties()`:

| Parameter          | Type   | Default                                                               | Description                                                                                           |
| ------------------ | ------ | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `name`             | `str`  | **Required**                                                          | Party name, used for identification and dependency references                                         |
| `root`             | `str`  | **Required**                                                          | Source root directory path                                                                            |
| `third_party`      | `str`  | `"GENERIC"`                                                           | Third-party type, built-in types include `"HAL"`, `"FREERTOS"`, `"LVGL"`, `"FATFS"`, etc.             |
| `build_type`       | `str`  | `"OBJECT"`                                                            | Build type: `OBJECT` (object files), `STATIC` (static library), `SHARED` (shared library)             |
| `source_exts`      | `set`  | `{".c", ".cpp", ".cxx", ".cc", ".c++", ".s", ".S", ".asm"}`           | Source file extension set                                                                             |
| `header_exts`      | `set`  | `{".h", ".hpp", ".hxx", ".hh"}`                                       | Header file extension set                                                                             |
| `recursive`        | `bool` | `True`                                                                | Whether to recursively scan subdirectories                                                            |
| `depends`          | `list` | `[]`                                                                  | List of dependent Parties, can also be set via `DependOn()` method                                    |
| `exclude_dirs`     | `set`  | `{}` (additionally excludes `{"doc", "docs", "examples"}` by default) | List of excluded subdirectories                                                                       |
| `exclude_prefixes` | `set`  | `{}`                                                                  | List of excluded filename prefixes                                                                    |
| `exclude_suffixes` | `set`  | `{"copy", "template"}`                                                | List of excluded filename suffixes (not affected by extension, e.g. `aa_copy.c` will not be compiled) |
| `defines`          | `dict` | `{}`                                                                  | Macro definitions, e.g. `{"DEBUG": "1"}` is equivalent to `-DDEBUG=1`                                 |
| `include_macros`   | `dict` | `{}`                                                                  | Internal macros within the Party, e.g. `{"XXXX": "1"}` is equivalent to `-DXXXX=1`                    |
| `params`           | `dict` | `{}`                                                                  | Additional parameters passed to the Party                                                             |

### 2.1 `third_party` — Third-Party Type

The default value is `"GENERIC"`. For your own source directories, use `"GENERIC"` (this is the default, so you don't need to specify it explicitly).

Nimmake has built-in special types of third-party libraries. Setting `third_party` will automatically apply the corresponding build rules:

```python
# Use GENERIC type (default, for your own source directories)
my_code = hlp.Parties("MY_CODE", "src/my_code", third_party="GENERIC")

# Use HAL driver type
hal = hlp.Parties("HAL", "Drivers", third_party="HAL")
```

### 2.2 `build_type` — Build Type

| Type     | Description                                                            |
| -------- | ---------------------------------------------------------------------- |
| `OBJECT` | Compile to object files (`.o`), directly linked into the final program |
| `STATIC` | Compile to static library (`.a`), merged into the program at link time |
| `SHARED` | Compile to shared library (`.so`/`.dll`), loaded at runtime            |

```python
# Build as static library
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
)

# Build as object files (default)
core = hlp.Parties("CORE", "Core", build_type="OBJECT")
```

### 2.3 `source_exts` / `header_exts` — File Extensions

By default, Nimmake automatically recognizes common C/C++ source and header file extensions. If your project only contains specific file types, you can manually specify them to improve scanning efficiency:

```python
# Only scan .c source files
core = hlp.Parties("CORE", "Core", source_exts={".c"})

# Only scan .h header files
core = hlp.Parties("CORE", "Core", header_exts={".h"})
```

### 2.4 `recursive` — Recursive Scanning

The default is `True`, which recursively scans subdirectories. Set to `False` to only scan files in the root directory:

```python
# Disable recursive scanning, only scan the root directory
core = hlp.Parties("CORE", "Core", recursive=False)
```

### 2.5 `exclude_dirs` / `exclude_prefixes` / `exclude_suffixes` — Exclusion Rules

Used to filter out files that should not be compiled:

```python
core = hlp.Parties(
    "CORE",
    "Core",
    exclude_dirs={"test", "backup", "deprecated"},   # Exclude these subdirectories
    exclude_prefixes={"old_", "tmp_"},                # Exclude files starting with old_ or tmp_
    exclude_suffixes={"copy", "template", "bak"},     # Exclude files with copy/template/bak suffix
)
```

**Note:**

- `exclude_dirs` additionally excludes `{"doc", "docs", "examples"}` directories by default
- `exclude_suffixes` excludes `{"copy", "template"}` by default, meaning `main_copy.c`, `hal_template.c`, etc. will not be compiled
- Suffix exclusion is not affected by file extension, e.g. both `aa_copy.c` and `aa_copy.h` will be excluded

### 2.6 `defines` / `include_macros` — Macro Definitions

Both `defines` and `include_macros` are used for defining macros, but they differ in scope:

- `defines` — Externally visible macros, passed to other modules that depend on this Party
- `include_macros` — Macros used only within the current Party

```python
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},   # Equivalent to -DSTM32F407xx -DUSE_HAL_DRIVER
    include_macros={"HAL_MODULE_ENABLED": "1"},           # Only visible within HAL
)
```

### 2.7 `params` — Passing Parameters

Used to pass additional dynamic parameters to the Party, such as chip configuration information:

```python
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

core = hlp.Parties("CORE", "Core", params=PARTY_PARAM)
driver = hlp.Parties("Driver", "Drivers", third_party="HAL", params=PARTY_PARAM)
```

---

## 3. Built-in Party Types

Nimmake comes with the following pre-configured third-party library modules. Just set the `third_party` parameter to integrate them automatically without manually configuring build flags:

```python
PARTIES_DCT = {
    "GENERIC":  GenericParty,    # Generic type (default), for custom source directories
    "_DEFAULT": DefaultParty,    # System default Party, carries global default build parameters
    "COMMON":   CommonDir,       # Common directory
    "HAL":      HALDrivers,      # HAL hardware abstraction layer driver library
    "FATFS":    FatFS,           # FAT file system
    "FREERTOS": FreeRTOS,        # FreeRTOS real-time operating system
    "LVGL":     LVGL,            # LVGL graphics library
}
```

More built-in libraries will be added in the future (such as CherryUSB, FreeModbus, EasyLogger, etc.).

### Usage Examples

```python
# Integrate FreeRTOS
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")

# Integrate LVGL graphics library
lvgl = hlp.Parties("LVGL", "Middlewares/LVGL", third_party="LVGL")

# Integrate FATFS file system
fatfs = hlp.Parties("FatFS", "Middlewares/FatFS", third_party="FATFS")
```

---

## 4. Party Dependencies

### 4.1 Basic Dependency Setup

Use the `DependOn()` method to set dependencies between Parties. The dependent Party's `defines` and header file paths are automatically passed to the depending Party:

```python
core = hlp.Parties("CORE", "Core")
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
)

# Core depends on Driver — Core can access Driver's headers and macros during compilation
core.DependOn([driver])
```

### 4.2 Complex Dependency Chains

In real projects, multiple Parties may have complex dependency chains:

```python
# Define modules
core = hlp.Parties("CORE", "Core")
hal = hlp.Parties("HAL_Driver", "Drivers", third_party="HAL", build_type="STATIC")
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")
lvgl = hlp.Parties("LVGL", "Middlewares/LVGL", third_party="LVGL")

# Set up dependency chain: Core -> LVGL -> FreeRTOS -> HAL
core.DependOn([lvgl])
lvgl.DependOn([freertos])
freertos.DependOn([hal])
```

Dependencies affect:

- **Build order** — Dependent modules are compiled first
- **Header paths** — The dependent Party's header directories are automatically added to the depending Party's include search path
- **Macro definitions** — The dependent Party's `defines` are automatically passed to the depending Party

---

## 5. `_DEFAULT` Party

`_DEFAULT` is a special system Party that carries global default build parameters and header file paths.

In general, you don't need to manually operate the `_DEFAULT` Party. When you configure the chip and toolchain via `hlp.Config()` and `hlp.Refresh()`, `_DEFAULT` automatically includes system-level build flags and header paths. All other Parties implicitly depend on `_DEFAULT`.

---

## 6. Complete Example

Here is a typical Party configuration example for an STM32F407 project:

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper, BuildType

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

# Chip parameters (passed to each Party)
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

# ==========================================
# Define Party Modules
# ==========================================

# User application code
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)

# HAL driver library (built as static library)
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type=BuildType.STATIC.name,
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    params=PARTY_PARAM,
)

# FreeRTOS real-time operating system
freertos = hlp.Parties(
    "FreeRTOS",
    "Middlewares/FreeRTOS",
    third_party="FREERTOS",
    params=PARTY_PARAM,
)

# LVGL graphics library
lvgl = hlp.Parties(
    "LVGL",
    "Middlewares/LVGL",
    third_party="LVGL",
    params=PARTY_PARAM,
)

# ==========================================
# Set Dependencies
# ==========================================
core.DependOn([lvgl])
lvgl.DependOn([freertos])
freertos.DependOn([driver])

# ==========================================
# Define Build Target
# ==========================================
srcs = ["src_stm/startup_stm32f407xx.s"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

---

## 7. Party Advantages Summary

1. **Automatic scanning** — No need to manually list each source file; new files in the directory are automatically included in the build
2. **Modular management** — Divide code with different functions into independent Parties for a clear structure
3. **Dependency propagation** — Automatically passes header file paths and macro definitions, avoiding manual configuration
4. **Flexible exclusion** — Supports excluding files by directory, filename prefix, and filename suffix
5. **Built-in library support** — Pre-configured libraries like FreeRTOS, LVGL, FatFS, ready to use out of the box
6. **Parameter passing** — Pass dynamic parameters to Parties via `params`, adapting to different chip configurations
