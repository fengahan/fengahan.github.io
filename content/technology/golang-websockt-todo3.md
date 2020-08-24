---
title: "Golang之Websocket（发送消息)"
date: 2019-08-24T17:23:11+09:46
description: "这个是golang 实现websocket 三部曲的最后一招 发送frams 可以说是三部曲中相对简单的action了"
categories: [go, websocket,网络协议]
---
到这里websocket 就算实现完毕了,
这里仅仅针对127 个字节以下的数据进行发送 剩下的请根据websocket 协议自行补充.
``` go
/**
装包
**/
func sendhMessgeData(conn net.Conn)  {
	msgStr:="hello word"
	content:=[]byte(msgStr)
	msg:= "\x81"+string(rune(len(content)))+msgStr//string(rune(len(content))取单字符的assic码
	conn.Write([]byte(msg))
}

```

下面献上完整的代码
``` go
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
			continue
		}else {
			go handler(conn)
		}

	}
}
func handler(conn net.Conn) {
	defer conn.Close()
	for  {
		//判断是否已经握手
		rmtAddr:=conn.RemoteAddr().String()
		if _,ok:=Client[rmtAddr];!ok {
			fmt.Println("start Hand")
			hr:=doHand(conn)
			if hr==nil {
				fmt.Println("hand Success")
				Client[rmtAddr]="Y"
			}else {

				fmt.Println("hand Error"+hr.Error())
				delete(Client,rmtAddr)
				break
			}
		}else {
			err,msg:=getMessageData(conn)
			if err !=nil {
				fmt.Println(err)
				break
			}
			sendhMessgeData(conn)
			fmt.Println(msg)
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
	sh := sha1.New()
	sh.Write([]byte(handAccString))
	accByte:=sh.Sum(nil)
	accept:=base64.StdEncoding.EncodeToString(accByte)
	resStr := "HTTP/1.1 101 Switching Protocols\r\n"
	resStr = resStr + "Sec-WebSocket-Accept: " + string(accept) + "\r\n"
	resStr = resStr + "Connection: Upgrade\r\n"
	resStr = resStr + "Upgrade: websocket\r\n\r\n"

	_,ee:=conn.Write([]byte(resStr))
	if ee!=nil {
		return err
	}
	return nil
}

/**
装包
*/
func sendhMessgeData(conn net.Conn)  {
	msgStr:="hello word"
	content:=[]byte(msgStr)
	msg:= "\x81"+string(rune(len(content)))+msgStr
	conn.Write([]byte(msg))
}
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
	var realLen byte
    var recData []byte
	var mask []byte
	var rec []byte
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

//获取Http头部信息
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
