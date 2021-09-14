---
title: Centos7使用Nginx伪装V2Ray代理服务
date: 2021-09-04 17:04:31.258
updated: 2021-09-05 16:19:30.539
url: /archives/centos7使用nginx伪装v2ray代理服务
categories: 软件
tags: nginx | v2ray
---

1.官方脚本安装v2ray服务
```bash
// 安装可执行文件和 .dat 数据文件
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
systemctl enable v2ray
systemctl status v2ray
systemctl start v2ray
```

2.安装nginx服务
```bash
yum install epel-release nginx
systemctl status nginx
systemctl enable nginx
systemctl start nginx
```

3.安装证书
```bash
sudo yum install epel-release snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
systemctl status snapd
systemctl start snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

4.配置v2ray服务
```json
{
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [{
    "port": 入站端口,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "客户端ID",
          "level": 1,
          "alterId": 68
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/伪装路径",
        "headers": {
          "Host": "外网域名"
        }
      }
    },
    "listen": "127.0.0.1"
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

5.配置nginx服务
```bash
server {
    if ($host = 外网域名) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name 外网域名;
    return 404; # managed by Certbot


}

server {
    listen       443 ssl http2;
    server_name 外网域名;
    charset utf-8;

    # ssl配置
    ssl_certificate fullchain.pem_path; # managed by Certbot
    ssl_certificate_key privkey.pem_path; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    root /usr/share/nginx/html;

    location / {
        index index.html;
    }

    location /伪装路径{
      proxy_redirect off;
      proxy_pass http://127.0.0.1:入站端口;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

6.重启服务
```bash
systemctl restart v2ray
systemctl restart nginx
```







