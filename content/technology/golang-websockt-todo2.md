---
title: "Golang之Websocket（接收消息)"
date: 2019-08-22T21:42:18+03:33
description: "上部分大概讲了下服务端和客户端 针对websocket协议的握手,这次我们再聊聊第二个步骤,针对消息(frams)的处理,大家看代码吧"
categories: [go, websocket,网络协议]
---

具体大家看下代码实现，这里不做过多的赘述,改方式仅仅实现简单的解析 ，然而实际使用中要复杂的多,
下面的代码只是为了熟悉流程，以及锻炼下从文档到代码的能力
```go
/**
解包
*/
func getMessageData(conn net.Conn) (error,string) {
	var m  =make([]byte,256)
	_,er:=conn.Read(m)
	if er!=nil {
		delete(Client,conn.RemoteAddr().String())
		return er,""
	}
	var playDataLen byte  
	var realLen byte //真实的数据长度
 var recData []byte //经过运算的recData 原数据 就是客户端发来是什么样就是什么样
	var mask []byte  //mask 与recData 按位异或运算可得rec
	var rec []byte   //得到的真是数据
	fin:=m[0]>>7
	if fin==0 {//fin 为0 代表有后续消息 fin为1 代表已经是最后一条消息这里不关心
		playDataLen=m[1] & 127
	}else{
		//数据消息长度
		playDataLen=m[1] & 127
	}
	if playDataLen<126 {
		realLen=playDataLen
		mask=m[2:6]
		recData=m[6:]
	}else if playDataLen==126 {
		mask=m[3:7]
		realLen=m[2] +m[3]
		recData=m[7:]
	}else{
	  // 这里默认当做127个字节处理 为什么i<11 呢  有效负载长度：7位，7 + 16位或7 + 64位 
		//当为127是就是64位8个字节前两个字节是fin+op+resv等等所以第三个字节到第11个字节是真实际长度
		for i:=3;i<11;i++  {
			realLen+=realLen
		}
		mask=m[11:15]
		recData=m[15:]
	}
	for i:=0;i<int(realLen);i++  {
		r:= recData[i] ^ mask[i%4]//这里是百度的没找到出处
		rec=append(rec,r)
	}
	return nil,string(rec)
}
```