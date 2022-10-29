# 小米手机App中下载视频提取

## 手机备份

只备份选中的App，比如极客时间，备份完后将备份文件传至电脑，手机上备份文件位置：/SD卡/MIUI/backup/AllBackup下，根据时间命名

## 修改文件头

使用UltraEdit这样的工具打开备份文件，比如这里的fm.bak（frontend master），然后找到`41 4E`的位置，将其之前的内容都删除，然后保存

## 使用安卓工具包解压缩文件

使用安卓工具android-backup-extractor中的abe.jar来进行解压缩，使用如下命令

```bash
# 通过-jar指定abe.jar的位置，后跟备份文件的位置，再跟生成的普通压缩文件的位置
java -jar abe.jar fm.bak fm.tar
```
解压之后，即可找到文件中的非压缩的视频文件

