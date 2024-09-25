---
title: "[Docker] Dockerfile 이해와 활용"
excerpt: ""

categories:
  - Docker
tags:
  - [Docker]

permalink: /docker/dockerfile

toc: true
toc_sticky: true

date: 2024-09-19
last_modified_at: 2024-09-19
---

# 📘 Dockerfile이란?
- Docker 이미지를 작성할 수 있는 스크립트 파일이다.
- 이미지는 이 스크립트를 기반으로 생성되며, Dockerfile 문법으로 작성된다.
- Dockerfile을 활용하여 사용자 정의 이미지를 만들 수 있고, 실제 배포에 많이 사용되는 기술이다.

---

# 📝 Dockerfile 기본 문법
- Dockerfile은 텍스트 파일 형식으로, 어떤 에디터로도 작성할 수 있다.
- 기본 구조는 `명령 인수` 형태이다.
- 명령어는 가독성을 위해 대문자로 작성하는 것이 관례이다.

---

# 🧱 Dockerfile 주요 명령어

#### FROM
- 베이스 이미지를 지정하는 명령어
- Dockerfile에서 반드시 있어야 하며, 보통 가장 첫 줄에 위치한다.
    ```dockerfile
    FROM openjdk:11-jdk-slim
    ```
- 도커는 계속 층으로 여러겹 이미지를 차고차곡 쌓는 개념이다. 베이스 이미지란, 그 층 중에서 가장 밑바닥의, 기본이 되는 이미지고 FROM이 이 이미지를 지정해준다.

---
{: .style1}

#### LABEL
* 이미지에 메타데이터를 추가하는 기능
* 키-값 쌍으로 정보를 저장하며, 보통 저자, 버전, 작성일자 등을 넣는다.
    ```dockerfile
    LABEL version="1.0.0" 
    LABEL maintainer="example@email.com"
    ```  

---
{: .style1}

#### CMD
* 컨테이너가 시작될 때 가장 먼저 실행할 기본 명령을 지정
* Dockerfile 내에서 한 번만 사용 가능하며, 여러 개 있을 경우 마지막 것만 적용된다.
    ```dockerfile
    CMD ["java", "-jar", "app.jar"]
    ```
    * `java`: Java 런타임을 실행한다. 
    * `-jar`: 다음에 오는 파일이 실행 가능한 JAR 파일임을 지정한다.
    * `app.jar`: 실행할 JAR 파일의 이름이다.
 
---
{: .style1}

#### RUN
* 이미지 빌드 과정에서 실행할 명령을 지정
* 새로운 레이어를 생성하여 패키지(프로그램) 설치 등의 작업을 수행한다. (이미지 레이어를 쌓아나가는 과정)
    ```dockerfile
    RUN apt-get update # 패키지 정보 업데이트
    RUN apt-get install -y apache2
    ```
    * `-y` 옵션: 설치 과정에서 나오는 모든 프롬포트에 자동으로 "yes"라고 답하게 한다.

---
{: .style1}

#### ENTRYPOINT
* 컨테이너가 시작될 때 실행할 명령을 지정
* **CMD와 달리** docker run 명령의 인자로 **덮어쓰기가 되지 않는다.**
    ```dockerfile
    ENTRYPOINT ["/bin/sh"]
    ```
---
{: .style1}

#### EXPOSE
* 컨테이너의 특정 포트를 외부에 오픈하는 설정
    ```dockerfile
    EXPOSE 8080
    ```
* `docker run -p` 옵션과 유사하다. 
  * 차이점은 `-p` 옵션은 컨테이너의 특정 포트를 외부에 오픈하고, 해당 포트를 호스트 PC의 특정 포트와 매핑시키는 것까지 알아서 해주지만, `EXPOSE`는 단순히 포트를 열어주기만 한다.
  * 따라서 도커파일에 EXPOSE로 포트를 열어주었더라도, `docker run`을 할 때, `-p` 옵션을 쓰게 된다.

---
{: .style1}

#### ENV
* 컨테이너 내의 환경 변수를 설정
* 설정한 환경 변수는 `RUN`, `CMD`, `ENTRYPOINT` 명령에도 적용된다.
    ```dockerfile
    ENV MYSQL_ROOT_PASSWORD=password # mysql 슈퍼관리자인 root ID에 대한 password란에 패스워드 설정하기
    ENV MYSQL_DATABASE=dbname # DB 지정해주기
    ENV MYSQL_USER=user
    ENV MYSQL_PASSWORD=pw #mysql 추가 사용자 ID, 패스워드 설정
    ```
---
{: .style1}

#### WORKDIR
* 이후 명령어들이 실행될 작업 디렉토리를 설정
    ```dockerfile
    # /app 작업 디렉토리로 설정
    WORKDIR /app

    # 이후 명령어들은 모두 /app 디렉토리에서 실행됨
    RUN npm install
    ```

---

# 🏭 이미지 빌드하기

```
docker build --tag <이미지명> -f <Dockerfile명> .
```
명령 끝에 있는 '.'(점)은 현재 디렉토리를 빌드 컨텍스트로 사용하라는 의미이다.
이미지를 빌드할 때, 명령을 실행하는 현재 디렉토리가 빌드 컨텍스트가 된다.

기본적으로 Docker는 빌드 컨텍스트 내에서 Dockerfile을 찾는다.

---

<p class='ref'>참고</p>
- [https://velog.io/@tjdwns2243/dockerfile-%EB%AC%B8%EB%B2%95-%EC%9E%91%EC%84%B1%EB%B2%95](https://velog.io/@tjdwns2243/dockerfile-%EB%AC%B8%EB%B2%95-%EC%9E%91%EC%84%B1%EB%B2%95){: target='_blank'}