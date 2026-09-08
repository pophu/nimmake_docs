# 欢迎来到 Nimmake 文档

一款专为 ARM 和 RISC-V 架构优化的轻量级跨平台构建工具。基于 Python 的 MCU 固件开发构建系统，轻松编译 ARM、RISC-V 及其他常见微控制器的 C/C++ 源码。

内置丰富的 MCU 和工具链配置，自动生成所需的编译标志。以第三方库的方式添加代码，无需逐个手动添加源文件、头文件或宏定义；可设置各库之间的依赖关系。特别适合快速开发和原型验证。

## 特性

- 多工具链支持 — 使用 armgcc、LLVM/Clang 或 armclang 编译固件。
- 模块化第三方集成 — 预置 FreeRTOS、LVGL、FatFS、EasyLogger、CherryUSB、FreeModbus 等模块。
- 增量构建 — 分析需要重新编译的文件，仅编译发生变更的部分。
- 多进程构建 — 并行编译，可配置并发任务数 (-j)。
- Ninja 后端 — 可选的 Ninja 构建模式，获得更快的构建速度 (-n 或 --ninja)。
- 编译数据库 — 生成 compile_commands.json，为 IDE 和静态分析提供支持 (--compiledb)。
- 构建缓存 — 缓存编译产物，加速重构建 (--cache)。
- 预演模式 — 预览即将构建的内容，不实际执行编译 (--dry-run)。
- Party 系统 — 以可复用的 "party" 模块组织第三方库的标志和配置。

## 安装

```bash
pip install nimmake
```

## 用法

```bash
nimmake -f Nimmake.py
nimmake --version
nimmake --compiledb
nimmake --ninja
nimmake --verbose
nimmake --dry-run
nimmake -j4
nimmake -c
nmmk -f Nimmake.py

```

## git

[nimmake](https://github.com/pophu/nimmake)
