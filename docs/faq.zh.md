# Nimmake 常见问题解答 (FAQ)

**Nimmake** 是一个基于 Python 的跨平台构建工具，专为 ARM 和 RISC-V 架构的 MCU 固件开发优化。支持 armgcc、LLVM/Clang、armclang 等多种工具链，内置大量 MCU 和工具链配置，可自动生成编译参数。

---

## 基础入门

### 1. Nimmake 适合哪些平台？

构建脚本基于纯粹的 Python 语法，支持 **Windows**、**Linux**、**macOS** 等主流平台。

同时适应不同的终端类型（PowerShell、CMD、Linux Shell 等），避免了不同终端命令格式不兼容的问题。

### 2. 工具链目录是否需要加入系统环境变量？

**不需要**。在 `Nimmake.py` 中直接配置工具链目录，系统会自动找到工具链，无需修改系统 PATH：

```python
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-none-eabi-gcc14\bin",
    "TOOL": "gcc",
    "TOOL_PREFIX": "arm-none-eabi-",
})
```

以上三个参数不设置时，使用默认值：

| 参数          | 默认值                   | 说明           |
| ------------- | ------------------------ | -------------- |
| `TOOLPATH`    | 系统 PATH 中的工具链目录 | 工具链安装路径 |
| `TOOL`        | `"gcc"`                  | 工具链类型     |
| `TOOL_PREFIX` | `""`（空字符串）         | 工具链前缀     |

### 3. 如何设置芯片类型？

通过 `nimmake.datasets` 模块导入内置的芯片配置，使用 `hlp.Config()` 设置：

```python
from nimmake.datasets import CORTEX_M4_CFG

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
hlp.set_cfg("linkscript", "STM32F407XX_FLASH.ld")
```

内置的芯片配置包括各 Cortex-M 系列（M0/M3/M4/M7 等）和 RISC-V 的配置。`hlp.Config()` 会设置 CPU、FPU、ABI、Arch 等芯片相关参数，`hlp.Refresh()` 会自动根据芯片配置生成对应的编译标志。

也可以通过厂商型号直接匹配：

```python
from nimmake.datasets import VENDOR_MODEL_OF, Vendor

CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")
hlp.Config(CFG)
```

或者手动设置底层参数：

```python
hlp.Update({
    "TARGET_CPU": "cortex-m4",
    "TARGET_FPU": "fpv4-sp-d16",
    "TARGET_ABI": "hard",
    "TARGET_THUMB": "1",
    "TARGET_ARCH": "arm",
})
```

### 4. 如何设置工具链？

通过 `hlp.Update()` 设置工具链相关参数，然后调用 `hlp.Refresh()` 使配置生效：

```python
# 定义多种工具链配置
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

# 选择工具链
hlp.Update(toolchain_gcc)
hlp.Refresh()  # 必须调用 Refresh() 使工具链配置生效
```

---

## 源码与编译目标

### 5. 如何添加需要编译的源码？

有两种方式添加源码：

**方式一：手动列出源文件**

通过 `hlp.Program()` 方法定义程序目标并指定源文件列表：

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

**方式二：使用 Party 系统（推荐）**

将源码目录作为 Party 注册，Nimmake 自动扫描目录下的源文件。后续新增文件无需修改 `Nimmake.py`：

```python
core = hlp.Parties("CORE", "src/Core")
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL")
```

### 6. 如何设置编译目标？

Nimmake 支持四种目标类型：

| 方法                                | 产物                        | 说明                   |
| ----------------------------------- | --------------------------- | ---------------------- |
| `hlp.Program(name, sources)`        | 可执行文件（`.elf`/`.exe`） | 可执行程序目标         |
| `hlp.Library_STATIC(name, sources)` | 静态库（`.a`）              | 在链接时合并到程序中   |
| `hlp.Library_SHARED(name, sources)` | 动态库（`.so`/`.dll`）      | 在运行时加载           |
| `hlp.Phony(name, targets)`          | 无实际文件                  | 虚拟目标，用于组合命令 |

```python
# 程序目标
app = hlp.Program("firmware", sources=["main.c"])
hlp.DefaultTarget(app)

# 静态库目标
mylib = hlp.Library_STATIC("mylib", sources=["lib.c"])

# 动态库目标
myso = hlp.Library_SHARED("myso", sources=["so.c"])
```

当定义了多个目标时，必须通过 `hlp.DefaultTarget()` 指定默认构建目标。

### 7. 如何生成库文件？

**生成静态库（`.a`）：**

```python
# 方式一：将整个构建目标设为静态库
mylib = hlp.Library_STATIC("mylib", sources=["lib.c", "utils.c"])
hlp.DefaultTarget(mylib)

# 方式二：将某个 Party 编译为静态库再链接
driver = hlp.Parties(
    "Driver",
    "src/Drivers",
    third_party="HAL",
    build_type="STATIC",  # 编译为静态库
)
```

**生成动态库（`.so`/`.dll`）：**

```python
myso = hlp.Library_SHARED("myso", sources=["so.c"])
hlp.DefaultTarget(myso)
```

---

## Party 系统

### 8. 如何添加第三方库（Party）？

通过 `hlp.Parties()` 方法添加第三方库，Nimmake 会自动扫描该目录下的源文件并参与编译：

```python
PARTY_PARAM = {
    "CPU": hlp._cfg.cpu,
    "ABI": hlp._cfg.abi,
    "FPU": hlp._cfg.fpu,
    "MODEL": hlp._cfg.model,
}

# 用户代码（GENERIC 类型，默认值）
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)

# HAL 驱动（内置类型）
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

`hlp.Parties()` 的主要参数：

| 参数               | 类型   | 默认值                                                      | 说明                                                        |
| ------------------ | ------ | ----------------------------------------------------------- | ----------------------------------------------------------- |
| `name`             | `str`  | 必填                                                        | Party 名称                                                  |
| `root`             | `str`  | 必填                                                        | 源码根目录                                                  |
| `third_party`      | `str`  | `"GENERIC"`                                                 | 第三方类型（`"HAL"`、`"FREERTOS"`、`"LVGL"`、`"FATFS"` 等） |
| `build_type`       | `str`  | `"OBJECT"`                                                  | 构建类型（`OBJECT`、`STATIC`、`SHARED`）                    |
| `source_exts`      | `set`  | `{".c", ".cpp", ".cxx", ".cc", ".c++", ".s", ".S", ".asm"}` | 源文件扩展名                                                |
| `header_exts`      | `set`  | `{".h", ".hpp", ".hxx", ".hh"}`                             | 头文件扩展名                                                |
| `recursive`        | `bool` | `True`                                                      | 是否递归扫描子目录                                          |
| `depends`          | `list` | `[]`                                                        | 依赖的 Party 列表                                           |
| `exclude_dirs`     | `set`  | `{}`                                                        | 排除的子目录（额外默认排除 `{"doc", "docs", "examples"}`）  |
| `exclude_prefixes` | `set`  | `{}`                                                        | 排除文件名前缀                                              |
| `exclude_suffixes` | `set`  | `{"copy", "template"}`                                      | 排除文件名后缀                                              |
| `defines`          | `dict` | `{}`                                                        | 宏定义，如 `{"DEBUG": "1"}` 等价于 `-DDEBUG=1`              |
| `include_macros`   | `dict` | `{}`                                                        | Party 内部包含宏                                            |
| `params`           | `dict` | `{}`                                                        | 传递给 Party 的额外参数                                     |

### 9. 如何设置 Party 之间的依赖？

通过 Party 对象的 `DependOn()` 方法设置依赖关系。依赖方的 `defines` 和头文件路径会自动传递给被依赖方：

```python
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)
hal = hlp.Parties("HAL_Driver", "src/Drivers", third_party="HAL")
freertos = hlp.Parties("FreeRTOS", "Middlewares/FreeRTOS", third_party="FREERTOS")

# 设置依赖链：Core -> FreeRTOS -> HAL
core.DependOn([freertos])
freertos.DependOn([hal])
```

依赖关系会影响：

- **编译顺序** — 被依赖的模块先编译
- **头文件路径** — 被依赖方的头文件目录自动加入搜索路径
- **宏定义** — 被依赖方的 `defines` 自动传递给依赖方

---

## 编译参数与调试

### 10. 如何查看当前编译参数？

在 `Nimmake.py` 脚本中，调用 `hlp.Refresh()` 后使用 `print(hlp.Flags)` 即可查看当前所有编译参数：

```python
from nimmake.Helper import Helper

hlp = Helper()
# ... 配置工具链和芯片等 ...
hlp.Refresh()
print(hlp.Flags)
```

`hlp.Flags` 会输出格式化的字符串，包含以下内容：

| 标志类别    | 说明           |
| ----------- | -------------- |
| `CFLAGS`    | C 编译器标志   |
| `CXXFLAGS`  | C++ 编译器标志 |
| `ASFLAGS`   | 汇编器标志     |
| `ARFLAGS`   | 静态库归档标志 |
| `LINKFLAGS` | 链接器标志     |
| `DEFINES`   | 宏定义         |

### 11. 如何修改编译参数？

Nimmake 提供了多种方式修改编译参数：

**`set_cfg` — 修改关键参数：**

```python
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")
```

**`Append` / `Prepend` — 追加标志：**

```python
hlp.Append(LIBS="m")                          # 末尾追加链接库
hlp.Prepend(LIBS="c")                         # 开头插入链接库
hlp.Append(DEFINES={"XXXX": "123"})           # 追加宏定义
hlp.Prepend(DEFINES={"YYYY": "456"})          # 开头插入宏定义
hlp.Append(TOOLPATH=toolpath2)                # 追加工具链路径
```

**`Update` — 批量更新：**

```python
hlp.Update(toolchain)
hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": "gcc", "TOOL_PREFIX": "arm-none-eabi-"})
```

**直接赋值：**

```python
hlp["TOOLPATH"] = [toolpath2]
```

### 12. 如何查看编译过程错误及调试方法？

Nimmake 提供了多个命令行调试选项：

| 命令                                | 说明                                              |
| ----------------------------------- | ------------------------------------------------- |
| `nimmake -f Nimmake.py --verbose`   | 显示每条执行的完整命令                            |
| `nimmake -f Nimmake.py --dry-run`   | 预览将要执行的操作，不实际编译                    |
| `nimmake -f Nimmake.py --compiledb` | 生成 `compile_commands.json`，供 IDE 静态分析使用 |
| `nimmake -f Nimmake.py --ninja`     | 使用 Ninja 后端构建，速度更快                     |
| `nimmake -f Nimmake.py -j4`         | 4 线程并行编译                                    |
| `nimmake -f Nimmake.py --cache`     | 启用编译缓存，加速重新构建                        |
| `nimmake -f Nimmake.py -c`          | 清理构建产物                                      |

在 `Nimmake.py` 脚本中，也可以通过代码设置：

```python
hlp.DRY_RUN = True    # 等价于 --dry-run
hlp.VERBOSE = True    # 等价于 --verbose
```

---

## Phony 目标与高级用法

### 13. 如何设置 Phony 目标？

通过 `hlp.Phony()` 方法定义虚拟目标（不产生实际文件），用于组合多个构建步骤或执行特定命令：

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

执行 `nimmake flash` 即可一键编译并烧录。

---

## 平台与工具链

### 14. ARM32 的程序，如何用 Clang 编译？

1. 下载对应的工具链：[arm llvm toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases)
2. 将 `TOOL` 设置为 `"clang"`，`TOOL_PREFIX` 为空字符串：

```python
hlp.Update({
    "TOOLPATH": r"D:\LLVM\arm-llvm\bin",
    "TOOL": "clang",
    "TOOL_PREFIX": "",
})
hlp.Refresh()
```

3. 参考示例：<https://github.com/pophu/nimmake/tree/main/samples/09_arm_llvm>

### 15. STM32CubeMX 生成的代码，如何用 Nimmake 编译？

请参考示例项目：<https://github.com/pophu/nimmake/tree/main/samples/05_arm32>

基本步骤：

1. 将 CubeMX 生成的 `Core/` 和 `Drivers/` 目录作为 Party 注册
2. 设置芯片类型为对应的 STM32 型号
3. 配置 `linkscript` 指向 CubeMX 生成的 `.ld` 文件
4. 设置对应的宏定义（如 `STM32F407xx`、`USE_HAL_DRIVER`）

### 16. 能否编译 picolibc 和 newlib 用于 MCU 开发？

当前正在开发中，后续版本将支持。

---

## 框架搭建与贡献

### 17. MCU 芯片制造商如何通过 Nimmake 搭建自己的开发框架？

芯片制造商可以方便地搭建自己的开发框架，通过 Nimmake 模块快速搭建开发环境，无需重复造轮子。

**推荐项目架构：**

- 构建目标放到 `Targets` 目录，每个板子有一个目录，每个 demo 有一个目录
- CMSIS、Driver 驱动库以及第三方库公共的部分，放到 `Studio` 目录下
- 芯片制造商将自己的底层驱动放到 `Drivers` 目录下

```txt
Studio
├── Drivers          # 芯片底层驱动
├── CMSIS            # CMSIS 标准接口
├── FreeRTOS         # 第三方库
└── Targets          # 演示程序
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

Nimmake.py 通过内置的 `HAL`、`FreeRTOS` 等第三方库类型，快速构建自己的开发框架。不同的芯片演示程序作为 `Targets` 目录的 PCB 子目录，方便快速演示芯片的 API 用法及项目编译。

### 18. 如何贡献代码到 Nimmake？

Nimmake 是开源项目，托管在 GitHub 上：

- 仓库地址：<https://github.com/pophu/nimmake>
- 文档地址：<https://nimmake-docs.readthedocs.io/en/latest/>
- 许可证：MIT License

**贡献流程：**

1. **Fork 仓库** — 在 GitHub 上 Fork <https://github.com/pophu/nimmake> 到你的账户
2. **克隆代码** — `git clone https://github.com/<your-username>/nimmake.git`
3. **创建分支** — `git checkout -b feature/your-feature`
4. **修改代码** — 在 `src/nimmake/` 目录下修改源代码
5. **添加测试** — 在 `tests/` 目录下添加对应的测试用例
6. **提交 PR** — 推送到你的 Fork 仓库，然后在 GitHub 上提交 Pull Request

**项目结构：**

| 目录                        | 说明                                                                 |
| --------------------------- | -------------------------------------------------------------------- |
| `src/nimmake/`              | 核心源代码（`Helper.py`、`Backends.py`、`executor.py`、`cli.py` 等） |
| `src/nimmake/builders/`     | 构建器模块                                                           |
| `src/nimmake/datasets/`     | 芯片配置（`chips.py`、`presets.py`、`tools.py`、`vendor.py`）        |
| `src/nimmake/flags/`        | 标志配置                                                             |
| `src/nimmake/thirdParties/` | 第三方库集成                                                         |
| `samples/`                  | 示例项目（`basic`、`stm32`、`arm32`、`riscv32` 等）                  |
| `tests/`                    | 测试代码                                                             |
