[clone_error](git/clone_error_eof.md)  
[环境变量](git/run_path.md)   
git 记住账号密码  
```
[core]
## 主要是后面这两行, 如果不想保存, 则删除即可;  
[credential]
    helper = store
## 保存  
[remote "origin"]
[branch "master"]
```
### git 操作  
git log  
如何撤销已经push上去的commit    
git提供了好几种方法做到这点, 其中最简单的方法就是用git revert命令, 后面跟上你要撤销的commit id即可。  
git commit  
```
#  提交当前本地变动, 添加变动描述;  
git commit -am 更新  
git commit -am "add readMe file"  

git commit -am 等于下面两句,   
git add .
git commit -m "some str"

让 change id 发生变更  
git commit --amend  

```
git push  
```
#  把本地的, 当前分支, 推送到远端的, 当前分支上  
git push origin  
```

git fetch  
```
#  抓取远端的 当前分支
git fetch origin  

#  抓取远端的 feature/v5.0 分支
git fetch origin feature/v5.0  
```  
fetch 和 pull 区别  
```
git pull = git fetch + git merge  
git fetch 的时候只是将remote的origin进行update  但是并没有在local的branch进行merge  
```
git add  
```
git add -A    添加所有变化  
git add -u    添加被修改(modified)和被删除(deleted)文件, 不包括新文件(new)  
git add .      添加新文件(new)和被修改(modified)文件, 不包括被删除(deleted)文件  
```
git status  
```
位于分支 master
您的分支领先 'origin/master' 共 3 个提交。
  （使用 "git push" 来发布您的本地提交）
无文件要提交, 干净的工作区
```
#### 分支管理  
❀ 查看本地分支  
git branch  
前面带 *, 高亮的是当前分支  
```
test  
dev  
release  
* master  
```
❀  新建分支-切换分支  
```
#  查看远端分支  
git branch -r  
#  检出远端分支, 并切换分支  
git fetch origin    
git checkout -b branchName origin/branchName  
#  新建本地分支  
git branch branchName  
#  切换本地分支  
git checkout branchName  

#  新建本地分支, 并切换到本地分支    
git checkout -b branchName  
等于以下两步     
1.. git branch branchName
2.. git checkout branchName  

#  本地分支推送到远端  
1.. git push origin localBranchName  
2.. git push  #  会提醒  
➜  test git:(t_f2) git push
fatal: The current branch t_f2 has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin t_f2
再执行 git push --set-upstream origin t_f2  
#  删除本地分支  
git branch -d branchA  
#  删除远端分支  
git push origin --delete branchB  
```
如果看错分支了;  
```
假设应该在 branchA 上改代码, 结果在 branchB 上;  
git branch = branchB  
#  保存当前工作进度, 会分别对暂存区和工作区的状态进行保存;  
git stash   或者 git stash save 'message'  
git checkout branchA  
git stash pop  
#  清空  
git stash clear 
```
#### tag  
```
#  查看标签列表  
git tag  
#  新建标签  
git tag v3.1.0  
#  push标签到远程  
git push origin v3.1.0  
#  删除本地标签  
git tag -d v3.1.0  
#  删除远程标签  
git push origin :refs/tags/v3.1.0  
```
### git merge  
```
#  当前分支是 release, merge 远端的 feature 分支到本地;  
git merge origin/feature  
假设有冲突, 先解决冲突, 我习惯用 intellij android-studio 标记并解决冲突;  
➜  test git:(test) ✗ git status
On branch test
Your branch is up to date with 'origin/test'.

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   submodule.java
```
需要提交刚才的 merge 操作;  
git commit  
出现提示信息  
```
Merge remote-tracking branch 'origin/merge-review' into test

# Conflicts:
#       submodule.java
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#       /Users/alex/workSpace/temp/git_test/.git/modules/submodule/test/MERGE_HEAD
# and try again.


# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch test
# Your branch is up to date with 'origin/test'.
#
# All conflicts fixed but you are still merging.
#
# Changes to be committed:
#       modified:   submodule.java
```
按 : (就是冒号), 输入 wq, 按 enter (就是回车);  
```
➜  test git:(test) ✗ git commit
[test 2f9162b] Merge remote-tracking branch 'origin/merge-review' into test
系统会帮我们插播一条 commit message;  
```

### 修改提交内容
#### 修改上一次的提交内容  
假设用的是 gerrit 这个 code review 工具;  
```
第01次  System.out.println("test 1");  
领导认为, 要添加一句 System.out.println("test 2");

1.. 开始修改代码 mock  ...
2.. 
git add . 
git commit --amend  
按 esc, 按: (冒号), 按 wq, 按 enter (回车);  
branchName 是分支名字, 写自己的分支名字, 例如 master  
3.. 
//  这里是 code review 的人员;  
//  --receive-pack='git receive-pack --reviewer=zhangsan@host.com --reviewer=lisi@host.com' 
git push origin HEAD:refs/for/branchName  

git push --receive-pack='git receive-pack --reviewer=zhangsan@host.com --reviewer=lisi@host.com' origin HEAD:refs/for/branchName  
```
git reset   
```
git reset 节点回退到上一次, 保留本地代码  
git reset --hard  节点回退到上一次, 废弃本地代码  
git reset --hard  aaaaaaaaa    节点回退到某一个节点, 废弃本地代码  
#  回退一个版本  
git reset --hard HEAD~  
git reset  
等于  git reset HEAD  
等于  git reset --soft  HEAD~1  
#  这一步操作等于撤回 git add 对 .idea/.gitignore 的影响  
git reset HEAD .idea/.gitignore  
git reset -- filename.java    
```
#### 修改前几次的提交内容  
```
#  2是倒数第几个commit  
git rebase -i HEAD~2   
会出来一个命令, 把 pick 改为 edit   
修改你的代码, 
git rebase --continue  
git push --review    
```
### git submodule  
子模块  
```
touch .gitmodules  
git submodule add https://gitlab.com/Alex_Cin/mdreader.git module/mdreader
git submodule add https://gitlab.com/Alex_Cin/andfun.git module/andfun
git submodule init   #  初始化本地.gitmodules 文件  
git submodule update   #  主仓库和子模块关联最初的挂起点    
git submodule update --init --recursive  #  等于 init update 
git submodule update --remote  #    
git submodule update --remote --merge  #    
git submodule foreach --recursive git submodule init   
git submodule foreach --recursive git submodule update   
```
#### 注意事项  
首先执行 git status 确认当前状态, 是否有本地变更;  
如果牵涉到主仓库-子模块的概念, 要注意主仓库和子模块的关联点的问题;   
什么是正确的关联, 主仓库执行 git submodule update 之后, cd 到子模块, 子模块是在一个具体的分支, 而不是一个游离点;   
那么, 主仓库执行完 checkout, 首先要做的事情就是  
git pull  #  拉取远端最新的代码;  
git submodule update  #  主仓库和子模块关联最初的挂起点;  
git status  
```
➜  git_test git:(t_master) git status
On branch t_master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   submodule/test (new commits)

no changes added to commit (use "git add" and/or "git commit -a")
```
一般来讲, 切到一个新的分支, 不会出现主仓库代码变动的, 除非本地已经有这个分支, 而且代码没有 push;  
一般来讲, 主仓库和子模块是有对应的分支的, 例如都叫 feature;  
如果当前主仓库没有任何更新, 但是子模块有更新,   
开发阶段, 如果没有完成一个功能, 仅仅是提交代码, 不需要 jenkins 打包的时候, 是不需要关联最新的挂起点;  
如果想让主仓库更新最新的关联点, 那么就要执行 
git add .  
git commit -am '子模块更新的内容描述'';  
如果主仓库需要切换分支, 但是子模块保持原来的分支, 这样的管理策略不推荐的;  
一般来讲是主仓库 merge 一个其他的分支, 这样也会造成关联点无法对齐的问题;  
git status  
git pull  
git submodule update  
git status  
这个时候, 就会发现关联点不对了, 假设主仓库现在在 release 分支, merge 了 feature 分支,  
```
#  当前在 t_master merge t_f1  
➜  git_test git:(t_master) git merge origin/t_f1
warning: Failed to merge submodule submodule/test (commits don't follow merge-base)
Auto-merging test.java
CONFLICT (content): Merge conflict in test.java
Auto-merging submodule/test
CONFLICT (submodule): Merge conflict in submodule/test
Automatic merge failed; fix conflicts and then commit the result.

提示我子模块有冲突, 需要解决冲突;  
```
理论上我们也是需要把子模块切换到 release 分支, 并 merge 一下 feature 分支;  
如果执行了子模块的 merge 操作, 正常情况下, 会有冲突的, 这个时候, 需要解决冲突, 代码部分选择合并,  
有一个文件是系统生成的, 纯文本的, 但是没有后缀名, 点击关闭不用关注,  
```
➜  test git:(t_master) ✗ git status
On branch t_master
Your branch is ahead of 'origin/t_master' by 1 commit.
  (use "git push" to publish your local commits)

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   submodule.java
```
git diff  
```
diff --cc submodule/test
#  子模块的关联点油 79d13d3 变成 9105880  
index 79d13d3,9105880..0000000
--- a/submodule/test
+++ b/submodule/test
```
用 sourTree 打开子模块  
t_f2          分支最新的提交点是  910588085d11797037c42a87c679910a33792d15 [9105880]  
t_master  分支最新的提交点是  79d13d3400409829ea6efc1dbff40c0955265874 [79d13d3]  
我当前子模块在 t_f2 分支, 执行 git log 得到 910588085d11797037c42a87c679910a33792d15  
所以断定, 子模块的提交点变成  79d13d3400409829ea6efc1dbff40c0955265874, 也就是需要切换到 t_master 分支  
git checkout t_master  
git merge origin/t_f2   
git commit  #  不写 message, 按回车  
```
➜  test git:(t_master) ✗ git commit
[t_master 8fc3b54] Merge remote-tracking branch 'origin/t_f1' into t_master
```
回到主仓库, 执行  
git status  
```
On branch t_master
Your branch is ahead of 'origin/t_master' by 1 commit.
Merge remote-tracking branch 'origin/t_f1' into t_master
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:

	modified:   test.java

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   submodule/test
```
git add .  
git commit    
```
➜  git_test git:(t_master) ✗ git commit
[t_master fb2962f] Merge remote-tracking branch 'origin/t_f1' into t_master
```
再执行 git push  

#### 问题01    
子模块无法被追踪  
```
➜  gitfun git:(master) git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   module/andfun (modified content)

no changes added to commit (use "git add" and/or "git commit -a")
这个时候, 需要到对应的子模块下面, 执行   
git status
git add . 
git commit -am 'update'
git pull  
git push   
```
#### 怎么正确的管理  
先讲一下概念,   
主仓库 t_master, t_f1, t_f2, t_f3;  
子模块 t_master, t_f1, t_f2, t_f3;  
如果子模块 t_master 往上走 1 个 commit, 这样主仓库和子模块就无法对应上,  
    如果想对应上, 就在主仓库 执行  git status 再执行 git  submodule update 再执行 git commit -am '这个过程子模块在干什么了'  
    如果不想对应上, 也就是不想让其他人知道, 以为我的东西没做好, 那就不执行 submodule 和 commit ;  
假设主仓库的 t_f1 要 merge 到主仓库的 t_master 上, 也就是要对齐一下,   
    首先 git status 确保 t_f1 上没有新的变化了, 之后再切换到 t_master 分支上, 再执行 git merge origin/t_f1  
    不出意外的话, 会有冲突的, 首先是在主仓库上, 两个分支的改动点的冲突, 解决完成, 还剩一个是子模块关联点的冲突;  
    不用标记解决, 然后执行 git submodule update 再执行 git push;  

### 参考  
https://gitforwindows.org/  
https://git-scm.com/book/zh/v1/Git-工具-重写历史  
http://blog.csdn.net/wangyanchang21/article/details/51437934  
http://iissnan.com/progit/  
https://git-scm.com/book/zh/v1  
https://git-scm.com/book/zh/v2  
http://www.runoob.com/git/git-basic-operations.html  
https://www.yiibai.com/git/git_clone_operation.html  
https://github.com/geeeeeeeeek/git-recipes/wiki  
https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/index.html  

