---
title: git-jenkins-docker-argo 배포
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, CI / CD]
tags:
  [
    백엔드,
	CI / CD,
	gibhub,
	jenkins,
	ci,
	cd,
    docker,
    nginx,
    argo,
    slack,
    build number
  ]

toc: true
toc_sticky: true

---

# git-jenkins-docker-argo 배포

[깃허브-젠킨스-도커-nginx CI/CD 구축 실습](https://tomy8964.github.io/posts/깃허브-젠킨스-도커-nginx-CICD/) 

이 전에 깃 레포에 커밋을 하면 웹훅을 젠킨스에 날려서 젠킨스에서는 깃 레포를 클론 한 뒤 nginx 에 담아서 도커 이미지로 빌드해서 도커 허브에 올리는 것까지 실습하였다.

여기에 몇 가지 사항을 변경하였다.

1. **젠킨스 파이프라인** 파일을 사용하는 것으로 변경하였다.
2. 이 전에는 프로젝트를 빌드한 것을 도커 허브에 올리고 nignx에 담아서 빌드한 것 두 개를 올렸다.  → 이것을 **nginx에 담아서 빌드한 것 하나만 도커 허브**에 올리도록 수정하였다.
3. 빌드 중 성공 실패를 빠르게 알 수 있도록 **Slack에 알람**이 가도록 파이프라인에 추가하였다.
4. 빌드를 성공한 뒤 argo cd 레포를 수정하여서 바로 쿠버네티스로 배포할 수 있도록 추가하였다.
5. **이 때 빌드 버전을 명시하였다. →** 


* 💡 과거에 도커 허브에 올리는 빌드 파일을 latest로만 배포를 하다 보니 argo CD에서 수정을 감지하지 못하고 쿠버네티스에 배포하지 않는 에러가 발생하였었다. 
* 이를 방지 하기 위해 버전을 명시 하였다.


# 젠킨스 파일

```jsx
pipeline {
    agent any
    tools {
        nodejs "nodeJs"
    }

    stages() {
        stage('git clone') {
            steps() {
                slackSend(
                    channel: '#jenkins',
                    color: '#FFFF00',
                    message: "STARTED: Job ${env.JOB_NAME}"
                )
                git branch: 'develop', credentialsId: 'front-github', url: 'https://github.com/SwaveReleaseNote/FrontEnd-template.git/'
            }
        }

        stage('Build Nginx') {
            steps {
                script {
                    // Build and push the Nginx Docker image with the custom configuration
                    def nginxImage = docker.build('tomy8964/release_note_front', '--file Dockerfile .')

                    docker.withRegistry('https://registry.hub.docker.com/repository/docker/tomy8964/release_note_front/', 'dockerHubPwd') {
                        nginxImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }

            post {
                success {
                    slackSend(
                        channel: '#jenkins',
                        color: '#00FF00',
                        message: "SUCCESS: Job ${env.JOB_NAME} Build Front"
                    )
                }
                failure {
                    slackSend(
                       channel: '#jenkins',
                       color: '#FF0000',
                       message: "FAIL: Job ${env.JOB_NAME} Build Front"
                    )
                }
            }
        }

        stage('AgroCD Manifest Update') {
            steps {
                git credentialsId: 'tomy8964',
                        url: 'https://github.com/SwaveReleaseNote/argocd-front',
                        branch: 'main'

                sh "sed -i 's/# build-version:.*/# build-version: ${env.BUILD_NUMBER}/g' front.yaml"
                sh "sed -i 's/release_note_front.*/release_note_front:${env.BUILD_NUMBER}/g' front.yaml"
                sh "git add front.yaml"
                sshagent(credentials: ['git-ssh']) {
                    sh "git commit -m '[UPDATE] v${env.BUILD_NUMBER} image versioning'"
                    sh "git remote set-url origin git@github.com:SwaveReleaseNote/argocd-front.git"
                    sh "git push -u origin main"
                }
            }

            post {
                success {
                    slackSend (
                        channel: '#jenkins',
                        color: '#00FF00',
                        message: "SUCCESS: Job ${env.JOB_NAME} AgroCD Manifest Update"
                    )
                }
                failure {
                    slackSend (
                    channel: '#jenkins',
                    color: '#FF0000',
                    message: "FAIL: Job ${env.JOB_NAME} AgroCD Manifest Update"
                    )
                }
            }

        }
    }

    post {
        success {
            slackSend(
                channel: '#jenkins',
                color: '#00FF00',
                message: "SUCCESS: Job ${env.JOB_NAME}"
            )
        }
        failure {
            slackSend(
                channel: '#jenkins',
                color: '#FF0000',
                message: "FAIL: Job ${env.JOB_NAME}"
            )
        }
    }
}
```

# Dockerfile

```docker
# 1. Using the node image as the base image
FROM node:16-alpine as build

# 3. Set the working directory inside the container
WORKDIR /app

# 4. Copy the package.json and package-lock.json (or pnpm-lock.yaml) to the working directory
COPY package*.json ./

# 2. Install pnpm globally (you can remove this step if you already installed pnpm locally)
RUN npm install -g pnpm

# 5. Install dependencies using pnpm
RUN pnpm install

# 6. Copy the rest of the source code to the container's working directory
COPY . .

# 7. Build your TypeScript code
RUN pnpm build

# Use the official Nginx base image
FROM nginx:latest

# 이전 빌드 단계에서 빌드한 결과물을 /usr/share/nginx/html 으로 복사한다.
COPY --from=build /app/build /usr/share/nginx/html

# Remove the default Nginx configuration
RUN rm /etc/nginx/conf.d/default.conf

# Copy your custom Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/

# Expose the port Nginx will listen on
EXPOSE 3000

# Start Nginx when the container is run
CMD ["nginx", "-g", "daemon off;"]
```

# Nginx nginx.conf

```conf
server {
    listen 3000;
    # server_name mydomain.com;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
            proxy_pass         http://back-service:8080;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /api/sse {
            proxy_pass                  http://back-service:8080;
            proxy_set_header            Connection "";
            proxy_http_version          1.1;
            proxy_set_header            X-Accel-Buffering 'no';
            proxy_set_header            Content-Type 'text/event-stream';
            proxy_buffering             off;
            proxy_set_header            Cache-Control 'no-cache';
            chunked_transfer_encoding   on;
            proxy_read_timeout          86400s;
    }

    location /api/socket { #웹 소켓 연결을 위한 Endpoint
            proxy_pass http://back-service:8080;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
    }

    location /metrics {
        stub_status on;
        access_log off;
        allow all;
    }
}
```

# Slack 알람

![Untitled](/assets/img/2023-10-48/Untitled.png)

# ArgoCD Front.yaml

```yaml
# build-version: 180
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: urnr
  name: front
  labels:
    app: front
spec:
  # replicas: 3
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '9113'
      labels:
        app: front
    spec:
      containers:
        - name: front
          image: tomy8964/release_note_front:180
          imagePullPolicy: Always
          ports:
            - name: front
              containerPort: 3000
          resources:
            limits:
              cpu: 750m
              memory: 2048Mi
        - name: front-exporter
          image: 'nginx/nginx-prometheus-exporter:0.10.0'
          args:
            - '-nginx.scrape-uri=http://localhost:3000/metrics'
          ports:
            - name: ngnix-exporter
              containerPort: 9113
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - front
              topologyKey: kubernetes.io/hostname
```