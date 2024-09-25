---
title: "[Docker] Docker CLI 기본 명령어"
excerpt: ""

categories:
  - Docker
tags:
  - [Docker]

permalink: /docker/cli-basic-commands

toc: true
toc_sticky: true

date: 2024-09-18
last_modified_at: 2024-09-18
---

# 🐠 Docker CLI 기본 명령어

Docker CLI(Command Line Interface) 기본 명령어들은 Docker를 사용할 때 가장 자주 사용되는 핵심 명령어들이다.
[`docker run`](#docker-run), [`docker pull`](#docker-pull), [`docker push`](#docker-push), [`docker ps`](#docker-ps), [`docker logs`](#docker-logs), [`docker exec`](#docker-exec)에 대해 알아보자.

---

#### `docker run`
- **기능**: 새 컨테이너를 생성하고 시작하는 가장 기본적인 명령어이다. 
  - 이미지가 로컬에 없으면 자동으로 pull 한 후 실행한다.
- **사용 예**:
  ```
  docker run -d -p 80:80 nginx
  ```
- **설명**: nginx 이미지를 사용하여 컨테이너를 백그라운드(-d)에서 실행하고, 호스트의 80 포트를 컨테이너의 80 포트에 매핑(-p 80:80)한다.

---

#### `docker pull`
- **기능**: Docker Hub 또는 다른 레지스트리에서 이미지를 다운로드한다.
  -  이미지를 미리 다운로드할 때 사용한다.
- **사용 예**:
  ```
  docker pull ubuntu:latest
  ```
- **설명**: Ubuntu의 최신 이미지를 Docker Hub에서 로컬 시스템으로 다운로드한다.

---

#### `docker push`
- **기능**: 로컬 이미지를 Docker Hub 또는 다른 레지스트리에 업로드한다. 
  - 자신이 만든 이미지를 다른사람들과 공유하거나, 개인 레지스트리에 업로드할 때 사용한다.
- **사용 예**:
  ```
  docker push username/my-image:tag
  ```
- **설명**: 'my-image'라는 이름의 로컬 이미지를 Docker Hub의 'username' 계정으로 업로드한다.

---

#### `docker ps`
- **기능**: 실행 중인 컨테이너 목록을 보여준다.
  - 현재 시스템에서 실행 중인 컨테이너를 확인할 때 사용한다. 문제 해결이나 리소스 관리에 유용하다.
- **사용 예**:
  ```
  docker ps
  docker ps -a  # 모든 컨테이너 (중지된 것 포함)
  ```
- **설명**: 현재 실행 중인 컨테이너의 정보(ID, 이미지, 생성 시간, 상태 등)를 표시한다.

---

#### `docker logs`
- **기능**: 컨테이너의 로그를 확인합니다.
  - 애플리케이션의 동작을 모니터링하거나 문제를 진단할 때 중요하다.
- **사용 예**:
  ```
  docker logs container_id
  docker logs -f container_id  # 실시간 로그 확인
  ```
- **설명**: 지정된 컨테이너의 로그 출력을 보여준다. -f 옵션을 사용하면 실시간으로 로그를 확인할 수 있다.

---

#### `docker exec`
- **기능**: 실행 중인 컨테이너에 명령을 실행한다.
  - 실행 중인 컨테이너에 접근해 추가 명령을 실행할 때 사용한다. 디버깅이나 설정 변경 등에 유용하다.
- **사용 예**:
  ```
  docker exec -it container_id /bin/bash
  ```
- **설명**: 실행 중인 컨테이너에 대화형(-it) 셸(/bin/bash)을 실행한다. 이를 통해 컨테이너 내부에서 명령을 실행할 수 있다.