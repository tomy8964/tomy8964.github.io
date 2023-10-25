---
title: REST API 통신시 WebClient Exceeded limit on max bytes
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

# REST API 통신시 `WebClient Exceeded limit on max bytes`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/53f33b4d-991a-41a1-b322-a1e7aef6e7d0)

## 원인
* `WebClient`에 설정되는 `default codec`의 `buffer size`를 초과했을 때 발생했습니다.

## 해결 방법
* `buffer size`의 크기를 설정해주었습니다.