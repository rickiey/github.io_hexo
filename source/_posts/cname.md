---
title: hexo的404问题
date: 2017-12-24 23:13:14
tags: [hexo]
categories: hexo
---
## **hexo部署访问域名404问题**
+ 问题：hexo部署到github,github.io已绑定域名，访问出现404错误
+ 原因：CNAME文件缺失

<!-- more -->

3. github.io绑定域名后访问XXX.github.io会自动跳到域名，然而CNAME文件缺失导致域名解析失败，出现404
4. 解决：我直接创建了一个CNAME文件（我那个火狐居然不能创建，换360浏览器才创建的，不知道为啥），可以正常访问，但是hexo每次部署都将导致CNAME文件缺失，于是，可以在hexo工程目录下的source文件夹下创建CNAME文件，这样就可以部署时连CNAME文件一起部署
5. 这个问题官网居然没提到
6. 还有域名已经解析至XXX.github.io后就不要在github.io的setting里绑定域名了，这样的话XXX.githu.io这个域名和个人域名都可以用，如果绑定了域名访问XXX.githu.io会自动跳转到域名，域名出问题（域名解析出问题或域名到期）就不能访问了