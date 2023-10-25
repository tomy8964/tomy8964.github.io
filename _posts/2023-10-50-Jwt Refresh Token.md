---
title: Jwt Refresh Token
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드 , Security]
tags:
  [
    백엔드,
    Security,
    JWT,
    Refresh,
    Token,
    Selectors
  ]

toc: true
toc_sticky: true

---

# Jwt Refresh Token

## Refresh Token이란?

- 서버측 리소스에 접근할 때 클라이언트는 본인을 인증할 수 있는 엑세스 토큰으로 동작한다
    - 액세스 토큰은 Stateless하기 때문에 본인이 맞는지 확인할 수는 없다
- 리프레쉬 토큰은 사용자 인증이 아닌 새로운 액세스 토큰을 생성하는 용도로만 사용

## Refresh Token을 사용하는 이유?

- Access Token의 유효 기간을 짧게 설정
- Refresh Token의 유효 기간은 길게 설정
- 클라이언트는 둘 다 서버에 전송한다
- 탈취자는 access token을 탈취해도 짧은 유효 기간이 지나면 사용할 수 없다

## 토큰의 저장 장소

- 서버에서는 NoSQL이나 기타 데이터베이스에 저장할 수 있다. 그럼 클라이언트에서는 어디에 저장할 수 있을까? 브라우저 환경의 경우 흔히 생각하는 방법은 쿠키, 로컬 스토리지 등 다양한 곳이 있지만 스택오버플로우에서는 http-only 속성이 부여된 쿠키에 저장하는 것을 권장하고 있다.
- 왜냐면 해당 속성이 부여된 쿠키는 자바스크립트 환경에서 접근할 수 없기 때문이다. 그래서 XSS나 CSRF가 발생하더라도 토큰이 누출되지 않는다. 일반 쿠키나 브라우저의 로컬 스토리지는 자바스크립트로 자유롭게 접근할 수 있기 때문에 보안 측면에서는 권장되지 않는다.