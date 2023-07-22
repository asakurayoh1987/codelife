# git rebase 使用

示例：

当前需求你提交了三次commit，hash号分别为a->b->c（c是最新的一次commit），此时你想将这3个commit合并为一个，则需要进行如下操作：

```bash
# -i表示使用交互模式，x是rebase的起始节点，但rebase的内容不包它，也就是说这里的x是指a之前的那次提交
git rebase -i x
# 或者使用如下命令，表示操作最近的三次提交
git rebase -i HEAD~3
```

上述操作之后你可以看到大概如下界面：

```bash
pick a
pick b
pick c
# 省略其它
...
```

此时按字母“i”键，进入编辑模式，将b和c所在行的"pick"改为"s"，表示将该commit与前一个合并，修改完保存后（:wa）即可进入commit msg的编辑模式，此时会将三次commit时提交的msg都列出来，你可以选择性的进行更改，更改完保存后，则将三次commit合并为一个了

关于pick及相关指令的说明如下：

- pick：保留该commit（缩写:p）

- reword：保留该commit，但我需要修改该commit的注释（缩写:r）

- edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
- squash：将该commit和前一个commit合并（缩写:s）
- fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
- exec：执行shell命令（缩写:x）
- drop：我要丢弃该commit（缩写:d）

[来源链接1](https://juejin.cn/post/6844903600976576519)

[来源链接2](https://www.jianshu.com/p/4a8f4af4e803)

如果之前的commit已经提交过远端，在rebase后，要再次提交，此时需要使用`git push --force`