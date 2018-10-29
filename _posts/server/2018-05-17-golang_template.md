---
layout : life
title: golang template
category : server
duoshuo: true
date : 2018-05-17
---

# Golang template 常见问题
————不解决随意可以百度的问题

1. template 语法
* {{with .Var}} 在下文可以用{{.}}
* 判断相等用 {{eq .x .y}}
* 获取数组长度  {{len .papers}}
* 判断数组长度为0   {{eq (len .papers) 0}}

2. template 自定义函数
* FuncMaps
* 如果是ParseFiles,则template对象无法使用自定义函数，用如下方法解决
```
t, _ := template.New("user.html").Funcs(template.FuncMap{"showTime": func(ts int32) string {
			return time.Unix(int64(ts), 0).Format("2006-01-02 15:04:05")
		}}).ParseFiles("static/views/user.html", "static/views/ref.html")
```
* 调试——打印error