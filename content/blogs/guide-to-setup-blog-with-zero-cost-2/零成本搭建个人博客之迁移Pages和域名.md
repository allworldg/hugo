---
title: "零成本搭建个人博客之迁移Pages和域名"
date: 2022-05-27T10:45:55+08:00
draft: true
---

在上文[零成本搭建个人博客之搭建篇](https://allworldg.xyz/blogs/%E9%9B%B6%E6%88%90%E6%9C%AC%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E7%AF%87/)中，我将Hugo博客搭建到 Github Page上，后来考虑到用 Cloudflare cdn加速，干脆就把博客站点迁移到Cloudflare Page上，方便管理。
## 迁移至Cloudflare Page
Cloudflare Page支持直接从Github仓库拉取文件并且自动部署，无需额外创建github actions，同时自带cdn加速以及二级域名，整体配置比较简单。
1. 创建一个[CloudFlare](https://www.cloudflare.com/)账号
2. 创建page，允许cloudflare访问github账号上存放完整博客代码仓库。![](content/blogs/guide-to-setup-blog-with-zero-cost-2/images/Pasted%20image%2020220527113307.png)
	   Framework preset 选择 Hugo，Environment variables 自行设置成自己仓库中hugo的版本（该Page默认hugo框架版本太低，不设置无法自动构建）。其余默认即可。![](content/blogs/guide-to-setup-blog-with-zero-cost-2/images/Pasted%20image%2020220527113916.png)
3. 构建成功后，page免费送了一个域名，通过域名就可以直接访问站点了。为了让静态资源正常显示，不要忘记修改博客项目中config.toml配置文件， `baseURL = "https://hugo-c1e.pages.dev/"`![](content/blogs/guide-to-setup-blog-with-zero-cost-2/images/Pasted%20image%2020220527114229.png)

## 域名设置
尽管已经自带域名，不过我还是想要一个稍微有个人标识的域名，所以需要额外购买一个。购买的费用一般可以用网站广告收入来抵消 。域名厂商有很多，国内有