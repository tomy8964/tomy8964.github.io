---
title: 이벤트 전파 방지 - e.preventdefault();
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [프론트 , javaScript]
tags:
  [
    프론트,
    e.preventdefault();
  ]

toc: true
toc_sticky: true

---

# e.preventdefault();

## preventDefault란?

- preventDefault는 브라우저가 적용하는 **기본 동작을 방지**하는 역할을 한다.
- 이때 기본 동작은 이벤트의 종류에 따라 다르다. submit인 경우 form 데이터를 서버에 전송하고, 페이지를 새로 고침 하는 부분이 기본 동작이다.
- preventDefault를 사용하면 이러한 기본 동작을 하지 않게 한다.