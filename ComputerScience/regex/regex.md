[必须包含一个数字](library/regex_001.md)  
[只能是纯数字(大于等于 1 个字符)](library/regex_002.md)  
[必须包含一个英文 和 一个数字(一共 2 个字符以上, 只能包含英文和数字)](library/regex_003.md)  
[必须包含一个英文 和 一个数字(一共 2 个字符以上, 任意字符)](library/regex_004.md)  
[是正整数(单个字符)](library/regex_005.md)  
[只能是纯数字(大于等于 1 个字符)](library/regex_006.md)  
[非 0 开头的纯数字(大于等于 1 个字符)](library/regex_007.md)  
[是手机号码(第 0 位是 1, 第 1 位是: 3456789  剩下 9 位是 0-9), 长度必须是 11 位](library/regex_008.md)  

正则规则
^ 匹配起始位置   
$ 匹配结束位置  
\d 匹配数字 0-9  
\D 匹配任意非空格  
\w 匹配字母(a-zA-Z) 数字(0-9) 下划线_  
\W 匹配不是 （字母(a-zA-Z) 数字(0-9) 下划线_）  
\s 匹配空格   
\S 匹配非空格  
. 匹配除了\n以外的任意字符  
[abcde] 匹配 [] 内的枚举字符  
[^abcde] 匹配 非[] 内的枚举的  其他字符  
* 重复0次，或者更多次  
+ 重复1次，或者更多次  
? 重复0次，或者1次  
{n} 重复n次  
{n,} 重复n次，或者更多次  
{n,m} 重复n次到m次  
*? 重复任意次，但尽可能减少重复  
+? 重复1次，或者更多次，但尽可能减少重复  
?? 重复0次，或者1次，但尽可能减少重复  
{n,m}? 重复n次到m次，但尽可能减少重复  
x(?=y) 正向前查匹配，例如：Hello(?=World)  只有HelloWorld才能匹配成功  
x(?!y) 反向前查配，Hello(?!World)   Hello后面不能跟World  

### 参考  
https://github.com/ziishaned/learn-regex/blob/master/translations/README-cn.md  
