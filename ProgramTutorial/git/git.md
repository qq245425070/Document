cloning  
```
fatal: The remote end hung up unexpectedly 
fatal: early EOF 
fatal: index-pack failed
```
[clone_error](library/clone_error_eof.md)  
[update latest](library/update_latest.md)  
[环境变量](library/run_path.md)   
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
git merge  
```
#  远端的 feature/v5.0 分支, merge 到本地, 当前分支;  
git merge origin/feature/v5.0 
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
❀ 查看本地分支  
git branch  
前面带 *, 高亮的是当前分支  
```
test  
dev  
release  
* master  
```
新建分支-切换分支  
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
git push origin localBranchName  
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

git reset  
等于  git reset HEAD  
等于  git reset --soft  HEAD~1  

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
git submodule update   #  同步远端 submodule 源码  
git submodule foreach --recursive git submodule init   
git submodule foreach --recursive git submodule update   
❀ 在主目录下, 更新子模块有两种形式  
1..  git submodule foreach git pull  
2..  cd subProjectPath  
      git pull  
❀ 修改子模块代码  
如果需要修改 submoduleA 的代码, 正确的操作是, 切换到子模块目录下, 执行 git pull;  
修改完成之后, 会执行 git add -A, 再执行 git commit -am 'message', 再执行 git push;  
回到主仓库, 执行 git pull, git submodule update, git status;  
这个时候, 会带来一个问题, 那就是, submoduleA 并不是在 master 分支, 而是某一个游离点-挂起点;  
如果再要回到 submoduleA 修改代码, 一定要执行 git checkout branchName, git pull, git status;  
如果再要回到 submoduleA 修改代码, 如果忘记切换分支, 但是已经修改了代码  
        执行 git checkout branchName, 将HEAD从游离状态切换到 branchName 分支, 这时候, git 会报 Warning, 有一个提交没有在 branch 上,   
        记住这个提交的change-id(假如change-id为 aaaa)  
        执行 git cherry-pick aaaa 来将刚刚的提交作用在 branchName 分支上;  
        再执行 git status;  

```
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

