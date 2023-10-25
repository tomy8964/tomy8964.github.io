---
title: Unable to evaluate expression. Method threw exception 'org.hibernate.LazyInitializationException'.
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

# Unable to evaluate expression. Method threw exception 'org.hibernate.LazyInitializationException'.
![image](https://github.com/tomy8964/Spring-Study/assets/103511161/70d10b21-51e9-46bc-b0c0-cc664e9f9cb6)


## 원인
* 프록시를 초기화해야 하는데 영속성 컨텍스트가 없으므로 실제 엔티티를 조회할 수 없어서 발생하는 예외이다. 
* 즉, `no Session`, 같은 세션에 있는 영속화된 엔티티가 아니라서 조회가 안 된다는 것이다.

## 해결 방법
* `@Transaction`으로 감싸서 같은 논리적 트랜잭션에 속하게 하여서 해결하였습니다.