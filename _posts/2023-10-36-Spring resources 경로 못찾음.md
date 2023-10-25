---
title: SpringBoot - .jar 배포 후 resources 파일 경로 못 찾는 오류
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, 트러블 슈팅]
tags:
  [
    Spring,
    백엔드,
    트러블 슈팅
  ]

toc: true
toc_sticky: true

---

# SpringBoot `.jar` 배포 후 `resources` 파일 경로 못 찾는 오류

## 원인
* 보통 IDE등에서 개발하고 빌드하게 되면 파일 경로로 잘 접근하는데, `jar`로 배포하면 기존 파일 접근 코드로는 문제가 생긴다.

* 왜냐하면 유효하지 않은 경로가 되어버리게 때문이다.

* `jar` 파일은 루트 경로를 참고해보면 `jar:file:/` 로 시작하는 경로값을 가진다.

* 로컬에서는 실제 `resource` 파일인 `file:/` 로 해당 경로를 찾는다.

## 해결 방법
* 스프링에서는 여러가지 형태로 저장된 리소스를 패턴으로만 찾아서 가져올 수 있도록 도구를 제공한다.
```java
Resource[] resources = ResourcePatternUtils
    .getResourcePatternResolver(new DefaultResourceLoader())
    .getResources("classpath*:**");
```
* 위의 코드를 사용하면 resources 파일의 모든 파일의 경로를 배열로 가져온다.