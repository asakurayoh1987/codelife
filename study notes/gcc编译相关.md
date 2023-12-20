## 一、GCC编译过程

### 1.1 查看GCC编译过程

```bash
# 注意这里是大写
GCC -ccc-print-phases test.c
# 下面的是输出，当前环境是m2 mac，arm64
               +- 0: input, "test.c", c
            +- 1: preprocessor, {0}, cpp-output
         +- 2: compiler, {1}, ir
      +- 3: backend, {2}, assembler
   +- 4: assembler, {3}, object
+- 5: linker, {4}, image
6: bind-arch, "arm64", {5}, image
```

### 1.2 编译相关的命令

- 预编译，preprocessor

  ```bash
  gcc -E test.c -o test.i
  ```

- 生成汇编文件，compiler

  ```bash
  gcc -S test.i -o test.s
  ```

- 生成目标文件，backend、assembler

  ```bash
  gcc -c test.s -o test.o
  ```

- 生成可执行文件，linker、bind-arch

  ```bash
   gcc test.o -o test
  ```

## 二、Clang编译过程

查看本机Clang版本

```bash
objdump --version
```

### 2.1 查看Clang编译过程

```bash
clang -ccc-print-phases test.c
# 以下为输出
               +- 0: input, "test.c", c
            +- 1: preprocessor, {0}, cpp-output
         +- 2: compiler, {1}, ir
      +- 3: backend, {2}, assembler
   +- 4: assembler, {3}, object
+- 5: linker, {4}, image
6: bind-arch, "arm64", {5}, image
```

