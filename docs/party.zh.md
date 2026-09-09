# Party 第三方库集成

Party 是 Nimmake 最核心的功能之一。

Party 来源于 "thirdparty"（第三方库），意为将第三方代码或自己的源码目录作为一个整体模块来管理。一旦将某个目录定义为 Party，Nimmake 会自动扫描该目录下的源文件并参与编译。后续在该目录下新增文件，**无需修改 Nimmake.py 文件**。

---

## 1. Party 基本用法

### 1.1 最简单的 Party

```python
from nimmake.Helper import Helper

hlp = Helper()

# 将 src/Core 目录作为 Party，自动扫描其中的 .c/.cpp 等源文件
core = hlp.Parties("CORE", "src/Core")
```

仅需指定名称和根目录，Nimmake 会自动递归扫描该目录下所有支持的源文件，并加入编译。

### 1.2 带配置的 Party

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

## 2. Party 参数详解

`hlp.Parties()` 支持的完整参数如下：

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
| `exclude_dirs`     | `set`  | `{}`（额外默认排除 `{"doc", "docs", "examples"}`）          | 排除的子目录列表                                                       |
| `exclude_prefixes` | `set`  | `{}`                                                        | 排除的文件名前缀列表                                                   |
| `exclude_suffixes` | `set`  | `{"copy", "template"}`                                      | 排除的文件名后缀（不受扩展名影响，如 `aa_copy.c` 不会被编译）          |
| `defines`          | `dict` | `{}`                                                        | 宏定义，如 `{"DEBUG": "1"}` 等价于 `-DDEBUG=1`                         |
| `include_macros`   | `dict` | `{}`                                                        | Party 内部包含宏，如 `{"XXXX": "1"}` 等价于 `-DXXXX=1`                 |
| `params`           | `dict` | `{}`                                                        | 传递给 Party 的额外参数字典                                            |

### 2.1 `third_party` — 第三方类型

默认值为 `"GENERIC"`。对于自己编写的源码目录，使用 `"GENERIC"` 即可（这也是默认值，可以不显式指定）。

Nimmake 内置了一些特殊类型的第三方库，设置 `third_party` 后会自动应用对应的编译规则：

```python
# 使用 GENERIC 类型（默认，适用于自己的源码目录）
my_code = hlp.Parties("MY_CODE", "src/my_code", third_party="GENERIC")

# 使用 HAL 驱动类型
hal = hlp.Parties("HAL", "Drivers", third_party="HAL")
```

### 2.2 `build_type` — 构建类型

| 类型     | 说明                                       |
| -------- | ------------------------------------------ |
| `OBJECT` | 编译为目标文件（`.o`），直接链接到最终程序 |
| `STATIC` | 编译为静态库（`.a`），在链接时合并到程序中 |
| `SHARED` | 编译为动态库（`.so`/`.dll`），在运行时加载 |

```python
# 编译为静态库
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
)

# 编译为目标文件（默认）
core = hlp.Parties("CORE", "Core", build_type="OBJECT")
```

### 2.3 `source_exts` / `header_exts` — 文件扩展名

默认情况下，Nimmake 会自动识别常见的 C/C++ 源文件和头文件扩展名。如果你的项目只包含特定类型的文件，可以手动指定以提升扫描效率：

```python
# 仅扫描 .c 源文件
core = hlp.Parties("CORE", "Core", source_exts={".c"})

# 仅扫描 .h 头文件
core = hlp.Parties("CORE", "Core", header_exts={".h"})
```

### 2.4 `recursive` — 递归扫描

默认值为 `True`，会递归扫描子目录。如果只想扫描根目录下的文件，可以设置为 `False`：

```python
# 关闭递归扫描，只扫描根目录
core = hlp.Parties("CORE", "Core", recursive=False)
```

### 2.5 `exclude_dirs` / `exclude_prefixes` / `exclude_suffixes` — 排除规则

用于过滤不需要编译的文件：

```python
core = hlp.Parties(
    "CORE",
    "Core",
    exclude_dirs={"test", "backup", "deprecated"},   # 排除这些子目录
    exclude_prefixes={"old_", "tmp_"},                # 排除以 old_ 或 tmp_ 开头的文件
    exclude_suffixes={"copy", "template", "bak"},     # 排除后缀为 copy/template/bak 的文件
)
```

**注意：**

- `exclude_dirs` 默认额外排除 `{"doc", "docs", "examples"}` 目录
- `exclude_suffixes` 默认排除 `{"copy", "template"}`，即 `main_copy.c`、`hal_template.c` 等文件不会被编译
- 后缀排除不受文件扩展名影响，如 `aa_copy.c` 和 `aa_copy.h` 都会被排除

### 2.6 `defines` / `include_macros` — 宏定义

`defines` 和 `include_macros` 都用于定义宏，但作用范围不同：

- `defines` — 对外可见的宏定义，会传递给依赖该 Party 的其他模块
- `include_macros` — 仅在当前 Party 内部使用的宏定义

```python
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},   # 等价于 -DSTM32F407xx -DUSE_HAL_DRIVER
    include_macros={"HAL_MODULE_ENABLED": "1"},           # 仅 HAL 内部可见
)
```

### 2.7 `params` — 传递参数

用于向 Party 传递额外的动态参数，例如芯片配置信息：

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

## 3. 内置 Party 类型

Nimmake 预置了以下第三方库模块，设置 `third_party` 参数即可自动集成，无需手动配置编译标志：

```python
PARTIES_DCT = {
    "GENERIC":  GenericParty,    # 通用类型（默认），适用于自定义源码目录
    "_DEFAULT": DefaultParty,    # 系统默认 Party，承载全局默认编译参数
    "COMMON":   CommonDir,       # 公共目录
    "HAL":      HALDrivers,      # HAL 硬件抽象层驱动库
    "FATFS":    FatFS,           # FAT 文件系统
    "FREERTOS": FreeRTOS,        # FreeRTOS 实时操作系统
    "LVGL":     LVGL,            # LVGL 图形库
}
```

后续将加入更多内置库（如 CherryUSB、FreeModbus、EasyLogger 等）。

### 使用示例

```python
# 集成 FreeRTOS
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")

# 集成 LVGL 图形库
lvgl = hlp.Parties("LVGL", "Middlewares/LVGL", third_party="LVGL")

# 集成 FATFS 文件系统
fatfs = hlp.Parties("FatFS", "Middlewares/FatFS", third_party="FATFS")
```

---

## 4. Party 依赖

### 4.1 基本依赖设置

通过 `DependOn()` 方法设置 Party 之间的依赖关系。依赖方的 `defines` 和头文件路径（incs）会自动传递给被依赖方：

```python
core = hlp.Parties("CORE", "Core")
driver = hlp.Parties(
    "Driver",
    "Drivers",
    third_party="HAL",
    build_type="STATIC",
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
)

# Core 依赖 Driver — Core 编译时可以访问 Driver 的头文件和宏定义
core.DependOn([driver])
```

### 4.2 复杂依赖关系

在实际项目中，多个 Party 之间可能存在复杂的依赖链：

```python
# 定义各模块
core = hlp.Parties("CORE", "Core")
hal = hlp.Parties("HAL_Driver", "Drivers", third_party="HAL", build_type="STATIC")
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")
lvgl = hlp.Parties("LVGL", "Middlewares/LVGL", third_party="LVGL")

# 设置依赖链：Core -> LVGL -> FreeRTOS -> HAL
core.DependOn([lvgl])
lvgl.DependOn([freertos])
freertos.DependOn([hal])
```

依赖关系会影响：

- **编译顺序** — 被依赖的模块先编译
- **头文件路径** — 被依赖方的头文件目录自动加入依赖方的编译搜索路径
- **宏定义** — 被依赖方的 `defines` 自动传递给依赖方

---

## 5. `_DEFAULT` Party

`_DEFAULT` 是一个特殊的系统 Party，用于承载全局默认的编译参数和头文件路径。

一般情况下，你不需要手动操作 `_DEFAULT` Party。当你通过 `hlp.Config()` 和 `hlp.Refresh()` 配置芯片和工具链后，`_DEFAULT` 会自动包含系统级的编译标志和头文件路径。所有其他 Party 隐式依赖于 `_DEFAULT`。

---

## 6. 完整示例

以下是一个典型的 STM32F407 项目的 Party 配置示例：

```python
from nimmake.datasets import CORTEX_M4_CFG
from nimmake.Helper import Helper, BuildType

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

# 芯片参数（传递给各 Party）
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

# ==========================================
# 定义 Party 模块
# ==========================================

# 用户应用代码
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)

# HAL 驱动库（编译为静态库）
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type=BuildType.STATIC.name,
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""},
    params=PARTY_PARAM,
)

# FreeRTOS 实时操作系统
freertos = hlp.Parties(
    "FreeRTOS",
    "Middlewares/FreeRTOS",
    third_party="FREERTOS",
    params=PARTY_PARAM,
)

# LVGL 图形库
lvgl = hlp.Parties(
    "LVGL",
    "Middlewares/LVGL",
    third_party="LVGL",
    params=PARTY_PARAM,
)

# ==========================================
# 设置依赖关系
# ==========================================
core.DependOn([lvgl])
lvgl.DependOn([freertos])
freertos.DependOn([driver])

# ==========================================
# 定义构建目标
# ==========================================
srcs = ["src_stm/startup_stm32f407xx.s"]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

---

## 7. Party 的优势总结

1. **自动扫描** — 无需手动列出每个源文件，目录下新增文件自动加入编译
2. **模块化管理** — 将不同功能的代码划分为独立的 Party，结构清晰
3. **依赖传播** — 自动传递头文件路径和宏定义，避免手动配置
4. **灵活排除** — 支持按目录、文件名前缀、文件名后缀排除不需要的文件
5. **内置库支持** — 预置 FreeRTOS、LVGL、FatFS 等常用库，开箱即用
6. **参数传递** — 通过 `params` 向 Party 传递动态参数，适配不同芯片配置
