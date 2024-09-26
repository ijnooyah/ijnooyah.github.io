---
title: "[Docker] Spring Boot & Docker (feat. Redis, H2)"
excerpt: ""

categories:
  - Docker
tags:
  - [Docker]

permalink: /docker/docker-spring-boot

toc: true
toc_sticky: true

date: 2024-09-26
last_modified_at: 2024-09-26
---

해당 프로젝트는 [Redis 실습 프로젝트](https://ijnooyah.github.io/redis/learn-redis){: target='_blank'}를 재구성했다.  
프로젝트 패키지 구조도 조금 변경했으므로 [깃허브](https://github.com/ijnooyah/docker-exercise){: target='_blank'}에서 확인하길 바란다.

---

# 📄 Dockerfile 작성

Dockerfile을 작성하기 전에 Redis 실전 프로젝트까지 진행했던 부분을 수정해줘야 한다. `application.yml`에 코드를 수정 및 추가해 준다.  


```yml
spring:
  data:
    redis:
      host: redis
      port: 6379
```
Redis 컨테이너의 이름을 `redis`라고 지을건데 이 때 Redis를 포함한 서비스들은 내부 네트워크로 연결된다.  
따라서 `application.yml` 파일에 Redis의 호스트를 `redis`로 설정해야 한다.

> Docker Compose를 사용하면 여러 컨테이너를 하나의 프로젝트처럼 관리할 수 있으며, 기본적으로 이들 컨테이너는 동일한 네트워크에 연결된다. 즉, 컨테이너 내에서 서로의 서비스 이름으로 접근할 수 있다. 따라서, Redis 컨테이너의 이름을 redis로 설정했다면, 다른 컨테이너에서는 redis라는 호스트명으로 Redis에 접속할 수 있는 것이다.
{: .info}

또한 `spring.datasource.url`을 `jdbc:h2:mem:testdb`로 변경해준다.  
우리는 H2 데이터베이스를 도커 컨테이너로 구동할 예정이기 때문에 개발 환경에서는 메모리 내 데이터베이스를 사용하겠다.

---
{: .style1}

이제 Dockerfile을 작성해보자.  
프로젝트의 root 경로에 Dockerfile을 만들어준다.

```dockerfile
# BUILD stage
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app
COPY gradle gradle
COPY build.gradle settings.gradle gradlew ./
RUN ./gradlew --no-daemon dependencies
COPY src src
RUN ./gradlew --no-daemon clean build


# RUN stage
FROM eclipse-temurin:21-jre

COPY --from=builder /app/build/libs/*.jar /app/app.jar
ENTRYPOINT ["java"]
CMD ["-jar","/app/app.jar"]
```

이제 이 명령들을 하나씩 알아보자.

---
{: .style1}

#### 🏗️ BUILD stage

```dockerfile
FROM eclipse-temurin:21-jdk AS builder
```
- Eclipse Temurin의 JDK 21 버전을 기반 이미지로 사용한다.
- `AS builder`로 이 단계에 이름을 지정하여 나중에 참조할 수 있게 한다.

<br>

```dockerfile
WORKDIR /app
```
- 컨테이너 내부에 `/app` 디렉토리를 생성하고 작업 디렉토리로 설정한다. 이후 명령어 들은 `/app` 디렉토리 안에서 실행된다.
<br>

```dockerfile
COPY gradle gradle
```
- 현재 프로젝트의 `gradle` 폴더를 컨테이너의 `/app/gradle`로 복사한다. 
-  `gradle` 폴더에는 Gradle Wrapper 관련 파일들이 포함되어 있다.
  
<br>

```dockerfile
COPY build.gradle settings.gradle gradlew ./
```
- 세 개의 파일(`build.gradle`, `settings.gradle`,` gradlew`)을 컨테이너 내의 현재 디렉토리(`./` -> `/app`)로 복사한다.
  - `build.gradle`: 프로젝트의 주요 빌드 설정 파일로, 의존성, 플러그인, 태스크 등을 정의한다.
  - `settings.gradle`: 멀티 모듈 프로젝트의 설정을 정의하는 파일이다. 단일 모듈 프로젝트에서도 존재할 수 있다.
  - `gradlew`: Gradle Wrapper 실행 스크립트로, 특정 버전의 Gradle을 다운로드하고 실행하는 데 사용된다.
- 빌드 환경의 일관성을 유지하기 위해 버전이 고정된 gradle 이미지를 사용하지 않고 프로젝트에 지정된 버전의 gradle을 복사한 것이다.
  
<br>

```dockerfile
RUN ./gradlew --no-daemon dependencies
```
- 의존성을 다운로드 한다.
- 자세한 설명은 `RUN ./gradlew --no-daemon clean build`에서 하겠다.
<br>

```dockerfile
COPY src src
```
- 프로젝트의 소스 코드를 src 폴더를 컨테이너의 /app/src에 복사한다.

<br>

```dockerfile
RUN ./gradlew --no-daemon clean build
```
- Gradle 빌드를 실행한다.
- `--no-daemon` 옵션은 빌드 후 Gradle 데몬을 종료해 리소스를 절약한다.
- `clean` 이전 빌드 아티팩트를 지우고 `build` 새로운 빌드를 실행한다.
- `clean`을 사용하는 이유는 이전 빌드에서 남은 파일들이 영향을 줄 수 있는 가능성을 배제하기 위해 `clean`을 먼저 실행해서 깨끗한 상태에서 빌드를 실행한다. 
- 의존성 설치 명령과 빌드 명령을 분리해서 캐시 활용도를 높였다.

<br>

---
{: .style1}

#### 🚀 RUN stage

```dockerfile
FROM eclipse-temurin:21-jre
```
- 런타임 환경에서는 JDK가 아닌 JRE를 사용해 최종 이미지 크기를 줄인다.
  
<br>

```dockerfile
COPY --from=builder /app/build/libs/*.jar /app/app.jar
```
- 이전 빌드 스테이지(`builder`)에서 생성된 `.jar` 파일을 가져와 런타임 스테이지의 `/app` 디렉토리에 복사한다.
-  최종적으로 실행될 `.jar` 파일이 `/app/app.jar`로 배치된다.

<br>

```dockerfile
ENTRYPOINT ["java"]
```
- 컨테이너가 실행될 때 기본적으로 Java 명령어를 실행하도록 설정한다.
- 이 명령어는 변경할 수 없는 명령어이다. (자세한 것은 [ENTRYPOINT 설명](https://ijnooyah.github.io/docker/dockerfile#-entrypoint) 참고)

<br>

```dockerfile
CMD ["-jar", "/app/app.jar"]
```
- `java` 명령어의 기본 인자로 `.jar` 파일을 지정한다. 
- `ENTRYPOINT`와 결합하여 컨테이너가 시작되면 자동으로 `java -jar /app/app.jar` 명령어가 실행된다.

> 해당 도커 파일은 **멀티 스테이지 빌드**다.  
멀티 스테이지 빌드란, Dockerfile에서 여러 빌드 단계를 정의하고 각 단계의 결과를 최종 이미지로 가져오는 방식이다. 이 과정에서 불필요한 빌드 도구나 파일을 최종 이미지에 포함시키지 않아 **이미지 크기를 최소화**하고, **보안 및 성능을 개선**할 수 있다.

> 해당 도커파일에서는 
- **Stage1 (build)**에서는 jdk21과 Gradle을 포함한 무거운 빌드 환경에서 애플리케이션을 빌드한다.
- **Stage2 (run)**에서는 빌드된 애플리케이션만을 가져와, 가벼운 JRE 환경에서 실행한다. 

---

# 🎼 Docker Compose 작성

프로젝트 root 경로에 `docker-compose.yml` 파일을 만들어준다.

```yml
version: "3"
services:
  db:
    container_name: h2
    image: oscarfonts/h2:latest
    ports:
      - 1521:1521
      - 8081:81
    environment:
      H2_OPTIONS: -ifNotExists
    volumes:
      - ./h2/:/opt/h2-data
    restart: always
  redis:
    container_name: redis
    image: redis:alpine
    ports:
      - 6379:6379
    volumes:
      - ./data/:/data
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: dockermaster
    ports:
      - 8080:8080
    environment:
      SPRING_DATASOURCE_URL: jdbc:h2:tcp://h2:1521/dockermaster
      SPRING_DATASOURCE_USERNAME: sa
      SPRING_DATASOURCE_PASSWORD:
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
    depends_on:
      - db
      - redis
```

이제 이 명령들을 하나씩 알아보자.

---
{: .style1}

```yml
version: "3"
```
- Docker Compose 파일 형식의 버전을 지정한다.

<br>


```yml
services: 
```
- 실행할 서비스들을 정의한다. 이 파일에는 세개의 서비스(db, redis, app)가 정의되어 있다.

<br>


```yml
  db:
    container_name: h2
    image: oscarfonts/h2:latest
    ports:
      - 1521:1521
      - 8081:81
    environment:
      H2_OPTIONS: -ifNotExists
    volumes:
      - ./h2/:/opt/h2-data
    restart: always
```
- `container_name: h2`: 컨테이너 이름을 'h2'로 지정한다.
- `image: oscarfonts/h2:latest`: Docker Hub의 `oscarfonts/h2` 이미지의 최신 버전을 사용한다.
- `ports`: 호스트의 1521 포트를 컨테이너의 1521 포트로, 8081 포트를 81 포트로 매핑한다. H2는 기본적으로 1521 포트를 사용하고, 웹 콘솔은 81 포트를 사용한다.
- `environment`: 환경 변수를 설정한다. `ifNotExists` 옵션은 데이터베이스가 존재하지 않을 때만 생성하도록 한다.
- `volumes`: 호스트의 `./h2/` 디렉토리를 컨테이너의 `/opt/h2-data`에 마운트한다. 이 디렉토리는 H2 DB 파일을 저장하는 위치다.
- `restart: always`: 컨테이너가 종료되면 항상 자동으로 재시작되도록 설정한다.

<br>

```yml
redis:
  container_name: redis
  image: redis:alpine
  ports:
    - 6379:6379
  volumes:
    - ./data/:/data
```
- `container_name: redis`: 컨테이너 이름을 'redis'로 지정한다.
- `image: redis:alpine`: Docker Hub의 공식 Redis 이미지의 Alpine 버전을 사용합니다. Alpine 버전은 경량화된 Linux 배포판을 기반으로 하여 이미지 크기가 작다.
- `ports`: 호스트의 6379 포트를 컨테이너의 6379 포트로 매핑한다. 6379는 Redis의 기본 포트이다.
- `volumes`: 호스트의 `./data/` 디렉토리를 컨테이너의 `/data` 디렉토리에 마운트합니다. 이를 통해 Redis 데이터를 호스트 시스템에 영구적으로 저장할 수 있다.

---

# 🐳Docker로 실행하기
1. 프로젝트 루트 디렉토리에서 다음 명령어를 실행한다.
   ```
   docker compose up --build
   ```
2. 브라우저에서 `http://localhost:8080/swagger-ui/index.html`으로 접속해 API 문서를 확인한다.



