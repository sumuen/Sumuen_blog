---
title: XTLS vision_
author: Sumuen
date: '2023-04-09'
categories:
  - xlog
tags:
  - null
slug: null
---

[vision起源](https://github.com/XTLS/Xray-core/discussions/1295#discussion-4523044)

[实战](https://www.v2ray-agent.com/archives/1680104902581)

**lightsail 东京**

XTLS = tls in tls 

> XTLS 的核心逻辑在于使用真实的 TLS 将代理流量隐藏于互联网最常见的流量之中。
> 对于普通 TLS 代理协议（比如[原版 Trojan](https://trojan-gfw.github.io/trojan/)），用户的代理客户端与代理服务器建立一个真实的 TLS 连接，通过这个加密的隧道，应用程序（比如浏览器）再与目标服务器（比如 Google）建立一个 TLS 连接。这时浏览器与 Google 服务器就是端对端加密，任何人（包括代理服务）不能解密或者伪装发送信息，这就是所谓的 e2e。

## xtls-rprx-vision

> 1. 建立一条真实 TLS 链接
> 2. 在加密隧道内，跟踪浏览器与 Google 之间的通信，识别当前的通信是否为 TLS 1.3
> 3. 如不是，则使用普通 TLS 代理方法，继续使用加密隧道
> 4. 如是，则等待两端开始传输加密数据，代理客户端在第一个内层加密包中**插入 UUID，指示代理服务端：**我们准备裸奔了！
> 5. 从第二个内层加密包之后，XTLS 就不再对数据包做任何操作，只是单纯拷贝

其中

> - 代理发起的外层 TLS 是真实的，所以审查者无法破解
> - 裸奔的流量在代理方面只做单纯拷贝，如果审查者进行任何破坏，将会得到浏览器与 Google 服务器的正常 alert 响应，与一般访问无异
> - 裸奔的信号是 UUID，此信号是代理私有信息，以避免类似 [关于 23 3 3 判断部分的 代码特征的利用漏洞 Go#17](https://github.com/XTLS/Go/issues/17) 的漏洞，这个没研究

## TLS in TLS

uuid的那几个包到底在干嘛？

> 第一个包 		 代理客户端 -> 代理服务器  		"你好 本次代理访问的目标地址是 Google 这里是我的 UUID"
> 第二个包 		 代理客户端 <- 代理服务器  		"你好 收到了你的代理请求 请开始发送数据"
> 第三个包 	浏览器 -> 代理客户端 -> 代理服务器 -> Google 	"你好 Google 我将和你进行加密通话 我支持的加密方式有。。" (这个包也叫 TLS Client Hello)
> 第四个包 	浏览器 <- 代理客户端 <- 代理服务器 <- Google 	"你好 用户 这是 Google 证书 本次将使用 TLS_AES_128_GCM_SHA256 加密 让我们开始加密通话！" (这个包也叫 TLS Server Hello)
> 第五个包 	浏览器 -> 代理客户端 -> 代理服务器 -> Google 	"收到 让我们开始加密通话！"

五个关键包的特征

> 第一个包 	很短 	唯一的变数是目标地址 
> 第二个包 	很短 	几乎固定
> 第三个包 	短 	变化很少 几乎唯一的变数是目标 SNI
> 第四个包 	长 	变化较大
> 第五个包 	很短 	变化很少

特征明显  应对措施  短包填充

> 可以直观的感受到，它们的包长特征其实是十分明显的。在 **Vision** 中，我们的应对方法也十分简单，就是将每一个短包的长度填充至 900 到 1400 这么一个区间。注意，这个方法与传统的随机填充不同，我们不是盲目的在所有包都加上填充，而是基于我们对内部流量的分析，有针对性的，对 TLS 握手过程的标志性的包进行填充。

> 另一个比较隐蔽的特征叫做时序特征。受限于 TLS 握手的设计，你可以注意到，前面这几个包的顺序是固定的。也就是说，浏览器发送 TLS Client Hello 之后，必须等待 Google 发出 TLS Server Hello 这一信息，而这之后浏览器又必须发送一个"收到 让我们开始加密通话！"，如此后面的对话才可以正常进行。
> 如果把用户到服务端发一个包叫做 C，反方向一个包叫做 S。审查者是否能够根据 CSCSC 这种数据时序，它们的时间间隔特征来判定这是一个 TLS in TLS 连接？
> 我们认为现在还不足以下结论，Vision 并没有在这个方面做处理。
> 有很多开发者都提到 MUX 多路复用对抗 TLS in TLS 检测。它在这方面的确有混淆作用，假如多路复用两个不同的 TLS 连接在一个隧道里，就有机会会形成 CCSSCC 等各种不同的时序特征。

> 可以预期，翻墙的技术战争将从 **shadowsocks （加密）-> 主动探测 -> TLS -> 机器学习** 继续升级，是生存或者死亡，将很大程度上取决于这5个包的处理。

# 问答

> 问：新流控这么好，你们推荐所有人都使用它翻墙吗
> 答：正如其它开发者所言，对于当前的审查者，把所有鸡蛋都放在一个篮子里是十分危险的。所以我们鼓励大家善用每个工具的特性，分散特征。
>
> - [@klzgrad](https://github.com/klzgrad) [NaiveProxy](https://github.com/klzgrad/naiveproxy) 使用 Chrome 浏览器架构当作代理客户端以及 Caddy 当作代理服务端，可以高度模仿一个普通的 HTTPS 访问，它也包含的 TLS 握手以及其它特征包的填充
> - [@gfw-report](https://github.com/gfw-report) 仅仅花一个下午写出的[魔改 shadowsocks](https://github.com/net4people/bbs/issues/136) 通过注入额外数据的方式，改变传统 shadowsocks 无序流量的特征
> - 在纯自用的情形下，使用 MITM 代理，目前这种方式工具较少，可以参考 [https://github.com/KiriKira/Kirikira.moe/blob/master/articles-md/V2Ray%E7%9A%84%E8%BF%9B%E9%98%B6%E6%95%99%E7%A8%8B(2):MITM.md](https://github.com/KiriKira/Kirikira.moe/blob/master/articles-md/V2Ray的进阶教程(2):MITM.md)

> 对于未来你推荐国密比如 sm4 吗？
>
> 我们认为最安全的方法就是隐藏于互联网中最常见的流量之中。除了 HTTPS 以外，我们应该考虑什么样的出国流量是常见的。SSH？Remote Desktop？微信视频？Email（雾）？电波通信（大雾）？
> 而不是使用一个极其小众的加密和协议。
>
> `国密是指中国国家密码管理局（State Cryptography Administration，简称SCA）发布的密码算法标准，这些算法包括对称加密算法、非对称加密算法、数字签名算法、杂凑算法等等。国密算法在中国政府、军队、金融、电信等领域得到广泛应用。`
>
> `SM4是国密算法中的一种对称加密算法，也被称为商用密码SM4。它是一种分组密码，分组长度为128位，密钥长度可选为128位或256位，使用Feistel结构设计，加密过程包括轮函数和密钥扩展。SM4算法在中国政府和企业等领域得到广泛应用，但其在国际上的应用和认可度较低。`

![image-20230409022412349](https://easyimage.muzi.studio/i/2023/04/09/3pex4z-1.png)