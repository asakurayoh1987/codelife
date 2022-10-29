# .gitignore的exclude写法

## 前提

如果某个文件夹被直接以xxx/的方式ignore了，那其下的所有子目录及文件都无法被exclude：

```bash
application/
# 这一句不生效
!application/favicon.ico
```

## 写法

```bash
# 这里是为了表示父目录必须没被ignore
!application/
# ignore目录application下的所有子目录和文件
application/*
# exclude其下的favico.ico文件
!application/favico.ico
# ignore目录application/language下的所有目录和文件
application/language/*
# exclude目录application/language/gr/下的所有目录和文件
!application/language/gr/
```

## “/*”的意义

* dir/ 表示ignore名为dir的文件夹，并且git不会再去查看dir下的任何目录和文件，其下的exclude的表达式就不会生效。
* dir/* 表示的是ignore其下的所有，而不是dir本身，所以后续的exclude表达式才可以生效。