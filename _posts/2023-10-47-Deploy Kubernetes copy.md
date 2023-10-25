---
title: Deploy - Kubernetes
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, k8s]
tags:
  [
    백엔드,
	  k8s
  ]

toc: true
toc_sticky: true

---

# Deploy - Kubernetes

## Application 배포 진화 과정

![Untitled](/assets/img/2023-10-47/Untitled.png)

## Kubernetes 구조

![Untitled](/assets/img/2023-10-47/Untitled%201.png)

- **Master:** 노드를 제어하고 전체 클러스터를 관리하는 컨트롤러이며, 전반적인 제어기능을 합니다.
- **Node**: 작업자 노드로 컨테이너가 배포될 물리적 서버 또는 가상 머신입니다.
- **Pod**: 단일 노드에 배포된 하나 이상의 컨테이너 그룹으로, 여러 컨테이너를 그룹화하여 포드 단위로 관리할 수 있습니다.

### CI (Countinous Integration)

![Untitled](/assets/img/2023-10-47/Untitled%202.png)

Kubernetes에서 **기본적인 객체**는 배포와 관리를 담당합니다.

컨테이너화 및 배포되는 애플리케이션의 워크로드를 설명하는 객체입니다.

포드, 서비스, 볼륨 및 네임스페이스의 네 가지 객체 유형이 있습니다.

- **Pod**는 컨테이너화된 애플리케이션
- **볼륨**은 디스크
- **서비스**는 로드 밸런서
- **네임스페이스**는 패키지 이름

## Kubernetes 스펙

- Pod 객체 스펙 (예)

![Untitled](/assets/img/2023-10-47/Untitled%203.png)

- Deployment 컨트롤러 스펙(예)

![Untitled](/assets/img/2023-10-47/Untitled%204.png)

> 3개의 nginx 포드를 가져오는 ReplicaSet을 생성합니다.

- 서비스 객체 스펙(예)

![Untitled](/assets/img/2023-10-47/Untitled%205.png)

> 서비스는 “myapp” 레이블이 붙은 서비스만 선택하여 서비스에 넣고 Pod간 로드 밸런싱을 통해 **외부로 서비스를 제공**합니다.

## Kubernetes 배포 실습1(Using Kubernetes without installation)

### Minikube(Katacode)

[https://kubernetes.io/ko/docs/tutorials/hello-minikube/](https://kubernetes.io/ko/docs/tutorials/hello-minikube/)

- Launch Terminal

![Untitled](/assets/img/2023-10-47/Untitled%206.png)

### Depolyment 생성

```
$ kubectl create deployment hello-node --image=registry.k8s.io/echoserver:1.4
```

![Untitled](/assets/img/2023-10-47/Untitled%207.png)

> hello-node deployment 컨트롤러를 만들어서 이미지 경로의 인스턴스를 배포합니다.

kubectl get pods 명령어를 통해 확인합니다.

```
$ minikube dashboard
```

![Untitled](/assets/img/2023-10-47/Untitled%208.png)

> 대시보드를 사용하면 현재 워크로드 상태를 보고 내용을 편집할 수 있습니다.

Deployment, Pod, ReplicaSet이 안정적으로 제공되는 것을 확인할 수 있습니다.

### 서비스 생성

```
$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
$ minikube service hello-node
```

![Untitled](/assets/img/2023-10-47/Untitled%209.png)

> EXTERNAL -IP 프로비저닝

### 서비스 생성확인

```
$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

![Untitled](/assets/img/2023-10-47/Untitled%2010.png)

> kotacoda 환경에서는 포트 32729를 사용하여 hello-node 서비스에 액세스할 수 있습니다.

> 아래 명령을 실행하여 Kubenetes를 종료합니다.

```
$ kubectl delete service hello-node
$ kubectl delete deployment hello-node
$ minikube stop
$ minikube delete
```

## Kubernetes 배포 실습2(Local Kubernetes Clustering microservice-demo)

[minikube start](https://minikube.sigs.k8s.io/docs/start/)

### 도커 드라이버 유형의 미니큐브 시작

```
> minikube start --driver=docker
```

![Untitled](/assets/img/2023-10-47/Untitled%2011.png)

## 매니페스트 파일 적용

```
> git clone https://github.com/microservices-demo/microservices-demo
> cd microservices-demo
> kubectl apply -f ./release/kubernetes-manifests.yaml
```

![Untitled](/assets/img/2023-10-47/Untitled%2012.png)

> 매니페스트 설정파일을 kubectl apply 명령어를 통해 emailservice, frontend, cartservice 등 12개의 deployment와 service를 생성합니다.

### 매니페스트 파일 적용

```
> kubectl get pods
```

![Untitled](/assets/img/2023-10-47/Untitled%2013.png)

> 클러스터내 생성된 파드를 확인

### 서비스 확인

```
> kubectl port-forward deployment/frontend 8080
```

![Untitled](/assets/img/2023-10-47/Untitled%2014.png)

> 포트포워딩을 통해 서비스에 접속할 수 있습니다.

### 쿠버네티스 로컬 연습

![Untitled](/assets/img/2023-10-47/Untitled%2015.png)

> Virtual Box 가상머신을 활용해서 Docker 및 쿠버네티스, Jenkins를 통한 CI/CD를 충분히 연습해봅니다.

### 오라클 클라우드 배포

![Untitled](/assets/img/2023-10-47/Untitled%2016.png)

> 무중단 배포를 위해 쿠버네티스를 로컬 및 클라우드 환경에서 적용해봅니다.

클라우드 환경 사용시 무료 비용 한계를 꼭 확인해야 합니다.