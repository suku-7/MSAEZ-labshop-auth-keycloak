## Model
## MSAEZ-labshop-auth-keycloak-20250613
www.msaez.io/#/storming/labshopoauthkeycloak-0821

Keycloak을 Docker로 직접 구축하여 OAuth2.0 기반 인증 시스템을 설정하는 과정입니다.  
이전 랩과 달리 모든 설정을 수동으로 진행하며, 사용자 인증 후 토큰 기반으로 보호된 웹 리소스에 접근하는 메커니즘을 구현하고 테스트했습니다.  
최종적으로, Gateway 서비스를 통한 Keycloak 연동 및 권한별 웹 페이지 접근 제어를 성공적으로 확인했습니다.  

![스크린샷 2025-06-13 110622](https://github.com/user-attachments/assets/724de20b-65b9-4972-ae7b-7c6aead281a1)
![스크린샷 2025-06-13 111332](https://github.com/user-attachments/assets/6f637b08-b584-41d9-9fca-3ec48321b2d7)
![스크린샷 2025-06-13 111359](https://github.com/user-attachments/assets/df6757f8-f6bb-49a0-8ab7-5f3fc60ca2ff)
![스크린샷 2025-06-13 120008](https://github.com/user-attachments/assets/0e770534-c36f-4393-b3fa-8fb68603830f)
![스크린샷 2025-06-13 120015](https://github.com/user-attachments/assets/0235286d-51aa-430b-9e14-401d44c15826)
![스크린샷 2025-06-13 120030](https://github.com/user-attachments/assets/f92f5d8a-4a0b-4378-85fd-e98dcdf78c29)
![스크린샷 2025-06-13 122645](https://github.com/user-attachments/assets/cb45caae-587c-4127-be83-824a5ecd79fb)
![스크린샷 2025-06-13 122654](https://github.com/user-attachments/assets/2c7f3a57-1b84-4753-8f74-c5758b637db0)

---
## 터미널 작성 참고용

1. 사전 준비: Java SDK 설치
프로젝트 구동에 필요한 Java Development Kit (JDK)를 설치합니다.  
```
sdk install java
```
2. Keycloak 서버 실행 (Docker Compose)
Keycloak을 Docker Compose를 이용해 실행합니다. Keycloak은 기본적으로 9090 포트로 구동됩니다.  
(본 과정에서는 카프카와 주키퍼도 함께 실행되었음을 전제합니다.)  
```
cd keycloak
docker-compose up -d
```
3. Keycloak 관리자 콘솔 접근 설정
실행된 Keycloak 서버의 9090 포트를 외부에서 접근 가능하도록 설정하고, 웹 브라우저를 통해 관리자 콘솔에 접속합니다.  
(Gitpod 환경 등에서 포트 포워딩 및 공개 설정 필요)

4. Keycloak Realm 및 클라이언트 설정
접속한 Keycloak 관리자 콘솔에서 다음 설정을 진행합니다:  
12stmall Realm 생성: 인증 및 권한 관리를 위한 새로운 Realm을 생성합니다.  
토큰 유효 기간 수정: 발급되는 토큰의 유효 기간을 1시간으로 조정합니다.  

클라이언트 설정:  
OAuth 2.0 인증 흐름을 위한 클라이언트를 설정합니다.  
리다이렉트 URI를 https://8088-YOUR_GITPOD_URL/login/oauth2/code/12stmall 형식으로 수정하고, 크리덴셜(Credential)을 활성화합니다.  

5. Gateway 서비스 설정 (application.yml 수정)
인증 게이트웨이 역할을 하는 Gateway 서비스의 application.yml 파일을 수정하여 Keycloak 서버와 연동되도록 설정합니다.  
Keycloak server-url 및 client id, client_secret, redirect url 등 OAuth2.0 관련 설정을 정확히 반영합니다.  

6. 테스트 사용자 생성
Keycloak에서 인증 테스트를 위한 두 개의 사용자를 생성합니다.  

admin 사용자  
user@naver.com 사용자  

7. Gateway 및 Order 서비스 실행
설정 변경을 완료한 Gateway (8088 포트)와 Order (8082 포트) 서비스를 실행합니다.  
(Order 서비스의 Lombok 버전을 1.18.30으로 수정해야 할 수 있습니다.)  
```
cd gateway
mvn clean spring-boot:run
# Gitpod 환경에서 Gateway 포트(8088)를 public으로 설정해야 할 수 있습니다.

# 새 터미널/세션에서 실행
cd order
# order lombok 버전 1.18.30으로 수정 (필요시)
mvn clean spring-boot:run
```
8. 웹 기반 OAuth2.0 인증 흐름 테스트
브라우저 시크릿 모드 또는 새 세션을 열고 Gateway 서비스의 /orders 엔드포인트에 접속하여 OAuth2.0 인증 흐름을 테스트합니다.
```
# 웹 브라우저 시크릿 모드에서 다음 URL 접속
https://8088-YOUR_GITPOD_URL/orders
```
8088 포트로 접근 시, Keycloak(9090 포트)의 로그인 페이지로 리다이렉트 되는 것을 확인합니다.  
user@naver.com 계정으로 로그인 시, 인증 성공 후 orders 리소스에 정상적으로 접근하여 JSON 응답을 받는 것을 확인합니다.  
이는 브라우저가 API 서버에 액세스 요청 시, 인증이 필요하면 Keycloak으로 리다이렉트 되어 로그인 처리 후 토큰을 받아  
다시 8088 포트로 요청하여 정상적인 응답을 받는 OAuth 인증 방식을 보여줍니다.  

9. 리소스별 권한 제어 테스트 (Order 서비스 코드 수정 및 재실행)
Order 서비스의 특정 리소스에 대한 접근 권한을 제어하기 위해 order/security/ResourceSecurityConfig.java 파일을 수정하고 서비스를 재실행합니다.  
```
# order/security/ResourceSecurityConfig.java 파일 수정 (권한 설정 로직 추가)
cd order
mvn clean spring-boot:run
```
변경된 설정으로 각 엔드포인트에 접근하며 권한이 제대로 적용되는지 확인합니다.  
```
# 웹 브라우저에서 다음 URL들 접속
https://8088-YOUR_GITPOD_URL/orders/placeOrder
https://8088-YOUR_GITPOD_URL/orders/manageOrder
```
orders/placeOrder: user@naver.com으로 로그인 시 "Hi~ user@naver.com. Welcome to 12stMall. We've Good Products, You can place an Order."  
메시지를 확인하여 정상 접근을 검증합니다.  
orders/manageOrder: admin 권한이 필요한 리소스의 접근 제어가 올바르게 작동하는지 확인합니다.  
