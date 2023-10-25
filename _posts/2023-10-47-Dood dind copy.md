---
title: 도커 dood vs dind
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, docker]
tags:
  [
    백엔드,
	  docker
  ]

toc: true
toc_sticky: true

---

# 도커 dood vs dind

## DooD란?
- **DooD**는 Docker Out Of Docker의 약어
- 호스트 도커 데몬이 사용하는 **소켓**을 공유하여 도커 클라이언트 컨테이너에서 컨테이너를 실행시키는 것
- 새로 실행시킨 컨테이너는 도커 클라이언트 컨테이너와 형제 관계를 가진다.
- 따라서 테스트 환경이 도커 호스트 환경과 일치하다.
- 호스트와 도커 이미지를 공유한다.
  
## DooD 실행 방법

```
$ docker run -it -v /var/run/docker.sock:/var/run/docker.sock docker
```

- 호스트의 도커 소켓을 컨테이너에 마운트 시켜준다

# DinD란?

- **DinD**는 Docker in Docker의 약어
- 도커 컨테이너 내부에 호스트 도커 데몬과는 별개의 새로운 도커 데몬을 실행시키는 것 (도커 컨테이너 안에 도커 실행)
- 새로 실행된 도커 데몬 컨테이너에, 새로운 도커 클라이언트 컨테이너를 이용하여 명령을 내리는 것이 가능하다(도커 안의 도커한테 명령 가능)
- 새로 생성된 도커 환경은 컨테이너 환경이기 때문에, 테스트 후의 정리가 꽤나 간편하다
- 보안 문제가 있기 때문에 다들 DooD를 추천한다.

## DinD 실행 방법

- 새로운 도커 컨테이너를 실행할 때, `--privileged` 옵션을 이용한다 (컨테이너가 호스트 머신의 대부분의 권한을 얻는 옵션) → **보안상 매우 좋지 못하다**