# Android端解包并查看Url Schema

## 查看当前应用的包名

这里以小米主题为例，手机通过USB连接电脑，开启调试模式

```bash
adb shell dumpsys window | grep -E 'mCurrentFocus|mFocusedApp'
# 拿到的包名为com.android.thememanager
```

## 查看包的位置

```bash
adb shell pm path com.android.thememanager
# 得到的地址为：/data/app/~~n-274h86TY0SOj251822PQ==/com.android.thememanager-NvMJ3LampDLI6iM1VnIqpA==/base.apk
```

## 下拉包到本地

```bash
adb pull /data/app/~~n-274h86TY0SOj251822PQ==/com.android.thememanager-NvMJ3LampDLI6iM1VnIqpA==/base.apk
# 将手机端的包拉到本地
```

## 解包

```bash
apktool d base.apk
```

apktool可以去官网下载，mac上可以通过`brew install apktool`安装

## 查看Url Schema

解包之后就可以在生成的文件夹（apk同名）中找到`AndroidManifest.xml`，查看其中的内容，可以看到很多的`intent-filter`节，如下：

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:host="zhuti.xiaomi.com" android:path="/search" android:scheme="theme"/
>
</intent-filter>
```

对于这段，对应的url schema为`theme://zhuti.xiaomi.com/search`，分别对应`<data android:host="zhuti.xiaomi.com" android:path="/search" android:scheme="theme"/`中的`android:scheme`、`android:host`、`android:path`

PS：对应`action`表示此条url schema的动作

## 验证

手机端访问`https://kuyin.iflysec.com/ycyu/wepay.html`，然后地输入框中输入`theme://zhuti.xiaomi.com/search`，点击按钮访问该链接，可以拉起小米主题并进入搜索页面

PS：不要在微信中访问

<video src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/502675d943b0406b8a0534e53fe1dcf7.mp4" autoplay="false" controls style="zoom: 80%"></video>

