---
title: 3티어기반 HAProxy 및 Keepalived 적용
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, 아키텍처]
tags:
  [
    백엔드,
    아키텍처,
    웹 시스템,
    3티어,
    VM,
    리눅스,
    Docker,
    WS,
    Nginx,
    Was,
    Tomcat,
    DB,
    HAProxy,
    Keepalived
  ]

toc: true
toc_sticky: true

---

# 3티어기반 HAProxy 및 Keepalived 적용


- 아키텍처

![Untitled](/assets/img/2023-10-44/Untitled.png)

- 전제조건
    - HAProxy1 서버는 VM으로 작동 (192.168.35.141)
    - HAProxy1 서버는 VM으로 작동 (192.168.35.125)
    - WS(Web Server)는 Nginx
        - WS1: 192.168.35.86
        - WS2: 192.168.35.226
    - WAS(Web Application Server)는 Tomcat
        - WAS1: 192.168.0.103
        - WAS2: 192.168.0.104
    - DB(DataBase)는 MySQL(Maria)
        - DB: 192.168.0.105
    - WS,  WAS 및 DB는 구축되어 있다는 것을 전제로 HAProxy 설치 및 연동을 진
    행함.

![Untitled](/assets/img/2023-10-44/Untitled%201.png)

![Untitled](/assets/img/2023-10-44/Untitled%202.png)

![Untitled](/assets/img/2023-10-44/Untitled%203.png)

# HAProxy 서버 구축 (Load balancer 설정)

![Untitled](/assets/img/2023-10-44/Untitled%204.png)

- HAProxy는 트래픽이 많은 웹기반 솔루션의 부하를 효율적으로 분산시켜 줄
수 있는 무료 오픈소스 로드밸런서임. 또한 이중화를 위한 Failover 기능도 제
공함.
- host파일에 정보업데이트

![Untitled](/assets/img/2023-10-44/Untitled%205.png)

- HAProxy 설치
    - yum update -y
    - yum install bind-utils
    - yum install yum-utils
    - yum install haproxy

![Untitled](/assets/img/2023-10-44/Untitled%206.png)

![Untitled](/assets/img/2023-10-44/Untitled%207.png)

![Untitled](/assets/img/2023-10-44/Untitled%208.png)

![Untitled](/assets/img/2023-10-44/Untitled%209.png)

![Untitled](/assets/img/2023-10-44/Untitled%2010.png)

![Untitled](/assets/img/2023-10-44/Untitled%2011.png)

![Untitled](/assets/img/2023-10-44/Untitled%2012.png)

![Untitled](/assets/img/2023-10-44/Untitled%2013.png)

![Untitled](/assets/img/2023-10-44/Untitled%2014.png)

![Untitled](/assets/img/2023-10-44/Untitled%2015.png)

![Untitled](/assets/img/2023-10-44/Untitled%2016.png)

![Untitled](/assets/img/2023-10-44/Untitled%2017.png)

![Untitled](/assets/img/2023-10-44/Untitled%2018.png)

![Untitled](/assets/img/2023-10-44/Untitled%2019.png)

![Untitled](/assets/img/2023-10-44/Untitled%2020.png)

![Untitled](/assets/img/2023-10-44/Untitled%2021.png)

# HAProxy를 이용한 Failover구축(이중화) 설정

- 앞서 구축된 LB이외에 HAProxy는 HA(High Availability)-고가용성 기술을 제공함.
L4 & L7수준에서 이중화 기술을 제공함.
- 앞단(Frontend)의 HAProxy LB가 하나이고 만약에 이슈가 발생하여 작동이 안된
다면?

![Untitled](/assets/img/2023-10-44/Untitled%2022.png)

- 고가용성을 위해 Backup HAProxy을 하나 더 구축하여 Active HAProxy가 문제가
발생 시 자동으로 Failover되어 Backup HAProxy가 작동되도록 함.
    - 이를 위해 VIP 와 추가 HAProxy서비스의 IP 및 포트정보가 추가되어야 함.
    - VIP(Virtual IP)는 VRRP(Virtual Router Redundancy Protocol)을 기반으로 이중화
    구조를 구현하기 위한 가상 IP임. 또한 VRRP는 Active/Standby, Master/Slave
    구조의 이중화 시스템을 구현하기 위해 가상의 게이트웨이 기반의 라우
    팅 기술임.
    - 여기서는 WS1~WS2 범위까지만 이중화구조로 구현하였음.

| Host | IP | Port | Remark |
| --- | --- | --- | --- |
| VIP | 192.168.35.207 |  | Haproxy1~2 해당 |
| haproxy1 | 192.168.35.141 | 80 | Master(keepalived & LB) |
| haproxy2 | 192.168.35.125 | 80 | Backup(keepalived & LB) |
| ws1 | 192.168.35.86 | 80 | Nginx |
| ws2 | 192.168.35.226 | 80 | Nginx |

![Untitled](/assets/img/2023-10-44/Untitled%2023.png)

![Untitled](/assets/img/2023-10-44/Untitled%2024.png)

![Untitled](/assets/img/2023-10-44/Untitled%2025.png)

![Untitled](/assets/img/2023-10-44/Untitled%2026.png)

![Untitled](/assets/img/2023-10-44/Untitled%2027.png)

![Untitled](/assets/img/2023-10-44/Untitled%2028.png)

![Untitled](/assets/img/2023-10-44/Untitled%2029.png)

![Untitled](/assets/img/2023-10-44/Untitled%2030.png)

![Untitled](/assets/img/2023-10-44/Untitled%2031.png)

![Untitled](/assets/img/2023-10-44/Untitled%2032.png)

![Untitled](/assets/img/2023-10-44/Untitled%2033.png)

![Untitled](/assets/img/2023-10-44/Untitled%2034.png)

![Untitled](/assets/img/2023-10-44/Untitled%2035.png)

![Untitled](/assets/img/2023-10-44/Untitled%2036.png)

![Untitled](/assets/img/2023-10-44/Untitled%2037.png)

![Untitled](/assets/img/2023-10-44/Untitled%2038.png)

![Untitled](/assets/img/2023-10-44/Untitled%2039.png)

![Untitled](/assets/img/2023-10-44/Untitled%2040.png)

# 트러블 슈팅

* 처음에는 VIP라는 VM을 파서 그 VM의 IP를 넣어줘야 하는지 알았습니다. 
* 하지만 VIP는 중복성을 구현하고 네트워크 서비스의 고가용성을 보장하는 데 사용되는 가상 IP 주소이라는 것을 알고 만들어 놓은 VIP의 VM을 삭제하고 하니 잘 작동하였습니다.

* restart 중요