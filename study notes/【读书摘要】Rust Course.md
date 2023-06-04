# Rust Course

## 0. 环境准备

### 安装rustup

```bash
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

### 安装c语言编译器

rust会依赖`libc`和链接器`linker`

- macOS

  ```bash
  xcode-select --install
  ```

- linux

  一般按照对应发行版本的文档来安装gcc或clang，如果是ubuntu，则可安装build-essential

### 创建项目

```bash
# vcs=none表示不添加版本管理系统，否则会添加git版本控制
cargo new world_hello --vcs=none
```

```rust
// Rust 原生支持 UTF-8 编码的字符串
fn greet_world() {
    let southern_germany = "Grüß Gott!";
    let chinese = "世界，你好";
    let english = "World, Hello";
    let regions = [southern_germany, chinese, english];
    // Rust 的集合类型不能直接进行循环，需要变成迭代器，但2021 edition之后就可以直接写成 for region in regions，因为for隐式将regions转换成迭代器了
    for region in regions.iter() {
        println!("{}", &region);
    }
}
fn main() {
    greet_world();
}

```

### 执行

```bash
cd world_hello
cargo run
```

