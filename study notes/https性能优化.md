# https性能优化

## 一、实现环境搭建

### 1. nginx安装

```bash
# centOS
yum install nginx
# 安装之后可通过下面的命令查看nginx的位置
whereis nginx
```

### 2. 启动nginx

```bash
nginx
# 如果提示端口被占用，可以通过以下命令来查看端口
# 1. lsof -i
# 2. lsof -i:80 ，nginx默认是启在80端口
# 3. netstat -tunlp
```

### 3. https证书生成

[参见](https://github.com/FiloSottile/mkcert/blob/master/README.md)


