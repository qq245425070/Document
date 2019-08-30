定义变量  
```
不需要 var val def  直接定义即可;  
counter = 100 # 赋值整型变量  
miles = 1000.0 # 浮点型  
name = "John" # 字符串  
```

输出中文  
```
当前脚本, 头上加上  
# -- coding: UTF-8 --
```

接受命令行参数  
```
import sys

print '参数列表 hooks.dir=' + sys.argv[1] + ',desDir=' + sys.argv[2]
```

控制台输入输出  
```
print('hello, world')  
name = input()  
int (input('输入年龄'))  
```
字符串模板  
```
print('''字符串模板，
可以换行的
''')

print('Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125))
```
占位符  
%d	整数    
%f	浮点数    
%s	字符串   
%x	十六进制整数  
```
print('%d--%s' % (10, 'alex'))  
```

当前目录  
```
import os  
print os.getcwd()  
1.. 得到当前工作目录, 即当前 Python 脚本工作的目录路径: os.getcwd()
2.. 返回指定目录下的所有文件和目录名: os.listdir()
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
### 文本字符串  

### 正则表达式  

♬ r  
是为了告诉编译器这个string是个raw string，不要转意backslash '\' 。 例如，\n 在raw string中，是两个字符，\和n， 而不会转意为换行符。  
由于正则表达式和 \ 会有冲突，因此，当一个字符串使用了正则表达式后，最好在前面加上'r'。  

字符串模板  
${string}  
print('hello ' + str(i) + '.mp4')  
print(f'hello {i}.mp4')
print(F'hello {i}.mp4')

参考  
http://www.cnblogs.com/c-x-a/p/9333826.html  


♬ findall  
 ```
 (pattern: Pattern[AnyStr], string: AnyStr, flags: Union[int, RegexFlag]) -> list  
 ```
 
### 参考  
文件  
https://www.cnblogs.com/my1318791335/p/8681136.html  

