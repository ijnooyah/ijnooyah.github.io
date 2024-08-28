---
title: "[Server] API란? (feat. Endpoint)"
excerpt: ""

categories:
  - Server
tags:
  - [Server]

permalink: /server/api-feat-endpoint/

toc: true
toc_sticky: true

date: 2024-08-28
last_modified_at: 2024-08-28
---
# 👨‍💼 API란?
API(Applicaiton Programming Interface)란 소프트웨어 응용 프로그램에서 다른 소프트웨어 구성 요소 또는 서비스와 상호 작용을 하기 위한 인터페이스를 제공하는 프로그래밍 기술을 뜻한다.

> `Applicaiton`: 특정한 업무를 수행하기 위해 개발된 응용 소프트웨어  
`Programming`: 컴퓨터에 부여하는 명령을 만드는 작업, 수식이나 작업을 컴퓨터에 알맞도록 정리해서 순서를 정하고 컴퓨터 특유의 명령코드로 고쳐 쓰는 작업  
`Interface`: 사물과 사물 사이 또는 사물과 인간 사이의 경계에서, 상호간의 소통을 위해 만들어진 물리적인 매개체나 통신 규칙
{: .info}

API를 예시를 통해 이해해보자.  

`손님`이 레스토랑에서 `웨이터`를 불러 음식을 주문하면  
`웨이터`는 `셰프`에게 음식을 요청한다.  
`셰프`는 요리가 끝난후 `웨이터`에게 음식을 전달한다.  
`웨이터`는 `손님`에게 주문한 음식을 제공한다.  

![API 비유](/assets/images/posts_img/server/api/metaphor.png)

여기서 `손님`은 메뉴에 있는 음식을 요청하는 **응용 프로그램**이라면 `셰프`는 요리를 만들어 전달하는 **API 제공자**라고 할 수 있다.  
이 둘을 연결해주는 `웨이터`가 바로 **API**다. 

<b>API의 사전적인 정의</b>  
API는 여러 프로그램들과 데이터베이스, 기능들의 상호 통신 방법을 규정하고 도와주는 매개체이다. API는 데이터베이스가 아니지만, 액세스 권한이 있는 앱의 권한 규정과 서비스 요청에 따라 데이터나 서비스 기능을 제공하는 메신저 역할을 한다.

---
# 📎 Endpoint란?
엔드포인트(Endpoint)는 소프트트웨어나 네트워크에서 데이터를 주고받는 특정 지점이나 위치를 의미한다. 특히 API에서 "Endpoint"는 특정 기능이나 자원(Resource)에 접근하기 위해 클라이언트가 요청을 보낸 URL 경로이다.  

각 Endpoint는 특정 HTTP 메소드(GET, POST, PUT, DELETE 등)와 함께 사용되어 서버에서 특정 작업을 수행하도록 한다. Endpoint는 API의 구조와 기능을 정의하는 핵심 요소로, 클라이언트와 서버 간의 효율적이고 표준화된 통신을 가능하게 한다.

---
<p class="ref">참고</p>
- [https://moonspam.github.io/What-is-an-API/](https://moonspam.github.io/What-is-an-API/){: target="_blank"}
- [https://velog.io/@dongjun187/API란-무엇인가](https://velog.io/@dongjun187/API%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80){: target="_blank"}


