---
title: 웹 서버 구축하기 (1) nginx, mysql, node 설치 및 기본 설정하기
date: 2020-06-01 12:21:13
category: 'web-server'
thumbnail: './images/mac.jpg'
draft: false
---
![](./images/mac.jpg)

---
Nginx + Nodejs + Mysql 스택은 웹 서버 구축은 가장 쉬운 방법 중에 하나입니다.  
준비물은 리눅스 기반의 OS입니다.  
Amazon Linux2에 구축하였으며 명령어만 따라치면 서버가 구축될 것입니다.  

### 웹 서버 필수 설정
---
```bash
# yum 패키지를 업데이트 합니다.
sudo yum update

## ERROR : trying other mirroring... 등 yum install 안될 경우
sudo yum clean all

# 형상 관리 도구 설치 (svn 혹은 git)
sum yum install svn

# 사용할 웹 서버와 node, DB 포트 오픈 (80, 443:nginx, 3000:node, 3306:mysql)
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp

# reload 후 포트 확인
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

### Nginx 설치
---
```bash
# yum으로 설치 
sudo yum install -y nginx

## ERROR : yum 저장소에 nginx가 없으면
vi /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/운영체제이름/운영체제버전/$basearch/
gpgcheck=0
enabled=1

# 시작프로그램으로 등록하고 서비스를 시작한다.
sudo systemctl start nginx
sudo systemctl enable nginx

## ERROR : Job for nginx.service failed because the control process exited with error code. 
## See "systemctl status nginx.service" and "journalctl -xe" for details.
## 아래의 명령어로 트래킹
sudo journalctl -xe
```

### Mysql 설치
---
```bash
# default로 설치되는 maria db 제거
rpm -qa | grep maria*
sudo yum remove "패키지명" -y

# json을 제공하는 5.7 버전 이상으로 설치
sudo yum -y install http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo yum -y install mysql-community-server

# 시작프로그램으로 등록하고 서비스를 시작한다.
sudo systemctl start mysqld
sudo systemctl enable mysqld

## ERROR : Access denied for user 'root'@'localhost' (using password: NO)
# 계정 비밀번호 확인 후 접속
sudo cat /var/log/mysqld.log
sudo mysql -u root -p
비밀 번호 입력

# 접속 후 복잡한 패스워드 정책 무시 및 새로운 루트 계정 암호 설정
SET GLOBAL validate_password_policy=LOW;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '설정할 비밀번호';

# 외부 클라이언트에서 접속하기 위한 권한 부여
GRANT ALL PRIVILEGES ON *.* TO '아이디'@'%' IDENTIFIED BY '패스워드';
```

### Node 설치
---
```bash
# 2020년 4월 24일 현재 stable한 12 버전 이상 설치, npm은 node 설치 시 설치됨.
curl --silent --location https://rpm.nodesource.com/setup_12.x | sudo bash -
sudo yum -y install nodejs

# 버전 확인
npm -v, node -v

# express 프레임 워크 및 프로세스 매니저 pm2 설치
sudo npm install -g express 
sudo npm install -g pm2
```

### Nginx와 Nodejs 연동
---
```bash
# nginx 설정 편집 루트 설정은 nginx 버전마다 상이 1.16인 내 버전 기준
sudo vi /ect/nginx/nginx.conf
# 3000 포트에 serving하는 node 프로젝트 프록시 패스에 추가
server {
	...
	location / {
		proxy_set_header X-Real-IP $remote_addr; 
		proxy_set_header X_Forwarded_For $proxy_add_x_forwarded_for; 
		proxy_set_header Host $http_host; 
		proxy_set_header X_NginX_Proxy true; 
		proxy_pass http://127.0.0.1:3000/;		
	}
}
```

### 기타 설정 및 알쓸신잡
---
```bash
# sudo 치기 싫으면 root shell로 전환
sudo su 

# 내 계정 환경 변수까지 넘겨 root shell로 전환
sudo -s

# TCP 리스닝 상태의 포트와 프로그램 출력 포트 사용 여부 확인
netstat -ntlp

# 특정 포트 바인딩 여부 확인
netstat -an|grep 3000

# 패키지 설치 여부 확인
rpm -qa | grep 패키지명*
 
```

## 참고 자료
---
[https://www.lesstif.com/system-admin/rhel-centos-7-firewalld-22053128.html](https://www.lesstif.com/system-admin/rhel-centos-7-firewalld-22053128.html)

[https://galid1.tistory.com/296](https://galid1.tistory.com/296)

[https://zetawiki.com/wiki/MySQL_원격_접속_허용](https://zetawiki.com/wiki/MySQL_%EC%9B%90%EA%B2%A9_%EC%A0%91%EC%86%8D_%ED%97%88%EC%9A%A9)