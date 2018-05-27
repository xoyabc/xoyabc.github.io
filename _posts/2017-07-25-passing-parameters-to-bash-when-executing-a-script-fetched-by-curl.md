---
layout: post
title: curl方式执行脚本时传参问题 
categories: translate
description: curl方式执行脚本时传参问题 
keywords: python, 
---

翻译+润色
> [passing-parameters-to-bash-when-executing-a-script-fetched-by-curl](https://stackoverflow.com/questions/4642915/passing-parameters-to-bash-when-executing-a-script-fetched-by-curl/4642975)


应用场景：使用curl执行远程Web设备上的脚本且需要携带参数时。

通常执行发布机上的脚本时习惯使用以下方式：

```shell
curl http://example.com/script.sh | bash
```

### 若脚本涉及到传参时，则可使用以下任一方式:

```shell
1. curl http://example.com/script.sh | bash -s arg1 arg2
2. curl http://example.com/script.sh | bash /dev/stdin arg1 arg2
3. bash <( curl http://example.com/script.sh ) arg1 arg2
```

### 若参数中带有"-"，则可使用长选项"--"解决
```shell
curl http://example.com/script.sh | bash -s -- arg1 arg2
```
如参数为`-p blah -d blah`,则可使用以下命令执行
```shell
curl http://example.com/script.sh | bash -s -- -p blah -d blah
```
该方法不止适用于curl的输入，其他方式的输入也满足。例如下面这个是用`echo`输入的BASH脚本。

```Bash
echo 'i=1; for a in $@; do echo "$i = $a"; i=$((i+1)); done' | \
bash -s -- -a1 -a2 -a3 --long some_text
```

输出结果：
```Bash
1 = -a1
2 = -a2
3 = -a3
4 = --long
5 = some_text
```










