# H5播放aac音频调研报告

##为了节省大家时间，我先说结论：

1. 现网的aac音频中存在两种容器格式（可能不止，目前是根据建工提供的200条音频来分析的），一种是*ADTS*，另一种*MPEG-4*，见下方图示（使用工具MediaInfo查看所得）：

2. 对于ADTS与MPEG-4这两种容器封装的aac音频的支持情况如下：

|容器格式|H5（移动端）|安卓|iOS|
|:-:|:-:|:-:|:-:|
|ADTS|下载完整音频后可播放|流式播放|获取时长存在问题，如：只能播放前3秒|
|MPEG-4（Base Meida/Version 2）|流式播放|流式播放|流式播放|
|MPEG-4(Apple audio with iTunes info)|流式播放|不能播放（无法解码）|流式播放|

3. **综上，需要对现网的音频进行清洗，都使用MPEG-4(Base Media/Version 2)的容器格式，同时建议后缀名改为m4a，另外对于这种音频的MIME类型也建议设置一下（即http的响应头Content-Type的内容），应为audio/mp4，现网为application/octet-stream**


现网两种容器的音频文件的信息

![image-20190828093202877](/Users/ycyu/Library/Application Support/typora-user-images/image-20190828093202877.png)

![img](file:///var/folders/v_/dv4tgf_s7gb3qb6lqszlcgph0000gn/T/WizNote/e4dce824-b0e8-4d15-9d4a-aec4deb8526a/index_files/50786383.png)

***

以下是我调研过程中了解到的一些知识点，列出来分享一下，如果有误还望指正哈

## html5中的audio对音频的支持

首先说下关于音频格式的问题，通过MDN以及别的网站上看到的h5的audio对于各种音频的支持情况如下（随着浏览器的更新，这个数据可能已不准确）：

![image-20190828094402604](/Users/ycyu/Library/Application Support/typora-user-images/image-20190828094402604.png)

audio可以设置多个音频格式，让其自主选择一种可以播放的格式，从上图中可以看到，同时使用ogg+mp3或ogg+aac即可满足绝大多数浏览器的播放要求了，同时在之前调研一些h5端的音频库时（soundmanager、howler），其demo中使用的都是ogg或webm，原因可能是因为ogg和webm是一种开源的格式，不像mp3或aac存在版权问题，当然在天朝，mp3因为各种问题，其版权问题貌似也是不存在的。

## ffmepg压制aac音频

接下来说说aac，关于其格式的介绍我就不说了，百度有一堆结果，我们现网是将mp3转成aac的，为了测试，首先考虑到的就是使用ffmpeg这个工具来转换aac的音频，具体的内容可以看ffmpeg官方的介绍：https://trac.ffmpeg.org/wiki/Encode/AAC，转换的命令如下：
```
ffmpeg -i rBBGq1c5pQuAbcG6ALdm2g15itw184.mp3 -c:a libfdk_aac -vbr 4 -movflags faststart -y 1.m4a
```
说明一下：
* -i 后为输入文件
* -c:a 用于指定音频的编码器，这里使用的是libfdk_aac，原生内置的是编码器可以使用aac，据官方的描述是前者压出来的音质要好于后者，但我这种五音不全的人真心听不出来（后来又查了一下官方文档，官方推荐使用原生的aac编码器，说在128比特率时，音质好于libfdk_aac）
* -vbr 是用于使用VBR（可变码率）模式，可选值为1-5，越高音质越好，但也意味着文件越大，此处还可以使用CBR（固定码率模式），比如-b:a 128k，即指定音频的码率为128kb/s
* -movflags faststart 按官方描述，这就是能流式播放的关键，其描述大概意思为默认情况下mp4文件会将一些“关键信息”（我的理解是会影响到音频时长等信息的获取）写在音频流的最后，这就导致要完整下载音频之后才能播放，而这两个参数就是将这个“关键信息”移到文件的开头，从而实现边下载边播放（实验时发现没加这两个参数时也可以流式播放）
* -y 当文件已存在时直接覆盖

压出来后的文件在移动端页面上测试确实可以流式播放，但当将该音频交给安卓端同事（马良）进行测试时，说无法播放，因为解码不了，原因为此种方式压制出来的aac文件，其文件格式是这样的：
，此种通过ffmpeg是可以解码的，但客户端的解码器是自己写的，无法解码此种格式，虽然iOS端是可以正常播放的，于是放弃用ffmepg来压制

## 使用neroaac压制aac音频

从马良那了解，我们的音频是研究院的同事使用neroaacenc这个工具来压制，于是又折腾一番，此工具只支持输入文件为wav，命令为：
```
neroAacEnc -lc -q 0.5 -if out.wav -of 1.m4a
```
参数说明如下：
* -lc 指定编码格式为LC AAC（还可以指定-he和-hev2，对应为HE AAC和HEv2 AAC，这个具体百度哈）
* -q 用于指定音频的质量，取值范围是0-1，研究院用的是0.5
* -if和-of 用于指定输入文件和输出文件

压出来的文件格式为MPEG-4（Base Meida/Version 2），经测试在H5、安卓、iOS上都可以正常流式播放了

## 历史问题

关于现网同时存在ADTS与MPEG-4两种格式的问题，现在已经无从查起了，只记得最早也做过h5页面使用aac播放的需求，当时是胡总负责处理的，最后上线时发现要等音频下载完才能播放，当时也查了，但结论并没有公布出来，只是确认是aac文件的问题，目测就是这个封装容器的问题

## 资料

上面的描述只写了重要的部分，在调研过程中看了一些文章，觉得还是很有用处的，发出来有兴趣的可以了解一下哈
* https://developer.mozilla.org/en-US/Apps/Fundamentals/Audio_and_video_delivery/Cross-browser_audio_basics
* https://developer.mozilla.org/zh-TW/docs/Web/HTML/Supported_media_formats
* https://trac.ffmpeg.org/wiki/Encode/AAC
* http://html5doctor.com/multimedia-troubleshooting/
* http://html5doctor.com/html5-audio-the-state-of-play/
* https://github.com/wangjx9110/document/blob/master/audio_summary.md

