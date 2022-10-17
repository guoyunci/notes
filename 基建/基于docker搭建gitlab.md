### 搭建

#### 环境说明
- 操作系统：centos 7.6
- docker版本：19.03.12
- ip地址：121.40.45.89

***

#### 运行

1. 拉取镜像
```shell script
docker pull gitlab/gitlab-ce
```
2. 创建文件夹
```shell script
mkdir -p /home/gitlab
```

3. 运行docker命令
```shell script
docker run -d \
  --restart=always \
  --name=gitlab \
  -p 8443:443 -p 8080:80 -p 222:22 \
  -v /home/gitlab/config:/etc/gitlab \
  -v /home/gitlab/logs:/var/log/gitlab \
  -v /home/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce
```

4. 查看日志
```shell script
docker logs -f gitlab-ce
```

***

### 配置文件

#### 修改ip(或域名)和端口

```shell script
cd /home/gitlab/config
cp gitlab.rb gitlab.rb.bak
vim gitlab.rb
```

1. external_url修改
```shell script
external_url 'http://gitlab.schoolpi.net'
```

2. gitlab_ssh_host修改
```shell script
gitlab_rails['gitlab_ssh_host'] = 'gitlab.schoolpi.net'
```

3. gitlab_shell_ssh_port修改
```shell script
gitlab_rails['gitlab_shell_ssh_port'] = 222
```

#### 修改邮箱

1. 根据实际情况修改
```shell script
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "aliyun@163.com"
gitlab_rails['smtp_password'] = "123456"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

2. 发件人地址修改
```shell script
gitlab_rails['gitlab_email_from'] = 'aliyun@163.com'
```

#### 重启gitlab
```shell script
docker exec -it gitlab gitlab-ctl reconfigure
docker restart gitlab
```

***

### nginx配置

#### http配置
1. 新增文件/usr/local/nginx/conf/vhost/gitlab.conf
```shell script
server {
 listen 80;
 server_name gitlab.schoolpi.net;
 charset utf-8;
 access_log /home/wwwlogs/gitlab.access.log;
 error_log /home/wwwlogs/gitlab.error.log;
 client_max_body_size 3072m;
 location / {
   index index.html index.htm;
   proxy_pass http://121.40.45.89:8080;
   proxy_set_header           Host $host;
   proxy_set_header           X-Real-IP $remote_addr;
   proxy_set_header           X-Forwarded-For $proxy_add_x_forwarded_for;
 }
}
```

#### https配置

1. external_url修改
```shell script
external_url 'https://gitlab.schoolpi.net'
```

2. ssl证书配置
```shell script
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.schoolpi.net.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.schoolpi.net.key"
```
> 注: .crt后缀证书, 将原先.pem后缀证书修改为.crt证书即可.

3. 重定向HTTP请求HTTPS
```shell script
nginx['redirect_http_to_https'] = true
```

4. 编辑/usr/local/nginx/conf/vhost/gitlab.conf
```shell script
upstream gitlab{
        server 127.0.0.1:8443;
}
# 转发到容器
server{
        listen 80;
        listen 443 ssl;
        server_name gitlab.schoolpi.net;
        ssl_certificate   /home/gitlab/config/ssl/gitlab.schoolpi.net.crt;
        ssl_certificate_key  /home/gitlab/config/ssl/gitlab.schoolpi.net.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        ssl_ciphers "TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
        ssl_session_cache builtin:1000 shared:SSL:10m;

        charset utf-8;

        access_log  /home/wwwlogs/gitlab.access.log;
        error_log  /home/wwwlogs/gitlab.error.log;

        location / {
                proxy_pass https://gitlab;
                proxy_set_header X_FORWARDED_PROTO https;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;
        }
}
```

