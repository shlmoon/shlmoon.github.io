---
layout: post
title: Let's Encrypt SSL证书 
date: 2019-03-22 09:10:00
tags: https
---

备注：[Let's Encrypt SSL证书安装教程](https://zhuanlan.zhihu.com/p/24996258)

#### 事故场景
通过certbot安装Let's Encrypt https证书，在安卓手机的微信小程序中报错：`wx.request:fail ssl hand shake error`
<!-- more -->
#### 证书鉴定
通过[SSL服务器证书安装检查器](https://www.myssl.cn/tools/check-server-cert.html)
检查证书发现
```
证书2
证书使用者：	Let's Encrypt Authority X3
证书颁发者：	ISRG Root X1
有效期：	从 2016/10/6 到 2021/10/6
签名算法：	sha256RSA
公钥长度：	2048位 (RSA 算法)
SHA1指纹：	1b 23 67 53 54 fc ad 90 11 9d 88 07 50 15 ea 17 ad d5 27 d8
SHA256指纹：	73 1d 3d 9c fa a0 61 48 7a 1d 71 44 5a 42 f6 7d f0 af ca 2a 6c 2d 2f 98 ff 7b 3c e1 12 b1 f5 68
证书来源：	缺失证书
状态：	错误： 服务器缺少中间证书
```

#### 问题解决
通过[获取证书信息 及 下载中间证书](https://www.myssl.cn/tools/downloadchain.html)将letsencrypt证书cert1.pem生成中间证书文件chain.crt下载下来，打开中间证书文件chain.crt将里面的内容追加到fullchain1.pem，保存。解决问题。

