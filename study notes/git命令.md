# Git 命令

- git checkout . #本地所有修改的。没有的提交的，都返回到原来的状态
- git stash #把所有没有提交的修改暂存到 stash 里面。可用 git stash pop 回复。
- git reset --hard HASH #返回到某个节点，不保留修改，已有的改动会丢失。
- git reset --soft HASH #返回到某个节点, 保留修改，已有的改动会保留，在未提交中，git status 或 git diff 可看。
- git clean -df #返回到某个节点，（未跟踪文件的删除）
- git clean 参数
  - -n 不实际删除，只是进行演练，展示将要进行的操作，有哪些文件将要被删除。（可先使用该命令参数，然后再决定是否执行）
  - -f 删除文件
  - -i 显示将要删除的文件
  - -d 递归删除目录及文件（未跟踪的）
  - -q 仅显示错误，成功删除的文件不显示

**注：**

- git reset 删除的是已跟踪的文件，将已 commit 的回退。
- git clean 删除的是未跟踪的文件
- git clean -nxdf（查看要删除的文件及目录，确认无误后再使用下面的命令进行删除）
- git checkout . && git clean -xdf
- git clone --depth=1 进行浅拷贝，不克隆历史信息

# 解决命令行中文显示成 ascii 码的问题

- git config --global core.quotepath false
- 终端的字符集改成utf-8
