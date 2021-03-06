---
layout: post
title: Nginx配置文件相关
category: [Linux,Nginx]
comments: false
---

* content
{:toc}

#### 安装

```
https://aiylqy.com/archives/112.html
http://nginx.org/en/download.html
http://nginx.org/en/linux_packages.html#stable
自动安装(CentOS推荐)：
http://nginx.org/en/linux_packages.html#RHEL-CentOS

(node-nginx ->  hiproxy -> https://github.com/hiproxy/hiproxy/blob/master/README-zh.md)

wget http://nginx.org/download/nginx-1.14.0.tar.gz
wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz

cd /data/nginx-1.14.0/src

./configure --prefix=/data/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/data/src/pcre-8.41 --with-openssl=/data/src/openssl-1.1.0g --with-http_gzip_static_module

make & make install

ln -s /data/nginx/sbin/nginx /usr/local/bin/nginx

nginx -s stop or reload
nginx

开机自启动（centos7）
> systemctl enable nginx.service
> systemctl status nginx
```

#### centos yum平滑升级nginx

```
手动升级的时候根据V可查看configure arguments
> nginx -V 

> lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.6.1810 (Core) 
Release:	7.6.1810
Codename:	Core

> vim /etc/yum.repos.d/nginx.repo (OSRELEASE即为lsb_release的Release变量7)
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/OSRELEASE/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

查找最新稳定版(@nginx-stable表示在用的,nginx-stable表示最新版)
> yum list |grep nginx.x86_64
or
> yum list |grep nginx
如果出现提示Repodata is over 2 weeks old.则更新下服务包到本地缓存
> yum makecache fast

更新
> yum update nginx -y

查看版本
> nginx -v

测试配置文件是否正常
> nginx -t

重启
> nginx -s stop
> nginx
```


#### /alidata/server/nginx/conf/proxy.conf
```conf
proxy_redirect          off;
proxy_set_header        Host $host;
proxy_set_header        X-Real-IP $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   300;
proxy_send_timeout      300;
proxy_read_timeout      300;
proxy_buffer_size       4k;
proxy_buffers           4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```

#### /alidata/server/nginx/conf/nginx.conf
```conf
user  www www;
worker_processes  2;

error_log  /alidata/log/nginx/error.log crit;
pid        /alidata/server/nginx/logs/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process. 
worker_rlimit_nofile 65535;

events 
{
  use epoll;
  worker_connections 65535;
}


http {
	include       mime.types;
	default_type  application/octet-stream;

	#charset  gb2312;

	server_names_hash_bucket_size 128;
	client_header_buffer_size 32k;
	large_client_header_buffers 4 32k;
	#最大上传8M
	client_max_body_size 8m;

	sendfile on;
	tcp_nopush     on;

	keepalive_timeout 60;

	tcp_nodelay on;

	fastcgi_connect_timeout 300;
	fastcgi_send_timeout 300;
	fastcgi_read_timeout 300;
	fastcgi_buffer_size 64k;
	fastcgi_buffers 4 64k;
	fastcgi_busy_buffers_size 128k;
	fastcgi_temp_file_write_size 128k;

	gzip on;
	gzip_min_length  1k;
	gzip_buffers     4 16k;
	gzip_http_version 1.0;
	gzip_comp_level 2;
	gzip_types       text/plain application/x-javascript text/css application/xml;
	gzip_vary on;
	#limit_zone  crawler  $binary_remote_addr  10m;
	log_format '$remote_addr - $remote_user [$time_local] "$request" '
	              '$status $body_bytes_sent "$http_referer" '
	              '"$http_user_agent" "$http_x_forwarded_for"';
	include /alidata/server/nginx/conf/vhosts/*.conf;
}
```

#### /alidata/server/nginx/conf/vhosts/hello.com.conf
```conf

# 禁止通过ip访问
server {
  listen 80 default;
  server_name _;
  return 403;
}

# 通用tomcat/jsp
server {
    listen       80;
    server_name xx.com www.xx.com;
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location ~ \.(gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
  		root /home/xx/apache-tomcat-8.5.33/webapps/ROOT;
  		if_modified_since before;
        etag off;
        access_log off;
        expires 1d;
    }
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
        access_log off;
    }
}
# hello
server
{
	listen 80;
	server_name www.hello.com hello.com;
	index index.html index.php;
	rewrite ^(.*) http://www.hello.com$1 redirect;
}
server {
    listen       80;
    server_name hello.com;
    location / {
		index index.jsp index.html index.htm;
		proxy_pass http://127.0.0.1:8080;
		include proxy.conf;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location ~ \.(html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
  		root /home/tomcat-hello/webapps/ROOT;
  		access_log off;
      	expires 1d;
    }
    location /vue-h5-app1 {
       add_header Cache-Control "no-cache, no-store";  
       alias /home/tomcat-hello/webapps/ROOT/vue-h5-app1/;
       try_files $uri $uri/ /vue-h5-app1/index.html;
       index index.html index.htm;
       access_log off;
       expires 1d;
    }
    location /vue-h5-app2 {
       add_header Cache-Control "no-cache, no-store";  
       alias /home/tomcat-hello/webapps/ROOT/vue-h5-app2/;
       try_files $uri $uri/ /vue-h5-app2/index.html;
       index index.html index.htm;
       access_log off;
       expires 1d;
    }
    location /m-service/ {
        proxy_pass http://127.0.0.1:8080/backend-service/;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
	    access_log off;
    }
}
server {
    listen       443; 
    server_name  hello.com;  
    ssl                  on;  
    ssl_certificate      sslkey/hello.com_bundle.crt;
    ssl_certificate_key      sslkey/hello.com.key;
    ssl_session_timeout  5m;  
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;  
    ssl_ciphers ALL:!ADH:!EXPORT56:-RC4+RSA:+HIGH:+MEDIUM:!EXP; 
    ssl_prefer_server_ciphers   on;  
    location / {
		index index.jsp index.html index.htm;
		proxy_pass https://127.0.0.1:8443;
		include proxy.conf;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location ~ \.(html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
  		root /home/tomcat-hello/webapps/ROOT;
  		access_log off;
      	expires 1d;
    }
}
```

#### /alidata/server/nginx/conf/sslkey目录存放安全认证文件
```
hello.com.key
hello.com_bundle.crt
```

#### 用户密码认证页面 location /auth 
```
location /auth {
	auth_basic "secret";
	auth_basic_user_file /data/nginx/conf/passwd.db;
	proxy_pass http://127.0.0.1:8081;
	proxy_set_header   Host             $host;
	proxy_set_header   X-Real-IP        $remote_addr;
	proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
}
passwd.db的生成：
wget http://trac.edgewall.org/export/10770/trunk/contrib/htpasswd.py
chmod 700 htpasswd.py
./htpasswd.py -c -b passwd.db root 123456
```


