---
title: "[Network] CSRF"
excerpt: ""

categories:
  - Network
tags:
  - [Network]

permalink: /network/csrf/

toc: true
toc_sticky: true

date: 2024-08-23
last_modified_at: 2024-08-23
---
# 🦹 CSRF란?
CSRF(Cross Site Request Forgery)는 직역하면 사이트 간 요청 위조로 해석되며, 웹 취약점 중 하나이다.  
공격자가 희생자의 권한을 도용하여 특정 웹 사이트의 기능을 실행하게 할 수 있으며 이는 희생자의 의도와는 무관하게 이루어진다.

## 🎯 CSRF 공격 과정
<div class="mermaid">
sequenceDiagram
    autonumber
    participant User as 사용자👤
    participant Legit as 정상 웹사이트🌐
    participant Malicious as 악성 웹사이트😈

    User->>+Legit: 로그인 요청
    Legit-->>-User: 세션 생성 (쿠키 발급)
    Note over User,Legit: 사용자 인증 완료

    User->>+Malicious: 악성 사이트 방문
    activate Malicious
    Note over Malicious: CSRF 공격 준비
    Malicious->>Legit: 자동 요청 (사용자의 쿠키 포함)
    deactivate Malicious
    
    activate Legit
    Note over Legit: 요청 검증
    Legit->>Legit: 인증된 요청으로 처리
    deactivate Legit

    Note over User,Legit: 사용자 모르게 작업 수행됨
</div>

1. 사용자가 정상 웹사이트 로그인한다.
2. 로그인 후 세션이 생성되고 사용자의 브라우저에 쿠키가 저장된다.
3. 사용자가 (로그인 상태를 유지한 채) 악성 웹사이트를 방문한다.
4. 악성 웹사이트는 사용자 모르게 정상 웹사이트로 자동 요청을 보낸다. 이 요청에는 사용자의 인증 쿠키가 자동으로 포함된다.
5. 정상 웹사이트는 이 요청을 인증된 사용자의 정상적인 요청으로 간주하고 처리한다.

실제로 2008년 옥션에서 발생했던 관리자 계정 탈취 사건도  
관리자가 admin 권한으로 로그인 한 상태에서 피싱 메일을 열람하였고 피싱 메일에 숨겨진 CSRF 코드가 실행되어 관리자 권한을 탈취하게 된 것이다.

---
# 🛡️ CSRF 대응 방법
CSRF 공격을 방어하는 방법에는 여러가지가 있다.
그 중 주요 방법인
- CSRF 토큰 사용
- SameSite 쿠키 속성 설정
- Referer/Origin 헤더 검사

을 알아보도록 하자.

## 🏷️ CSRF 토큰 사용
CSRF 방어의 가장 일반적인 방법이다.  
서버에서 임의의 토큰을 생성해 각 요청에 포함시키고, 클라이언트는 이 토큰을 hidden 필드에 숨겨서 함께 전송한다.  
서버에서는 클라이언트에서 전달된 토큰과 서버에 저장된 토큰을 비교해 일치하는지 확인한다. 일치하지 않으면 위조된 요청으로 판단하고 처리하지 않는다.
## 🍪 SameSite 쿠키 속성 설정
쿠키를 특정 도메인이나 사이트에서만 전송하도록 제한하는 방법이다.  
쿠키에 `SameSite`속성을 `Strict` 또는 `Lax`로 설정한다.
Cross-site 요청 시 쿠키가 전송되지 않아 CSRF 공격을 방어할 수 있다.
## 🕵️‍♂️ Referer/Origin 헤더 검사
서버는 요청의 `Referer` 또는 `Origin` 헤더를 확인해 요청이 신뢰할 수 있는 출처에서 왔는지 검사한다. 만약 헤더 값이 예상한 출처와 일치하지 않으면 요청을 거부한다.


---
<p class="ref">참고</p>
- [https://tibetsandfox.tistory.com/11](https://tibetsandfox.tistory.com/11){: target="_blank"}


