# Nimmake Command-Line Parameters

Nimmake provides a rich set of command-line parameters for controlling build behavior, debugging, and optimizing build speed.

---

## `-f <file>` / `--file <file>`

Specifies the build script file. Defaults to `Nimmake.py` in the current directory.

```bash
nimmake -f Nimmake.py
nimmake -f build.py
nimmake --file my_build.py
```

---

## `-c` / `--clean`

Cleans build artifacts, equivalent to `--clean`.

```bash
nimmake -c
nimmake --clean
```

After cleaning, the next build will recompile all files.

---

## `--compiledb`

Generates a `compile_commands.json` file for use by IDEs (such as VSCode, CLion) and static analysis tools (such as clang-tidy, clangd).

```bash
nimmake --compiledb
```

The generated `compile_commands.json` contains the compilation command for each source file, helping IDEs provide accurate code completion, syntax highlighting, and static analysis.

---

## `--ninja` / `-n`

Uses Ninja as the backend build engine for faster incremental builds. Suitable for large projects.

```bash
nimmake --ninja
nimmake -n
```

The Ninja backend focuses on speed. The generated `build.ninja` file can be executed directly by Ninja. On first use, Ninja build files are generated; subsequent builds only compile changed parts.

---

## `--verbose`

Displays the full details of each compilation command, including the compiler path, all compilation flags, and parameters. Useful for debugging compilation issues.

```bash
nimmake --verbose
```

In the `Nimmake.py` script, you can also set this via code:

```python
hlp.VERBOSE = True
```

---

## `--dry-run`

Preview mode: shows the operations that would be performed without actually compiling any files. Useful for verifying whether the build plan is correct.

```bash
nimmake --dry-run
```

In the `Nimmake.py` script, you can also set this via code:

```python
hlp.DRY_RUN = True
```

---

## `-j <N>` / `--jobs <N>`

Sets the number of parallel compilation threads. `N` is the number of tasks to compile simultaneously.

```bash
nimmake -j4          # 4-thread parallel compilation
nimmake -j8          # 8-thread parallel compilation
nimmake --jobs 4
```

It is recommended to set this to the number of CPU cores for optimal compilation speed.

---

## `--cache`

Enables compilation caching, which caches build artifacts to disk. On rebuild, if source files have not changed, cached results are used directly, significantly speeding up builds.

```bash
nimmake --cache
```

The cache directory defaults to `.nimmake_cache` in the build directory.

---

## `--version`

Displays the current version of Nimmake.

```bash
nimmake --version
```

---

## Parameter Quick Reference

| Parameter            | Short | Description                            |
| -------------------- | ----- | -------------------------------------- |
| `--file <file>`      | `-f`  | Specify the build script file          |
| `--compile_commands` | `-c`  | Clean build artifacts                  |
| `--compiledb`        | —     | Generate `compile_commands.json`       |
| `--ninja`            | `-n`  | Build with Ninja backend               |
| `--verbose`          | —     | Display full compilation commands      |
| `--dry-run`          | —     | Preview mode, no actual compilation    |
| `--jobs <N>`         | `-j`  | Number of parallel compilation threads |
| `--cache`            | —     | Enable compilation caching             |
| `--version`          | —     | Display version number                 |
