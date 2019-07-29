◆ cloning  
```
fatal: The remote end hung up unexpectedly 
fatal: early EOF 
fatal: index-pack failed
```
[clone_error](library/clone_error_eof.md)  
[update latest](library/update_latest.md)  
[环境变量](library/run_path.md)   
git log  
如何撤销已经push上去的commit    
git提供了好几种方法做到这点，其中最简单的方法就是用git revert命令，后面跟上你要撤销的commit id即可。  

git commit  
```
//  提交当前本地变动, 添加变动描述;  
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
//  把本地的, 当前分支, 推送到远端的, 当前分支上  
git push origin  
```

git fetch  
```
//  抓取远端的 当前分支
git fetch origin  

//  抓取远端的 feature/v5.0 分支
git fetch origin feature/v5.0  
```  
fetch 和 pull 区别  
```
git pull = git fetch + git merge  
git fetch 的时候只是将remote的origin进行update  但是并没有在local的branch进行merge  
```
git merge  
```
//  远端的 feature/v5.0 分支, merge 到本地, 当前分支;  
git merge origin/feature/v5.0 
```
git checkout  
```
//  检出对应的分支名称  
git checkout -b feature/v5.0  
```
git add  
```

git add -A  添加所有变化  
git add -u  添加被修改(modified)和被删除(deleted)文件，不包括新文件(new)  
git add .   添加新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件  
```
git status  
```
位于分支 master
您的分支领先 'origin/master' 共 3 个提交。
  （使用 "git push" 来发布您的本地提交）
无文件要提交，干净的工作区
```
git submodule  
```
touch .gitmodules
git submodule add https://gitlab.com/Alex_Cin/mdreader.git module/mdreader
git submodule add https://gitlab.com/Alex_Cin/andfun.git module/andfun
```
### 分支管理  
查看本地分支  
git branch  
前面带*，高亮的是当前分支  
```
test  
dev  
release  
* master  
```
查看远端分支  
```
git branch -a  
```

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
### git pull 冲突  
```

Merge branch 'master' of https://gitlab.com/Alex_Cin/mdreader
# 请输入一个提交信息以解释此合并的必要性，尤其是将一个更新后的上游分支
# 合并到主题分支。
#
# 以 '#' 开始的行将被忽略，而空的提交说明将终止提交。




                                                                          [ 已读取 5 行 ]
^G 求助         ^O 写入         ^W 搜索         ^K 剪切文字     ^J 对齐         ^C 游标位置     M-U 撤销        M-A 标记文字    M-] 至括号      M-▲ 上一个
^X 离开         ^R 读档         ^\ 替换         ^U 还原剪切     ^T 拼写检查     ^_ 跳行         M-E 重做        M-6 复制文字    M-W 搜索下一个  M-▼ 下一个

```
直接按下 control + x  


### 参考  
git 下载  
https://gitforwindows.org/
  
http://blog.csdn.net/wangyanchang21/article/details/51437934  
http://iissnan.com/progit/  
https://git-scm.com/book/zh/v1  
https://git-scm.com/book/zh/v2  
http://www.runoob.com/git/git-basic-operations.html  
https://www.yiibai.com/git/git_clone_operation.html  
https://github.com/geeeeeeeeek/git-recipes/wiki  
https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/index.html  

