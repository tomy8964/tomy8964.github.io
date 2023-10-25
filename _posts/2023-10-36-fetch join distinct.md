---
title: OneToMany 관계의 entity를 Querydsl로 조회할 때 fetchjoin을 사용하면 데이터가 중복되어 조회될 수 있다.
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

# `OneToMany 관계의` `entity`를 `Querydsl`로 조회할 때 `fetchjoin`을 사용하면 데이터가 중복되어 조회될 수 있다.

## 해결 방법
```java
public SurveyDocument findSurveyById(Long surveyDocumentId) {
    SurveyDocument survey = queryFactory
            .selectFrom(surveyDocument)
            .leftJoin(surveyDocument.design, design).fetchJoin()
            .leftJoin(surveyDocument.date, dateManagement).fetchJoin()
            .leftJoin(surveyDocument.questionDocumentList, questionDocument).fetchJoin()
            .where(surveyDocument.id.eq(surveyDocumentId))
            .distinct()
            .fetchOne();

    return survey;
}
```
* `distinct`를 추가하여 중복된 row를 제거할 수 있다.
* 하지만 db에 날아간 쿼리의 결과에는 중복이 있으며
* 쿼리를 통해 db에서 가져온 데이터를 Entity 차원에서 중복을 제거하는 것이다.