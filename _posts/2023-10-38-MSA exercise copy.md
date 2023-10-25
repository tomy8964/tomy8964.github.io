---
title: MSA 실습
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [백엔드, MSA]
tags:
  [
    Spring,
    백엔드,
    MSA
  ]

toc: true
toc_sticky: true

---
# MSA 실습

# msaez 접속

- msaez 사이트 접속 : [https://www.msaez.io.com](https://www.msaez.io/#/)
- 깃허브 계정으로 로그인

# EventStorming Design

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/9ae185c7-61a4-4632-8270-aa75c4f96c0f)

# Hexagonal

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/1ec13cb6-c161-4e17-9161-9b92063ac041)

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/eaca3cce-2142-4e02-b28b-1fe4b73220c5)

# 쿠버네티스 확인(k8s)

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/7e59cfa2-2373-4005-a416-8f0437764bf0)

# Code Preview

# Github push

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/0554700c-71a6-43da-8611-936a189c9f6a)

# 코드 확인 및 수정

## `external/HelloServiceImpl.java`

```java
public class HelloServiceImpl implements HelloService {
	/*FallBack*/
	@Override
	public void hello(Hello hello){
		throw new UnsupportedOperationException("Unimplemented method 'hello'");
	}
}
```

# 터미널에서 `mvn spring-boot:run` 실행

## `InHello`

## `OutHello`

## `gateway`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/973935cc-6090-498f-b926-05668a0018d0)

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/6550d898-8270-441e-b807-387db5db65d9)

# 포트창에서 브라우저 클릭해서 확인

## 8081

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/422c42b7-1449-45fc-8b0f-a235108db85c)

## 8082

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/ada24854-0cd1-4a42-8764-745084298e28)

# httpie를 실행해서 확인

## `http POST [localhost:8088/hellos](http://localhost:8088/hellos) hello=”hello”`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/421f92dd-96e2-48fb-886b-efcebfdae410)

## `http POST [localhost:8088/](http://localhost:8088/hellos)worlds world=”world”`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/0bcd424a-cd12-4b36-bdf3-b7abe87379e7)

# 카프카로 서비스간 통신 (Pub/Sub)

## `kafka-console-consumer —bootstrap-server [localhost:9092](http://localhost:9092) —topic gcumsa2`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/ab2ba8b1-b7f8-4606-9e18-8c09fe2fb390)

## `http :8088/hellos id=1 hello=”hello”`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/0be0326f-5fba-4a38-b7e8-2100923ab976)

## `http :8088/worlds id=1 world=”helloworld”`

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/9df1cc95-dd70-4315-adce-bfaea17c8884)

![image](https://github.com/tomy8964/Spring-Study/assets/103511161/fd88403e-80b1-49e5-89fa-d0a2aa5e8a7b)

----
References: 가천 SW 아카데미