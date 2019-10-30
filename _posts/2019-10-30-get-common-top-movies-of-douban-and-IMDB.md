---
layout: post
title: 同时入选豆瓣 top250 及 IMDB top250 的影片
categories: linux
description: 获取入选豆瓣 top250 及 IMDB top250 的影片
keywords: linux, python, csv
---

## 说明

1，影片数据截止至 2019.10.28

2，豆瓣 top 250 影片信息使用之前 shell 的脚本获取

3，IMDB top 250 影片及排名信息数据来自 `bimzcy` 的rank4douban 项目

4，脚本运行环境为 Python 3.6.8

5，使用 `csv` 模块读取及写入 csv 文件


## csv 文件写入

使用 DictWriter 类，将字典映射到 csv 行中。

```python
#!/usr/bin/python3

import csv

f = open('names.csv', 'w')

with f:
    
    # 表头
    fnames = ['first_name', 'last_name']
    # 创建 writer 对象，将表头值传递给 fieldnames 参数
    writer = csv.DictWriter(f, fieldnames=fnames)    
    # 使用 writeheader 方法将表头数据写入到 csv 文件
    writer.writeheader()
    # 使用 writerow 方法将一行数据写入到 csv 文件中
    writer.writerow({'first_name' : 'John', 'last_name': 'Smith'})
```

输出：

```
$ cat names.csv 
first_name,last_name
John,Smith
Robert,Brown
Julia,Griffin
```

## 脚本内容

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import csv
import re
import os
'''
1，csv文件存放在子目录 data 中
2，00_all_movies.csv 为影片详细信息数据文件
3，01_IMDbtop250.csv 为 IMDB top250 影片信息文件
4，07_Douban_top250_movies.csv 为豆瓣 top250 片信息文件
5，common_movies.csv 存放共同影片
'''

# 获取豆瓣条目ID
def get_dbid_from_csv(filename, rowname):
    file = os.path.join('.', 'data', filename)
    pattern = re.compile(r'(?P<db_id>\d+)')
    with open(file, 'r', newline='', encoding='utf-8-sig') as f:
        reader = csv.DictReader(f)
        k = rowname
        LIST_DB = [ pattern.search(row[k]).group('db_id') for row in reader ]
        return (LIST_DB)


# 使用递归获取共同影片
def get_common_data(l1, *info_list): 
    for l in info_list: 
        common_list = [] 
        for i in l:
            if len(list(filter(lambda x: x == i, l1))) > 0:
                common_list.append(i) 
        l1 = common_list 
    return common_list


# 获取各个榜单影片排名
def get_rank_list(filename, douban_=False, CC_=False):
    rank_file = os.path.join('.', 'data', filename)
    f_db = open(rank_file, 'r', encoding='utf-8-sig')
    reader = csv.DictReader(f_db) 
    if douban_:
        pattern = re.compile(r'(?P<db_id>\d+)')
        rank_dict = { pattern.search(row['链接']).group('db_id'): (i+1) for i,row in enumerate(list(reader)) }
    elif CC_:
        rank_dict = { row['dbid']: row['spine'] for row in reader }
    else:
        rank_dict = { row['dbid']: row['rank'] for row in reader }
    return rank_dict


# 写入到csv文件
# 使用 encoding='utf-8-sig' 避免 windows 下中文乱码
def write_to_csv(filename, head_line, *info_list):
    file = os.path.join('.', 'data', filename)
    with open(file, 'w', encoding='utf-8-sig') as f:
        fnames = head_line
        writer = csv.DictWriter(f, fieldnames=fnames)
        writer.writeheader()
        writer.writerows(info_list)

# 查找共同影片详细信息
def search_movie(db_file, new_file, *info_list):
    # get rank info
    db_rank = get_rank_list('07_Douban_top250_movies.csv', douban_=True)
    imdb_rank = get_rank_list('01_IMDbtop250.csv')

    # get detailed movie info
    file = os.path.join('.', 'data', db_file)
    with open(file, 'r', encoding='utf-8-sig') as f:
            l = [] 
            reader = csv.DictReader(f) 
            for row in reader: 
                for i in info_list:  
                    if row['subject_id'] == i:
                        row['IMDB_rank'] = imdb_rank[i]
                        row['db_rank'] = db_rank[i]
                        l.append(row) 
            head_line = ['subject_id', 'type', '中文名', '年份', '片长', '评分', '评价人数', '国家', '语言', '类型', '主演', '导演', 'IMDB编号', 'IMDB_rank', 'db_rank']
            write_to_csv(new_file, head_line, *l)

if __name__ == '__main__':
    LIST_DB= get_dbid_from_csv('07_Douban_top250_movies.csv', '链接')
    LIST_IMDB= get_dbid_from_csv('01_IMDbtop250.csv', 'dbid')
    common_movies = get_common_data(LIST_IMDB, LIST_DB)
    search_movie('00_all_movies.csv', 'common_movies.csv', *common_movies)
```

## 结果

| 中文名             | 年份   | 导演                |
|-----------------|------|-------------------|
| 肖申克的救赎          | 1994 | 弗兰克·德拉邦特          |
| 教父              | 1972 | 弗朗西斯·福特·科波拉       |
| 教父2             | 1974 | 弗朗西斯·福特·科波拉       |
| 蝙蝠侠：黑暗骑士        | 2008 | 克里斯托弗·诺兰          |
| 十二怒汉            | 1957 | 西德尼·吕美特           |
| 辛德勒的名单          | 1993 | 史蒂文·斯皮尔伯格         |
| 指环王3：王者无敌       | 2003 | 彼得·杰克逊            |
| 低俗小说            | 1994 | 昆汀·塔伦蒂诺           |
| 黄金三镖客           | 1966 | 赛尔乔·莱昂内           |
| 搏击俱乐部           | 1999 | 大卫·芬奇             |
| 指环王1：魔戒再现       | 2001 | 彼得·杰克逊            |
| 阿甘正传            | 1994 | 罗伯特·泽米吉斯          |
| 盗梦空间            | 2010 | 克里斯托弗·诺兰          |
| 指环王2：双塔奇兵       | 2002 | 彼得·杰克逊            |
| 黑客帝国            | 1999 | 莉莉·沃卓斯基           |
| 飞越疯人院           | 1975 | 米洛斯·福尔曼           |
| 七武士             | 1954 | 黑泽明               |
| 七宗罪             | 1995 | 大卫·芬奇             |
| 上帝之城            | 2002 | 费尔南多·梅里尔斯         |
| 美丽人生            | 1997 | 罗伯托·贝尼尼           |
| 沉默的羔羊           | 1991 | 乔纳森·戴米            |
| 拯救大兵瑞恩          | 1998 | 史蒂文·斯皮尔伯格         |
| 千与千寻            | 2001 | 宫崎骏               |
| 绿里奇迹            | 1999 | 弗兰克·德拉邦特          |
| 这个杀手不太冷         | 1994 | 吕克·贝松             |
| 星际穿越            | 2014 | 克里斯托弗·诺兰          |
| 狮子王             | 1994 | 罗杰·阿勒斯            |
| 钢琴家             | 2002 | 罗曼·波兰斯基           |
| 摩登时代            | 1936 | 查理·卓别林            |
| 终结者2：审判日        | 1991 | 詹姆斯·卡梅隆           |
| 触不可及            | 2011 | 奥利维埃·纳卡什          |
| 惊魂记             | 1960 | 阿尔弗雷德·希区柯克        |
| 城市之光            | 1931 | 查理·卓别林            |
| 爆裂鼓手            | 2014 | 达米恩·查泽雷           |
| 致命魔术            | 2006 | 克里斯托弗·诺兰          |
| 萤火虫之墓           | 1988 | 高畑勋               |
| 天堂电影院           | 1988 | 朱塞佩·托纳多雷          |
| 记忆碎片            | 2000 | 克里斯托弗·诺兰          |
| 窃听风暴            | 2006 | 弗洛里安·亨克尔·冯·多纳斯马尔克 |
| 被解救的姜戈          | 2012 | 昆汀·塔伦蒂诺           |
| 机器人总动员          | 2008 | 安德鲁·斯坦顿           |
| 幽灵公主            | 1997 | 宫崎骏               |
| 控方证人            | 1957 | 比利·怀德             |
| 蝙蝠侠：黑暗骑士崛起      | 2012 | 克里斯托弗·诺兰          |
| 美国往事            | 1984 | 赛尔乔·莱昂内           |
| 你的名字。           | 2016 | 新海诚               |
| 寻梦环游记           | 2017 | 李·昂克里奇            |
| 勇敢的心            | 1995 | 梅尔·吉布森            |
| 三傻大闹宝莱坞         | 2009 | 拉吉库马尔·希拉尼         |
| 地球上的星星          | 2007 | 阿米尔·汗             |
| 摔跤吧！爸爸          | 2016 | 涅提·蒂瓦里            |
| 无耻混蛋            | 2009 | 昆汀·塔伦蒂诺           |
| 心灵捕手            | 1997 | 格斯·范·桑特           |
| 梦之安魂曲           | 2000 | 达伦·阿伦诺夫斯基         |
| 2001太空漫游        | 1968 | 斯坦利·库布里克          |
| 狩猎              | 2012 | 托马斯·温特伯格          |
| 发条橙             | 1971 | 斯坦利·库布里克          |
| 天使爱美丽           | 2001 | 让\-皮埃尔·热内         |
| 雨中曲             | 1952 | 斯坦利·多南            |
| 玩具总动员3          | 2010 | 李·昂克里奇            |
| 一次别离            | 2011 | 阿斯哈·法哈蒂           |
| 何以为家            | 2018 | 娜丁·拉巴基            |
| 飞屋环游记           | 2009 | 彼特·道格特            |
| 罗生门             | 1950 | 黑泽明               |
| 绿皮书             | 2018 | 彼得·法雷里            |
| 小鞋子             | 1997 | 马基德·马基迪           |
| 哈尔的移动城堡         | 2004 | 宫崎骏               |
| 龙猫              | 1988 | 宫崎骏               |
| 美丽心灵            | 2001 | 朗·霍华德             |
| 两杆大烟枪           | 1998 | 盖·里奇              |
| 三块广告牌           | 2017 | 马丁·麦克唐纳           |
| 头脑特工队           | 2015 | 彼特·道格特            |
| V字仇杀队           | 2005 | 詹姆斯·麦克特格          |
| 房间              | 2015 | 伦尼·阿伯拉罕森          |
| 猜火车             | 1996 | 丹尼·博伊尔            |
| 第六感             | 1999 | M·奈特·沙马兰          |
| 禁闭岛             | 2010 | 马丁·斯科塞斯           |
| 乱世佳人            | 1939 | 维克多·弗莱明           |
| 东京物语            | 1953 | 小津安二郎             |
| 楚门的世界           | 1998 | 彼得·威尔             |
| 荒蛮故事            | 2014 | 达米安·斯兹弗隆          |
| 玛丽和马克思          | 2009 | 亚当·艾略特            |
| 消失的爱人           | 2014 | 大卫·芬奇             |
| 血战钢锯岭           | 2016 | 梅尔·吉布森            |
| 驯龙高手            | 2010 | 迪恩·德布洛斯           |
| 杀人回忆            | 2003 | 奉俊昊               |
| 布达佩斯大饭店         | 2014 | 韦斯·安德森            |
| 爱在黎明破晓前         | 1995 | 理查德·林克莱特          |
| 猫鼠游戏            | 2002 | 史蒂文·斯皮尔伯格         |
| 疯狂的麦克斯4：狂暴之路    | 2015 | 乔治·米勒             |
| 忠犬八公的故事         | 2009 | 拉斯·霍尔斯道姆          |
| 风之谷             | 1984 | 宫崎骏               |
| 卢旺达饭店           | 2004 | 特瑞·乔治             |
| 哈利·波特与死亡圣器\(下\) | 2011 | 大卫·叶茨             |
| 聚焦              | 2015 | 汤姆·麦卡锡            |
| 死亡诗社            | 1989 | 彼得·威尔             |
| 怪兽电力公司          | 2001 | 彼特·道格特            |
| 爱在日落黄昏时         | 2004 | 理查德·林克莱特          |
| 花样年华            | 2000 | 王家卫               |
| 无间道             | 2002 | 刘伟强               |
| 天空之城            | 1986 | 宫崎骏               |

## REF

[Python CSV tutorial](http://zetcode.com/python/csv/)

[rank4douban](https://github.com/bimzcy/rank4douban/blob/master/update_snippets.py)




