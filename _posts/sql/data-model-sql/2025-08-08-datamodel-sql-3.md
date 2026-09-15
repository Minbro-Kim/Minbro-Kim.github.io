---
title: "[데이터모델과 SQL] 모델이 표현하는 트랜잭션"
excerpt: "모델이 표현하는 트랜잭션의 이해"

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, 트랜잭션, Transaction]

permalink: /sql/data-model-sql-3/

toc: true
toc_sticky: true

date: 2025-08-08
last_modified_at: 2025-08-08
---

## 트랜잭션(Transaction)

- 데이터베이스의 <mark>논리적 연산단위</mark>
- 하나의 <mark>업무 단위로 묶여서 처리(Commit)</mark> → 전부 실행  or 전부 취소 (`All or Nothing`; `원자성`)
- 모델의 트랜잭션은 업무적 트랜잭션보다 재사용성 이점↓ → 반드시 하나의 트랜잭션으로 처리
    - 업무적 트랜잭션은 사기 검사, 한도확인, 이체실행 등을 묶어 하나의 트랜잭션으로 처리
    - 모델 트랜잭션은 DB 입장에서 동시에 발생하는 이체실행(차감과 가산이 한 번에 실행)을 하나의 트랜잭션으로 처리
- ex. 계좌이체(출금계좌 차감 & /입금계좌 가산)
- 잘못된 트랜잭션 → 데이터 정합성 문제, 데이터 품질 하락

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-19.png)

- 주문과 주문상세를 필수적 관계로 표시
- 태생적 동시 발생 데이터
</div>

<div class="col-2-item" markdown="1">

![img.png](/assets/images/posts_img/sql/data-model-sql/sql-20.png)

- 주문과 주문상세를 선택적 관계로 표시

  → 습관적 표현으로 주의 필요
</div>
</div>


```text
//Pseudo Code
//A->B 계좌이체
Step1. 계좌이체API {
  잔고수정(고객번호=>A, 수정값=>현재잔고-이체금액);
  잔고수정(고객번호=>B, 수정값=>현재잔고+이체금액);
  commit();
  }	
```

- 차감과 가산이 완료된 수 커밋 수행
- 두 단계를 분리할 경우, 중간에 장애 발생 시 데이터 정합성 문제 발생

---