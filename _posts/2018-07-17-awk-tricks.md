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

