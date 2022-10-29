# Flutter学习笔记

## 安装

直接参考英文官网即可

## flutter doctor使用

检查环境问题，可以使用`flutter doctor -v`查看具体的详情

### 关于“Unable to find bundled Java version”

[参考](https://stackoverflow.com/questions/51281702/unable-to-find-bundled-java-version-on-flutter/68575967#68575967)

```bash
cd /Applications/Android\ Studio.app/Contents/jre
ln -s ../jre jdk
ln -s "/Library/Internet Plug-Ins/JavaAppletPlugin.plugin" jdk
flutter doctor -v
```

查看本机java安装的版本

```bash
/usr/libexec/java_home -V
```