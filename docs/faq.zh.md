# Nimmake 常见问题解答 (FAQ)

**Nimmake** 是一个基于 Python 的跨平台构建工具，专为 ARM 和 RISC-V 架构的 MCU 固件开发优化。支持 armgcc、LLVM/Clang、armclang 等多种工具链，内置大量 MCU 和工具链配置，可自动生成编译参数。

---

## 1. 适合windows linux mac 等平台吗

构建脚本基于纯粹的python语法，支持windows linux mac 等平台。

适应不同的终端类型，powershell cmd linuxshell等。

## 2. 工具链目录是否需要加入系统环境变量

不需要加入系统环境变量。在Nimmake.py中配置工具链目录， 系统会自动找到工具链

## 3. 如何查看当前 Nimmake.py 的编译参数

在 `Nimmake.py` 脚本中，调用 `hlp.Refresh()` 后使用 `print(hlp.Flags)` 即可查看当前所有编译参数。`hlp.Flags` 是一个属性，会输出格式化的字符串，包含以下内容：

- **CFLAGS**：C 编译器标志
- **CXXFLAGS**：C++ 编译器标志
- **ASFLAGS**：汇编器标志
- **ARFLAGS**：静态库归档标志
- **LINKFLAGS**：链接器标志
- **DEFINES**：宏定义

```python
from nimmake.Helper import Helper
hlp = Helper()
# ... 配置工具链和芯片等 ...
hlp.Refresh()
print(hlp.Flags)  # 查看所有编译参数
```

## 4. 如何添加需要编译的源码

通过 hlp.set_cfg() 方法修改基于工具链芯片相关的flags
通过 hlp.Program() 方法定义程序目标并指定源文件列表

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

## 5. 如何修改编译参数

通过 set_cfg 函数修改工具链标志。

```python
hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")
```

通过 APPEND 函数或 UPDATE 函数修改标志。

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

## 6. 如何设置编译目标

Nimmake 支持三种目标类型：
hlp.Program(name, sources) — 可执行程序目标
hlp.Library_STATIC(name, sources) — 静态库目标（.a）
hlp.Library_SHARED(name, sources) — 动态库目标（.so）

```python
# 程序目标
app = hlp.Program("firmware", sources=["main.c"])
hlp.DefaultTarget(app)

# 静态库目标
mylib = hlp.Library_STATIC("mylib", sources=["lib.c"])
hlp.DefaultTarget(mylib)

# 动态库目标
myso = hlp.Library_SHARED("myso", sources=["so.c"])
hlp.DefaultTarget(myso)
```

## 7. 如何添加第三方库

通过 hlp.Parties() 方法添加第三方库（称为 "Party"），支持配置根目录、第三方类型、构建类型、依赖、宏定义等：

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

hlp.Parties() 的主要参数：
name — Party 名称
root — 源码根目录
third_party — 第三方类型（如 "HAL"、"GENERIC" 等内置模块）
build_type — 构建类型（OBJECT、STATIC、SHARED）
source_exts — 源文件扩展名列表
header_exts — 头文件扩展名列表
recursive — 是否递归扫描子目录
depends — 依赖的 Party 列表
exclude_dirs / exclude_prefixes / exclude_suffixes — 排除规则
defines — 宏定义
params — 传递给 Party 的参数字典

## 8. 如何设置工具链

通过 hlp.Update() 设置工具链相关参数，然后调用 hlp.Refresh() 使配置生效：

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

# 选择工具链
hlp.Update(toolchain_gcc)
hlp.Refresh()  # 必须调用 Refresh() 使工具链配置生效
```

## 9. 如何设置芯片类型

通过 nimmake.datasets 模块导入内置的芯片配置，使用 hlp.Config() 设置：

```python
from nimmake.datasets import CORTEX_M4_CFG

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
hlp.set_cfg("linkscript", "STM32F407XX_FLASH.ld")
```

内置的芯片配置包括各 Cortex-M 系列（M0/M3/M4/M7 等）和 RISC-V 的配置。hlp.Config() 会设置 CPU、FPU、ABI、Arch 等芯片相关参数，hlp.Refresh() 会自动根据芯片配置生成对应的编译标志。
也可以通过 hlp.Update() 手动设置底层参数：

```python
hlp.Update({
    "TARGET_CPU": "cortex-m4",
    "TARGET_FPU": "fpv4-sp-d16",
    "TARGET_ABI": "hard",
    "TARGET_THUMB": "1",
    "TARGET_ARCH": "arm",
})
```

## 10. 如何设置party之间依赖

通过 Party 对象的 DependOn() 方法设置 Party 之间的依赖关系：

```python
core = hlp.Parties("CORE", "src/Core", params=PARTY_PARAM)
driver = hlp.Parties("Driver", "src/Drivers", third_party="HAL", params=PARTY_PARAM)

# 设置依赖：CORE 依赖 Driver
core.DependOn(driver)

# 双向依赖
driver.DependOn(core)
```

依赖关系会影响编译顺序和头文件/宏定义的传播。

## 11. 如何设置phony目标

通过 hlp.Phony() 方法定义伪目标（phony target），伪目标不产生实际文件，仅执行命令：

```python
# 定义命令目标
bin_cmd = hlp.Command("BIN", [
    f'{hlp["OBJCOPY"]} -O binary build/test.elf build/test.bin',
    f'{hlp["OBJCOPY"]} -O ihex build/test.elf build/test.hex',
])
flash_cmd = hlp.Command("FLASH", [
    "openocd -f interface/cmsis-dap.cfg -f target/stm32f4x.cfg ..."
])
# 定义 phony 目标
hlp.Phony("my", ["test", "BIN", "FLASH"])       # 组合目标
hlp.Phony("flash", [bin_cmd.name, flash_cmd.name])  # 烧录目标
hlp.Phony("bin", [bin_cmd.name])                 # 生成 bin 目标
```

## 12. 如何查看编译过程错误及调试方法

Nimmake 提供了多个命令行调试选项：

| 参数                                | 说明                                            |
| ----------------------------------- | ----------------------------------------------- |
| `nimmake -f Nimmake.py --verbose`   | 显示每条执行的完整命令                          |
| `nimmake -f Nimmake.py --dry-run`   | 预览将要执行的操作，不实际编译                  |
| `nimmake -f Nimmake.py --compiledb` | 生成 compile_commands.json，供 IDE 静态分析使用 |
| `nimmake -f Nimmake.py --ninja`     | 使用 Ninja 后端构建，速度更快                   |
| `nimmake -f Nimmake.py -j4`         | 4 线程并行编译                                  |
| `nimmake -f Nimmake.py --cache`     | 启用编译缓存，加速重新构建                      |
| `nimmake -f Nimmake.py -c`          | 清理构建产物                                    |

在 Nimmake.py 脚本中，也可以通过 hlp.DRY_RUN = True 或 hlp.VERBOSE = True 来设置。

## 13. 如何贡献代码 Nimmake

Nimmake 是开源项目，托管在 GitHub 上：
仓库地址：https://github.com/pophu/nimmake
文档地址：https://nimmake-docs.readthedocs.io/en/latest/
许可证：MIT License

贡献代码的一般流程：
Fork 仓库 — 在 GitHub 上 Fork https://github.com/pophu/nimmake 到你的账户
克隆代码 — git clone https://github.com/<your-username>/nimmake.git
创建分支 — git checkout -b feature/your-feature
修改代码 — 在 src/nimmake/ 目录下修改源代码
添加测试 — 在 tests/ 目录下添加对应的测试用例
提交 PR — 推送到你的 Fork 仓库，然后在 GitHub 上提交 Pull Request

项目结构：
src/nimmake/ — 核心源代码（Helper.py、Backends.py、executor.py、cli.py 等）
src/nimmake/builders/ — 构建器模块
src/nimmake/datasets/ — 芯片配置（chips.py、presets.py、tools.py、vendor.py）
src/nimmake/flags/ — 标志配置
src/nimmake/thirdParties/ — 第三方库集成
samples/ — 示例项目（basic、stm32、arm32、riscv32 等）
tests/ — 测试代码
