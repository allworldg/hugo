---
title: "零成本搭建个人博客之迁移Pages和域名"
date: 2022-05-25T10:45:55+08:00
draft: false
categories : ["个人博客搭建"]
tags : ["个人博客","环境搭建"]
---

在上文[零成本搭建个人博客之搭建篇](https://allworldg.xyz/blogs/%E9%9B%B6%E6%88%90%E6%9C%AC%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E7%AF%87/)中，我将Hugo博客搭建到 Github Page上，后来考虑到用 Cloudflare cdn加速，干脆就把博客站点迁移到Cloudflare Page上，方便管理。
## 迁移至Cloudflare Page
Cloudflare Page支持直接从Github仓库拉取文件并且自动部署，无需额外创建github actions，同时自带cdn加速以及二级域名，整体配置比较简单。
1. 创建一个[CloudFlare](https://www.cloudflare.com/)账号
2. 创建page，允许cloudflare访问github账号上存放完整博客代码仓库。![](https://img.allworldg.xyz/2022/05/9c7f9966c5d7ad50858f326cfbd58315.png)
	   Framework preset 选择 Hugo，Environment variables 自行设置成自己仓库中hugo的版本（该Page默认hugo框架版本太低，不设置无法自动构建）。其余默认即可。![](https://img.allworldg.xyz/2022/05/2b902470173744a61e74feb68f3af61b.png)
3. 构建成功后，page免费送了一个域名，通过域名就可以直接访问站点了。为了让静态资源正常显示，不要忘记修改博客项目中config.toml配置文件， `baseURL = "https://hugo-c1e.pages.dev/"`![](https://img.allworldg.xyz/2022/05/3d226d806db093aaa5fc307fa1c07e0e.png)

## 域名设置
尽管已经自带域名，不过我还是想要一个稍微有个人标识的域名，所以需要额外购买一个。购买的费用一般可以用网站广告收入来抵消 。域名厂商有很多，国内有 腾讯云、阿里云。优点是在国内备案较容易，缺点是必须实名制。国外有[Godady](https://www.godaddy.com/)、[NameSilo](https://www.namesilo.com/)等。

经过简单比较，我最终选择在NameSilo上购买，好处是域名价格购买和续费相对较便宜，支持支付宝，免费的WHOIS Doamin 保护。先设置NameSilo：

1. 创建账号，购买域名，进入Manage My Domains -> Domain Console 。由于使用cloudflare加速需要把NameServer设置成cf，所以自行修改。
	```bash
	arch.ns.cloudflare.com  
	bailey.ns.cloudflare.com
	```

	![](https://img.allworldg.xyz/2022/05/ef297273cba302a59f79d44b6685035b.png)

设置CloudFlare（CF)：
1. 首先自行添加个人域名。
![](https://img.allworldg.xyz/2022/05/5899b0e43fb0220e3fe2f5850ad12331.png)
2. 然后进入Pages，绑定域名![](https://img.allworldg.xyz/2022/05/4b4a00bb92c5985a9e60925c9e3c7426.png)
3. 最后在个人域名里DNS设置中添加一个CName解析，解析到原来免费送的域名即可。![](https://img.allworldg.xyz/2022/05/42e202ce31f5a25fe81504ff06113cd1.png)



