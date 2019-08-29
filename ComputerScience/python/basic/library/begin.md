定义变量  
```
不需要 var val def  直接定义即可;  
counter = 100 # 赋值整型变量  
miles = 1000.0 # 浮点型  
name = "John" # 字符串  
```
当前目录  
```
import os  
print os.getcwd()  
1.. 得到当前工作目录, 即当前 Python 脚本工作的目录路径: os.getcwd()
2.. 返回指定目录下的所有文件和目录名: os.listdir()
```
输出中文  
```
当前脚本, 头上加上  
# -- coding: UTF-8 --
```
拷贝文件  
```
#  hooks_file0  是原始文件, 绝对路径;  
#  child  是目标位置, 文件夹, 绝对路径;   
shutil.copy(hooks_file0, child)  
```
递归遍历文件夹  
```
import os
import shutil

filePath = "/Users/alex/workSpace/git/ebook"

def gci(path):
    files = os.listdir(path)
    for fi in files:
        child = os.path.join(path, fi)
        if os.path.isdir(child):
            if os.path.basename(child) == 'hooks':
                print ('目录=' + child)
                shutil.copy(hooks_file0, child)
                shutil.copy(hooks_file1, child)
                shutil.copy(hooks_file2, child)
            gci(child)
        # else:
        #     print ('文件=' + child)


gci(filePath)

```
### 参考  
文件  
https://www.cnblogs.com/my1318791335/p/8681136.html  

