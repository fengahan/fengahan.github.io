---
title: "今天有大佬在群里问go的[]uint8怎么转string"
date: 2019-08-14T16:05:58+05:30
description: "之前写过 因为在数据库连表查询的时候会出现null 这个null 在go的sql查询会返回[]uint8 觉得大家都会 也没记录，然后大佬们说百度出来的都不行我才又打算写出来的"
categories: [go]
---

```golang

var user_nickname []uint8
b := make([]byte, len(user_nickname))
for i, v := range user_nickname {
			b[i] = byte(v)
}
fmt.Println(string(b))

```		
		
