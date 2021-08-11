---
title: "简单的讲讲Redis分布式hash到底是啥情况"
date: 2019-11-25T13:45:07+07:10
description: "话说分布式一致性Hash可以理解为 放在什么圆环上。长篇大论的讲的我没兴趣看下去，其实最简单的一句话就是普通hash 是按照key值找server 而分布式hash 是按照key值所在的server(hash值)的范围找server。"
categories: [redis,算法]
---

话说分布式一致性Hash可以理解为 放在什么圆环上。长篇大论的讲的我没兴趣看下去，其实最简单的一句话就是普通hash 是按照key值找server 而分布式hash 是按照key值所在的server(hash值)的范围找server。