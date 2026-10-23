
# 5주차 DevOps 실습: Docker 기초 및 nginx 실행

## 1. 실습 목표
* 하이퍼바이저(VM)와 컨테이너의 구조적 차이점을 이해한다.
* Docker 이미지와 컨테이너의 개념 및 관계를 파악한다.
* `docker run`, `ps`, `logs`, `exec`, `stop`, `rm` 등 기본 명령어를 익히고 활용한다.
* 서로 다른 파이썬 버전 실행 및 Port Forwarding을 통해 컨테이너 격리 환경을 확인한다.
* GitHub Issue, Branch, PR(Pull Request) 기반의 DevOps 작업 흐름을 실습한다.

---

## 2. 주요 개념 정리

(1) VM(가상머신) vs Container
* VM (가상머신): 하드웨어 가상화 방식. 각 VM마다 독립된 Guest OS가 필요하여 무겁고 시작 속도가 상대적으로 느림 (기가바이트 단위).
* **Container**: 호스트 OS 커널을 공유하며 프로세스 수준으로 실행 환경을 격리. 가볍고 빠르게 시작 가능 (메가바이트 단위).

### (2) 이미지(Image)와 컨테이너(Container)
* **Image**: 애플리케이션과 실행 환경을 함께 압축한 읽기 전용(Read-Only) 템플릿 (붕어빵 틀).
* **Container**: 이미지를 실행한 인스턴스로, 독립된 프로세스 및 쓰기 계층(Writable Layer)을 가짐 (붕어빵).

---

## 3. 주요 명령 및 실습 내용

### ① 환경 확인 및 테스트 실행
```bash
# Docker 버전 및 기본 설치 확인
docker --version

# 테스트용 hello-world 이미지 실행 (로컬에 없으면 Docker Hub에서 Pull 후 실행)
docker run hello-world
