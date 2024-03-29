```
#  表示命令行注释  
```
### 目录和文件  
```
#  显示文件或目录
ls  
#  当前工作目录 路径
pwd
#  切换目录  
cd  
#  切换到根目录
cd  /  
#  去主目录, home目录 
cd  ~  
#  返回上级目录	
cd  ..  
#  返回多级  
cd  ../..  
cd  ../../..  

❀ 打开文件夹   
cd /Users/alex/WorkSpace/Linux/leetcode/leet/src/main/java/com/alex/leet  
#  ubuntu  
nautilus .   
#  mac  
open .
#  打开home 目录
nautilus  home  
#  打开DownLoads 目录
nautilus  Downloads  
#  获得文件写权限  
chmod o+rwx hello.java  
#  在当前目录下, 创建ProgramFiles文件夹
mkdir ProgramFiles 
#  希望创建 test/book/look 目录;  创建多及目录, 需要加 -p 参数;  
mkdir -p test/book/look  
#  在当前目录下, 创建Sample.txt 文件  
touch Sample.txt  
#  往Sample.txt 文件上, 追加并换行写入 Hello
echo Hello >> Sample.txt
#  往Sample.txt 文件上, 清空并写入 Hello  
echo Hello > Sample.txt    
#  查看文件内容  
cat Sample.txt 
#  文件复制 重命名
cp hello.text  /usr/share/hello.java
#  删除文件
rm -rf filePath  
rm -rf /alex/work/sample  
-f  force 
-i  interactive  #  进行交互式 删除
-r -R  recursive  #  递归删除所有子目录, 子文件
-v  verbose  #  详细地进行的步骤

❀ find  
find /Users/alex/WorkSpace/Studio/AndFun -cmin -9  
参考  
http://man.linuxde.net/find-2 

❀ 找出 最大的文件  
du -ah . | sort -nrk 1 | head 
参考  
https://blog.csdn.net/tzh_linux/article/details/51192472    


❀ 在当前目录下, 查找 segment01 文件夹  
find ./ -name segment01
查询结果  
.//page02/segment01
.//page01/segment01


❀ 获取 域名对应的ip  
ping www.baidu.com  

❀ 今天是 一年中的第几天  
cal -j  
```
 
### ps  
//  按照名字, 杀死进程  
pkill hello  
ps -f|grep java
```
UID       PID       PPID      C     STIME    TTY       TIME         CMD  
501  9208   392   0  5:00下午 ttys000    0:00.00 grep java  

UID      ：程序被该 UID 所拥有  
PID      ：就是这个程序的 ID   
PPID    ：则是其上级父程序的ID  
C          ：CPU使用的资源百分比  
STIME ：系统启动时间  
TTY     ：登入者的终端机位置  
TIME   ：使用掉的CPU时间。  
CMD   ：所下达的是什么指令  

获取 pid  
pgrep java  
1656  # uid  
10571  # pid  

杀死进程 kill -9 pid  
kill -9 10571  
```  

```

```


### 压缩与解压缩  
```
// 解压缩  
tar -zxvf   sample.rar  

❀ tar   
tar -cvf leet.tar leetcode  

tar -cvf leet.zip /Users/alex/leetcode  
这样创建的,  解压之后, 会有 目录 [Users/alex/leetcode]  

cd /Users/alex 
tar -jcvf leet.tar.bz2 leetcode
tar -jcvf AndFun.tar.bz2 AndFun
tar -zcvf AndFun.tar.gz AndFun


-c, --create  create a new archive 创建一个归档文件
- C: 要在指定目录下进行解压缩, 
-v, --verbose verbosely list files processed 显示创建归档文件的进程
-f, --file=ARCHIVE use archive file or device ARCHIVE  后面要立刻接被处理的档案名； 如果有多个选项 , f 必须在最后； -zcf   demo.tar.gz  
-x：解压  
-t：查看内容  
-u：更新原压缩包中的文件
-r：向压缩归档文件末尾追加文件  
-j: 通过bzip2进行压缩与解压缩,  此时文档名最好（必须）为 *.tar.bz2  
-J: 通过xz进行压缩与解压缩,  此时文档名最好（必须）为 *.tar.xz
-z: 通过gzp进行压缩与解压缩, 此时文档名最好（必须）为 *.tar.gz
```

❀ zip 语法  
```
//  tmp 文件目录下, 压缩 所有的 .txt 文件
zip test.zip *.txt tmp  

cd /Users/alex 
zip leet.zip *  leetcode  
```
❀ gzip 语法  
```
gzip [-cdtv#] 文件名  
zcat *.gz  
参数：  
-c:将压缩的数据输出到屏幕上, 可通过数据流重定向来处理；  
-d:解压的参数；  
-t:可以用来验证一个压缩文件有没有错误；  
-v:可以显示出"原文件/压缩文件"的压缩比等信息；  
-#：压缩等级, 1最快, 但是压缩比最差, -9最慢, 但是压缩比最高, 默认为6；  
```
❀  常见压缩文件格式  
```
*.gz   gzip程序压缩的文件  
*.bz2  bzip程序压缩的文件  
*.tar  tar程序打包的数据, 没有经过压缩  
*.tar.gz  tar程序打包后, 并经过gzip程序压缩  
*.tar.bz2 tar程序打包后, 并经过bzip程序压缩  
*.zip     zip程序压缩的文件  
*.rar     rar.程序压缩的文
```
❀ 参考  
https://blog.csdn.net/zmzwll1314/article/details/81746487  
https://www.cnblogs.com/jxldjsn/p/8029431.html  
http://eksliang.iteye.com/blog/2109693  
