---
title: '웹 서버 구축하기(feat.aws+nginx+node.js)'
date: 2020-07-04 12:21:13
category: 'server'
thumbnail: './images/hello.png'
draft: true
---

<center><img src="{{site.baseurl}}/assets/post_img/freety_main.png"></center>
<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화가 되어 있습니다.  
    </span>
</div>

## 개요
대학생 벤처 창업 연합 동아리 S.O.P.T(Shout Our Passion Together)의 20기 AppJam의 일환으로 서버 개발에 참여한 후기 입니다.  
## 프로젝트  
'Freety - 지갑없이 예뻐져요.' 어플리케이션 개발
## 주제   
헤어 디자이너와 일반인 실습 모델의 매칭 서비스 플랫폼입니다.
## 개발기간  
2017년 06월 24일 (토)~ 2017년 07월 08일 (토)
## 개발목표
클라이언트 개발자에게 최대한 RESTFul한 API를 제공합니다.  
참여한 해커톤에서 대상을 수상합니다.
## 개발스펙
node.js (ECMA Script2016)
MySQL  
AWS RDS, EC2, S3  
## 개발도구
Atom  
MySQL workbench  
FileZilla  
Git Bash  
Postman
## 소스 코드
[저장소](https://github.com/leveloper97/Server)에 있습니다.
## API
![freety_api](/assets/post_img/freety_api.PNG)
## ERD
![fretty_erd](/assets/post_img/freety_erd.PNG)
## AWS-EC2
EC2 인스턴스를 생성하였습니다.  
실 상용화 목적이 아닌 해커톤 목적이였기 때문에 프리티어로 구축하였습니다.  
키 파일(.pem)은 putty-keygen을 통해 ppk를 생성 후 팀원들과 공유하였습니다.  
프로세스 매니저로는 node.js 전용 pm2를 설치해 배포하였고 status, log 등을 활용하여 장애 발생 시 대응하였습니다.  
## Database
AWS RDB를 사용하여 인스턴스를 생성하였고 MySQL workbench를 사용해 스키마 및 데이터를 연동하였습니다.  
ORM을 구성하기에는 시간이 부족했고 그렇다고 복잡한 쿼리문을 매번 작성하는 것은 번거로운 일이었습니다.  
따라서 데이터베이스 뷰를 생성해 최대한 쿼리문을 단순하게 구현하였습니다.  
트랜잭션(커밋-롤백)은 따로 구현하지 않았고 비동기 콜백 함수를 통해 에러를 리턴하게 하였습니다.  
## config
config 폴더에서 DB의 connection pool 및 aws_config 등의 config를 일괄 관리하였습니다.  
## EC2 TimeZone
AWS의 region을 Seoul으로 설정했음에도 불구하고 moment(), date() 등에서는 UTC가 출력 되었습니다.  
```sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime``` 
명령어를 통해 해결하였습니다.  
## [JWT (Json Web Token)](https://jwt.io/)
stateless한 토큰 기반의 인증 시스템은 세션이나 DB 등 인증 정보를 담아주지 않아 서버에게 부담을 줄여줍니다.  
서버는 단지 토큰 검증의 작업만을 진행합니다. 또한, JWT의 해싱 알고리즘은 쿠키보다는 좀 더 보안을 높일 수 있습니다.
## CORS 
쿠키를 따로 사용하지 않았고 http 통신만을 제공하여 문제가 된 것은 없었습니다.  
다만, 추후 https로의 redirecting 또는 멀티 도메인에서 서버가 작동하게 하고 싶다면  
`Access-Control-Allow-Origin: *`  
를 헤더에 추가해야 합니다.
## API 구현
router의 속한 파일은 일반적으로 하나당 하나의 요청을 처리하게 구현하였습니다.  
콜백형식은 아래와 같습니다.  
1. db pool의 connection을 받는다.  
2. 쿼리문을 수행한다.  
3. connection을 release한다.  
4. HTTP status와 json 형식의 데이터를 반환한다.  

각 순서는 콜백 형식으로 수행하게 구성하였습니다.  
## 콜백 지옥
이벤트 기반의 비동기 단일 쓰레드를 사용하는 node.js 특성 상 상황에 따라 콜백 지옥에 빠질 수 밖에 없습니다.  
es6의 promise 패턴도 고려하였으나 async 모듈를 require해 series, waterfall, parallel 등을 통해 콜백 지옥을 최소화 하였다.
## 로그인 처리  
sns 로그인을 처리하기 위해 login 라우터 안에서 login/sns와 login/email로 파싱하였습니다.  
쿼리가 정상적으로 완료되면 200 status를 반환하였고 그외의 경우 503 status를 반환합니다.
## 비밀번호 
암호화 라이브러리 bcrypt-nodejs를 활용해 암호화하여 저장하였습니다.
## IAM Role
프리티어지만 EC2 용량에 한계가 있기 때문에 noti용으로 사용하습니다.  
또한 AWS 리소스에 접근 권한을 설정하여 SQL 인젝션으로 부터 조금이라도 보호하기 위해 IAM Role을 사용하였습니다.  
EC2에 권한을 주었고 RDB나 S3의 리소스에 접근 퍼미션을 줘서 access key와 secret key를 매번 설정하지 않게 하였습니다.
## multer
body-parser 미들웨어는 multi-part, form-date 등의 바이너리 데이터를 처리할 수 없습니다.  
따라서 이미지 파일 업로드의 목적으로 multer 모듈를 통해 AWS S3의 bucket과 연동 되도록 구현하였습니다.
## express
express 프레임 워크를 사용해 HTTP 통신을 파싱이나 헤더, 바디 작성 없이 간결하게 구현하였습니다.
## 필터 적용
클라이언트에서 선택한 필터들만 적용하기 위해 요청을 파싱해야 하였습니다.  
replacer와 JSON.stringify로 직렬화 후 switch문을 통해
동적 쿼리문을 작성하였고 req로 반환하였습니다.  
## [시연 영상](https://drive.google.com/file/d/0B-bbZmOdnzOzWi13VW9uUHg5dE0/view?usp=sharing)
## 후기
참가한 해커톤에서 최우수상(12팀 중 2등) 및 인기상을 수상하였습니다.  
시간이 부족해 쪽지 기능을 socket.io가 아닌 relm을 통해 구현해서 아쉬웠습니다.  
HTTPS 프로토콜을 지원하지 않았습니다. 최소한 openSSL 인증서로 보안 서버를 구축하였으면 더 좋았을 것입니다.
API 문서화를 구글 시트를 통해 하였는데 swagger를 활용하지 못해 아쉬웠습니다.  
세 명의 팀원이 있었지만 각 자 맡은 기능을 구현하느냐 코드 리뷰는 물론 git flow를 활용하지 않아 아쉬웠습니다.  
nginx를 붙이지 않았고 로드 밸런싱, 샤딩 등 대용량 요청 처리가 고려되지 않았습니다.  
쿼리문을 간결하게 짠다는 이유로 뷰를 구성했지만 실 서비스 시, 테이블의 구조 변경에 취약하다는 약점이 있습니다.  
그래도 짧은 시간 동안 눈에 보이는 결과물을 만들었고, 상도 받고 실력도 많이 향상되었습니다.  