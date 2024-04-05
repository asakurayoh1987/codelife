# Mac M2 Rust交叉编译

## 一、添加配置文件

项目下添加`.cargo/config.toml`用来配置构建目标相关

```bash
mkdir -p .cargo
touch .cargo/config.toml
```

## 二、构建windows平台下的二进制包

```bash
# 安装MinGW-w64（为Windows的GNU环境）
brew install mingw-w64
# 添加Windows的目标架构
rustup target add x86_64-pc-windows-gnu
```

在项目的`.cargo/config.toml`添加构建目标

```toml
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
```

可以通过`x86_64-w64-mingw32-gcc --version`来验证linker是否安装成功

```bash
# 指定target来进行构建
cargo build --release --target x86_64-pc-windows-gnu
```

成功的话就会在`target/x86_64-pc-windows-gnu/release`目录下生成Windows可执行文件

## 三、构建CentOS7.9平台下的二进制包

```bash
# 用于交叉编译，可以生成静态链接的Linux二进制文件，非常适合在不同Linux发行版之间移植
brew install FiloSottile/musl-cross/musl-cross
# 验证是否安装成功
x86_64-linux-musl-gcc --version
# 添加目标架构
rustup target add x86_64-unknown-linux-musl
```

通过以下命令可以查看目标是否安装

```bash
rustup target list | grep x86_64-unknown-linux-musl
```

如果对应的target后面有`(installed)`就表示已安装

在项目的`.cargo/config.toml`添加构建目标

```toml
[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-musl-gcc"
```

构建项目

```bash
cargo build --target x86_64-unknown-linux-musl --release
```

## 四、项目样例

`.cargo/config.toml`内容

```toml
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"

[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-musl-gcc"
```

`src/lib.rs`

```rust
pub mod encryptor {
    use aes::Aes128;
    use base64::prelude::*;
    use block_modes::block_padding::Pkcs7;
    use block_modes::{BlockMode, Ecb};
    use sha1::{Digest, Sha1};

    type Aes128Ecb = Ecb<Aes128, Pkcs7>;

    pub fn aes3d(content: &str, secret: &str) -> Result<String, Box<dyn std::error::Error>> {
        let mut hasher = Sha1::new();
        hasher.update(secret.as_bytes());
        let keysha1 = hasher.finalize();

        let mut hasher = Sha1::new();
        hasher.update(&keysha1);
        let real_key = hasher.finalize();
        let key_bytes = &real_key[..16];

        let cipher = Aes128Ecb::new_from_slices(key_bytes, Default::default())?;
        let encrypted_data = cipher.encrypt_vec(content.as_bytes());
        let encoded = BASE64_STANDARD.encode(encrypted_data);
        Ok(encoded)
    }
}
```

`src/main.rs`

```rust
use std::io::{self, Read, Write};

use crypto_tool::encryptor::aes3d;

fn main() {
    match aes3d("17755105631", "!iflytek") {
        Ok(encrypted) => println!("Encrypted: {}", encrypted),
        Err(e) => eprintln!("Error: {}", e),
    }

    let mut stdout = io::stdout();
    stdout.write_all(b"Press Enter to exit...").unwrap();
    stdout.flush().unwrap();
    io::stdin().read(&mut [0]).unwrap();
}
```

`Cargo.toml`

```toml
[package]
name = "crypto-tool"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
aes = "0.7"
base64 = "0.22.0"
block-modes = "0.8.1"
hex = "0.4.3"
sha-1 = "0.10.1"
```

