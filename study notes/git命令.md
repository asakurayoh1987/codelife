# Git使用
## Git 命令

- git switch 切换分支，比起git checkout有更精细的检查
  - git switch other-branch
  - git switch - # 切回之前的分支，类似于`cd -`
  - git switch remote-branch # 切回远程分支并开始追踪

- git restore 将文件恢复到上次提交的版本
  - git restore --staged some-file.py \# Unstage changes made to a file, same as "git reset some-file.py"
  - git restore --staged --worktree some-file.py \# Unstage and discard changes made to a file, same as "git checkout some-file.py"
  - git restore --source HEAD~2 some-file.py \# Revert a file to some previous commit, same as "git reset commit -- some-file.py"

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

## 解决命令行中文显示成 ascii 码的问题

- git config --global core.quotepath false
- 终端的字符集改成utf-8
- git config --global i18n.commitencoding utf-8
- git config --global i18n.logoutputencoding utf-8

## 只clone部分仓库

```bash
# 通过指定--no-checkout来不进行实际上的仓库克隆
git clone --no-checkout https://github.com/derrickstolee/sparse-checkout-example
cd sparse-checkout-example
# 设置只匹配仓库根目录下的文件
git sparse-checkout init --cone
# 此时就只会checkout根目录下的文件
git checkout main
# 后期如果想下载指定的文件夹下的文件只需通过下面的命令：只下载service下的common文件夹
git sparse-checkout set service/common
```

这里有关于`sparse-checkout`更详细的[介绍](https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/)

## worktree的使用

```bash
git branch
# * dev
# master

git worktree list
# /.../some-repo  ews5ger [dev]

git worktree add -b hotfix ./hotfix master

# Preparing worktree (new branch 'hotfix')
# HEAD is now at 5ea9faa Signed commit.

git worktree list
# /.../test-repo         ews5ger [dev]
# /.../test-repo/hotfix  5ea9faa [hotfix]

cd hotfix/  # Clean worktree, where you can make your changes and push them
```

此时进入`hotfix`文件夹后，就会切到hotfix分支，进行修改后，可以合并到master分支，然后通过`git worktree remove hotifx`来清除worktree

更多详细介绍参见[这里](https://opensource.com/article/21/4/git-worktree)

