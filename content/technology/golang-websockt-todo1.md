---
title: "Golang之Websocket（握手)"
date: 2019-08-21T17:22:48+06:33
description: "空闲时间不多,而且之前对这块不是特别深入,所以呢这部分打算分三部分写分别为 1.握手  2.发送Frames, 3.接收Frames"
categories: [go, websocket,网络协议]
---

空闲时间不多,而且之前对这块不是特别深入,所以呢这部分打算分三部分写分别为 1.握手  2.发送Frames, 3.接收Frames
关于websocket 的握手大家可以wiki上面看一下,下面是我的golang 实现捂手部分
参考文献[RFC6455][1] 和 [webcoket分析][2]

```go
package main

import (
	"fmt"
	"net"
	"strings"
	"crypto/sha1"
	"encoding/base64"
)
const SRCKEY  ="258EAFA5-E914-47DA-95CA-C5AB0DC85B11"
var Client =make(map[string]string)
func main() {
	n,err:=net.Listen("tcp",":8080")
	if err !=nil {
		panic(err)
	}
	for  {
		conn,err:=n.Accept()
		if err!=nil {
			fmt.Println(err)
		}
		go handler(conn)
	}
}
func handler(conn net.Conn) {
	defer conn.Close()
	for  {
		//判断是否已经握手
		rmtAddr:=conn.RemoteAddr().String()//获取客户端的地址。
		if _,ok:=Client[rmtAddr];!ok {
			fmt.Println("start Hand")
			hr:=doHand(conn)
			if hr==nil {
				fmt.Println("hand Success")
				Client[rmtAddr]="Y"
			}else {
				fmt.Println("hand Error"+hr.Error())
				break
			}
		}else {
			getMessageData(conn)
		}
	}
}
/**
  握手函数
 */
func doHand(conn net.Conn) error {
   var msgContent =make([]byte,1024)
   _,err:= conn.Read(msgContent)
	if err!=nil {
		return err
	}
   msgContentString:=string(msgContent)
   //获取Sec-WebSocket-Key
   var secKey string
	headers:=getHttpHeader(msgContentString)
	if v,ok:=headers["Sec-WebSocket-Key"];ok{
		secKey=v
	}else {
		return fmt.Errorf("%s","Sec-WebSocket-Key not found"+msgContentString)
	}
	handAccString:=secKey+SRCKEY
	//先进行sh1加密,把加密后的值再进行base64加密得到Sec-WebSocket-Accept的值
	sh := sha1.New()
	sh.Write([]byte(handAccString))
	accByte:=sh.Sum(nil)
	accept:=base64.StdEncoding.EncodeToString(accByte)
	//此处顺序无关紧要
	resStr := "HTTP/1.1 101 Switching Protocols\r\n"
	resStr = resStr + "Sec-WebSocket-Accept: " + string(accept) + "\r\n"
	resStr = resStr + "Connection: Upgrade\r\n"
	resStr = resStr + "Upgrade: websocket\r\n\r\n"
	_,ee:=conn.Write([]byte(resStr))//写入http请求头
	if ee!=nil {
		return err
	}
	return nil
}
/**
    解析头部
    这部分内容是当websocket连接Tcp服务端时要发送的信息
    GET / HTTP/1.1
    Host: 127.0.0.1:8080
    Connection: Upgrade
    Pragma: no-cache
    Cache-Control: no-cache
    Upgrade: websocket
    Sec-WebSocket-Version: 13
    Accept-Encoding: gzip, deflate, br
    Accept-Language: zh-CN,zh;q=0.9
    Sec-WebSocket-Key: e+D+Ox4LvEbt42S7lC4Y+A==
    Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
**/
func getHttpHeader(content string) map[string]string {
	headers := make(map[string]string, 10)
	rows := strings.Split(content, "\r\n")

	for _,row := range rows {
		if len(row) >= 0 {
			words := strings.Split(row, ":")
			if len(words) == 2 {
				headers[strings.Trim(words[0]," ")] = strings.Trim(words[1], " ")
			}
		}
	}
	return headers
}

```
  [1]: http://www.rfcreader.com/#rfc6455
  [2]: https://www.cnblogs.com/songwenjie/p/8575579.html