### config
```sh
# 全局配置文件 ~/.gitconfig
git config --global user.name "[name]"
git config --global user.email "[email address]"
# 启用有帮助的彩色命令行输出
git config --global color.ui auto 
# 设置commit时使用的编辑器为vim
git config --global core.editor vim

# project级别的配置文件 .git/config
git config user.name "[name]"
git config user.email "[email address]"

# remote
# 列出远程主机名和远程主机的网址
git remote -v
# 查看主机详细信息
git remote show <远程主机名>
# 改名
git remote rename <原主机名> <新主机名>
```

### 常用指令
```sh
# clone到mylibgit目录并切换到master分支
git clone -b master https://github.com/libgit2/libgit2 mylibgit

# 第一列表示暂存区状态，第二列表示工作区状态(本地): 新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。
git status -s

# 文件比较
# 此命令比较的是工作目录中当前文件和暂存区域快照之间的差异。 也就是修改之后还没有暂存起来的变化内容。 比较本地未add到stage区的file和stage区的file比较
git diff <file> 
# 比较stage的file和commit的file比较. 可选参数 file，不加则比较全部的文件。--staged 和 --cached 是同义词，效果一样。
git diff --staged [file]
git diff --cached [file]

# 将某个文件(使用add命令添加到暂存区后)回退到上次提交到暂存区后的状态（注意不是上次commit后的状态,ps: 可以使用上面的reset命令将暂存区的文件拉下来再使用checkout命令回退）
git checkout -- <file>
# 取消某个文件的暂存（从 stage -> unstage ）
git reset HEAD <file>

# commit
git commit -m "Fix: benchmarks for speed"
# -a选项，自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤。（注意是已跟踪过的文件，新文件还是需要add）
git commit -a -m 'added new benchmarks'
# 重新提交（作用：将上一次的commit和本次的commit合成一个commit）
git commit --amend

# 查看提交历史
# 查看最后一次提交，可以选择加上head参数 git log -1 head
git log -1
# 在每次提交的下面列出所有被修改过的文件
git log --stat
# 美化成一行显示, 图形化显示当前分支提交情况
git log --oneline --graph
# 现实这个项目所有分支的提交情况
git log --oneline --graph --all

# 将所有本地分支提交上传到 GitHub
git push <远程主机名> <本地分支名>:<远程分支名>
git push origin devlop:devlop
```

### 分支
```sh
# 查看本地分支 -r远程 -a所有
git branch
# 切换到指定分支并更新工作目录 | git checkout [branch-name]
git switch "[branch-name]"
# 创建本地分支dev | git branch [branch-name]
git switch -c "[branch-name]"
# 删除本地分支dev | git branch -d 
git branch -d "[branch-name]"
# 删除远程分支dev
git push origin --delete dev
# 查看本地分支的最后一次提交。
git branch -v

# 分支合并： 在指定分支合并dev分支 使用--no-ff模式
git switch master
git merge dev
git branch -d "[branch-name]"
```

### 同步更改
如果你在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们。(merge 冲突)
```sh
# fetch: 下载远端跟踪分支的所有历史
# git fetch --all # 下载所有主机远端跟踪分支的所有历史（并不会修改工作目录中的内容，从服务器更新数据，只会获取数据然后需要自己手动合并）
# 从远程的origin仓库的最新的master分支下载到本地，并新建为一个temp分支。
git fetch <远程主机名> <远程分支名>:<本地分支名>  # git fetch origin master:temp
git fetch origin     # 从服务器更新所有分支信息; 或者使用 git fetch origin master 只更新master分支
git merge origin/master   # 将远程master分支合并到本地

# --no-ff: 禁止快进模式，对master分支非常友好，建议合并时都加上这个参数
git switch master
git merge --no-ff -m "merge with no-ff" hotfix
git branch -d hotfix

# 使用变基
# git rebase <basebranch> <topicbranch> 
# 将server分支rebase到master分支
git rebase master server
git switch master
git merge server
git branch -d server

# git pull 是 git fetch 和 git merge 的结合。推荐使用fetch和merge代替pull。
git pull <远程主机名> <远程分支名>:<本地分支名>  # 将远程分支拉取并合并到本地分支
git pull origin master:master
```

### 打标签
```sh
# 列出已有标签
git tag
# 给某次commit打上标签 v1.2
git tag -a v1.2 f8603ca
# 查看打标信息
git show v1.2
# 将标签信息推送到服务器。 git push origin --tags 推送所有不在远程的标签
git push origin <tagname>
# 删除本地标签。 删除后执行 git push origin --tags 是不是也能将远程仓库上的标签删掉？
git tag -d <tagname>
# 删除远程仓库的标签
git push origin --delete <tagname>
```

### .gitignore 文件
```sh
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

### submodule
```sh
git submodule add https://github.com/chaconinc/DbConnector
cat .gitmodules

# 克隆含有子模块的项目 git clone 命令传递 --recursive 选项，它就会自动初始化并更新仓库中的每一个子模块
# 推荐使用 git clone --recursive XXXXXX
git submodule init #用来初始化本地配置文件
git submodule update # 从该项目中抓取所有数据并检出父项目中列出的合适的提交

# 查看所有子模块，子模块前面有一个-，说明子模块文件还未检入（空文件夹）
git submodule
```

``` sh
回退：
HEAD 是当前分支引用的指针，它总是指向该分支上的最后一次提交。 HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。
要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。

保存现场：
git add .
git stash
git stash list
git stash pop （git stash apply stash@{0} && git stash drop） ： 恢复现场在删除
```

