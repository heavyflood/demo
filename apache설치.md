# 1. Apache 설치 - CentOS 7

## 1) Apache 설치 확인

    yum list installed | grep httpd

## 2) Apache 설치

    yum install -y httpd

## 3) 주요 디렉토리 설명

-   conf : 웹 서버의 주요 설정 파일인 httpd.conf, MIME 형식을 지정하기 위한 파일인 magic 파일이 있는 곳
-   conf.d : 아파치의 주요설정을 분리 해서 저장 하는 곳, httpd.conf 설정내용을 분리하여 이곳에 저장하면, httpd.conf 파일에서 불러와서 사용하게 됩니다. httpd.conf 파일 맨 마지막에 ‘IncludeOptional conf.d/*.conf’ 구문이 있습니다.
-   logs : 로그파일이 저장 되는 디렉토리
-   modules : 아파치 모듈 설치디렉토리

## 4) 서비스 활성화 및 시작

    systemctl enable httpd
    systemctl start httpd

#  2. Web과 WAS의 분리

## >> apache mod_Proxy 사용

    vim /etc/httpd/conf/httpd.conf
    
    <VirtualHost *:80>
      DocumentRoot "/usr/share/tomcat/webapps/demo"
      ServerName 106.10.33.28
      ProxyPass /img !
      ProxyPass /js !
      ProxyPass / http://106.10.33.28:8080/
      ProxyPassReverse / http://106.10.33.28:8080/
    </VirtualHost>
    
    <Directory "/usr/share/tomcat/webapps/demo/">
      Options FollowSymLinks
      AllowOverride None
      Require all granted
    </Directory>
