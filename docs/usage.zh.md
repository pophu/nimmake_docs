# 基本用法

## 芯片类型

编译依赖芯片的类型，例如arch,fpu float等。
nimmake内置大量芯片类型，您无需手动设置。您只需要指定芯片类型即可。

例子1: cpu类型定义

```py
from nimmake.datasets import CORTEX_M4_CFG, TOOL_OF
CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)
```

例子2： 厂商型号定义

```py
from nimmake.datasets import CORTEX_M4_CFG, VENDOR_MODEL_OF, Vendor
CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")
hlp.Config(CFG)
```

例子3： 外部toml文件定义
NIMMAKE_CFG.toml

```py
from nimmake.datasets import CORTEX_M4_CFG, VENDOR_MODEL_OF, Vendor
hlp.TOML()
```

## 工具链及工具链目录

如果工具链以及全局安装，您无需设置工具链目录。
系统默认工具链为gcc, 如果您使用的gcc,也无需设置工具链。否则您需要设置工具链及工具链的前缀。

```py
from nimmake.datasets import CORTEX_M4_CFG, TOOL_OF
from nimmake.Helper import Helper

toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

print("== Welcome to  Nimmake! ==")
hlp = Helper()

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})

hlp.Refresh()
```

## 编译标志

在你的源码目录中新建 Makefile.py 文件。
在此文件中设置工具链信息和工具路径，并指定 CPU 架构。Nimmake 将帮助你获取编译标志。

你可以通过打印 hlp.Flags 来获取编译标志，并在输出中检查这些标志。

```python
from nimmake.datasets import CORTEX_M4_CFG, TOOL_OF
from nimmake.Helper import Helper

toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

print("== Welcome to  Nimmake! ==")
hlp = Helper()

CFG = CORTEX_M4_CFG.clone()
hlp.Config(CFG)

hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})

hlp.Refresh()

print(TOOL_OF(tool, prefix))

print(hlp["CFG"])

print("====== 01 ==========")
print(hlp.Flags)
print("====== 01 ==========")
```

## 如何修改编译标志

1. 你可以通过 set_cfg 函数修改工具链标志。

hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")

2. 你可以通过 APPEND 函数或 UPDATE 函数修改标志。

   hlp.Append(LIBS="m")
   hlp.Prepend(LIBS="c")
   hlp.Append(DEFINES={"XXXX": "123"})
   hlp.Prepend(DEFINES={"YYYY": "456"})
   hlp["TOOLPATH"] = [toolpath2]
   hlp.Append(TOOLPATH=toolpath2)
   hlp.Update(toolchain)
   hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
   print(TOOL_OF(tool, prefix))

# 如何添加源码

我们通过类似第三方库的模式添加源码。把外部目录或自己定义的目录作为一个 party，添加整个 party。

1. 自动搜索指定扩展名的文件作为源码。
2. 排除某个子文件夹中的文件。
3. 排除文件名以特定前缀开头的文件。
4. 排除文件名以特定后缀结尾的文件。
5. 设置特定宏以及内部包含宏。

```
driver = hlp.Parties(
    name="Driver", root="src_stm/Drivers",
    source_exts=["*.c"],
    header_exts=["*.h"],
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""}
)
```

## 设置目标

你可以添加包含源文件的目标。可以添加多个目标，但需要设置默认目标。

```
srcs = [
    "src_stm/startup_stm32f407xx.s",
]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```

## 通过mcu 厂商模型获得编译参数

```py
from nimmake.datasets import CORTEX_M4_CFG, TOOL_OF, VENDOR_MODEL_OF, Vendor
from nimmake.Helper import Helper
toolpath_armgcc = r"D:\LLVM\arm-none-eabi-gcc14\bin"
tool = "gcc"
prefix = "arm-none-eabi-"

print("== Welcome to  Nimmake! ==")
hlp = Helper()
CFG = CORTEX_M4_CFG.clone()
CFG = VENDOR_MODEL_OF(Vendor.ST, "STM32F407")

hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
hlp.Refresh()

```
