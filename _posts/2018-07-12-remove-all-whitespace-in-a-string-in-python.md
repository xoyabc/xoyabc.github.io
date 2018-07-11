---
layout: post
title: python中删除空白字符串
categories: python
description: python中删除空白字符串
keywords: python, pexpect
---


python中删除字符串中的空白字符。

主要参考StackOverflow答案总结。

空白字符一般指以下几种字符：space, tab, linefeed, return, formfeed, and vertical tab。中英文对照表如下：

| 英文 | space | tab | linefeed | return | formfeed | vertical tab |
| :---: |:---: |:---: |:---: |:---: |:---: |:---: |
| 中文| 空格 | 水平制表符 | 换行 | 回车 | 换页 | 垂直制表符 |
| 转义字符 | ' ' | '\t' | '\n' | '\r' | '\f' | '\v' |
| ASCII name | SP | HT | NL | CR | FF | VT |
| English name | space | horizontal tab | newline | carriage return | form feed | vertical tab |
| REF | [string.html](https://docs.python.org/2/library/string.html) | [whitespace.html](https://infohost.nmt.edu/tcc/help/pubs/python/web/whitespace.html) |


## 去除空格

- 去除左右两边的空格

使用`str.strip():`

``` python
In [1]: sentence = ' hello  apple'

In [2]: sentence.strip()
Out[2]: 'hello  apple'
```

- 去除所有空格

使用`str.replace()`

``` python
In [3]: sentence = ' hello  apple'

In [4]: sentence.replace(" ", "")
Out[4]: 'helloapple'
```

## 去除空白字符

 - 去除所有的空白字符

使用`str.split()`及`join`

``` python
In [10]: sentence = '  hello  apple   \n  \r   \t '

In [11]: "".join(sentence.split())
Out[11]: 'helloapple'
```

使用正则

``` python
In [13]: import re

In [14]: sentence = '  hello  apple   \n  \r   \t '

In [15]: pattern = re.compile(r'\s+')

In [16]: sentence = re.sub(pattern, '', sentence)

In [17]: print sentence
helloapple
```

使用`str.translate()`

 ``` python
In [78]: sentence = '  hello  apple     \n\r \n\r'

In [79]: print sentence
  hello  apple     
 


In [80]: print sentence.translate(None, ' \n\t\r')
helloapple
 ```

 - 只去除左边的空白字符
 
 使用`str.lstrip()`
 
 ``` python
In [29]: sentence = '  hello  apple  '

In [30]: sentence.lstrip()
Out[30]: 'hello  apple  '
 ```
 
 使用正则
 ``` python
In [35]: import re

In [36]: sentence = '  hello  apple  '

In [37]: sentence = re.sub(r"^\s+", "", sentence, flags=re.UNICODE)

In [39]: sentence
Out[39]: 'hello  apple  '
 ```
 
 - 只去除右边的空白字符
 
 使用`str.rstrip()`

``` python
In [40]: sentence = '  hello  apple  '

In [41]: sentence.rstrip()
Out[41]: '  hello  apple'
```
 
 使用正则
 
 ``` python
 In [43]: import re

In [44]: sentence = '  hello  apple  '

In [45]: sentence = re.sub(r"\s+$", "", sentence, flags=re.UNICODE)

In [46]: sentence
Out[46]: '  hello  apple'
 ```
 
 - 仅去除重复的空白字符
 
 使用正则
 
 ``` python
 In [66]: import re

In [67]: sentence = '  hello  apple     '

In [68]: sentence = " ".join(re.split("\s+", sentence, flags=re.UNICODE))

In [69]: sentence
Out[69]: ' hello apple '
 ```
 
综上，`str.strip()`会移除字符串中开头与结束(左右两侧)的空白字符，中间部分的空白字符是不会移除的。
strip方法中可自定义要移除的字符，如下面这个示例，移除的为字符串中两侧的逗号
``` python
In [85]: ",1,2,3,".strip(",")
Out[85]: '1,2,3'

```

```
strip does a rstrip and lstrip (removes leading and trailing spaces, tabs, returns and form 
feeds, but it does not remove them in the middle of the string).
```

## 参考

[remove-all-whitespace-in-a-string-in-python](https://stackoverflow.com/questions/8270092/remove-all-whitespace-in-a-string-in-python)


