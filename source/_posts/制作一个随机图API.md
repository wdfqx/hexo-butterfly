---
title: 制作一个随机图API
categories:
  - 记录
tags:
  - CDN
  - EdgeOne
  - Serverless函数
  - 随机图API
  - Backblaze
abbrlink: 9e1ff682
date: 2025-11-09 00:18:18
---

## 0.引言

众所周知啊， **API（Application Programming Interface/应用程序编程接口）** 类型那叫一个千奇百怪，啥都有，我随随便便用EdgeOne Pages Functions做了个随机图API，至于你问我做它想干啥……我也不知道~~（等会，你关心这个干嘛？）~~，API端点也直接公开，反正EdgeOne的Functions免费无限用，你爱用就用（不是）

## 1.API端点

API端点域名为`randomimg.987632.xyz`，用法如下：

- 随用户User-Agent自适应图像：[https://randomimg.987632.xyz/img/ua](https://randomimg.987632.xyz/img/ua)
- 获取横屏图像用于桌面端：[https://randomimg.987632.xyz/img/deskop](https://randomimg.987632.xyz/img/deskop)
- 获取竖屏图像用于移动端：[https://randomimg.987632.xyz/img/mobile](https://randomimg.987632.xyz/img/mobile)

## 2.具体实现：

图源存放在Backblaze B2上，Backblaze免费提供10GB的存储空间和10GB的流量，空间对于随机图而言几乎用不完~~（你家webp图片能用10个G啊？）~~，但10GB如果全部用于公网流出就有些不够用了，所以我选择使用EdgeOne进行缓存，中国大陆速度尚可

图片分为横屏和竖屏两类，分别对应桌面端（`/img/deskop`）和移动端（`/img/mobile`），全部重命名为纯数字文件名，为传输优化而全部采用webp格式节省大小。

EdgeOne Pages Functions作为用户访问的入口端点（即上文提到的端点），收到请求后根据HTTP请求的Path进行处理，区分横屏、竖屏、自适应三类，在内部生成一个随机数，范围硬编码在代码里，图源我没有设置防盗链故未作处理（如果你要自己部署自定义后端端点的API请自行解决好防盗链问题），更多细节详见我发布的源码，见此处：[https://github.com/naki-im/edgeone_randomimg_api](https://github.com/naki-im/edgeone_randomimg_api)

## 3.具体代码实现

> 温馨提示：本项目所用的代码仅供参考学习使用，下载后请在24小时内删除。

此处展示的是自适应图像的处理逻辑

```js
export async function onRequest({ request }) {
  try {
    const ua = request.headers.get("user-agent") || "";
    const isMobile = /Android|iPhone|iPad|iPod|Mobile|BlackBerry|IEMobile|Opera Mini/i.test(ua);
    const isDesktop = !isMobile && /Windows NT|Macintosh|X11|Linux|Mac OS X|Ubuntu/i.test(ua); //判定用户设备类型

    let type;
    if (isDesktop) type = "h";
    else if (isMobile) type = "s";
    else type = Math.random() < 0.5 ? "s" : "h";

    const randIntInclusive = (min, max) =>
      Math.floor(Math.random() * (max - min + 1)) + min; //随机数生成模块

    const num = type === "s" ? randIntInclusive(1, 2942) : randIntInclusive(1, 833);
    const url = `https://b2.randomimg.987632.xyz/${type}/${num}.webp`; //拼接回源URL
    //如果你要自部署的话请尽量自行搭建图源

    const upstream = await fetch(url);
    const arrayBuffer = await upstream.arrayBuffer();
	
    return new Response(arrayBuffer, { 
      status: upstream.status,
      headers: {
        "Content-Type": "image/webp",
        "Cache-Control": "no-cache"
      }
    }); //向用户返回随机的图片
  } catch (err) { 
    return new Response(
      JSON.stringify({ error: "Internal Error", msg: String(err) }),
      { status: 500, headers: { "Content-Type": "application/json" } }
    ); // 故障处理逻辑
  }
}
```
