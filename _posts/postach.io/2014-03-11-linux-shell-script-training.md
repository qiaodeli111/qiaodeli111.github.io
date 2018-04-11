---
layout: post
title: "Linux Shell Script training"
description: ""
category: "Coding"
---
{% include JB/setup %}

**1、写脚本实现，可以用shell、perl等。在目录/tmp下找到100个以abc开头的文件，然后把这些文件的第一行保存到文件new中。**    

<!-- more --> 
    
    #!/bin/bash
    cd /tmp
    for file in `ls abc* | head -100`; do
        head -1 $file >> new
    done
 
***分析***


先`ls abc* | head -100` 找到100个以abc开头的文件，然后把每个文件的第一行都存到new文件里

* * *

**2、写脚本实现，可以用shell、perl等。把文件b中有的，但是文件a中没有的所有行，保存为文件c，并统计c的行数。**  

    
    
    #!/bin/bash
    diff A B > tmpfile
    grep -A2 '^[0-9]\{1,\}a[0-9]\{1,\}' tmpfile | grep '^> ' | cut -d " " -f 2 > C
    rm -rf tmpfile
    wc -l C
    
***分析***

先diff一下并且把结果放到tmpfile里面。  
diff的结果，如果是A文件里没有但是B文件里有的，那么将会是类似于如下的结果：  `5a6 > xxxxxxxxxxxx`

我们需要的内容是”>”后面的内容，所以需要吧tmpfile做一下处理：

`grep -A2 '^[0-9]{1,}a[0-9]{1,}' tmpfile`可以找到有类似5a6起的相邻两行，然后再把每一组的第二行找出来，然后再把"> "之后的实体内容找出来，这些内容就是真正的A里面没有但是B里有的内容了。然后得到的内容放到C里。

C文件的行数就很简单了，wc -l C，搞定。