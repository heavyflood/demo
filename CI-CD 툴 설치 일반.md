## Pre-Install

 **yum update / JDK 설치**
   
    yum -y update && upgrade
    // ==> update && upgrade centos

    yum install -y java-1.8.0-openjdk.x86_64 
    yum install java-1.8.0-openjdk-devel.x86_64
    // ==> install openjdk

## SVN 설치

**설치여부 확인**

    # svn
    -bash: svn: command not found
    # rpm -qa | grep subversion

**설치**

    # yum -y install subversion

**설정** 

    # mkdir svn_repos //최상위 폴더 생성
    # vim /etc/sysconfig/svnserve //최상위 경로 설정
      OPTIONS="-r /svn_repos"
    
    # cd svn_repos 
    
    //프로젝트 repository 생성
    # svnadmin create --fs-type fsfs /app
    # cd app/conf
    
    # vim svnserve.conf
    //아래 주석제거
    anon-access = read
    auth-access = write
    password-db = passwd
    authz-db = authz
 
    //사용자생성
    # vim passwd
    [users]
    heavyflood = 0811

    //권한 설정
    # vim authz
    [/]
    heavyflood = rw


**프로젝트 기본 폴더 생성**

    //기본 에디터 설정
    # cd ~
    # vi .bash_profile
    
    <생략>
    SVN_EDITOR=/usr/bin/vim
    export SVN_EDITOR
    
    # source .bash_profile
    
    //폴더 생성
    svn  mkdir  명령어로  trunk, tags, branches를 각각 만든다.
    저장 후 빠져나온다(:wq  입력)
    “c”를 입력한다.

    # svn mkdir svn://127.0.0.1/app/trunk

**서비스 시작**

    # service svnserve start
    #
    # ps -ef | grep svnserve | grep -v grep
    root 17869 1 0 17:22 ? 00:00:00 /usr/bin/svnserve --daemon --pid-file=/run/svnserve/svnserve.pid --threads --root /svn/repos
    #
    # netstat -anp | grep svnserve
    tcp 0 0 0.0.0.0:3690 0.0.0.0:* LISTEN 17869/svnserve


## Jenkins 설치

    #  wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
    #  rpm --import https://jenkins-ci.org/redhat-stable/jenkins-ci.org.key
    #  yum install jenkins
    
    //설치확인
    # rpm -qa | grep jenkins
    
    //포트변경 (일반적으로 9090)
    # vi /etc/sysconfig/jenkins
  
    //서비스 시작
    # systemctl enable jenkins
    # systemctl start jenkins

## Tomcat 설치

**설치**

    # yum list installed | grep tomcat
    # yum install -y tomcat*
    # systemctl enable tomcat
    # systemctl start tomcat

**root로 서비스 하기**

 **[$CATALINA_BASE]/conf/serverl.xml**

    <Host  name="localhost"  appBase="webapps" unpackWARs="true"  autoDeploy="true">

**[$CATALINA_BASE]/conf/Catalina/localhost/ROOT.xml**

    <Context  path="" docBase="/업로드폴더/blog.war" reloadable="true" crossContext="true">


## Apache 설치

**설치**

    # yum list installed | grep httpd
    # yum install -y httpd
    
    # cd /etc/httpd
    -   conf : 웹 서버의 주요 설정 파일인 httpd.conf, MIME 형식을 지정하기 위한 파일인 magic 파일이 있는 곳
    -   conf.d : 아파치의 주요설정을 분리 해서 저장 하는 곳, httpd.conf 설정내용을 분리하여 이곳에 저장하면, httpd.conf 파일에서 불러와서 사용하게 됩니다. httpd.conf 파일 맨 마지막에 ‘IncludeOptional conf.d/*.conf’ 구문이 있습니다.
    -   logs : 로그파일이 저장 되는 디렉토리
    -   modules : 아파치 모듈 설치디렉토리


    # systemctl enable httpd
    # systemctl start httpd

**Tomcat 연동 (mod_proxy)**

    # vim /etc/httpd/conf/httpd.conf
    
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
