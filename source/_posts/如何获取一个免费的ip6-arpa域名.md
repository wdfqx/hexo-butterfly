---
title: 如何获取一个免费的ip6.arpa域名
categories:
  - 记录
tags:
  - IPv6
  - ip6.arpa
  - 域名
  - DNS
  - CDN
  - CA（证书颁发机构）
abbrlink: f8911cd
date: 2025-11-07 21:46:36
---

## 0.引言

~~众所周知~~ （其实几乎没人知道），有一种**TLD（Top Level Domain/顶级域名）** 叫作`.arpa`，是互联网的第一个TLD，后期停止作为公网解析的域名，用作互联网基础设施，例如 **反解域名（Reverse DNS Zone）**，具体的如`in-addr.arpa`（IPv4的反解域名）、`ip6.arpa`（IPv6的反解域名）、`e164.arpa`（手机号码的反解域名，一般用于VOIP等业务），但反解域名也是域名，所以它在技术上也能提供常规的解析记录，只不过我们现在一般不这么干就是（战术摊手.webp）

## 1.开始获取

本文通过[Webdom](https://www.webdom.cn)注册ip6.arpa域名，完全免费，ip6.arpa域名最大的特点就是可以托管到Cloudflare，方便使用（尽管我不知道有啥用）

首先点此进入Webdom平台，点击右上角登录/注册按钮进入登录页面，如果你有账号就输入账密登录（那你还来看我这篇文章作甚？），没有账号就选择注册，按要求填好验证码、邮箱、密码，示例如下图：
![Webdom注册页面](https://static.987632.xyz/img/20251108001154944.png)

点击注册后，进入面板，点击注册域名，选择一个前缀，点击注册，系统会自动分配一个域名，过程很简单就不配图了~~（其实是当时忘了截图）~~

注册完之后，打开后台面板就能看到你注册好的域名了
![Webdom后台](https://static.987632.xyz/img/20251108002934949.png)

点击旁边的“DNS管理”即可进入NS服务器配置页面，这里我配置好就不演示了
![配置好的NS服务器](https://static.987632.xyz/img/20251108003212431.png)

托管到Cloudflare是个人就会，有手就行，不演示了，等待域名被激活就行，大概三五分钟。

## 3.为ip6.arpa签发SSL/TLS证书

Cloudflare默认以Google Trust Services为签发SSL/TLS证书的CA，但ip6.arpa域名相对特殊，这种域名在常规的CA处会被拒签证书，所以我们需要发起更改CA的请求，给出的命令是这个：

```bash
curl --location --request PATCH 'https://api.cloudflare.com/client/v4/zones/<zone_id>/ssl/universal/settings' --header 'X-Auth-Email: 你的CF注册邮箱' --header 'X-Auth-Key: 你的CF全局APIKey' --header 'Content-Type: application/json' --data-raw '{"enabled":true,"certificate_authority":"ssl_com"}'
```



按照网上的其他教程，这时你需要准备注册邮箱和Global API Key请求api.cloudflare.com发起更改请求，把CA换成SSL.COM以获得有效的SSL/TLS证书，但2021年以后注册的Cloudflare账号已经不提供Global Key了，官方的理由是出于安全
**那咋办啊？凉拌炒鸡蛋么？**
放心，不会的，API Key不能用了就用Token，反正API永远在那

我们需要准备这些，用于请求更换CA：

- 域名的区域ID
- API Token
- 一个curl命令行

创建一个API令牌，这样写就行了

![创建API Token](https://static.987632.xyz/img/20251108005720493.png)

然后获取你的区域ID，直接点进你添加的域名的“概述”页面，右边的侧栏有：
![域名的区域ID](https://static.987632.xyz/img/20251108010128433.png)

然后，你就可以开始进行更换请求的执行了，打开终端敲下这串命令，看下输出

```bash
curl --location --request PATCH 'https://api.cloudflare.com/client/v4/zones/<zone_id>/ssl/universal/settings' --header 'Authorization: Bearer 你的API Token' --header 'Content-Type: application/json' --data-raw '{"enabled":true,"certificate_authority":"ssl_com"}'
```

如果不出意外，终端的输出会是这个，就代表你成功了：

```json
{"result":{"enabled":true,"certificate_authority":"ssl_com"},"success":true,"errors":[],"messages":[]}
```

然后？没有然后了，添加解析，用去吧，完结，撒花
