---
layout: post
title: 64 行 shell 获取豆瓣电影 Top250 信息
categories: linux
description: 获取豆瓣电影 Top250 信息
keywords: linux, pup, jq
---

## 说明

1，使用 `pup` 解析 html 元素，`jq` 解析 json 数据，详细介绍见参考链接

2，获取片名、年份，国家/地区，类型，导演，评分，评价人数信息，将其分别存在一个独立的文件中

3，将以上独立的文件合并到一个文件中，文件格式为 csv

4，使用 iconv 将文件格式由 UTF-8 转为 GB18030，避免 Windows 下文件格式乱码

## 代码

```shell
#!/bin/bash
# **********************************************************
# * Author        : xoyabc
# * Email         : lxh1031138448@gmail.com
# * Last modified : 2019-10-02 20:58
# * Filename      : get_douban_top250_movies.sh
# * Description   : get douban top 250 movies info
# **********************************************************

TEMP_DIR="temp"
TEMP_MV_INFO="douban_mv_info"
URL="${TEMP_DIR}/url"
NAME="${TEMP_DIR}/name"
YEAR="${TEMP_DIR}/year"
REGION="${TEMP_DIR}/region"
GENRE="${TEMP_DIR}/genre"
DIRECTOR="${TEMP_DIR}/director"
RATING_SCORE="${TEMP_DIR}/rating_score"
RATING_NUM="${TEMP_DIR}/rating_num"

RESULT_FILE="top250_movies.csv"

[ ! -d ${TEMP_DIR} ] && mkdir ${TEMP_DIR}
>${URL} && >${NAME} && >${YEAR} && >${REGION} && >${GENRE} && >${DIRECTOR} && >${RATING_SCORE} && >${RATING_NUM}

# 循环遍历
for i in $(seq 0 25 225)
do
        link="https://movie.douban.com/top250?start=${i}&filter="
        # 将页面保存到临时文件中
        curl -s "${link}" > ${TEMP_MV_INFO}
        # URL
        cat ${TEMP_MV_INFO} |pup 'div[class="hd"] a attr{href}' >> ${URL}
        # other ways
        #cat douban_mv_info |pup 'div[class="hd"] json{}' |jq -r '.[]|.children|.[]|select(.href != null) |.href'
        #cat douban_mv_info |pup 'div[class="hd"] json{}' |jq -r '.[]|.children[0]|.href'

        # movie name，片名
        cat ${TEMP_MV_INFO} |pup 'div[class="hd"] json{}' |jq -r '.[]|.children|.[]|select(.children != null)|.children[0]|.text' >> ${NAME}

        # year，年份
        cat ${TEMP_MV_INFO} |sed 's#&nbsp;##g' |pup 'div[class="bd"] p[class=""] text{}' |awk --posix -F "/" '!/^[[:space:]]+$/ && !/^$/ && /[0-9]{4}/ { sub(/^[[:blank:]]*/,"",$1);sub(/\(.*\)/,"",$1);print $1 }' >> ${YEAR}

        # region，国家/地区
        cat ${TEMP_MV_INFO} |sed 's#&nbsp;##g' |pup 'div[class="bd"] p[class=""] text{}' |awk --posix -F "/" '!/^[[:space:]]+$/ && !/^$/ && /[0-9]{4}/ { print $(NF-1)}' >> ${REGION}

        # genre，类型
        cat ${TEMP_MV_INFO} |sed 's#&nbsp;##g' |pup 'div[class="bd"] p[class=""] text{}' |awk --posix -F "/" '!/^[[:space:]]+$/ && !/^$/ && /[0-9]{4}/ { print $NF}' >> ${GENRE}

        # director，导演
        cat ${TEMP_MV_INFO} |pup 'div[class="bd"] p[class=""] text{}' |awk --posix -F "[/ ]+" '!/^[[:space:]]+$/ && !/^$/ && !/[0-9]{4}/{print $3}'  >> ${DIRECTOR}

        # rating_score，评分
        cat ${TEMP_MV_INFO} |pup 'div[class="star"] span[property="v:average"] text{}' >> ${RATING_SCORE}

        # rating_num，评价人数
        cat ${TEMP_MV_INFO} |pup 'div[class="star"] span:last-child text{}' |grep -Eo '[0-9]{1,}' >> ${RATING_NUM}
done

# 将片名、年份，国家/地区，类型，导演，评分，评价人数，链接等信息合并到csv文件中
paste -d ',' ${NAME} ${YEAR} ${REGION} ${GENRE} ${DIRECTOR} ${RATING_SCORE} ${RATING_NUM} ${URL} > ${RESULT_FILE}
# 添加表头
sed -i '1i 片名,年份,制片国家/地区,类型,导演,评分,评价人数,链接' ${RESULT_FILE}
iconv -f utf8 -t gb18030 ${RESULT_FILE} -o ${RESULT_FILE}
```

## 结果
去年（2018）共有三部影片入选，分别是《何以为家》、《我不是药神》、《头号玩家》。
![douban_top_250.png](https://i.loli.net/2019/10/02/GcpmQTSqd9jkxsU.png)

## REF

[pup](https://github.com/ericchiang/pup)

[jq](https://stedolan.github.io/jq/)

[如何解决 iconv: illegal input sequence at position](https://my.oschina.net/repine/blog/898156)
