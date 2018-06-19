---
title: gitlab 维护命令
date: 2018-06-13 13:39:23
categories: 
	- gitlab
tags:
	- linux
	- centos6.5
	- gitlab
	- git
---


### 不用gitlab自带的nginx
```
vim /etc/gitlab/gitlab.rb
nginx['enable'] = false
```

### 设置服务端口域名

```
external_url 'http://xxx.com'
unicorn['port'] = 8800

```

### 更改配置文件后，重新设置生效
```
gitlab-ctl reconfigure

```

### 启动，重启，停止,查看状态，查看日志,查看某组件日志
```
gitlab-ctl start
gitlab-ctl restart
gitlab-ctl stop
gitlab-ctl stop
gitlab-ctl tail 
gitlab-ctl tail  unicorn
```
### gitlab无法push或clone的错误:JWT::DecodeError (Nil JSON web token): lib/gitlab/workhorse.rb:120:in `verify_

问题出在反代的配置上:nginx或者apache的反代应该反代到 http://gitlab-workhorse; 而不应该反代到http://127.0.0.1:8080

修改

```
vim /etc/nginx/nginx.conf  

user git;
```
添加gitlab配置

```
vim /etc/nginx/conf.d/gitlab.conf

upstream gitlab-workhorse {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket;
}
# HTTPS server

        #proxy_cache_path  /data/nginx/cache  levels=1:2    keys_zone=STATIC:10 inactive=24h  max_size=1g;
server {
        listen       80  default_server;
        #server_name  git.okycz.com;
        #ssl                  on;
        #ssl_certificate
        #ssl_certificate_key

        #ssl_session_cache    shared:SSL:1m;
        #ssl_session_timeout  5m;

        #ssl_ciphers  HIGH:!aNULL:!MD5;
        #ssl_prefer_server_ciphers  on;

        proxy_read_timeout 1800;
        proxy_send_timeout 1800;
        #proxy_connect_timeout 600;
        #send_timeout  600;
        #keepalive_timeout 600s;

        location / {
            proxy_pass http://gitlab-workhorse/;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Via "nginx";
            #proxy_cache STATIC;
            #proxy_cache_valid  200 1d;
            #proxy_cache_use_stale  error timeout invalid_header updating http_500 http_502 http_503 http_504;
        }

        location ~ ^/(assets)/ {
                root /opt/gitlab/embedded/service/gitlab-rails/public;
                gzip_static on; # to serve pre-gzipped version
                expires max;
                add_header Cache-Control public;
         }

        #rewrite ^(.*)$  https://$host$1 permanent;
}

```


