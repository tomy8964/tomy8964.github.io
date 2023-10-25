---
title: gitlab-jenkins CI / CD를 위한 시스템 아키텍처 구축실습
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, CI / CD]
tags:
  [
    백엔드,
	CI / CD,
	giblab,
	jenkins,
	ci,
	cd
  ]

toc: true
toc_sticky: true

---

# gitlab-jenkins CI / CD를 위한 시스템 아키텍처 구축실습



# 서론
## CI/CD의 배경

소프트웨어의 규모가 커지고 복잡해지면서 분업과 협업은 필수가 되었습니다. 이 분업과 협업의 과정에서 코드의 Merge 과정은 더욱 더 까다로워졌으며 테스트하는 데에는 더 큰 자원을 소비하게 되었습니다. 이러한 배경 속에서 CI와 CD가 탄생하게 되었습니다.

## CI/CD의 정의

![Untitled](/assets/img/2023-10-45/Untitled.png)

### CI

CI는 `**빌드/테스트 자동화**` 과정을 의미하는 용어로 개발자를 위한 자동화 프로세스이며, `**지속적인 통합(Continuous Integration)**`을 의미합니다. 쉽게 GitHub에 특정 브랜치(master)에 새로운 커밋이 될 때 마다, 해당 코드를 바탕으로 빌드하고 사용자가 미리 만들어둔 테스트 코드를 실행하여 문제가 있는지 없는지를 체크하는 과정을 자동화 한것을 의미합니다.

### CD

CD는 `**배포 자동화 과정**`을 의미하는 용어로 `**지속적 서비스 제공(Continuous Delivery)**` 또는 `**지속적 배포(Continuous Deployment)**`를 의미합니다. 기존에는 빌드 후 문제가 없다고 판단되면 실제 서버든, 클라우드 환경의 서버 환경에 합쳐진 코드(빌드 된 상태의)를 올리는 과정을 하며 이를 `배포한다` 라고 하였습니다. 그런데 위에서 설명한 CI의 과정이 되어 있다면, 우리는 배포 마저도 CI가 완료되는 시점에 자동으로 실행하면 됩니다. 이를 CD라고 합니다.

### 목적

이 실습의 목적은 CI/CD를 위한 시스템 아키텍처 구축을 실제로 해보고 이에 따라 CI/CD 파이프라인의 동작 방식 및 CI/CD의 구성을 이해할 수 있으며 배포 주기 단축 및 빌드/테스트/배포 자동화의 편리함을 느낄 수 있다.

### 사용 하는 툴

- CI: Gitlab
- CD: Jenkins
- WAS: tomcat
- VM: Oracle Vmware
- docker

## CICD 아키텍처

![Untitled](/assets/img/2023-10-45/Untitled%201.png)

# 본론

## 전제조건

- docker 및 docker-compsose가 미리 설치되어 있어야 함
- 우선 원활한 설치를 위해 selinux설정을 disabled 함
- /etc/seliux/config 에서 disabled 함

![Untitled](/assets/img/2023-10-45/Untitled%202.png)

![Untitled](/assets/img/2023-10-45/Untitled%203.png)

## CI 구축

## Jenkins 설치

- docker pull jenkins/jenkins:lts

![Untitled](/assets/img/2023-10-45/Untitled%204.png)

다운로드 받은 이미지를 이용하여 jenkins 실행

- docker run -d -p 8181:8080 --restart=always --name jenkins -u root jenkins/jenkins:lts

![Untitled](/assets/img/2023-10-45/Untitled%205.png)

- 192.168.35.226:8181 접속

![Untitled](/assets/img/2023-10-45/Untitled%206.png)

- docker의 jenkins bash로 접속하여 해당 경로의 암호파일을 열어 접속한다.
    - 초기 암호 확인
    - Bash mode에서 먼저 패키지 업데이트를 해주고 vim 설치
        - apt-get update
        - apt-get install vim
    - vi 편집기로 열어 복사
    
    ![Untitled](/assets/img/2023-10-45/Untitled%207.png)
    

![Untitled](/assets/img/2023-10-45/Untitled%208.png)

- 초기 설정 후 처음 관리자(root) 설정

![Untitled](/assets/img/2023-10-45/Untitled%209.png)

![Untitled](/assets/img/2023-10-45/Untitled%2010.png)

## gitlab 설치

- gitlab-CE 버전을 docker compose 기반으로 설치
- 메모리 8GB, 2 코어
- docker compose 설치
    
    ``` 
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/dockercompose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```
    
    ![Untitled](/assets/img/2023-10-45/Untitled%2011.png)
    
- docker compose version 확인
    - docker-compose —version 확인
    - /etc/sudoers에서 경로 추가 및 재시작
    
    ![Untitled](/assets/img/2023-10-45/Untitled%2012.png)
    
    - Permission denied → docker-compose 폴더에 권한 부여
        - chmod +x /usr/local/bin/docker-compose


* 💡 이때, 저는 vm의 os를 CentOS를 사용하고 있기 때문에 docker-compose —version을 실행하면


![Untitled](/assets/img/2023-10-45/Untitled%2013.png)


* 라는 오류가 발생하였습니다.

* 이에 따라 낮은 버전의 docker-compose를 다운 받았습니다.

![Untitled](/assets/img/2023-10-45/Untitled%2014.png)


- docker-compose —version 확인

![Untitled](/assets/img/2023-10-45/Untitled%2015.png)

## giblab 설치 및 실행

- 홈 디렉토리로 이동 후 docker-compose.yml 파일 생성
- gitlab 설치한 vm의 ip 주소를 확인하고 수정

```
gitlab:
 image: "gitlab/gitlab-ce:latest"
 restart: always
 hostname: "192.168.45.111"
 container_name: gitlab
 environment:
	GITLAB_OMNIBUS_CONFIG: |
		external_url 'http://192.168.45.111'
		# Add any other gitlab.rb configuration here, each on its own line
 ports:
	- "9090:80"
	- "1022:22"
	- "443:443"
 volumes:
	- "~/gitlab/config:/etc/gitlab"
	- "~/gitlab/logs:/var/log/gitlab"
	- "~/gitlab/data:/var/opt/gitlab"
	- "~/gitlab/backups:/var/opt/gitlab/backups"
```

- docker-compose up -d

![Untitled](/assets/img/2023-10-45/Untitled%2016.png)

- 방화벽 설정 및 재시작

![Untitled](/assets/img/2023-10-45/Untitled%2017.png)

- 해당 서버 IP 및 포트로 접속(http://192.168.45.111:9090/)

![Untitled](/assets/img/2023-10-45/Untitled%2018.png)

- 초기 암호 확인

NHAEf8WN7boJTdj5y4exgERpz2pAOY6muzep79//0rQ=

![Untitled](/assets/img/2023-10-45/Untitled%2019.png)

- 암호 변경 및 재로그인

## Gitlab 프로젝트 생성 및 테스트

- gitlab에 해당 프로젝트 및 설정을 하고 작동 테스트
- 프로젝트 생성

![Untitled](/assets/img/2023-10-45/Untitled%2020.png)

- 해당 프로젝트 git 테스트

![Untitled](/assets/img/2023-10-45/Untitled%2021.png)

![Untitled](/assets/img/2023-10-45/Untitled%2022.png)

## giblab과 jenkins 연동

### Jenkins

- gitlab 관련 플러그인 설치 (git, gitlab)

![Untitled](/assets/img/2023-10-45/Untitled%2023.png)

- 프로젝트 생성

![Untitled](/assets/img/2023-10-45/Untitled%2024.png)

- 해당 프로젝트의 구성 메뉴
    - 소스 관리 탭에서 git

![Untitled](/assets/img/2023-10-45/Untitled%2025.png)

![Untitled](/assets/img/2023-10-45/Untitled%2026.png)

- 빌드 유발 탭 설정
    - webhook 설치 시 사용:
    - **http://192.168.219.170:8181/project/saproject**

![Untitled](/assets/img/2023-10-45/Untitled%2027.png)

- 빌드 확인

![Untitled](/assets/img/2023-10-45/Untitled%2028.png)

## Gitlab 및 jenkins 와 연동을 위해 webhook 설정

- 우선 jenkins에서 아래와 같이 “빌드유발” 부분에서 고급버튼을 누르고 “Secret token”을 발생시킴. 이 토큰을 메모해놓고 gitlab 시스템에 입력해야 함.

![Untitled](/assets/img/2023-10-45/Untitled%2029.png)

- gitlab Webhooks 설정

![Untitled](/assets/img/2023-10-45/Untitled%2030.png)

- 생성된 webhooks 테스트

![Untitled](/assets/img/2023-10-45/Untitled%2031.png)

![Untitled](/assets/img/2023-10-45/Untitled%2032.png)

- webhook을 테스트 하면 젠킨스에서 빌드를 실행함

![Untitled](/assets/img/2023-10-45/Untitled%2033.png)

## CD 구축

### Build Steps

- jenkins에서 job 구성에서 Build Steps에서 **Invoke Gradle script** 추가
- **Use Gradle Wrapper** 와 **Make gradlew executable** 선택
- task에는 **clean**과 **bootWar** 설정

![Untitled](/assets/img/2023-10-45/Untitled%2034.png)

### 빌드할 Spring 프로젝트 설정

- war 파일로 만드는 was에 올라갈 프로젝트 설정
- build gradle에 id war 와 **bootWar** 추가

```
plugins {
	id 'java'
	id 'war'
	id 'org.springframework.boot' version '2.7.12'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

bootWar{
	archiveBaseName="springproject"
	archiveVersion="1.0.1-SNAPSHOT"
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	implementation 'org.springframework.boot:spring-boot-starter-web'
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

- Jenkins 빌드 후 결과 확인

![Untitled](/assets/img/2023-10-45/Untitled%2035.png)

### 플러그인 설치

- jenkins에 **Deploy to container Plugin** 설치

![Untitled](/assets/img/2023-10-45/Untitled%2036.png)

### WAS(tomcat) 에 배포하기 위한 계정 생성

- **tomcat/tomcat-uesrs.xml** 파일에 사용자 계정과 권한 추가

![Untitled](/assets/img/2023-10-45/Untitled%2037.png)

### tomcat manager(배포할때 필요) 사용하기 위한 설정

- **manager.xml**

![Untitled](/assets/img/2023-10-45/Untitled%2038.png)

<aside>
💡 WAS의 웹페이지 접속 시: 404 NOT FOUND 오류 발생

</aside>

```docker
~$ mv webapps webapps2

~$ mv webapps.dist/ webapps
```

### 젠킨스에 빌드 후 조치 설정

- 젠킨스 job의 ‘**빌드 후 조치**’ 설정
- **Deploy war/ear to a container** 추가
- 배포할 war 파일의 경로 설정
- 배포될 경로 설정 (이후에 /demo로 수정함)
- **tomcat-users**에 설정한 계정 등록

![Untitled](/assets/img/2023-10-45/Untitled%2039.png)

### 배포 테스트

<aside>
💡 gitlab 푸쉬 → jenkins에서 war로 빌드 → jenkins에서 WAS로 war 배포

</aside>

- gitlab에 프로젝트 푸쉬

![Untitled](/assets/img/2023-10-45/Untitled%2040.png)

- 젠킨스 빌드 실행

![Untitled](/assets/img/2023-10-45/Untitled%2041.png)

![Untitled](/assets/img/2023-10-45/Untitled%2042.png)

- WAS 배포 확인
- [`http://172.16.48.186:8080/manager/html`](http://172.16.48.186:8080/manager/html) 로 설정한 계정 정보로 접속
- **/demo** 배포 확인
    
    ![Untitled](/assets/img/2023-10-45/Untitled%2043.png)
    
- 톰캣 webapps에 배포된 파일 확인

![Untitled](/assets/img/2023-10-45/Untitled%2044.png)

# 결론

## 결과

* CI/CD 구축 실습 결과로는 빌드, 테스트 및 배포 프로세스를 자동화하여 소프트웨어 배포 주기에 필요한 시간을 줄였으며 오류를 감소시켰고 안정적이고 일관된 소프트웨어 배포를 할 수 있게 되었습니다.

## 느낀점

* CI/CD 구축 실습을 통해 전체적인 CICD 과정을 알게 되었으며 CI와 CD에 대한 정의와 탄생하게 된 배경, 필요한 이유 그리고 적용했을 때의 장점을 알게 되었습니다. 
* 그 전에는 말로만 듣고 이해했을 때의 CICD 파이프라인을 직접 구축을 해보니 CICD가 어떻게 구성되었으며 어떻게 작동하는지 이제 완벽히 알게 되었으며 CICD가 주는 편리함에 대해 느끼게 되었습니다.