---
title: 通过 certbot 给网站部署 Let’s Encrypt SSL 安全证书
date: 2019-04-01 12:12:28
tags:
---

## http 不安全

当部署完网站，你迫不及待打开 chrome，输入网站域名，敲下回撤，页面便展现在你的眼前。这时候可能会注意到域名的左边，赫然显示着“不安全”。这太难看了，没法忍。

chrome 显示不安全的其中一个原因就是网站没有配置安全证书，使用的是 HTTP 而不是 HTTPS。

http 是一个传输网页内容的协议，本身不带加密，是明文传输的。而 https 可以理解为“ HTTP over SSL/TLS ”，这是为了安全，为 http 协议上加了一层 SSL/TLS 安全协议。

## SSL/TLS 是什么？

SSL（ Secure Sockets Layer） 和 TLS（Transport Layer Security） 是同一个东西的不同阶段，可以理解为一个东西，都是安全协议。

Secure Sockets Layer 翻译为“安全套接层”，所以 HTTP over SSL/TLS ” 就是带“安全套接层”的 http 协议”，既然带上了“安全套”，那肯定是安全得多了。

## 如何部署 https ？

部署 https 不仅仅是为了安全，各大互联网企业和一些相关的基金会也在推，可以给一个网站部署 https 几乎是必须的。那么要怎么部署呢？

你只需要有一张被信任的 CA （ Certificate Authority ）也就是证书授权中心颁发的 SSL 安全证书，并且将它部署到你的网站服务器上。一旦部署成功后，当用户访问你的网站时，浏览器会在显示的网址前加一把小绿锁，表明这个网站是安全的，当然同时你也会看到网址前的前缀变成了 https ，不再是 http 了。

以前比如 Godaddy 、 GlobalSign 等机构签发的证书一般都很贵，为了推进 https 的普及，EEF 电子前哨基金会、 Mozilla 基金会和美国密歇根大学成立了一个公益组织叫 ISRG （ Internet Security Research Group ），这个组织从 2015 年开始推出了 Let’s Encrypt 免费证书

而我的个人网站也是利用了 Let’s Encrypt 提供的免费证书部署 https。

## 通过 Certbot 部署 Let’s Encrypt 证书

Certbot 是 Let’s Encrypt 发布的官方客户端。利用它可以完全自动化的获取、部署和更新安全证书。

以下步骤是根据[官方网站](https://certbot.eff.org/)和网上的一些教程实际操作的。

我的服务器系统是 CentOS7，web 服务器是 nginx，个人网站域名是 [vdorchan.com](vdorchan.com)，我将为 [vdorchan.com](vdorchan.com) 和所有子域名 *.vdorchan.com 开启 HTTPS 功能（在 2018 年 3 月 14 日，Let's Encrypt 的执行董事 Josh Aas对外宣布，他们的通配符证书正式上线，用户可以基于此特性轻松部署 / 开启所有子域名的 HTTPS 功能。）。

### 1、获取客户端

```bash
cd /tmp
$ wget https://dl.eff.org/certbot-auto

# 给所有用户（a）添加执行权限（x）
$ chmod a+x ./certbot-auto

$ ./certbot-auto --help
```

### 2、确认版本
客户端需要支持 ACME v2 版本，才可以申请通配符证书，官方介绍 Certbot 0.22.0 版本支持新的协议版本，所以首先确认下版本号

```bash
$ ./certbot-auto --version
certbot 0.32.0
```

### 3、证书部署

客户在申请 Let's Encrypt 证书的时候，需要校验域名的所有权，我选择 dns-01 方式（。

dns-01 方式通过给域名添加一个 DNS TXT 记录来验证，申请通配符证书，只能用这种方式。

```bash
./certbot-auto certonly  -d *.vdorchan.com  -d vdorchan.com --manual --preferred-challenges dns
```

两次确认后，你可以看到类似下面的输出

```bash
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.vdorchan.com with the following value:

nCBuVBZsIK6fz4GDGA4wMOB1ZCw4jutwbKZsC0CrkXA

Before continuing, verify the record is deployed.
```

这里是在要求配置 DNS TXT 记录，从而校验域名所有权，也就是判断证书申请者是否有域名的所有权。

上面的输出要求你在 _acme-challenge.vdorchan.com 配置一条 TXT 记录，等记录生效之后，才能继续。

我使用的域名服务器是 GoDaddy，配置如下图：

然后本地可以使用 dig 命令来确认记录是否生效

```bash
dig  -t txt  _acme-challenge.vdorchan.com @8.8.8.8
```

确认包含了类似下面所示的内容

```bash
;; ANSWER SECTION:
_acme-challenge.vdorchan.com. 599 IN  TXT "nCBuVBZsIK6fz4GDGA4wMOB1ZCw4jutwbKZsC0CrkXA"
```

确认生效后，回车执行

执行成功后，证书被保存到了下列的目录

```bash
$ ls /etc/letsencrypt/live/vdorchan.com
cert.pem  chain.pem  fullchain.pem  privkey.pem  README
```

### 4、校验证书信息

```bash
openssl x509 -in  /etc/letsencrypt/archive/vdorchan.com/cert1.pem -noout -text

# 关键输出可以看到* .vdorchan.com 和 vdorchan.com
Authority Information Access:
    OCSP - URI:http://ocsp.int-x3.letsencrypt.org
    CA Issuers - URI:http://cert.int-x3.letsencrypt.org/

X509v3 Subject Alternative Name:
    DNS:*.vdorchan.com, DNS:vdorchan.com
```

### 5、配置 nginx

```bash
$ vi /etc/nginx/nginx.conf

server {
    listen 443 ssl;
    server_name vdorchan.com www.vdorchan.com;

    ssl_certificate /etc/letsencrypt/live/vdorchan.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/vdorchan.com/privkey.pem;
}

# 配置重定向跳转，http 自动跳转 https

server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  vdorchan.com www.vdorchan.com;
    return 301   https://$server_name$request_uri;
}
```

### 重载 nginx，证书生效

```bash
systemctl reload nginx
```

## 关于 certbot-auto 和 certbot

certbot-auto 和 certbot 本质上是完全一样的；不同之处在于运行 certbot-auto 会自动安装它自己所需要的一些依赖，并且自动更新客户端工具。因此在你使用 certbot-auto 情况下，只需运行在当前目录执行即可。

```bash
./certbot-auto
```