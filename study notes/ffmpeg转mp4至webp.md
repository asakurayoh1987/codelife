# ffmpeg 将 mp4 视频转换为 webp 格式动图

在 React Native 等技术实现的 app 中，为了添加简单的动效或宣传动画，一种方法是通过应用 webp 或 gif 格式的动态图片以减少对视频的播放控制层面的开发功夫。 若源文件为视频格式，如 .mp4 或 .mov， 则需要转换格式。虽然网上有几款转换器能实现视频至 webp 动图的格式转换，但由于渲染质量、尺寸等的设置限制，往往效果未能完美。

ffmpeg 是十分强大的视频录制及格式转换的命令行工具，使用 ffmpeg 可以实现高度自定义的格式转换。本文将以 mp4 格式为例，介绍将视频转换为 webp 动图格式的方法。

### 关于 WebP

WebP，非官方英文读法是 weppy ‘/wepi/‘。

#### 属性及特点：

1. 文件格式是 .webp
2. 无损压缩或有损压缩皆可
3. 与 JPEG 格式图片相比，WebP 格式下的有损图片能比尺寸小 25-34%
4. 与 PNG 格式图片相比，WebP 格式下的无损图片能比尺寸小 25%
5. WebP 支持无损的透明涂层，类似带 alpha channel 的 PNG 文件
6. WebP 支持动画，类似带动画的 GIF 文件

总的来说，WEBP 格式文件能够实现比 JPEG、PNG 和 GIF 更小的文件体积。

### MP4 转为 WebP 遇到的问题

基于 WebP 图片属性无需播放控制及更小的文件尺寸的考量，最近需将 MP4 格式的动画短视频转为 WebP 格式实现用于 app 中的短时动画。一开始的方法是使用在线的 Video to WebP 转换器 [ezgif](https://ezgif.com/video-to-webp) 来转换格式。在线的可视化转换器操作上虽然方便，但却出现了以下问题：

1. ezgif 支持导出的最大webp分辨率只有 800px，难以自定义导出尺寸
2. 使用 10fps 的帧率（frame per second）导出的 webp 的视觉效果不佳
3. 使用 20fps 或更高帧率导出的 webp 图片的色彩损失严重。若画面使用较多色彩或渐变色，导出的 WebP 动画文件部分画面会出现马赛克效果，视觉体验不佳。

由于以上问题，我也尝试使用其他在线视频-webp转换器，例如 [OnlineConverter](https://www.onlineconverter.com/video-to-webp) 和 [hnet](https://hnet.com/video-to-webp/)，然而他们要么不支持循环播放设置，要么导出文件的色彩渲染马赛克问题更加严重。

另外，若源视频需要保密，那么在线编辑器的服务器可能会将视频存入服务器缓存，存在一定的风险。

### 解决方案

由于视频转换出的 WebP 属于动画模式，网上可以找到的支持此类格式的转换器数量很少，亦没有找到一款可以在本地运行的软件。不过可以找到一些通过使用 ffmpeg 的 libwebp 组件实现的转换。可惜网上的案例命令行大都笼统，没有给出太具体的 MP4 至 WebP 及设置相关导出参数的案例。 终于，在参考了 [ffmpeg libwebp 的相关文档](http://ffmpeg.org/ffmpeg-all.html#libwebp) 之后，总结出了以下基于 ffmpeg 命令行在 MacOs 上的将视频转为 WebP 动画的方法：

## 1. 使用 Homebrew 安装 ffmpeg

打开 terminal.app, 输入以下命令安装 ffmpeg：

```text
brew install ffmpeg
```

安装成功后，验证 ffmpeg 已安装：

```text
which ffmpeg
```

执行以上命令后若 terminal 返回类似 `/usr/local/bin/ffmpeg` 的结果，则表示 ffmpeg 已成功安装。

## 2. 设置并执行视频格式至 WebP 的转换命令

以下命令行可以将名为 input.mp4 文件转化为帧率为20帧每秒，循环播放，默认渲染预设效果，分辨率为 800px宽 600px 高的无损的文件名为 output 的 .webp 文件：

```text
ffmpeg -i input.mp4 -vcodec libwebp -filter:v fps=fps=20 -lossless 1 -loop 0 -preset default -an -vsync 0 -s 800:600 output.webp
```

若希望转出的 output.webp 动画只播放一次，有损，压缩级别为3（0-6，默认为4，越高效果越好），质量为70（0-100，默认为75，越高效果越好），越舍渲染为图片，可使用以下命令：

```text
ffmpeg -i input.mp4 -vcodec libwebp -filter:v fps=fps=20 -lossless 0 -compression_level 3 -q:v 70 -loop 1 -preset picture -an -vsync 0 -s 800:600 output.webp
```

### 主要选项:

- 将每秒帧率设为20: `-filter:v fps=fps=20`
- 设为导出为无损质量: `-lossless 1`
- 设为循环播放: `-loop 0`。 设为不循环播放： `-loop 1`
- 设置预设渲染模式 `-preset default` , 可按视频画面内容类型设置 `picture`, `photo`, `text`, `icon`, `drawing` 或 `none`。选择合适的渲染模式可导出更小的 webp 文件。 http://ffmpeg.org/ffmpeg-all.html#Options-28
- 将导出 webp 文件分辨率设为 800px*600px： `-s 800:600`

以上方法也适用于其他主流视频格式导出为 webp 或 gif 动画，更多转换选项，请参考 [ffmpeg 相关文档](http://ffmpeg.org/ffmpeg-all.html#libwebp)。