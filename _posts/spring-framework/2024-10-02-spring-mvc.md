---
title: "[Spring Framework] Spring MVC"
excerpt: ""

categories:
  - Spring Framework
tags:
  - [Spring Framework]

permalink: /spring-framework/spring-mvc/

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---
# 🖼️ MVC 패턴이란?

MVC 패턴은 애플리케이션 개발 시 **Model**, **View**, **Controller**의 세 가지 요소로 나누어 개발하는 디자인 패턴이다. 이를 통해 애플리케이션의 UI(사용자 인터페이스)와 비즈니스 로직을 분리하여 **유지보수**와 **확장성**을 높일 수 있다. 이 패턴은 각 영역이 명확하게 구분되어 각 역할에만 집중할 수 있게 해준다.

- **Model**: 애플리케이션의 **데이터와 비즈니스 로직**을 담당한다. 데이터의 상태를 표현하고 저장하거나, 데이터를 조작하는 기능을 제공한다.
- **View**: 사용자가 보는 **UI 요소**(텍스트, 버튼 등)를 담당한다. **Model**에서 전달받은 데이터를 화면에 표시한다.
- **Controller**: **사용자의 요청**을 받아 Model과 View 사이의 **상호작용**을 관리한다. 사용자가 요청한 작업을 Model에게 전달하고, 처리 결과를 View에 전달하여 화면을 업데이트한다.

## 🎯 MVC 패턴을 사용하는 이유

MVC 패턴을 사용하면 **데이터 처리**와 **UI 처리**를 분리할 수 있기 때문에 각 구성 요소가 자신의 역할에만 집중할 수 있다. 또한, 시스템의 **결합도를 낮추어** 유지보수가 쉬워지고, 코드의 **재사용성**과 **확장성**이 높아진다.

---

# 📊 Model, View, Controller

![alt text](/assets/images/posts_img/spring-framework/spring-mvc/mode-controller-view.png)

## 🧩 Model (모델)
Model은 애플리케이션의 데이터와 그 데이터를 처리하는 비즈니스 로직을 포함한다. 클라이언트의 요청을 처리하여 결과 데이터를 생성하고, 그 데이터를 View에 전달한다. Model은 데이터의 상태를 유지하고, 데이터를 조작하는 로직을 제공한다.

## 👁️ View (뷰)
View는 **Model**에서 전달받은 데이터를 이용하여 **화면에 표시할 리소스**를 생성한다. 웹 애플리케이션에서 View는 HTML 페이지, PDF, 엑셀 파일, JSON, XML 등 다양한 형식으로 출력할 수 있다.

## 🎮 Controller (컨트롤러)
Controller는 클라이언트의 요청을 **직접 처리**하는 엔드포인트다. 요청을 받아서 **Model**에게 작업을 지시하고, 처리된 결과 데이터를 **View**에 전달하여 응답을 생성한다. Controller는 **Model**과 **View** 사이의 상호작용을 관리하며, 클라이언트와 서버 사이의 중개자 역할을 한다.

---

# 🧬 MVC1과 MVC2

## 1️⃣ MVC1

![alt text](/assets/images/posts_img/spring-framework/spring-mvc/mvc1.png)

MVC1은 초기의 패턴으로, 브라우저에서 요청이 들어오면 JSP(View)에서 **Model** 객체를 사용해 데이터를 처리하고, 응답을 직접 JSP가 담당하는 방식이다. 하지만 이 방식은 **비즈니스 로직**과 **프레젠테이션 로직**이 JSP에 혼재하게 되어, 유지보수가 어려워질 수 있다. 

## 2️⃣ MVC2

![alt text](/assets/images/posts_img/spring-framework/spring-mvc/mvc2.png)

MVC2는 이러한 단점을 보완하여 **Controller**가 요청을 먼저 받아 **비즈니스 로직 처리**를 하고, 그 결과를 JSP(View)에 전달하는 방식이다. Controller는 서블릿으로 구현되어 요청 처리의 중심 역할을 수행하며, View와의 분리를 통해 **구조적 복잡성**은 늘어날 수 있지만 유지보수성과 확장성이 크게 향상되었다.

---

# 🍃 Spring MVC

Spring MVC는 **MVC2 패턴**을 발전시켜 만든 **웹 애플리케이션 프레임워크**다. Spring MVC는 다음과 같은 구성 요소로 이루어져 있다.

## 📡 DispatcherServlet (디스패처 서블릿)
DispatcherServlet은 **Front Controller**로서 모든 클라이언트의 요청을 받는다. 이후 요청을 처리할 적절한 컨트롤러로 **위임**하고, 응답이 완료되면 클라이언트에게 **렌더링된 결과**를 반환한다.

## 🗺️ HandlerMapping (핸들러 매핑)
HandlerMapping은 들어온 요청을 처리할 **적절한 컨트롤러를 매핑**합니다. 요청 URL에 맞는 컨트롤러를 찾아주는 역할을 한다.

## 🔌 HandlerAdapter (핸들러 어댑터)
HandlerAdapter는 매핑된 컨트롤러를 실행시키는 역할을 한다. 이를 통해 다양한 형태의 컨트롤러를 유연하게 처리할 수 있다.

## 🕹️ Controller (컨트롤러)
Controller는 클라이언트의 요청을 처리하고, 결과 데이터를 **Model**에 담아 반환한다. 이 데이터는 **View**로 전달되어 화면에 표시된다.

## 📦 ModelAndView (모델 앤 뷰)
Controller는 결과 데이터를 **ModelAndView** 객체로 반환한다. 이 객체에는 **Model 데이터**(비즈니스 로직 결과)와 **View 이름**(렌더링할 화면 정보)이 함께 담겨 있다.

## 🔬 View Resolver (뷰 리졸버)
View Resolver는 Controller에서 전달받은 **View 이름**을 실제 **View 템플릿 파일**로 변환하여 찾아준다. 예를 들어, JSP 파일이나 Thymeleaf 파일을 탐색한다.

## 📺 View (뷰)
View는 Controller에서 전달받은 데이터를 이용해 **최종적으로 화면에 표시될 결과**를 생성한다. View 템플릿 엔진이 결과 화면을 만들어 클라이언트에게 응답한다.

---

# ⚙️ Spring MVC의 동작 흐름

![alt text](/assets/images/posts_img/spring-framework/spring-mvc/flow.png)

1. 클라이언트가 서버에 요청을 보내면, **DispatcherServlet**이 이를 먼저 받는다.
2. DispatcherServlet은 요청을 **HandlerMapping**에 전달하여 적절한 **Controller**를 찾는다.
3. Controller는 **비즈니스 로직을 처리**할 Service를 호출하여 작업을 수행한다.
4. Service는 데이터베이스 접근이 필요할 경우 **DAO**에 요청을 위임하여 데이터베이스 작업을 수행한다.
5. DAO는 DB에서 가져온 데이터를 **DTO** 혹은 **Entity** 객체로 변환하여 Service로 전달한다.
6. Service는 처리 결과를 Controller로 반환하고, Controller는 이를 **Model**에 담아 View로 전달한다.
7. DispatcherServlet은 **ViewResolver**를 통해 알맞은 View 템플릿을 찾아 요청을 전달한다.
8. 최종적으로 View는 데이터를 렌더링하여 클라이언트에게 응답한다.






