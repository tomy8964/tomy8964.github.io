---
title: Querydsl QEntity 재생성 오류 (Attempt to recreate a file for type com.xxx.Qxxx)
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

# Querydsl QEntity 재생성 오류 (Attempt to recreate a file for type com.xxx.Qxxx)
## 원인

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/84f95011-6601-4331-9802-c203ba0c96b5)

* Querydsl의 QEntity가 이미 존재하여서 재생성 오류가 발생

## 해결 방법
![image](https://github.com/tomy8964/Spring-Study/assets/103511161/17fe84f6-c48b-4594-bcff-37a14d41c6b0)
* gradle에서 querydsl을 `clean`
* 다시 `build` -> `QEntity` 생성
