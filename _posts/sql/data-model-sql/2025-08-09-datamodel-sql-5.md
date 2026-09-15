---
title: "[데이터모델과 SQL] 본질식별자 vs. 인조식별자"
excerpt: "본질식별자과 인조식별자 및 인조식별자 사용의 문제"

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, 식별자, 본질식별자, 인조식별자]

permalink: /sql/data-model-sql-5/

toc: true
toc_sticky: true

date: 2025-08-09
last_modified_at: 2025-08-09
---

## 1. 본질식별자와 인조식별자

- `본질식별자`: 업무에 의해 만들어진 식별자
- `인조식별자` : 본질식별자의 복잡한 구성을 피하기 위해 인위적으로 만든 식별자
  <br>
  <br>
  <br>
  <br>

| 주문번호 | 상품번호 | 주문수량 |
| :---: | :---: | :---: |
| 10 | 1234 | 1 |
| 11 | 1222 | 4 |
| 12 | 123 | 3 |
{: style="display: table !important; width: auto !important; margin: 0 auto !important;"}


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">

- **본질식별자**

![img.png](/assets/images/posts_img/sql/data-model-sql/sql-21.png)

```plsql
INSERT INTO 주문상품 VALUES(10,1234,1);
```

</div>
<div class="col-2-item" markdown="1">

- **인조식별자**

![img.png](/assets/images/posts_img/sql/data-model-sql/sql-22.png)

```plsql
INSERT INTO 주문상품 VALUES(주문상품번호SEQ.NEXTVAL, 10, 1234, 1);
```

</div>
</div>


→ 인조식별자 - 주문상품번호SEQ라는 시퀀스 객체를 생성하고  `NEXTVAL` 기능을 이용

⇒ 불필요한 시퀀스만 생성

---

## 2. 인조식별자 사용시 문제
### 가. 중복 데이터로 인한 품질 문제

- <mark>기본키를 인위적으로 생성</mark>함에 따라 중복된 데이터 발생 우려
- ex. 인조식별자만 다르고 같은 데이터가 오류로 삽입된 경우
    - 본질식별자의 경우 기본키 제약조건(Unique, Not null)에 의해 DBMS에서 원천 차단

---

### 나. 불필요한 인덱스 생성

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">

- **본질식별자**

```plsql
SELECT *
FROM 주문상품
WHERE 주문번호 = :B1;
```

</div>
<div class="col-2-item" markdown="1">

- **인조식별자** 

```plsql
SELECT *
FROM 주문상품
WHERE 주문번호 = :B1
	AND 상품번호 = :B2;
```

</div>
</div>


→ 데이터에 접근할 때 기본 패턴은 주문번호와 상품번호의 인덱스가 필요한 형태

→ 인조식별자로 기본키를 구성하면 <mark>인덱스 추가 생성</mark> 필요

- 본질식별자 - PK: 주문번호+상품번호
- 인조식별자 - PK:  주문상품번호, IX1: 주문번호+상품번호

⇒  인덱스 용량과 DML 성능에 영향

---