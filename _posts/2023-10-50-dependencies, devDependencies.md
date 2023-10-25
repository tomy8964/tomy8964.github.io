---
title: dependencies, devDependencies
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [프론트 , etc]
tags:
  [
    프론트,
    dependencies,
    devDependencies,
    라이브러리
  ]

toc: true
toc_sticky: true

---

# dependencies, devDependencies

## 차이점

- dependencies에는 애플리케이션 동작과 연관된 라이브러리가 있다
- devDependencies에는 애플리케이션 동작과 직접적인 연관은 없지만, 이름 그대로 개발할 때 필요한 라이브러리가 있다

## 구분하는 이유는?

- 배포할 때 어떤 라이브러리를 포함할지 정해서 빌드 시간도 줄이고, 배포할 때 불필요한 라이브러리를 포함시키지 않기 위해
- dependencies에 설치된 라이브러리만 배포할 때 포함된다.