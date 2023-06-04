# Rust实战

## 第5章 深入理解数据

### 5.1 位模式和类型

```rust
fn main() {
  let a: f32 = 42.42;
  // 这里也可以使用i32
  let frankentype: u32 = unsafe {
    // 这会要求Rust在不影响任何底层的位数据的情况下，将一个f32直接解释成一个u32
    std::mem::transmute(a)
  };
  println!("{}", frankentype);
	// 这里的"{:032b}"的意思将输出格式化为一个二进制的值，并且要占用32位，位数不足则在其左侧补零
	// 这是借助std::fmt::Binary这个trait来实现的
  // 如果是"{}"则会调用std::fmt::Display这个trait，如果是"{:?}"则会调用std::fmt::Debug这个trait
	println!("{:032b}", frankentype);
  
  let b: f32 = unsafe {
    std::mem::transmute(frankentype)
  };
  println!("{}", b);
  assert_eq!(a, b);
}

```

### 5.2 整数的生存范围

