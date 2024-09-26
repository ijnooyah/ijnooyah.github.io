---
title: "[Docker] 자주 사용하는 Docker Compose 명령어 정리"
excerpt: ""

categories:
  - Docker
tags:
  - [Docker]

permalink: /docker/docker-compose

toc: true
toc_sticky: true

date: 2024-09-25
last_modified_at: 2024-09-25
---

# 👑 기본 사용법
`docker compose` 명령어는 기본적으로 `docker-compose.yml` 파일이 있는 디렉토리에서 실행해야 한다.  
파일 이름이 다르거나 다른 위치에 있는 경우 `-f` 옵션을 사용한다.

```
docker compose -f ./docker-compose.prod.yml up
```
PS C:\Users\ijnoo> docker run -d -p 1521
---

# 🍰 주요 명령어

#### `docker compose build [service]`
- `docker-compose.yml`에 정의된 서비스의 이미지를 빌드한다.
- 서비스를 지정하지 않으면 모든 서비스를 빌드한다.

#### `docker compose up <service>`
- 서비스에 필요한 모든 컨테이너를 생성하고 시작한다.
- 이미지가 없거나 변경된 경우 자동으로 빌드한다.
- `--build` 옵션을 추가하면 강제로 이미지를 다시 빌드한다. (모든 이미지를 새로 빌드해서 컨테이너를 생성하고 싶다면 아래 명령어를 사용한다.)
   ```
   docker compose up --build
   ```
 - 실행 중인 서비스를 중단하려면 `Ctrl + C`를 사용한다.

#### `docker compose down <service>`
- 해당 프로젝트의 컨테이너를 중지하고 제거한다.
- 네트워크와 볼륨도 함께 제거된다.
- 따라서 잠시 중단한 뒤 다시 시작하고 싶다면 `docker compose stop` 혹은 docker compose가 실행되고 있는 터미널에서 `Ctrl + C`를 통해 중단해야 한다.

#### `docker compose stop <service>`
- 컨테이너를 중지한다. (제거하지 않음)
- 서비스를 지정하지 않으면 모든 서비스를 중지한다.

#### `docker compose start <service>`
- 중지된 컨테이너를 시작한다.

#### `docker compose restart <service>`
- 컨테이너를 재시작한다. (`stop` + `start`)
- 이미지 변경 후 재시작하려면 빌드후 재시작한다:
   ```
   docker compose build <service>
   docker compose restart <service>
   ```

#### `docker compose ps`
- 현재 동작하는 컨테이너를 보여준다.

#### `docker compose logs <service>`
- 해당 서비스의 로그를 출력한다.


---

<p class='ref'>참고</p>
- [https://velog.io/@zero-black/Docker-docker-compose-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC](https://velog.io/@zero-black/Docker-docker-compose-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC){: target='_blank'}