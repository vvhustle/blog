---
title: 라인 챗봇 개발 후기
date: 2018-11-21 00:00:00 +0300
description: 
cover: /assets/post_img/line-chat.jpg
tags: chatbot, nodejs
---
<center><img src="{{site.baseurl}}/assets/post_img/line-chat.jpg"></center>
<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화가 되어 있습니다.  
    </span>
</div>

## 개요
라인 API를 이용하여 날씨 정보를 제공하는 Node.js 기반의 챗봇을 구현하였습니다.  
이 문서에는 주로 리눅스 구축에 대한 정리와 개발 과정을 포함하였습니다.  
서버 코드는 [여기](https://github.com/leveloper97/weather-bot)에 있습니다.
## 개발 배경
서울의 날씨 정보를 좀 더 쉽게 얻기 위해 개발하였습니다.  
## 개발 기간
2018년 12월 01일 ~ 2018년 12월 02일  
## 개발 목표
Nginx + SSL을 직접 구축합니다.  
Node.js + express 구성을 학습합니다.  
## 개발 스펙
Node.js  
centOS7 (NAVER CLOUD PLATFORM)
## 개발 도구
VisualStudio  
FileZilla  
Putty  
Git Bash
## 라인 계정 설정
라인 채널 및 개발자 계정 생성, 봇 등록 등은 가이드가 잘 되어 있어 어렵지 않습니다.  
다만, 웹훅을 처리하기 위한 url이 반드시 https 프로토콜을 지원해야합니다.  
[공식 문서]를(https://developers.line.biz/en/) 참고하시면 좋습니다.
## 도메인 설정
SSL을 발급 받기 위해서는 도메인이 필수적으로 필요합니다.  
가비아에서 가장 싼 도메인(1년 2000원)을 발급 받았습니다.  
도메인을 받으실 때는 최초 1년은 낮은 가격이지만 그 이후부터는 10~20배가 넘는 가격이 책정되니 참고하시면 좋을 것 같습니다.
## 서버 구축하기
NAVER CLOUD PLATFORM는 AWS EC2 구축하는 방식과 크게 다르지 않았습니다.  
다만, ACG라는 규칙이 있었고 80 포트가 default로 등록되어 있지 않아 애를 먹을 수 있으니 관련 설정을 꼭 확인하시길 바랍니다.
## 도메인과 서버 연결하기
가비아에서 받은 도메인을 네이버 클라우드 플랫폼에서 받은 서버에서 연결하는 과정을 진행해야 합니다.  
도메인의 관리툴에서 DNS 설정 부분에서 클라우드 서버의 public-IP을 입력합니다.
## firewalld 설정
centos7의 방화벽 데몬은 firewalld입니다. 또한 ipstables 명령어 기능을 firewall-cmd 명령어가 대체합니다.  
따라서 firewalld를 설치하여 사용하거나 systemctl 설정을 바꿔 iptables를 사용해야 합니다.  
~~~powershell
yum install firewalld  
systemctl start firewalld  
systemctl enable firewalld  
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https  
firewall-cmd --permanent --zone=public --add-port=80/tcp  
firewall-cmd --permanent --zone=public --add-port=443/tcp  
~~~
## 포트 포워딩 (with nginx)
포트는 일반적으로 사용되는 8080(http)와 4443(https)를 사용하였습니다.  
따라서 80, 443과의 포트 포워딩이 필요했습니다.  
Nginx을 설치하면 포트 포워딩을 쉽게 할 수 있습니다.  
1. 저장소를 추가합니다.  
```sudo vi /etc/yum.repos.d/nginx/repo```
2. 자신의 환경에 맞게 저장소 내용을 작성합니다.  
```powershell
[nginx]  
name=nginx repo  
baseurl=http://nginx.org/packages/centos/7/$basearch/  
gpgcheck=0  
enable=1
```
3. 설치를 진행합니다.  
```sudo yum install -y nginx```
4. 웹서버 포트를 방화벽에서 열어줍니다.  
```
firewall-cmd --permanent --zone=public --ad-service=http  
firewall-cmd --permanent --zone=public --ad-service=https  
firewall-cmd --reload
```
5. nginx를 실행합니다.  
~~~
systemctl start nginx  
~~~

자동 실행 옵션  
```systemctl enable nginx```  
nginx 설정 파일을 자신의 서버에 맞게 수정 해야합니다.  
```vi /etc/nginx/conf.d/default.conf```
bot.conf를 생성하여 80 포트와 웹 서버를 바인딩하였습니다.
## masquerade
따로 외부로 데이터를 보내는 일이 없다고 생각해 특별히 지정하지 않았습니다.  
만약을 사용한다면 마스커레이딩을 추가해줘야 합니다.   
```firewall-cmd --permanent --add-masquerade```
## SSL(HTTPS) 설정
무료 인증서인 Let's Encrypt를 사용하였습니다.  
소스 다운로드  
```git clone https://github.com/letsencrypt/letsencrypt```  
의존 파일 설치  
```./letsencrypt-auto --help```  
플러그인을 사용하는 방법도 있지만, 수동으로 인증서를 발급 받았습니다.  
```./letsencrypt-auto certonly --manual```  
이후에는 가이드 대로 진행하면 됩니다.  
다만, URL 인증 단계가 있는데 nginx에서 파싱하던지 express 서버에서 요청을 처리하는 방식으로 해결하면 된니다.  
인증이 완료되면 /etc/letsencrypt/live/ 위치에 4개의 key 파일이 생성됩니다.  
## nginx + SSL
SSL을 express 서버에 직접 붙일 수 도 있지만, 이왕 사용하게 된 nginx에 설치하였습니다.  
[여기](https://mozilla.github.io/server-side-tls/ssl-config-generator/)에 설정을 명시하면 문서를 만들어 줍니다.  
config 파일에 ssl과 관련된 정보를 업데이트 해야 합니다.  
아래의 key 파일들을 ```/etc/letsencrypt/live/``` 의 key 파일로 넣어 줍니다.  
- ssl_certificate
- ssl_certificate_key 
- ssl_truested_certificate
- ssl_dhparam  

## 웹 어플리케이션 서버
node와 미들웨어로 express를 사용하여 HTTP 및 HTTPS를 지원하였습니다.  
파일은 서버를 구동시키기 위한 app.js와 웹훅으로 메시지를 날려줄 reply.js로 구성하였습니다.  
외부 API 호출을 위해서 request 모듈을 사용하였습니다.  
## 외부 API
한국에도 다양한 날씨를 제공해주는 API가 존재하지만 토큰을 받는 과정이 까다롭고 시간이 오래 걸립니다.  
따라서 날씨 정보를 받아오기 위해 [외부 API](https://openweathermap.org/api)를 사용하였습니다.  
데이터 응답이 영어로 오는 단점이 있지만 기능이 크게 많지 않아 node 서버에서 직접 파싱하였습니다. 
## OpenSSL 인증서 갱신
무료 인증서 LETSENCRYPT의 인증 기간은 90일입니다. 따라서 90일 지나면 https가 적용되지 않습니다.  
인증서 갱신 방법은 간단합니다.  
서버로 접속해 letsencrypt 옵션을 켜주면 됩니다.  
```./letsencrypt-auto --help```  
위의 명령어의 결과 성공적으로 수행된다면  
```/etc/letsencrypt/live/프로젝트이름/```
의 경로에 새로운 인증서 파일 및 심볼릭 링크가 추가 된 것을 확인하실 수 있습니다. 
만약 자동적으로 심볼릭 링크가 생성되지 않는다면  letsencrypt 폴더로 이동하셔서 
```./certbot-auto renew``` 명령어로 갱신하면 됩니다.  
## 실행
라인 ID 등록 @ksj4636z 혹은  
[링크](https://line.me/R/ti/p/%40ksj4636z)에 있는 QR 코드를 통해 친구를 추가합니다.  
## 예시 화면
![line](../assets/post_img/line.jpg)
## Spadework
centos7에 node 및 npm 설치 후 npm install이 안되는 경우가 있는데 yum install openssl로 해결하였습니다.  
Let's Encrypt의 인증서 유효기간은 90일입니다.  
etc/nginx/conf.d 아래의 default.conf의 파일 내용을 복사하여 a~c로 시작하는 파일명으로 만들어주면 먼저 로드합니다.  
저 같은 경우는 bot.conf 파일을 생성하여 작성하였습니다.  
ssl_dhparam의 2048 키 생성 시, 키 디렉토리에서 생성하거나 out의 디렉토리를 설정해야 합니다.  
## 참고 자료
개발을 진행하는데 있어 아래의 웹 사이트에서 많은 도움을 받았습니다.  
[ncloud](http://docs.ncloud.com/ko/networking/networking-1-2.html)  
[ssh 포트](https://se34.tistory.com/47)  
[챗봇 구축](https://m.blog.naver.com/PostView.nhn?blogId=n_cloudplatform&logNo=221245743135&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F)  
[let's encrypt](https://blog.outsider.ne.kr/1178)  
[날씨 API](https://openweathermap.org/api) 