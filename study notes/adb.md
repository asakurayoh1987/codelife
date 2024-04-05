# Adb 常用命令

## 查看当前已连接的设备

```bash
adb devices
```

## 查看当前运行应用的包名

```bash
adb shell dumpsys window | grep mCurrent
# 直接打印当前的package与activity
adb shell dumpsys window | grep -E 'mCurrentFocus' | awk '{print $3}'
```