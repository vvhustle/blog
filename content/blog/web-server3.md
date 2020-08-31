---
title: 웹 서버 구축하기 (3) Let's Encrypt로 https 인증 받기
date: 2020-06-03 12:21:13
category: 'web-server'
thumbnail: './images/mac.jpg'
draft: false
---
![](./images/mac.jpg)

<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화되어 있습니다.  
    </span>
</div>

---
이전 글에서 웹 서버의 로직을 만들었다.  
이번에는 https 무료 인증 업체 Let's Encrypt을 통해 OpenSSL 인증서를 발급 받겠다.  

## 도메인 설정

SSL을 발급 받기 위해서는 도메인이 필수적으로 필요하다.  
가비아에서 가장 싼 도메인(1년 2000원)을 발급 받도록 한다.  
최초 1년은 낮은 가격이지만 그 이후부터는 10~20배가 넘는 가격이 책정되니 참고하자.  

## 도메인과 내 서버 연결하기

가비아에서 받은 도메인을 네이버 클라우드 플랫폼에서 받은 서버에서 연결하는 과정을 진행한다.  
발급 받은 도메인 업체에의 관리툴에서 DNS 설정 부분에서 클라우드 서버의 public-IP을 입력해주자.  

## SSL(HTTPS) 설정

Let's Encrypt의 소스 다운로드한다.  
```bash
git clone https://github.com/letsencrypt/letsencrypt
```  
의존 파일 설치한다.    
```bash
./letsencrypt-auto --help
```  
플러그인을 사용해도 되지만, 수동으로 인증서를 발급 받는다.  
```bash
./letsencrypt-auto certonly --manual
```  
인증이 완료되면 /etc/letsencrypt/live/ 위치에 4개의 key 파일이 생성된다.  

## nginx + SSL

SSL을 express 서버에 직접 붙일 수 도 있지만, 이왕 사용하게 된 nginx에 설치한다.  
[여기](https://mozilla.github.io/server-side-tls/ssl-config-generator/)에 설정을 명시하면 문서를 만들어 준다.    
config 파일에 ssl과 관련된 정보를 업데이트한다.  
아래의 key 파일들을 
- ssl_certificate
- ssl_certificate_key 
- ssl_truested_certificate
- ssl_dhparam  

```bash
/etc/letsencrypt/live/
```
의 key 파일로 넣어 준다.  

etc/nginx/conf.d 아래의 default.conf의 파일 내용을 복사하여 a~c로 시작하는 파일명으로 만들어주면 먼저 로드한다.  
따라서 a~c로 시작하는 파일명으로 config를 만들어 먼저 로드되게 하자.  

## OpenSSL 인증서 갱신

무료 인증서 LETSENCRYPT의 인증 기간은 90일이다.  
서버로 접속해 letsencrypt 옵션을 켜줘야한다.  
```bash
./letsencrypt-auto --help
```  
위의 명령어의 결과 성공적으로 수행되었다면  
```
bash
/etc/letsencrypt/live/프로젝트이름/
```
의 경로에 새로운 인증서 파일 및 심볼릭 링크가 추가 된 것을 확인할 수 있다. 
만약 자동적으로 심볼릭 링크가 생성되지 않는다면  letsencrypt 폴더로 이동하셔서 
```bash
./certbot-auto renew
``` 
명령어로 갱신하면 된다.  

## 참고 자료

[SSH 포트](https://se34.tistory.com/47)  
[Let's encrypt](https://blog.outsider.ne.kr/1178)  