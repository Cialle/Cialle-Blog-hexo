---
title: 论如何配合宝塔与git搭建个人Hexo个人博客
date: 2022-06-03 21:28:00
author: Cialle
categories:
  - 其他
tags:
  - hexo
  - git
  - 宝塔
---

## 介绍

本文讲述如何使用宝塔来搭建Hexo个人博客，希望对你有帮助

文章注意事项
[username] 代表你自定义的用户名
如果提示权限不足 请将命令前加sudo
例如 `mkdir /home/git/`  改为 `sudo mkdir /home/git/`


## 本地前置安装

- [git下载](https://git-scm.com/downloads)
- [node.js下载](https://nodejs.org/zh-cn/)

不过多介绍windows安装方式，git别忘记设置邮箱以及昵称。
cnpm[可选] 安装后所有npm命令可用cnpm替代（如果后续npm太慢可用考虑使用）：
命令行输入：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 本地安装hexo

命令行输入：

```bash
npm install -g hexo-cli
```

## 服务器安装宝塔

[宝塔面板下载(bt.cn)](https://www.bt.cn/new/download.html)

等待出现![image-20220601202526678](https://img.cialle.com/image-20220601202526678.png)
则即可按照外网面板地址进入，输入账号密码，绑定宝塔账号。
然后安装宝塔自带套件（懒狗必备）
![image-20220601202614762](https://img.cialle.com/image-20220601202614762.png)
这将是一个漫长的过程，在此期间可先进行下一步


## 服务器安装git

查看git版本
ssh输入

```bash
git --version
```

如果提示
`git version xx.xx.xx`
则表示git安装已安装成功，无需重新安装git。
如果报错，则需要进入进行git官网[Linux/Unix下载](https://git-scm.com/download/linux)

## 服务器git仓库配置

创建服务器新用户密码 

```bash
sudo adduser [username]
sudo passwd [username]
```

分配权限

```bash
usermod [username] -G wheel
```

如果ubuntu没有wheel组则添加至adm 或 admin组:

```bash
usermod [username] -G adm
或者
usermod [username] -G admin
```

在本机电脑终端输入：

```bash
ssh-keygen -t rsa -C 你的@邮箱.com
```

然后一直回车，新建一个密钥。
再你创建的时候命令行会提示你密钥文件的路径
复制id_rsa.pub文件中的内容备用。



回到服务器终端，切换用户

```bash
su - [username]
```

创建.ssh文件夹

```bash
mkdir .ssh
```

新建并编辑authorized_keys 将刚刚id_rsa.pub公钥中的内容，复制粘贴到文件里，保存退出。

```bash
vim .ssh/authorized_keys
```



创建git目录，修改所有权和用户权限。并建立git仓库修改权限

```bash
mkdir /home/git/
chown -R [username]:[username] /home/git/
chmod -R 755 /home/git/
cd /home/git/
git init --bare Blog.git
chown [username]:[username] -R Blog.git
```

新建钩子文件 post-receive

```bash
vim /home/git/Blog.git/hooks/post-receive
```

进入文本编辑器，粘贴下面两行。

```bash
#!/bin/bash
git --work-tree=/home/hexo --git-dir=/home/git/Blog.git checkout -f
```

保存退出。然后修改文件权限

```bash
chmod +x /home/git/Blog.git/hooks/post-receive
```

创建blog目录

```bash
mkdir /home/hexo/
chown -R [username]:[username] /home/hexo/
chmod -R 755 /home/hexo/
```



## 宝塔设置

如果你看到这里，并且一点点跟着操作，那么上边的宝塔应该也是安装完毕了
之后就很简单了，直接网站-添加站点

![image-20220602001600302](https://img.cialle.com/image-20220602001600302.png)

域名位置填写你的域名。
根目录填写/home/hexo或你自己定义的位置

## 如何使用

到目前为止，已经全部架设完毕了。

只需要在本地

```
hexo clean
hexo g
hexo d
```

就可以在网站直接访问到了。

**关于hexo使用方法，请参考网络其他教程~这里只讨论如何让你的网站跑起来。**
