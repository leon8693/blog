---
layout: post
title: "awk and sed"
date: 2014-05-05 11:05
comments: true
categories: Other
description: "awk and sed"
tags: [awk,sed]
keywords: linux,awk,sed
---


###AWK
通过-F分割段落并将输出的内容，以一行2个组成2列
```
#文本内容
cat test
"111" : "33333"
"222" : "44444"
"333" : "55555"
"444" : "66666"

#结果
awk -F'[":]' 'NR%2{printf $5" ";next}{print $5}' test
33333 44444
55555 66666
```
