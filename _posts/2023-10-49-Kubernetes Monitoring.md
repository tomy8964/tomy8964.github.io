---
title: Kubernetes Monitoring
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, k8s]
tags:
  [
    백엔드,
    k8s,
    Monitoring,
    Helm,
    Prometheus,
    Grafana
  ]

toc: true
toc_sticky: true

---

# Kubernetes Monitoring

> Prometheus and Grafana


# 모니터링이란?

> 시스템의 **관측가능성**(Observability)를 확보합니다.


- 시스템 관리에서 모니터링이란 **메트릭(Metric)**이라는 시계열(Time-series)로 표현한 수치를 지속적으로 **측정 및 수집**함으로써 **시스템**이 어떤 **상태**에 있는지 **확인**
- CPU 사용률(Utilization), Ram 이용률(Usage), 트래픽 지연 시간(Latency) 등 숫자로 표현되는 데이터를 시간 별로 수집하고 시각화

# Architecture

![Untitled](/assets/img/2023-10-49/Untitled.png)

- kubernetes 리소스의 metric(사용 현황) 정보를 수집(prometheus)하여 대시보드로 제공(grafana) 할 수 있도록 하기 위해 **prometheus**와 **grafana** 툴을 이용합니다.
- Prometheus는 메트릭(Metric)을 포함한 시계열(Time-series) **데이터를 수집**하고 이를 기반으로 **쿼리(Query)**하거나 **경고(Alerting)** 할 수 있게 해주는 오픈 소스 모니터링

# Helm

> Helm 저장소(repositoty)를 등록해야 합니다.

- kubenetes Helm은 컨테이너화된 애플리케이션의 배포와 관리를 돕는 kubernetes용 **패키지 관리자**
- Deployment, Service, Ingress 와 같은 kubernetes 리소스를 사용하여 애플리케이션을 관리
- **Helm chart**는 kubernetes 리소스와 응용 프로그램 간의 종속성, 템플릿 및 다양한 메타 정보를 관리하기 위한 패키지 정보를 담고 있는 파일
- 스테이징, 프로덕션, 개발 등 다양한 환경에서 쉽게 애플리케이션 배포
- 버전관리, 배포방식, 롤백 등의 기능도 가지고 있기 때문에 릴리즈를 쉽게 관리

## Helm 설치

[Helm 설치 링크](https://helm.sh/docs/intro/install/)

> Helm은 서비스를 쉽게 설치할 수 있는 툴입니다.


> 서비스 배포에 필요한 Image, Volumn, 환경설정 등을 정의한 helm chart를 통해 빠르고 쉽게 배포하게 해 줍니다.


![Untitled](/assets/img/2023-10-49/Untitled%201.png)

- Helm-CLI 툴로 helm 명령어를 사용해서 kubernetes master node의 API Server에 요청하여 배포를 수행합니다.
- 따라서, helm을 수행하는 terminal(PC 또는 VM등)에 kubernetes cluster 인증/인가 정보를 담고 있는 kube configuration 파일(yaml 파일)이 있어야 합니다.
- Jenkins와 같은 CI/CD 툴에서 CD파트에 helm을 이용할 수 있습니다.

## Helm 레포지토리 등록

- Helm을 설치했다면 버전 확인을 합니다.

![Untitled](/assets/img/2023-10-49/Untitled%202.png)

- 모니터링 툴을 설치하기 위해 레포지토리를 등록하고 확인합니다.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

- 다운로드를 원한다면 `git clone https://github.com/prometheus-community/helmcharts.git`하고 관련 폴더에서 yaml 파일을 참조합니다.

# Prometheus

## 프로메테우스(Prometheus) 아키텍처

[https://prometheus.io/](https://prometheus.io/)

![Untitled](/assets/img/2023-10-49/Untitled%203.png)

![Untitled](/assets/img/2023-10-49/Untitled%204.png)

- 프로메테우스 서버가 각각의 노드(컴포넌트)가 수집한 데이터를 가져오는 구조입니다.
- 서버에서 클라이언트가 동작 중인 상태를 확인하면 주기적으로 클라이언트에 접속해서 데이터를 가져오는 pull 방식으로 동작합니다.

## Prometheus Components

**Prometheus Server**

- 데이터를 수집하고 저장하는 메인 서버
- target 메트릭 데이터 수집
- HTTP 엔드포인트를 설정하여 서버에서 메트릭을 수집하도록 지원
- HAProxy, StatsD, Graphite와 같은 서비스를 지원

**Alertmanager**

- 설정한 규칙에 따른 알림 처리

**Pushgateway**

- Exporter를 이용하여 수집이 어려운 작업에 대한 메트릭 pushing 지원

**Grafana**

- 데이터 시각화를 위한 Web UI 및 대시보드 지원

**Client Libraries**

- Java, Scala, Go, Python, Ruby 등의 언어에 대한 프로메테우스 연동 라이브러리

## Prometheus 설치

```
helm install prometheus-stack prometheus-community/kube-prometheus-stack
```

![Untitled](/assets/img/2023-10-49/Untitled%205.png)

## Prometheus 실행

```
kubectl get po
kubectl port-forward svc/prometheus-stack-kube-prom-prometheus 9090:9090
```

![Untitled](/assets/img/2023-10-49/Untitled%206.png)

![Untitled](/assets/img/2023-10-49/Untitled%207.png)

- [localhost:9090](http://localhost:9090) 접속
- 기간별 kube_service_info에 대한 자료를 수집하고 리스트화하여 보여줍니다.

![Untitled](/assets/img/2023-10-49/Untitled%208.png)

# Grafana

## grafana 설치

> prometheus의 데이터를 읽어 대시보드를 제공하는 툴입니다.


![Untitled](/assets/img/2023-10-49/Untitled%209.png)

- 그라파나(Grafana)는 메트릭을 시각화 해주는 오픈 소스 도구
- Graphite, Prometheus, InfluxDB 등 다양한 데이터베이스와 메트릭 수집 시스템을 지원하고, 하나의 대시보드에 동시에 여러 메트릭 시스템들의 지표를 표시

## grafana 설치 확인

> 계정은 admin, 초기 비밀번호는 prom-operator로 접속


```
kubectl get services
kubectl port-forward svc/prometheus-stack-grafana 3000:80
```

![Untitled](/assets/img/2023-10-49/Untitled%2010.png)

- [localhost:3000](http://localhost:3000) 접속

![Untitled](/assets/img/2023-10-49/Untitled%2011.png)

## grafana의 prometheus 연동

- DATA SOURCES 클릭

![Untitled](/assets/img/2023-10-49/Untitled%2012.png)

![Untitled](/assets/img/2023-10-49/Untitled%2013.png)

- Prometheus 클릭
- URL은 prometheus의 정보를 받아올 수 있는 주소와 포트로써, 프로메테우스의 서버 정보입니다.
- [http://prometheus-stack](http://prometheus-stack/)-kube-promprometheus:9090
- Save & test 클릭

![Untitled](/assets/img/2023-10-49/Untitled%2014.png)

## 대시보드 작성

- Grafana Labs에서 사용하고자 하는 대시보드 템플릿의 ID 확인

![Untitled](/assets/img/2023-10-49/Untitled%2015.png)

- import dashboard에 붙여넣고 Load (저는 11074 id를 사용했습니다)

![Untitled](/assets/img/2023-10-49/Untitled%2016.png)

## 대시보드 확인

![Untitled](/assets/img/2023-10-49/Untitled%2017.png)