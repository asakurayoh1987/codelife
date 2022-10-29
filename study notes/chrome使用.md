# chrome使用

## 关于headless检查方式

```bash
./Google\ Chrome --headless --disable-gpu --dump-dom http://arh.antoinevastel.com/bots/areyouheadless
./Google\ Chrome --headless --disable-gpu --screenshot http://arh.antoinevastel.com/bots/areyouheadless
```

## 自动打开控制台

```bash
./Google\ Chrome --auto-open-devtools-for-tabs
```

### **获取和编译chromium**

> https://www.npmjs.com/package/headless-chromium

### **Linux安装依赖**

> yum install git python bzip2 tar pkgconfig atk-devel alsa-lib-devel \
>
> bison binutils brlapi-devel bluez-libs-devel bzip2-devel cairo-devel \
>
> cups-devel dbus-devel dbus-glib-devel expat-devel fontconfig-devel \
>
> freetype-devel gcc-c++ glib2-devel glibc.i686 gperf glib2-devel gtk2-devel \
>
> gtk3-devel java-1.*.0-openjdk-devel libatomic libcap-devel libffi-devel \
>
> libgcc.i686 libgnome-keyring-devel libjpeg-devel libstdc++.i686 libX11-devel \
>
> libXScrnSaver-devel libXtst-devel libxkbcommon-x11-devel ncurses-compat-libs \
>
> nspr-devel nss-devel pam-devel pango-devel pciutils-devel \
>
> pulseaudio-libs-devel zlib.i686 httpd mod_ssl php php-cli python-psutil wdiff \
>
> xorg-x11-server-Xvfb

### **设置代理获取chromium代码**

git 设置代理：

> git config --global http.proxy http://127.0.0.1:1080
>
> git config --global https.proxy https://127.0.0.1:1080
>
> git config --global --unset http.proxy
>
> git config --global --unset https.proxy

全局代理：

> export http_proxy="http://127.0.0.1:1080"
>
> export https_proxy="https://127.0.0.1:1080"

### **启动Headless模式**

> chrome --headless --remote-debugging-port=9222 https://chromium.org --disable-gpu

### **Chromium默认编译不支持音视频的播放**

在args.gn文件中增加编译参数

> proprietary_codecs = true
>
> ffmpeg_branding = "Chrome"

### **Chromium中视频不自动播放**

chromuim 66 版本以后的内核，在默认情况下和标签已经不能自动播放了。需要用户点击触发后才播放，或者要把播放设置为静音模式才可自动播放。

解决方法是，启动参数中增加以下参数来关闭这个默认策略

> –autoplay-policy=no-user-gesture-required

### **加速编译**

Chromium官方文档中提供了一些可以加速编译的GN编译项

> symbol_level = 0
>
> blink_symbol_level= 0
>
> enable_nacl = false

### **其他编译参数**

> is_debug：这个选项值可以为true或者false。当为true时编译debug版本，false时编译release版本。is_component_build：这个选项值可以为true或者false。当为true时将chromium代码编译成多个小的dll，false时代码编译成单个dll。一般我们编译debug版本时，设置is_component_build = true，这样每次改动编译链接花费的时间就会减少很多。编译release版本时，设置is_component_build = false，这样就可以把所有代码编译到一个dll里面。
>
> target_cpu：这个选项值为字符串，控制我们编译出的程序所匹配的cpu。编译32位x86版本设置成target_cpu =”x86″，编译64位x64版本设置成target_cpu =”x64″。如果我们没有显式指定target_cpu的值，那么target_cpu的值为编译它的电脑所用的cpu类型。通常target_cpu的值为x86会比x64编译速度更快，并且支持增量编译。另外如果设置了target_cpu =”x86″，也必须设置enable_nacl = false，否则编译速度会慢很多。
>
> enable_nacl：这个选项值可以为true或者false。控制是否启用Native Client，通常我们并不需要。所以把其值设置成enable_nacl = false。
>
> is_clang：这个选项值可以为true或者false。控制是否启用clang进行编译。目前m63 clang编译还不稳定，所以这个选项设置成is_clang = false。m64开始支持clang编译。
>
> ffmpeg_branding=”Chrome” proprietary_codecs=true。这个两个选项是控制代码编译支持的多媒体格式跟chrome一样，支持mp4等格式。
>
> symbol_level：其值为整数。当值为0时，不生成调试符号，可以加快代码编译链接速度。当值为1时，生成的调试符号中不包含源代码信息，无法进行源代码级调试，但是可以加快代码编译链接速度。当值为2时，生成完整的调试符号，编译链接时间比较长。
>
> is_official_build：这个选项值可以为true或者false。控制是否启用official编译模式。official编译模式会进行代码编译优化，非常耗时。仅发布的时候设置成is_official_build = true开启优化。

 

### **GN编译命令**

> \# 生成编译目录
>
> gn gen out/Default
>
> \# 设置编译目录的编译参数
>
> gn args out/Default
>
> \# 查看编译目录的编译参数
>
> gn args --list out/Default
>
> \# 启动编译
>
> ninja -C out/Default
>
> \# headless_shell编译
>
> ninja -C out/Release headless_shell
>
> headless_shell编译参数
>
> import("//build/args/headless.gn")
>
> is_component_build = false
>
> is_debug = false
>
> symbol_level = 0
>
> blink_symbol_level= 0
>
> enable_nacl = false
>
> proprietary_codecs = true
>
> ffmpeg_branding = "Chrome"

### **启动参数**

Chrome：

> ./out/Default/chrome --headless --no-sandbox --ignore-certificate-errors --ignore-ssl-errors --disable-gpu --disable-software-rasterizer --remote-debugging-port=9222 https://www.baidu.com

headless_shell:

> ./out/Release/headless_shell --no-sandbox --ignore-certificate-errors --ignore-ssl-errors --disable-gpu --disable-software-rasterizer --remote-debugging-address=0.0.0.0 --remote-debugging-port=9222 https://www.baidu.com

### **headless_shell进程指令**

查看headless_shell进程是否存在

> ps -ef | grep headless_shell

### **关闭headless_shell进程**

> pkill -f '(chrome)?(--headless)'

### **VirtualBox虚拟机远程调试**

> 启动参数增加–remote-debugging-address=0.0.0.0 --remote-debugging-port=9222。
>
> 关闭虚拟机中操作系统的防火墙，或者开放9222端口。
>
> VirtualBox设置端口转发，从子系统9222到主机任意可用端口。
>
> 浏览器打开chrome://inspect/#devices开始调试。

 

### **调试相关**

> 调试地址：chrome://inspect/#devices
>
> 一些调试接口：
>
> http://127.0.0.1:9222/json 查看已经打开的Tab列表
>
> http://127.0.0.1:9222/json/version : 查看浏览器版本信息
>
> http://127.0.0.1:9222/json/new?http://www.baidu.com : 新开Tab打开指定地址
>
> http://127.0.0.1:9222/json/close/8795FFF09B01BD41B1F2931110475A67 :关闭指定Tab,close后为tab页面的id
>
> http://127.0.0.1:9222/json/activate/5C7774203404DC082182AF4563CC7256 : 切换到目标
>
> chromium C++与javascript互操作
>
> 仿照extensions_v8::LoadTimesExtension
>
> 在ChromeContentRendererClient的函数RenderThreadStarted()中注册thread->RegisterExtension(extensions_v8::XXXXExtension::Get());

[Headless Chrome使用](https://zhuanlan.zhihu.com/p/29207391)