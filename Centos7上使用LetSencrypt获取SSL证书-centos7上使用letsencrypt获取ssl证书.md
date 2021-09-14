---
title: Centos7上使用LetSencrypt获取SSL证书
date: 2021-09-05 16:17:50.648
updated: 2021-09-05 16:19:56.875
url: /archives/centos7上使用letsencrypt获取ssl证书
categories: 软件 | 网站
tags: ssl
---

1.安装snapd
```bash
安装snapd
$ sudo yum install epel-release
$ sudo yum install snapd
$ sudo systemctl enable --now snapd.socket
$ sudo ln -s /var/lib/snapd/snap /snap

更新至最新
$ sudo snap install core; sudo snap refresh core
```

2.安装certbot
```
确保接下来的命令是最新安装的snapd，需要先删除再安装
$ sudo yum remove certbot
$ sudo snap install --classic certbot
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

3.nginx下配置 
```bash
获取证书，并自动修改nginx配置
sudo certbot --nginx -d 域名	
或
获取证书，需要手动修改nginx配置
sudo certbot certonly --nginx -d 域名
```

4.测试自动续租
```bash
$ sudo certbot renew --dry-run
如果没有错误信息，则会自动续租证书
```

备注:
certbot会自动创建续租服务
```bash
$ systemctl --type=timer | grep snap
snap.certbot.renew.timer     loaded active waiting Timer renew for snap application certbot.renew
```





