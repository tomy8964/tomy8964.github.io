---
title: 자바에서 process bulider를 통해 파이썬 파일을 실행했을때 무한 로딩 오류
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

# 자바에서 process bulider를 통해 파이썬 파일을 실행했을때 무한 로딩 오류
```java
builder = new ProcessBuilder("python", "./src/main/resources/python/python2.py", String.valueOf(surveyDocumentId));

builder.redirectErrorStream(true);
Process process = builder.start();

// 자식 프로세스가 종료될 때까지 기다림
process.waitFor();

//// 서브 프로세스가 출력하는 내용을 받기 위해
br = new BufferedReader(new InputStreamReader(process.getInputStream(),"UTF-8"));

String line = br.readLine();
```
## 원인
* 자바에서 `process builder`를 통해 파이썬 파일을 실행시키고 파이썬에서 `print`한 결과를 자바에서 `String line`으로 받는데
* 이때 파이썬 파일에서 결과말고도 다른 **print문이 너무 많아 무한 로딩 오류**가 발생하였습니다.

## 해결 방법
* 파이썬 파일에서 필요한 결과에 대해서만 print
* 나머지 필요 없는 print 문 삭제
