# Mac相关

## 安装软件提示已损坏

```bash
sudo spctl --master-disable
sudo xattr -d com.apple.quarantine /Applications/iTerm.app
```

## 安装命令行基础工具

node.js、python3等都依赖此

```shell
xcode-select --install
```

## 使用nginx

默认情况下80端口被unix自带的apache占用了，所以要先关闭apache服务

```shell
# 启动
sudo apachectl start
# 关闭
sudo apachectl restart
# 重启
sudo apachectl stop
```

关闭apache服务后，要设置开机启动项，避免下次80端口继续被apache占用

```shell
# 关闭开机启动
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
# 开启开机启动
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

