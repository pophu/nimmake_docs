# 高级用法

## 如何构建 PC 程序

请参考 samples\02_win_exe 目录。执行 nimmake，将生成 test.exe 文件。

## 如何使用 arm-none-eabi-gcc 构建 ARM32 MCU 程序

从 [xpack arm gcc](https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack/) 下载工具链。

请参考以下目录：
samples\03_stm32_led_reg
samples\04_arm32_led_cmsis
samples\05_arm32
samples\06_arm32_lib

执行 nimmake，将生成 test.elf 文件。

## 如何使用 clang 构建 ARM32 MCU 程序

从 [arm llvm toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases) 下载工具链。

请参考以下目录：
samples\09_arm_llvm

## 如何使用 arm-none-eabi-gcc 构建 RISC-V MCU 程序

从 [xpack riscv](https://xpack-dev-tools.github.io/riscv-none-elf-gcc-xpack/) 下载工具链。

请参考以下目录：
samples\07_riscv32
samples\08_riscv64
