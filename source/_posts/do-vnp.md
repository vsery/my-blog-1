---
title: 利用digital ocean轻松翻墙
date: 2017-04-03 13:53:00
categories: 工具类
tags:
  - digital ocean
  - shadowsocks
  - vpn
---
> 没错，就是教你怎么翻墙，去你妹的防火墙！去你妹的审核不通过！

# digital ocean注册账号
首先到[digital ocean](https://www.digitalocean.com/)去注册一个帐号。或者[点击这里](https://m.do.co/c/1b83177d2929)使用我的邀请链接进入官网注册帐号，这样子你就有$10的优惠卷。

注册的过程中，需要缴纳$5的“入会费”。网上很多教程几乎都会跟你说不要绑定信用卡，而使用paypal的缴费。[点击这里](https://www.paypal.com/cn/home)进入paypal的中文网。

注册账号这里就不详说了，弄得时候没想过要记录下来，就没截图什么的。而且网上也有很多教程。

# 开启主机

点击Create Droplets建立主机。这时候会看到选择主机的界面。

## 选择镜像

这个选择是看个人爱好的，我选的是ubuntu 16.04 x64。

![选择镜像](http://upload-images.jianshu.io/upload_images/1626912-f97c0fff8cfea29f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 选择套餐

因为我只是需要搭建一个shadowsocks/vpn服务，所以5刀的套餐对我来说已经足够了。

![选择套餐](http://upload-images.jianshu.io/upload_images/1626912-82dd56f4ce7a48c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 是否添加储存

选择不加存储就行了。

![是否添加储存](http://upload-images.jianshu.io/upload_images/1626912-ddfc8d657540a7db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 选择机房

听说是要根据你本地的网络的运营商来选的？详细的还是要去网上找一下答案。不过网上也有说纽约或者三藩市的会比较好，我本人选的是三藩市的机房。

![机房的选择](http://upload-images.jianshu.io/upload_images/1626912-3533d673cf1895f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 其他选项

其他的，我基本上就是默认的。如果考虑到安全什么的，可以尝试加个ssh keys?

![others](http://upload-images.jianshu.io/upload_images/1626912-824b1f0348f5d768.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 点击创建，开启主机

然后点击`create`按钮提交，就会看到主机创建成功了。开启主机，约过了一分钟左右，我的注册邮件里就收到了一封邮件：

![](http://upload-images.jianshu.io/upload_images/1626912-64e19ae0461d9399.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 设置主机

根据邮件的内容，我拿到我的主机的IP地址，用户名以及密码。接下来，就是连接到这台主机去设置vps或者shadowsocks（下文简称ss）了。我个人比较推荐的是ss。
vps是全局代理，开启了之后访问百度等国内的网站都要跑一遍三藩市，好慢...而ss就稍微智能一点。

## 连接到主机

一般来讲，使用命令行ssh到主机是比较方便的，不过digital ocean也在网页页面提供了操作界面。


### 登录

```sh
$ ssh root@xxx.xxx.xxx.xxx
```

- xxx.xxx.xxx.xxx: 主机的ip地址

输入帐号密码（邮件里边给的）。第一次登录的时候，需要我们更换密码。

## shadowsocks

### 安装ss

安装pip

```sh
$ sudo apt-get install python-pip
```

安装ss

```sh
$ pip install ss
```

如果安装过程出现一下报错（我就是...）：
```sh
➜  ~ pip install virtualenv
Traceback (most recent call last):
  File "/usr/bin/pip", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python3.4/site-packages/pip/__init__.py", line 215, in main
    locale.setlocale(locale.LC_ALL, '')
  File "/usr/lib64/python3.4/locale.py", line 592, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
```
别慌，更新一下pip就行了

```sh
$ sudo pip install --upgrade pip
```

### 配置ss
```sh
$ sudo vim /etc/ss.json//这里的json文件是自己创建的，不是系统自带
```

配置文件的内容大致如下：
```json
{
  "server":"服务器的ip",
  "server_port":"ss服务的端口",
  "local_address":"127.0.0.1",
  "local_port":1080,
  "password":"密码",
  "timeout":300,
  "method":"aes-256-cfb",
  "fast_open":false
}
```

### 启动ss客户端

前两步很简单，可是有人就纳闷了安装好了不知道怎么用，其实可以用sslocal -help 来查看帮助就知道了
 
```sh
sslocal -c /etc/ss.json
```

这样ss，就基本搭建完毕了。

### 设置开机启动

设置开机启动是为了以防我们主机人为或者其他不可描述的因素而关机/宕机了，不需要再次连上去启动。
```sh
$ sudo vi /etc/rc.local
```

编辑`rc.local`，在末尾加上：

```md
/usr/local/bin/ssserver -c /etc/ss.json -d start
```

完成！接下来就是windows下安装shadowsocks，填写代理ip地址，账户，密码，然后启动就行了。


![ss设置](http://upload-images.jianshu.io/upload_images/1626912-54a0f96665a84f34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

什么，你用mac/linux?

## vps

这个教程挺多的，这里不说了（我真的很懒）。

很多都是通过pptpd来搭建，不过如果你嫌麻烦的话，可以直接用一个[github上边写好的脚本](https://github.com/viljoviitanen/setup-simple-pptp-vpn)。运行一下脚本，就基本构建成功了。不过需要到设置文件里边修改一下端口跟密码。

```sh
# 获取远程脚本文件
$ wget https://raw.github.com/viljoviitanen/setup-simple-pptp-vpn/master/setup.sh

# 运行脚本
$ sudo sh setup.sh

# 修改配置文件
$ sudo vim /etc/ppp/chap-secrets
```

配置文件大概如下：

```
#/etc/ppp/chap-secrets
# Secrets for authentication using CHAP
# client server secret IP addresses
vpn pptpd fAj-JwC-rLu *
```

字段分别表示：

- client：vpn （用户名） 
- server：pptpd（服务器类型）
- secret：fAj-JwC-rLu（密码）      
- IP addresses：* （*代表允许所有的ip链接）

最后重启一下pptpd服务：

```sh
$ service pptpd restart
```

### 客户端的设置

这里就拿win10系统为例：

到设置里边，选择VPN > 添加VPN连接 > 填写对应的信息，就可以了。


![win10网络](http://upload-images.jianshu.io/upload_images/1626912-bfe3539e946abae9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)