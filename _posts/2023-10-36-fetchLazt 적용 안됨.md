---
title: OnetoOne 관계에서 fetch = FetchType.LAZY 적용 안될 시
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

# `@OnetoOne` 관계에서 `fetch = FetchType.LAZY` 적용 안될 시

## 원인
* `cascade = CascadeType.ALL` 이 적용 되어 있을 시 소유 엔터티의 모든 작업(가져오기 작업 포함)이 연결된 엔터티로 계단식으로 전달되기 때문에 `fetch EAGER` 처럼 작동됨

## 해결 방법
* `cascade = CascadeType.ALL` 제거 