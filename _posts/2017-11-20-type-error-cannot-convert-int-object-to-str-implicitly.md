---
layout: post
title: python中TypeError:Can't convert int object to str implicitly报错的解决办法
categories: python translate 
description: python中TypeError:Can't convert int object to str implicitly报错的解决办法
keywords: python
---

本文主要介绍了python中“TypeError: Can't convert 'int' object to str implicitly"报错的解决办法。

原文出处：

[TypeError: Can't convert 'int' object to str implicitly](https://stackoverflow.com/questions/13654168/typeerror-cant-convert-int-object-to-str-implicitly)

## 问题

我在尝试着写一个文字游戏，遇到了一个函数错误，这个函数实现的功能是：在你输入完字符后，就会消耗你的技能分。刚开始时报错信息显示我在试图用一个整数减去
一个字符，对应代码为“balance - strength”，这个错误很明显，因此我将其改为“strength = int(strength)”修复了... 但是现在我遇到了一个以前从未见过的
错误(o(╯□╰)o我是一个新手)，我不知道它试图在告诉我什么以及如何修复它。

以下为该函数对应的代码：

```python
def attributeSelection():
    balance = 25
    print("Your SP balance is currently 25.")
    strength = input("How much SP do you want to put into strength?")
    strength = int(strength)
    balanceAfterStrength = balance - strength
    if balanceAfterStrength == 0:
        print("Your SP balance is now 0.")
        attributeConfirmation()
    elif strength < 0:
        print("That is an invalid input. Restarting attribute selection. Keep an eye on your balance this time!")
        attributeSelection()
    elif strength > balance:
        print("That is an invalid input. Restarting attribute selection. Keep an eye on your balance this time!")
        attributeSelection()
    elif balanceAfterStrength > 0 and balanceAfterStrength < 26:
        print("Ok. You're balance is now at " + balanceAfterStrength + " skill points.")
    else:
        print("That is an invalid input. Restarting attribute selection.")
        attributeSelection()
```

以下为运行此部分代码后的报错信息：
```python
Your SP balance is currently 25.
How much SP do you want to put into strength?5
Traceback (most recent call last):
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 205, in <module>
    gender()
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 22, in gender
    customizationMan()
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 54, in customizationMan
    characterConfirmation()
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 93, in characterConfirmation
    characterConfirmation()
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 85, in characterConfirmation
    attributeSelection()
  File "C:\Python32\APOCALYPSE GAME LIBRARY\apocalypseGame.py", line 143, in attributeSelection
    print("Ok. You're balance is now at " + balanceAfterStrength + " skill points.")
TypeError: Can't convert 'int' object to str implicitly
```

有人知道如何解决这个问题吗？先行感谢。

(译者注：提问者报错信息中涉及较多，部分为其项目代码文件。为缩短报错信息，我将提问者所提到的函数部分代码粘贴到本机后，运行完对应的报错信息如下)

```python
Your SP balance is currently 25.
How much SP do you want to put into strength?5
Traceback (most recent call last):
  File "test.py", line 26, in <module>
    attributeSelection()
  File "test.py", line 20, in attributeSelection
    print("Ok. You're balance is now at " + balanceAfterStrength + " skill points.")
TypeError: cannot concatenate 'str' and 'int' objects
```

## 答案

你不能将整型(int)与字符串(string)连在一起。你需要使用'str'函数将整型(int)转换为字符型(string)，或者使用'formatting'格式化输出。

将

`print("Ok. Your balance is now at " + balanceAfterStrength + " skill points.")`

改为：

({}  .format方式)

`print("Ok. Your balance is now at {} skill points.".format(balanceAfterStrength))`

或改为：

(使用str函数转换类型)

`print("Ok. Your balance is now at " + str(balanceAfterStrength) + " skill points.")`

或按照下面的一条评论所提及的那样做，使用','将不同的字符串传递给print函数，而不是使用'+'连接。

(涉及的评论为：你不能使用','连接字符串；你可以用','将参数分开传递给print函数，这些参数会以空格分割，一个接一个的打印出来)

`print("Ok. Your balance is now at ", balanceAfterStrength, " skill points.")`

## 总结

当同时打印字符及整型变量时，有以下几种方式来避免“TypeError”报错。

假设变量temp = 3，要输出的内容为the number you input is 3.

1.使用str强制将整型转换为字符型

`print 'the nume you input is ' + str(temp)`

2.使用格式化输出(python2中适用，“format % values”形式)，详细使用方法可参考官方文档：[string-formatting](https://docs.python.org/2/library/stdtypes.html#string-formatting)

`print 'the nume you input is %s' % temp`

3.使用" str.format()"(python2.6以上)格式化输出，详细使用方法可参考官方文档：[string-formatting](https://docs.python.org/3/library/string.html#string-formatting)

`print 'the nume you input is {}' .format(temp)`

4.使用逗号将变量和字符串分隔

`print 'the nume you input is' , temp`

## % 及 .format() 两种格式化输出对比

更多实例对比请参考：

https://pyformat.info/
https://github.com/ulope/pyformat.info

```python
基本输出
Old    '%s %s' % ('one', 'two')
New    '{} {}'.format('one', 'two')
Output    one two

Old    '%d %d' % (1, 2)
New    '{} {}'.format(1, 2)
Output    1 2

#右对齐
Old    '%10s' % ('test',)
New    '{:>10}'.format('test')
Output        test    #test左边有六个空格 

#左对齐
Old    '%-10s' % ('test',)
New    '{:<10}'.format('test')
Output    test       #test右边有六个空格   

#字典 
person = {'first': 'Jean-Luc', 'last': 'Picard'}
New    '{p[first]} {p[last]}'.format(p=person)
Output    Jean-Luc Picard

#列表
data = [4, 8, 15, 16, 23, 42]
New    '{d[4]} {d[5]}'.format(d=data)
Output    23 42

#Accessing arguments by position:
>>> '{0}, {1}, {2}'.format('a', 'b', 'c')
'a, b, c'
>>> '{}, {}, {}'.format('a', 'b', 'c')  # 3.1+ only
'a, b, c'
>>> '{2}, {1}, {0}'.format('a', 'b', 'c')
'c, b, a'
>>> '{2}, {1}, {0}'.format(*'abc')      # unpacking argument sequence
'c, b, a'
>>> '{0}{1}{0}'.format('abra', 'cad')   # arguments' indices can be repeated
'abracadabra'  

#Accessing arguments by name:
>>> 'Coordinates: {latitude}, {longitude}'.format(latitude='37.24N', longitude='-115.81W')
'Coordinates: 37.24N, -115.81W'
>>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
>>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
'Coordinates: 37.24N, -115.81W'
```


















