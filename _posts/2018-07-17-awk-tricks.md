---
layout: post
title: "Tricks for awk"
description: ""
category: ["Tools"]
---
{% include JB/setup %}

> 需求：
> 一个文件内有多行，还有不定量的列，要求找出每行中最大的一个数字

使用awk实现：

    cat list | awk '{max[NR]=0; for(n=1;n<=NF;n++) { if($n>max[NR]) max[NR]=$n; }}END{for(i=1;i<=NR;i++) print max[i]}'

输入输出示范：

    $ cat list | awk '{max[NR]=0; for(n=1;n<=NF;n++) { if($n>max[NR]) max[NR]=$n; }}END{for(i=1;i<=NR;i++) print max[i]}'
    400
    9999
    1234
    1234
    1234
    $ cat list
    200 300 400 100
    123 2122 345 9999
    222 444 333 1234
    222 444 333 1234
    222 444 333 1234
    222 444 333 1234


> 需求：
> 统计每个IP对HTTP server的请求数

awk实现（这里假设access_log每行的第一列就是访问者的IP地址）：

   awk '{array[$1]++}END{for(i in array){print i, "accessed ", array[i], " times"}}' access_log | sort -gr | head -20

*简单解释一下：在awk内部使用变量无需事先声明，如果变量是列表，那么列表的每个新的元素的值会默认设置为0。而awk又是流处理器，前面的部分相当于对每行进行操作，得到的数组就会是`array[123.123.123.123]=1`这种形式。在END块里直接用一个循环输出列表中的每个元素就能得到每个IP访问了多少次了。*


