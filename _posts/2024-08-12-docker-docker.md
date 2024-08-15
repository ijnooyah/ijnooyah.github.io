---
title: "[Docker] 도커(Docker)란 무엇인가?"
excerpt: "도커(Docker)에 대해 알아보기"

categories:
  - Docker
tags:
  - [Docker]

permalink: /docker/docker/

toc: true
toc_sticky: true

date: 2024-08-12
last_modified_at: 2024-08-12
---
# 🐬 도커(Docker)
도커는 컨테이너 기반의 오픈소스 가상화 플랫폼이다.  
도커는 리눅스 컨테이너 기술을 자동화해 쉽게 사용할 수 있게 도와준다.(리눅스 컨테이너 기술은 아래에 후술하겠다.)  
실무에서 일하다보면 개발환경을 설정하는 것은 정말 쉬운일이 아니라는 것을 느껴봤을 거다.   
도커는 이러한 복잡한 작업들을 도커 이미지와 도커 컨테이너를 사용해 기존 가상화방식보다 성능을 업그레이드 시켰다.

## 🫙 컨테이너(Container)
![vm-docker](/assets/images/posts_img/docker/docker/vm-docker.png)

컨테이너는 프로세스가 격리되어서 동작하는 기술을 말하며 가상화 기술 중 하나이다.

기존의 가상화 방식은 주로 OS를 가상화한다. 하이퍼바이저를 사용해 호스트 OS 위에 게스트 OS를 올려 호스트 OS를 공유한다.  

도커의 컨테이너는 게스트 OS를 두지 않고 호스트 OS 커널을 바로 사용하며 각각은 격리된 프로세스로 실행된다. 리눅스에서는 이 방식을 리눅스 컨테이너 방식이라고 한다.  
하이퍼바이저와 게스트 OS가 없다는 면에서 도커는 매우 가벼워진 것이다.

## 🖼️ 이미지(Image)
도커 공식 홈페이지에서는 이미지를 이렇게 설명하고 있다. 
> Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.  
Container images become containers at runtime and in the case of Docker containers – images become containers when they run on Docker Engine.

Docker 컨테이너 이미지는 애플리케이션을 실행하는데 필요한 모든 것(코드, 런타임, 시스템툴, 시스템 라이브러리, 세팅 등)을 포함한 가볍고 독립적이고 실행가능한 소프트웨어 패키지를 말한다.   

정리하자면 이미지는 특정 프로세스를 실행하기 위한 (컨테이너 생성, 실행에 필요한) 모든 파일과 설정값(환경)을 지닌 것이다. 때문에 더이상 의존성 파일을 컴파일하고 이것저것 설치할 필요 없다.

## 😎 정리
- 도커를 사용하면 애플리케이션을 컨테이너화하여, 어떤 환경에서도 동일하게 실행할 수 있다. ('내 컴퓨터에서는 되는데?'라는 문제를 해결해준다.)

---

<p class="ref">참고</p>
- [https://terms.naver.com/entry.naver?docId=3578612&cid=59088&categoryId=59096](https://terms.naver.com/entry.naver?docId=3578612&cid=59088&categoryId=59096)
- [https://www.docker.com/resources/what-container/](https://www.docker.com/resources/what-container/)
- [https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- [https://tecoble.techcourse.co.kr/post/2021-08-14-docker/](https://tecoble.techcourse.co.kr/post/2021-08-14-docker/)
- [https://devlog-wjdrbs96.tistory.com/279](https://devlog-wjdrbs96.tistory.com/279)

