# Nimmake 命令行参数

Nimmake 提供了丰富的命令行参数，用于控制构建行为、调试和优化构建速度。

---

## `-f <file>` / `--file <file>`

指定构建脚本文件。默认为当前目录下的 `Nimmake.py`。

```bash
nimmake -f Nimmake.py
nimmake -f build.py
nimmake --file my_build.py
```

---

## `-c` / `--clean`

清理构建产物，等价于 `--clean`。

```bash
nimmake -c
nimmake --clean
```

清理后，下次构建将重新编译所有文件。

---

## `--compiledb`

生成 `compile_commands.json` 文件，供 IDE（如 VSCode、CLion）和静态分析工具（如 clang-tidy、clangd）使用。

```bash
nimmake --compiledb
```

生成的 `compile_commands.json` 包含每个源文件的编译命令，可帮助 IDE 实现准确的代码补全、语法高亮和静态分析。

---

## `--ninja` / `-n`

使用 Ninja 作为后端构建引擎，获得更快的增量构建速度。适合大型项目。

```bash
nimmake --ninja
nimmake -n
```

Ninja 后端专注于速度，生成的 `build.ninja` 文件可被 Ninja 直接执行。首次使用时会生成 Ninja 构建文件，后续仅编译变更的部分。

---

## `--verbose`

显示每条编译命令的完整细节，包括编译器路径、所有编译标志和参数。用于调试编译问题。

```bash
nimmake --verbose
```

在 `Nimmake.py` 脚本中，也可以通过代码设置：

```python
hlp.VERBOSE = True
```

---

## `--dry-run`

预览模式：显示将要执行的操作，但不实际编译任何文件。用于验证构建计划是否正确。

```bash
nimmake --dry-run
```

在 `Nimmake.py` 脚本中，也可以通过代码设置：

```python
hlp.DRY_RUN = True
```

---

## `-j <N>` / `--jobs <N>`

设置并行编译的线程数。`N` 为同时编译的任务数量。

```bash
nimmake -j4          # 4 线程并行编译
nimmake -j8          # 8 线程并行编译
nimmake --jobs 4
```

建议设置为 CPU 核心数，以获得最佳编译速度。

---

## `--cache`

启用编译缓存，将编译产物缓存到磁盘。重新构建时，如果源文件未发生变化，则直接使用缓存，大幅加速构建。

```bash
nimmake --cache
```

缓存目录默认为构建目录下的 `.nimmake_cache`。

---

## `--version`

显示 Nimmake 的当前版本号。

```bash
nimmake --version
```

---

## 参数速查表

| 参数                 | 简写 | 说明                         |
| -------------------- | ---- | ---------------------------- |
| `--file <file>`      | `-f` | 指定构建脚本文件             |
| `--compile_commands` | `-c` | 清理构建产物                 |
| `--compiledb`        | —    | 生成 `compile_commands.json` |
| `--ninja`            | `-n` | 使用 Ninja 后端构建          |
| `--verbose`          | —    | 显示完整编译命令             |
| `--dry-run`          | —    | 预览模式，不实际编译         |
| `--jobs <N>`         | `-j` | 并行编译线程数               |
| `--cache`            | —    | 启用编译缓存                 |
| `--version`          | —    | 显示版本号                   |
