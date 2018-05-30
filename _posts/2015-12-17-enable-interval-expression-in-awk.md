---
layout: post
title: 启用awk中的正则间隔表达式
categories: awk
description: 启用awk中的间隔表达式
keywords: linux, awk
---

本文为`翻译+实例`结合。

awk中若使用间隔表达式`{m,n}`，需要配合--posix 或--re-interval参数才会生效。

## 原文

```shell
       r{n}
       r{n,}
       r{n,m}    
One or two numbers inside braces denote an interval expression.If there is one number in 
thebraces, the preceding regular expression r is repeated n times. If there are two numbers
separated by a comma, r is repeated n to m times. If there is one number followed by a 
comma, then r is repeated at least n times.Interval expressions are only available if either 
--posix or --re-interval is specified on the com-mand line.
```

## 翻译

```
大括号与在其中的一个或两个数字组合表示一个间隔表达式。若大括号中只含有一个数字，则重复执行前面的正则表达"r"n次。若其中含有以逗号分开的两个数字n与m，

则重复执行表达式"r"n到m次(n<m)。若数字n后跟一个逗号，则至少^重复执行表达式"r"n次。只有在命令行中指定了--posix 或--re-interval参数后，间隔表达式才

会生效。
```

## 实例

用awk提取出test.txt文本中末尾是两个8的手机号。

test.txt文本内容为：
```
18295089368
1895089368
185089368
182089368
17888888888
17884432254
17888132435
17812266688
18295089368
18235089368
13335508387
15575089368
```
- 不加上述两个参数前
```shell
root@network test$awk '/1[3578]{1}[0-9]{7}8{2}/' test.txt
root@network test$awk '/1[3578][0-9][0-9][0-9][0-9][0-9][0-9][0-9]88/' test.txt          
17888888888
17812266688
```
- 加上二者任意一个参数
```
root@network test$awk --re-interval '/1[3578]{1}[0-9]{7}8{2}/' test.txt
17888888888
17812266688
root@network test$awk --posix '/1[3578]{1}[0-9]{7}8{2}/' test.txt
17888888888
17812266688
```

可以看到,若不加--re-interval或--posix参数时，匹配后无结果输出，加上之后即正常输出。

## Ref
[awk --posix](http://blog.chinaunix.net/uid-21505614-id-289437.html)
