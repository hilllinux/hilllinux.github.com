---
layout: post
title: "利用 ngx_openresty 打造简易的 WAF (一) "
description: "如何获取真实客户端IP"
keywords: "ngx_openresty, 配置, 使用"
category: Linux,openresty,nginx
tags: [ngx_openresty, redis, LNMP]
---

#### WAF 和 ngx_openresty
WAF (Web Application Firewall) 网站应用防火墙。
对 GET/POST/COOKIE 的内容进行正则匹配，阻止常见的URL SQL注入攻击，屏蔽常见的扫描软件的UA，一定程度上增加网站被黑客黑掉的难度。

nginx 对客户端请求处理有多个处理阶段，可以利用 ngx_openresty 在 access
阶段 Lua 代码的灵活性，处理多种可能的威胁。


<!-- more -->
