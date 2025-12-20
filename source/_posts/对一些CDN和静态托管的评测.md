---
title: 对一些CDN和静态托管的评测结果
categories:
  - 记录
tags:
  - CDN
  - HTTPS
  - 域名
  - IPv4
  - IPv6
  - SSL
  - TLS
  - 互联网
  - 记录
  - DNS
slug: comments-about-some-CDNs-and-hosting-services
abbrlink: 7a44deef
date: 2025-09-19 23:42:26
---
## 0.引言
我在优化本网站在中国大陆地区的访问速度时曾使用过各家免费CDN/静态托管服务， ~~写不出东西遂水一篇文~~ 在此分享些我对各家的看法和一些测试结果（本篇提到的所有服务都无需备案）
本篇使用的测速方法为[itdog网站测速-IPv4版](https://www.itdog.cn/http/) | i[tdog网站测速-IPv6版](https://www.itdog.cn/http_ipv6/)，IPv4和IPv6（如果支持）均有
### 0.1 免责声明
本站推荐的任何网络服务均为合法服务，个人观点仅供参考，请勿用于任何违反法律法规的用途，安全绿色上网
## 1.[Vercel](https://vercel.com)
本站现在（2025-09-19）在用的托管服务，每月提供100GB的免费流量额度，但不支持IPv6，仅支持IPv4，网络托管在AWS服务上，总体速度很快，但近期疑似有一些地区出现污染现象，默认的 `*.vercel.app` 会被SNI阻断，需要绑定自己的域名，注册无门槛，用量限制较严格。
测速使用的域名：本站域名
坑点：每月只有1M（100万次）请求额度，若超出免费额度的话可能会有封号的风险。
[官网：vercel.com](https://vercel.com)
![Vercel-IPv4测速结果](https://static.987632.xyz/img/20250920000637557.webp)
## 2.[Netlify](https://netlify.com)
我曾使用过Netlify作为托管，整体的体验和Vercel大体相同，都是基于AWS的服务器IP，同时支持IPv4和IPv6访问，默认域名的访问速度不错，没有被SNI阻断
坑点：如果你没有在自己的Team里移除掉所有绑定的自定义域，你将无法再将这个域名绑定到其他的Teams,需要在[Netlify支持论坛](https://answers.netlify.com)发帖求助staff才能解决。如果超出免费额度网站将被Netlify关停。
测速的结果如下，测试域名：Netlify默认域名
IPv4结果：
![Netlify-IPv4](https://static.987632.xyz/img/20250920002713244.webp)
IPv6结果：
![Netlify-IPv6](https://static.987632.xyz/img/20250920002851411.webp)
可以看到速度还是非常不错的，但缺点也和Vercel一样，部分地区有一些访问失败，疑似污染或阻断。
## 3.[Tenent EdgeOne](https://edgeone.ai)
这个CDN服务我没才用上不久，看了下这家是免费提供中国大陆地区加速节点的（需要备案，且各家的中国大陆地区节点速度都大差不差，遂略去），不备案使用默认的海外节点在中国大陆的访问速度尚可但不算优秀，略逊于Vercel和Netlify。这家是支持IPv6的，但需要用户手动选择在加速服务中开启（当然，如果你使用EdgeOne Pages的话当我没说，这个是真不支持，至少我不知道在哪里开）。
坑点：Tencent EdgeOne在IPv6的优化不如IPv4，主要原因是其IPv6网络是基于新加坡/Aceville Pte.ltd的节点而非其自家的优质节点。
这个CDN是支持高级回源策略的，腾讯云将其视为对标CloudFlare的产品
![回源配置策略](https://static.987632.xyz/img/20250920004809066.webp)
上结果，测试域名为`r2.naki-im.top`：
IPv4测速结果：
![EdgeOne-IPv4](https://static.987632.xyz/img/20250920004134380.webp)
IPv6测速结果：
![EdgeOne-IPv6](https://static.987632.xyz/img/20250920004229892.webp)
很明显的，IPv6节点质量不如IPv4。

## 4.[CloudFlare](https://www.cloudflare.com)
这家是老牌CDN服务商，成立已有十几年（since 2009？），加速节点什么样子想必都是心知肚明的，看起来半死不活。免费版套餐非常慷慨，无限流量和DDoS防护，也提供诸如R2等（本文讲加速/托管服务，遂略去）。官方宣称是无流量和请求数限制，实际看来也确实如此。官方还声称其服务 **无法被打死** ，我不知道怎么回事，不予置评。
坑点：整体来看半死不活的，中国大陆地区大范围的超时和解析失败，用起来比较难受。（别用优选IP和自定义节点杠我，但你杠就是你对）
上测试结果，测试域名blog-74z.pages.dev：
IPv4测速结果：
![CloudFlare-IPv4](https://static.987632.xyz/img/20250920010404682.webp)
IPv6测速结果：
![CloudFlare-IPv6](https://static.987632.xyz/img/20250920010504597.webp)
可以看到，中国大陆地区基本上全境都有访问阻断问题。

## 5.[Github Pages](https://pages.github.com/)
GitHub Pages的网络质量想必都是心知肚明，移动屏蔽，其他半死不活，时常有SNI阻断和IP封禁或者DNS污染，官方提供无限流量和请求数，以及每个仓库5GB的网页存储空间。 **绑定自定义域无法解决访问速度问题。** 
坑点：常年有DNS污染、SNI阻断、移动屏蔽，半死不活，用起来极其难受。
直接上测速图，测试域名naki-im.github.io：
IPv4测速结果：
![GitHub-Pages-IPv4](https://static.987632.xyz/img/20250920011302436.webp)
IPv6测速结果：
![GitHub-Pages-IPv6](https://static.987632.xyz/img/20250920011357701.webp)
可以看到，基本上中国大陆地区全境有阻断和访问失败，IPv6环境下稍微好一点（但也没好哪去）。