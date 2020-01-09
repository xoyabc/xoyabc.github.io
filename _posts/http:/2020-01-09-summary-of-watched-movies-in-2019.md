---
layout: post
title: 2018-2019观影记录
categories: movie
description: 2018-2019观影记录
keywords: python, movie
---

查看了最近一年(2018.12.14-2019.12.30)的票根，统计了上面的片名、日期、影院及票价信息。接着又调用豆瓣API获取了对应影片的年份、片长、评分、评价人数、国家、语言、类型、主演及导演。

## 1，看片数量

共计 247 部
![movie-2019-total-count.png](https://i.loli.net/2020/01/09/csSgfEDJzlxhpCv.png)

## 2，影片年份统计

19 年最多，共计 72 部
![movie-2019-year.png](https://i.loli.net/2020/01/09/czGrQW8CujLlegE.png)

## 3，二刷的电影

- 过春天
- 他们已不再变老
- 撞死了一只羊
- 哪吒之魔童降世
- 徒手攀岩

![movie-2019-seen-twice.png](https://i.loli.net/2020/01/09/HrDemvZkwzbWcJ1.png)

## 4，类型

排名前三的依次为喜剧，剧情，爱情。
![movie-2019-rank-type.png](https://i.loli.net/2020/01/09/vR5cNOF2nlYoh4W.png)

## 5，主演

观看超过一部的主演、影片数量及片名见下：
| 主演        | 影片数 | 片名                         |
|-----------|-----|----------------------------|
| 北野武       | 6   | 小心恶警，大佬，花火，菊次郎的夏天，奏鸣曲，座头市  |
| 黄渤        | 3   | 被光抓走的人，疯狂的外星人，我和我的祖国       |
| 克林特·伊斯特伍德 | 3   | 不可饶恕，廊桥遗梦，骡子               |
| 三船敏郎      | 2   | 战国英豪，蜘蛛巢城                  |
| 王千源       | 2   | 钢的琴，你是凶手                   |
| 伊莲娜·雅各布   | 2   | 给西奥的信，蓝白红三部曲之红             |
| 齐雅拉·马斯楚安尼 | 2   | 212号房间，我在伊朗长大              |
| 斯特兰·斯卡斯加德 | 2   | 被涂污的鸟，外出偷马                 |
| 刘德华       | 2   | 扫毒2天地对决，旺角卡门               |
| 基里安·墨菲    | 2   | 风吹麦浪，酒会                    |
| 仲代达矢      | 2   | 切腹，影武者                     |

![movie-2019-rank-start.png](https://i.loli.net/2020/01/09/ywmX4gUvj8eF1MH.png)

## 6，导演top10

观看超过一部的导演、影片数量及片名见下：
| 导演            | 影片数 | 片名                                           |
|---------------|-----|----------------------------------------------|
| 北野武           | 9   | 心恶警，大佬，花火，坏孩子的天空，菊次郎的夏天，那年夏天，宁静的海，玩偶，奏鸣曲，座头市 |
| 维姆·文德斯        | 6   | 爱丽丝城市漫游记，柏林苍穹下，德州巴黎，地球之盐，公路之王，里斯本的故事         |
| 黑泽明           | 5   | 梦，影武者，战国英豪，蜘蛛巢城，姿三四郎                         |
| 罗伯托·罗西里尼      | 3   | 德意志零年，火山边缘之恋，罗马，不设防的城市                       |
| 查理·卓别林        | 3   | 城市之光，凡尔杜先生，摩登时代                              |
| 斯坦利·库布里克      | 3   | 2001太空漫游，巴里·林登，杀手                            |
| 克林特·伊斯特伍德     | 3   | 不可饶恕，廊桥遗梦，骡子                                 |
| 克日什托夫·基耶斯洛夫斯基 | 3   | 蓝，白，红                                        |
| 李安            | 3   | 少年派的奇幻漂流，双子杀手，卧虎藏龙                           |
| 维克多·弗莱明       | 2   | 乱世佳人，绿野仙踪3D                                  |
| 阿涅斯·瓦尔达       | 2   | 阿涅斯论瓦尔达，五至七时的克莱奥                             |
| 奥逊·威尔斯        | 2   | 公民凯恩，历劫佳人                                    |
| 侯孝贤           | 2   | 海上花，最好的时光                                    |
| 肯·洛奇          | 2   | 对不起，我们错过了你，风吹麦浪                              |
| 朱塞佩·托纳多雷      | 2   | 海上钢琴师，西西里的美丽传说                               |
| 陈凯歌           | 2   | 孩子王，我和我的祖国                                   |
| 王家卫           | 2   | 旺角卡门，一代宗师                                    |
| 大卫·雷奇         | 2   | 死侍2：我爱我家，速度与激情：特别行动                          |
| 宫崎骏           | 2   | 龙猫，千与千寻                                      |
| 雅克·欧迪亚        | 2   | 希斯特斯兄弟，锈与骨                                   |

![movie-2019-rank-director.png](https://i.loli.net/2020/01/09/xXQdSqYsjhZk9ew.png)

## 7，票价

除去活动票、免费票，最便宜的是电影博物馆的女狙击手（5），最贵的是双子杀手（159），平均票价46。

![movie-2019-ticket-price.png](https://i.loli.net/2020/01/09/RZrtU8vzy4qPlaf.png)

## 8，观看城市

共计4个城市，北京119部，漯河3部，西安2部，杭州1部。
![movie-2018-count.png](https://i.loli.net/2018/12/16/5c15ce633864c.png)
![movie-2019-city.png.png](https://i.loli.net/2020/01/09/OUBtdyz4KvLG9i8.png)

## 9，影院

+ top3 影院
    - 中国电影资料馆（小西天店） 93部
    - 泰禾影城（立水桥店）      45部
    - 当代MOMA百老汇电影中心    32部

![movie-2019-cinemas.png](https://i.loli.net/2020/01/09/hevOPEN9w2WUIfy.png)

+ top5观看方式
    - 商业院线，102部，新片居多
    - 艺术影院，92部，经典老片居多
    - 北影节，21部
    - 欧盟影展，17部
    - 北野武影展，8部

![movie-2019-watch-method.png](https://i.loli.net/2020/01/09/9g2JmLQIUT5buM3.png)

## 10，一些难忘的场次

见备注列

![movie-2019-unforgettable-showings-mobile.png](https://i.loli.net/2020/01/09/Aga9IJTjY16WFKV.png)

完~~~