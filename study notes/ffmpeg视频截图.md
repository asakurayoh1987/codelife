# 利用ffmpeg进行视频截图

## 提取视频中所有帧

```bash
ffmpeg -i test.mp4 frames/%04d.jpg -hide_banner
```
参数说明：
* -i	指定输入文件
* -hide_banner	隐藏处理时的编译信息

提取当前视频中每一帧到当前目录下的frames文件夹中，截图的文件命名格式为“4位数字.jpg”。

## 提取视频中指定时间开始的指定帧数

```bash
ffmpeg -i test.mp4 -ss 00:00:01.000 -vframes 16 frames/%04d.jpg -hide_banner
```
参数说明：
* -ss	指定开始处理的时间
* -vframes	需要提取的帧数

## 提取视频中的截图并调整分辨率

```bash
ffmpeg -i test.mp4 -vf scale=375:-1 frames/%04d.jpg
```
参数说明：
* -vf	指定视频过滤器，这里scale表示进行缩放，宽度为375px，高度-1表示按比例缩放

## 按指定的时间间隔提取

```bash
ffmpeg -i test.mp4 -vf fps=2 frames/%04d.jpg -hide_banner
```
参数说明：
* -vf	指定视频过滤器，可以使用多个，这指指定的是fps=2，即每秒2帧，也可以使用如fps=1/5，表示每5秒截1帧

## 提取视频中的关键帧（Index Frames）

```bash
ffmpeg -i test.mp4 -vf "select=eq(pict_type\,I)" -vsync vfr frames/thumb%04d.jpg -hide_banner
```
参数说明：
* -vf	指定视频过滤器，这里使用的过滤表达式稍复杂，其中`pict_type\,I`表示图片类型为索引，即关键帧，`eq()`表示符合条件，`select=eq(pict_type\,I)`表示选取类型为关键帧的图片。
* -vsync vfr	指定使用可变比特率的视频同步方式，如果不指定此参数，ffmpeg除提取除关键帧之外的帧。

