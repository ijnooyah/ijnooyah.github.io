---
title: "[Server] Web Server와 WAS"
excerpt: "Web Server와 WAS에 대해 알아보기"

categories:
  - Server
tags:
  - [Server]

permalink: /server/web-was/

toc: true
toc_sticky: true

date: 2024-08-12
last_modified_at: 2024-08-12
---
# 🍌 Web Server
Web Server란 **웹 브라우저(클라이언트)**로부터 **HTTP 요청**을 받아 **정적 컨텐츠**(.html, css 등)를 제공하는 프로그램을 말한다. 예시로 Apache Server, Nginx 등이 있다.

## 🍓 Web Server 기능
- 클라이언트로부터 **HTTP 요청**을 받을 수 있다.
- 정적 컨텐츠 요청시
  - **정적 컨텐츠를 제공**할 수 있다.
- 동적 컨텐츠 요청시
  - **WAS로 전달**하여 WAS가 처리한 결과를 클라이언트에 전달한다.

---

# 🌲 WAS(Web Application Server)
WAS(Web Application Server)란 **웹 서버(Web Server) + 웹 컨테이너(Web Container)**를 결합한 서버를 의미한다. 예시로 Tomcat, Jeus 등이 있다.

## 🏈 WAS 기능
- 클라이언트로부터 **HTTP 요청**을 받을 수 있다. (대부분의 WAS는 Web Server 내장) 
- 요청에 맞는 정적 컨텐츠를 제공할 수 있다.
- **DB 조회**나 다양한 로직 처리를 통해 **동적 컨텐츠**를 제공할 수 있다.

## 🌱 웹 컨테이너(Web Container)
**"동적인 처리를 하는 곳"**  
웹 컨테이너는 JSP나 Servlet이 실행될 수 있는 프로그램으로 **서블릿 컨테이너(Servlet Container)**라고도 한다.

---

# 🐩 WAS와 Web Server를 같이 사용하는 이유
WAS만으로도 웹 서비스를 제공할 수 있지만 앞에 Web Server를 붙여 같이 사용하게 되면 여러가지 장점이 있다.

**역할 분리를 통한 성능 향상**  
정적 컨텐츠는 Web Server, 동적 컨텐츠는 WAS가 담당  
&rarr; 전체 서비스의 성능 향상

**여러 대의 WAS 로드밸런싱**  
WAS가 처리해야하는 요청을 여러 WAS가 나누어서 처리할 수 있도록 설정  
&rarr; 서버 부하를 줄여 전체 서비스의 처리량 상승 및 가용성 향상

**캐싱 및 압축**  
Web Server는 정적 리소스에 대한 요청을 캐시하고 압축  
&rarr; 사용자에게 더 빠른 응답 제공 가능

**보안**  
- Web Server는 리버스 프록시 또는 DMZ(비무장 지대) 역할  
- 외부와 내부 네트워크 격리시킬 수 있음  
&rarr; 공격자가 직접 WAS에 접근하는 것을 방지
- SSL/TLS를 설정해 HTTPS를 통한 암호화된 연결을 지원 가능  
&rarr; 데이터 전송 속도 향상 및 안전한 데이터 통신 가능

---

<p class="ref">참고</p>
- [https://velog.io/@waoderboy/%EC%9B%B9-%EC%84%9C%EB%B2%84-WAS-%EC%9B%B9-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88](https://velog.io/@waoderboy/%EC%9B%B9-%EC%84%9C%EB%B2%84-WAS-%EC%9B%B9-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88)
- [https://velog.io/@suhongkim98/CGI%EC%99%80-%EC%84%9C%EB%B8%94%EB%A6%BF-JSP%EC%9D%98-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0](https://velog.io/@suhongkim98/CGI%EC%99%80-%EC%84%9C%EB%B8%94%EB%A6%BF-JSP%EC%9D%98-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- [https://hstory0208.tistory.com/entry/%EC%99%9C-WAS%EC%99%80-Web-Server%EB%A5%BC-%EA%B0%99%EC%9D%B4-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B1%B8%EA%B9%8C](https://hstory0208.tistory.com/entry/%EC%99%9C-WAS%EC%99%80-Web-Server%EB%A5%BC-%EA%B0%99%EC%9D%B4-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B1%B8%EA%B9%8C)
- [https://www.youtube.com/watch?v=mcnJcjbfjrs](https://www.youtube.com/watch?v=mcnJcjbfjrs)

