---
title: golang祖安助手
categories:
- 小知识
- golang
tags:
- golang
- 脚本
src: //eatboooo.gitee.io/img/background/golong_zaun_h.jpg
---
# 祖安助手
分为两个版本
+ 文件读取版本
+ 网络请求版本
## 文件读取版本
读取本地的 test.txt 文件，把文件内容用分号隔开，以数组的形式存入到内存中，同时监听键盘按键，每次触发按键后在数组中依次读取语句存入到剪切板中。
```go
package main

import (
	"fmt"
	"gitee.com/veni0/robotgo"
	"io/ioutil"
	"strings"
)

//该方法用于读取文件
func getFile(name string) [] string {
	data, err := ioutil.ReadFile(name) //读取文件
	if err != nil {
		fmt.Println("File reading error", err)
		return nil
	}
	fmt.Println("Contents of file:", string(data)) //打印文件内容
	split := strings.Split(string(data), ";")      //用分号隔开文件内容，并以数组的形式返回
	return split
}

//该方法用于把string字符串存入到剪切板中
func putMsg(msg string) {
	clipboard.WriteAll(msg)
}

func main() {
	fmt.Println("---祖安助手！---")
	msg := getFile("test.txt")   //读取文件
	eve := robotgo.AddEvent("v") //开始监听
	//robotgo.KeyTap("a", "ctrl")
	//全局监听事件
	fmt.Println("---请按v键---")
	// 循环遍历数组
	for i := 0; eve; i++ {
		if i < len(msg) {
			putMsg(msg[i])
		} else {
			i = 0
			putMsg(msg[0] + "\n") //当 i 超过数组长度时，修改 i 的值并且把 msg【0】 存入到剪切板中。
		}
		fmt.Println("---你按下v键---", "v")
		eve = robotgo.AddEvent("v")
	}
}
```

## 网络请求版本
向网站 [骂人宝典](https://nmsl.shadiao.app/) 发送 get 请求，把 response 转换成string存入到剪切板中。
```go
package main

import (
	"fmt"
	"gitee.com/veni0/robotgo"
	"github.com/atotto/clipboard"
	"io/ioutil"
	"net/http"
)

//该方法用于模拟发送get请求
func GetData() string {
	client := &http.Client{}
	resp, err := client.Get("https://nmsl.shadiao.app/api.php?lang=zh_cn") //发送get请求,并接收response
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body) //读取response
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(string(body))
	return string(body) //以string的形式返回
}

//该方法用于把string字符串存入到剪切板中
func putMsg(msg string) {
	clipboard.WriteAll(msg)
}

func main() {
	fmt.Println("---祖安助手！---")
	putMsg(GetData()) 
	eve := robotgo.AddEvent("v")
	
	for eve {
		putMsg(GetData())
		fmt.Println("---你按下v键，剪切板生成了以下内容---", "v")
		eve = robotgo.AddEvent("v")
	}
}
```