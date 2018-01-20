---
title: 使用Fiddler调试微信jssdk
date: 2016-06-12 22:34:54
tags:
	- 微信
	- Fiddler
categories: 微信开发
---
### 使用Fiddler调试微信 js sdk

#### PC端调试
1. 打开fiddler hosts，`菜单 -> Tools -> hosts`，填入内容格式为：targethost hostname
例如：192.168.1.19 xxx.xxx.com
2. 在浏览器直接访问 xxx.xxx.com 就可以了
3. 如果在微信调试工具中调试，需要另外将微信调试工具的代理设置为：本机ip，端口号8888
<!--more-->
#### 手机端调试
1. 在pc端的fiddler中，开启 Allow remote computers to connect `如何开启请自行百度`
2. 重启fiddler
3. 手机连上wifi，跟电脑处于通过局域网
4. 修改wifi中的HTTP代理，设置为本机ip，端口号8888

>注意，想要通过js sdk的验证，你的域名需在微信开放平台上设置为安全域名