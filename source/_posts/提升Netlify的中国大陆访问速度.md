---
title: 提升Netlify的中国大陆访问速度
categories:
  - 记录
tags:
  - HTTPS
  - TLS
  - SSL
  - 互联网
  - 域名
  - IPv4
  - IPv6
  - CDN
  - DNS
slug: improve-the-pagespeed-of-netlify-in-china-mainland
abbrlink: b0c1b58e
date: 2025-10-03 18:19:23
---

## 0.引言

9月中旬的时候我评测过各家的托管/CDN服务，对Netlify的评价是速度很好（[见此处](../posts/7a44deef/)），但Netlify默认CNAME在近期出现了阻断的现象，中国大陆有超过10个节点超时（失败率10%以上），对我而言不可接受，遂心生一计试图优选IP解决。
图为Netlify的访问速度
IPv4访问速度：

![Netlify中国大陆访问速度-IPv4](https://static.987632.xyz/img/20251003183002496.png)

IPv6访问速度：
![Netlify中国大陆访问速度-IPv6](https://static.987632.xyz/img/20251003183331660.png)

## 1.实现原理

Netlify的IP段是所有网站共享的（包括其官网`netlify.com`），但默认分配给我们使用的IP段质量虽然还可以但由于CNAME的目标地址`<projectname>.netlify.app`被DNS污染和/或IP/SNI阻断，导致该域名成为了网站的故障点，我们要做的就是绕过这个故障的域名直接访问其IP地址。 **经过测试，默认分配的IP地址多在新加坡，优缺点上文已讲**

经过我的试验，Netlify并不严格校验用户是否将域名的解析记录设置到其提供的CNAME域名（即`<projectname>.netlify.app`），而`netlify.com`使用的IP地址也可以响应我们的域名的请求（也就是基于SNI的流量路由），所以我们可以在自定义域设置A记录和AAAA记录（Netlify支持IPv6）指向更快的IP地址，Netlify是可以正常为我们的网站签发有效的SSL/TLS证书的，使用的CA为Let‘s Encrypt。

## 2.上手操作

### 2.1 获取高速的IP地址

打开itdog网站测速（[IPv4版点此](https://www.itdog.cn/http/) | [IPv6版点此](https://www.itdog.cn/http_ipv6/)），分别对`https://netlify.com` 进行测试，获取速度快的IP
IPv4测试解析：
![Netlify解析IPv4](https://static.987632.xyz/img/20251003184822551.png)

看到解析结果是这两个IP，是Amazon的Anycast地址，速度很快：

```txt
3.33.186.135
15.197.167.90
```

把IPv4地址记录好备用，然后接着测试IPv6地址解析：

![Netlify解析IPv6](https://static.987632.xyz/img/20251003185052300.png)

可以看到IP地址比较多，挨个测试，得到这样的比较快的IP：

```txt
2406:da12:53f:c100::1f5
2406:da18:b3d:e201::1f5
```

IPv6地址记录下来，稍后添加DNS记录

### 2.2 添加DNS记录

首先在Netlify处添加好自定义域名，按照指引操作即可，不难

> 注意：如果是第一次添加新的域名的话需要进行域名所有权验证，按照指引操作即可，是正常现象。

添加好之后如下图，显示你的自定义域为Primary domain：
![在Netlify处添加好自定义域](https://static.987632.xyz/img/20251003185617430.png)

然后，在你的DNS提供商处添加域名解析记录， **不要使用Netlify要求填入的`<projectname>.netlify.app`，否则会前功尽弃**  ，添加A记录和AAAA记录，记录的值填写你选出来速度快的IP地址，IPv4和IPv6的记录建议各两条就可以了， **过多的解析节点反而对优化访问速度不利** 。

我这里以Cloudflare为例，在下图中的各字段填写对应的值：
![Cloudflare添加DNS记录](https://static.987632.xyz/img/20251003190246158.png)


>类型选择A或者AAAA 
>名称这里填写你的域名，也就是在Netlify那里填写的自定义域名
>IPv4（IPv6）地址这一栏填写你选出的高速IP,注意一条记录只能写一个IP地址
>*如果你想省事也可以直接使用我做好的优选域名`netlify.987632.xyz`*
>2025/10/06更新：还有个邪修方法是直接把域名CNAME到`www.netlify.com`

添加完成后，等待DNS刷新，大概三到五分钟后浏览器这个自定义域名就能正常访问了，Netlify自动申请了SSL/TLS证书进行HTTPS加密（对HTTPS的原理有兴趣可[试读这篇文章](../posts/a3e6861f/)），测试一下打开速度，绿到发麻
![优选IP后的Netlify测速](https://static.987632.xyz/img/20251003191100448.png)

## 3.完结撒花

恭喜你，成功地把不稳定的Netlify默认CNAME换掉，做到了网站在中国大陆秒开，希望本篇对你有所帮助
