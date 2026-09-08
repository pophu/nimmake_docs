# BasicUsage

## chip

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

## toolchain and tool path

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

## compile flags

In your src dir, New file Makefile.py.
You will set the toolchain info and tool path in this file. and you will designate the CPU architectures. Nimmake will help you get the compile flags.

you can print hlp.Flags to get the compile flags. You will check the flags in the output.

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
# hlp.TOML()

############
# TOOL
############
hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
print(TOOL_OF(tool, prefix))

print(hlp["CFG"])

hlp.Refresh()

print("====== 01 ==========")
print(hlp.Flags)
print("====== 01 ==========")
```

## how to modify flags

1. you can modify the toolchain flags by the set_cfg function.

hlp.set_cfg("linkscript", "src_stm/STM32F407XX_FLASH.ld")
hlp.set_cfg("cpu", "cortex-m4")

2. you can modify the flags by APPEND function or UPDATE function.

   hlp.Append(LIBS="m")
   hlp.Prepend(LIBS="c")
   hlp.Append(DEFINES={"XXXX": "123"})
   hlp.Prepend(DEFINES={"YYYY": "456"})
   hlp["TOOLPATH"] = [toolpath2]
   hlp.Append(TOOLPATH=toolpath2)
   hlp.Update(toolchain)
   hlp.Update({"TOOLPATH": toolpath_armgcc, "TOOL": tool, "TOOL_PREFIX": prefix})
   print(TOOL_OF(tool, prefix))

# how to add source

We add source code in a third-party library style. Treat an external directory or your own directory as a party, and add the entire party.

1. Automatically search for files with specified extensions as source code.
2. Exclude files from specific subdirectories.
3. Exclude files with specific prefixes in their filenames.
4. Exclude files with specific suffixes in their filenames.
5. Set specific macros and internal include macros.

```
driver = hlp.Parties(
    name="Driver", root="src_stm/Drivers",
    source_exts=["*.c"],
    header_exts=["*.h"],
    defines={"STM32F407xx": "", "USE_HAL_DRIVER": ""}
)
```

## set TARGET

You can add the target with srcs. Multiple targets can be added, but you should set default target.

```
srcs = [
    "src_stm/startup_stm32f407xx.s",
]
t = hlp.Program("test", sources=srcs)
hlp.DefaultTarget(t)
```
