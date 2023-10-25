---
title: ProcessBuilder를 사용하여 python 파일을 실행할 때 Process exited with code 9009
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

# ProcessBuilder를 사용하여 python 파일을 실행할 때 `Process exited with code 9009`

## 원인
```java
builder = new ProcessBuilder("python", substring, String.valueOf(surveyDocumentId));

builder.redirectErrorStream(true);
Process process = builder.start();

// 자식 프로세스가 종료될 때까지 기다림
int exitCode;
try {
    exitCode = process.waitFor();
} catch (InterruptedException e) {
    // Handle interrupted exception
    exitCode = -1;
}

if (exitCode != 0) {
    BufferedReader errorReader = new BufferedReader(new InputStreamReader(process.getErrorStream()));
    String errorLine;
    System.out.println("Error output:");
    while ((errorLine = errorReader.readLine()) != null) {
        System.out.println(errorLine);
    }
}

System.out.println("Process exited with code " + exitCode);
```

* `processbulider`에 “`python`" 를 인자로 주어 파이썬 파일을 실행할 때, 시스템은 시스템의 `PATH` 환경 변수에서 `Python` 실행 파일을 찾으려고 시도합니다. 이때 문제가 발생하였습니다.

## 해결 방법
* 파이썬 재설치
* `PATH` 환경 변수를 재설정하여 해결했습니다.