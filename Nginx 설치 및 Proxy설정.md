# 1. Nginx 설치 - CentOS

## 1) Nginx 저장소 추가

    vim /etc/yum.repos.d/nginx.repo
    
    [nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/7/$basearch/
	gpgcheck=0
	enabled=1
	

## 2) 설치

    yum install -y nginx

## 3) Nginx 활성화 및 시작

    systemctl start nginx
    systemctl enable nginx

# 2. Nginx Proxy 설정

    vim /etc/nginx/conf.d/default.conf
    
    server {
        root /usr/share/tomcat/webapps/demo;
        listen       80;
        server_name  localhost;
    
        #charset koi8-r;
        #access_log  /var/log/nginx/host.access.log  main;
    
        location / {
            #root   /usr/share/nginx/html;
            #index  index.html index.htm;
            proxy_pass http://106.10.33.28:8080;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
        }
    
        location /img/ {
            autoindex on;
        }
    
        location /js/ {
            autoindex on;
        }
