---
title: 一些git小坏坏们
date: 2017-08-15 20:41:15
tags: [git]
---
### diff
* `git diff --cached`
显示当前暂存区与上次提交间的差异；这些内容将在运行 "git commit"命令时就会被提交。

* `git diff HEAD`
显示工作目录与上次提交时之间的所有差别，这些内容都会在执行"git commit -a"命令时被提交。
<!--more-->
* `git diff commit1 commit2`
显示两次提交之间的所有差别

* `git diff branch1 branch2`
显示两个分支之间的所有差别，如果省略掉`branch1`，则显示当前分支与`branch2`之间的差别

* `git difftool commit1 commit2 <filename>`
显示两次提交之间`<filename>`路径下的文件的内容差别

* `git diff --stat`
显示有哪些文件被改动，有多少行被改动

### log
* `git log --pretty=full`
打印详细的log信息，相较`git log`命令，除了显示Commit，还显示了Author信息

* `git log --pretty=oneline`
查看远程仓库提交记录

* `git log --author="<username>"`
查看名叫`<username>`的作者提交的记录

* `git log --stat`
查看每次提交的文件变更的统计信息

* `git log -p <filename>`
查看`<filename>`目录下的文件每次详细修改内容

* `git log --since=<time>`
查看指定time的所有提交记录，例如`git log --since=2017-01-01`指2017年1月1日的提交记录，`git log --since=”1 years 1 day 30 minutes ago“`指1年1个月30分钟之前的提交记录

### reset
* `git reset --soft <commit>`
index和工作目录中的内容不作任何改变，仅仅把`HEAD`指向`<commit>`，执行完毕后，自从`<commit>`以来的所有改变都会显示在git status的"Changes to be committed"中,所有修改的文件被放入暂存区
* `git reset --hard <commit>`
自从`<commit>`以来在工作目录中的任何改变都被丢弃，并把`HEAD`指向`<commit>`
* `git reset --mixed <commit>`
工作区中文件的修改都会被保留，不会丢弃，但是也不会被标记成"Changes to be committed"，但是会打出什么还未被更新的报告。
* `git reset --hard HEAD`
回滚上一次的提交

### stash
* `git stash`
储存当前修改过的被追踪的文件和暂存的变更
* `git stash -a “<commit message>”`
将新加入的代码文件同时存储
* `git stash list`
查看所有暂存的信息,包括id
* `git stash apply stash@{<id>}`
恢复指定`<id>`的暂存且不从栈中删除该暂存
* `git stash pop stash@{<id>}`
恢复指定`<id>`的暂存且从栈中删除该暂存
* `git stash drop stash@{<id>}`
删除指定`<id>`的暂存

### add
* `git add -p <filename>`
将`<filename>`文件的修改拆分提交

### commit
* `git commit --amend`
将提交与git目录中的最新提交合并

### rebase
* `git rebase -i`
压缩多个Commit，输入命令后，将进入可交互模式，其中有几个参数
    * `reword`    修改commit信息
    * `squash`    合并上一个commit
    * `fixup`     合并上一个commit且不保存当前commit信息

### reflog
* `git reflog`
查看所有分支的所有操作记录,包括commit和reset的操作，已经被删除的commit记录

### blame
* `git blame <filepath>`
逐行显示文件，并在每一行的行首显示commit号,提交者,最早的提交日期

### cherry-pick
* `git cherry-pick --no-commit <commit_id>`
把另一个本地分支的commit修改应用到当前分支,`--no-commit`参数是防止`cherry-pick`每一次应用更改都commit一次
