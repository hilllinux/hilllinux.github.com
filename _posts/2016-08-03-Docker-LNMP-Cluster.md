---
layout: post
title: "Docker LNMP 集群配置"
description: "试着用docker 打造集群方案，测试项目集群下的效果"
keywords: "Docker, Docker compose, LNMP, 集群"
category: Docker
tags: [Docker, LNMP, Nginx]
---

#### 一. 环境搭建

在Ubuntu 14.04 上搭建

```bash
sudo apt-get install apt-transport-https ca-certificates
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
lsb_release -c
vim /etc/apt/sources.list
apt-get update
apt-get install -y linux-image-extra-$(uname -r) apparmor init-system-helpers lsb-base libdevmapper1.02.1 libsystemd-journal0 docker-engine
docker run -t -i ubuntu:12.04 /bin/bash
```
<!-- more -->

编写Dockerfile

