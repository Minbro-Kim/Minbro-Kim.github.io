---
title: "[데이터모델과 SQL] 정규화(Normalization)"
excerpt: "데이터 정규화"

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, 정규화, Normalization]

permalink: /sql/data-model-sql-1/

toc: true
toc_sticky: true

date: 2025-08-06
last_modified_at: 2025-08-06
---

## 1. 제1정규형: <mark>모든 속성은 반드시 하나의 값 소유</mark>

- 한 속성에 <mark>다중 값</mark> → 개발 복잡성↑ 속성의 의미 모호

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![다중값 문제 예시](/assets/images/posts_img/sql/data-model-sql/sql-1.png)

- 시청 가능 OTT의 값을 추출하기 어려움
</div>

<div class="col-2-item" markdown="1">
![엔티티 추가 해결 예시](/assets/images/posts_img/sql/data-model-sql/sql-2.png)

- 엔티티를 추가하여 다중 값 해결
- 시청 가능한 OTT의 개수가 늘어도 문제X
</div>
</div>

---

- <mark>중복 데이터를 속성으로 분리</mark>

  → 계속된 속성추가로 DBMS에서 테이블 Lock 발생, 향후 모델 변경 힘듦

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-3.png)

- 시청가능 ott를 일일이 열(속성)으로 추가
- ott를 3개까지만 추가 가능
- 4개이상의 ott를 추가하기 위해서는 매번 속성 추가 필요
</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-4.png)

- 엔티티를 추가하여 다중 연결의 제약을 해소
- 안정적 서비스
</div>
</div>

---

## 2. 제2정규형: <mark>엔터티의 일반속성은 주식별자 전체에 종속적</mark>

- <mark>함수종속성(Functional Dependency)</mark>
    - 데이터들이 어떤 기준값에 의해 종속되는 현상
    - `결정자(Determinant)`: 기준값
    - `종속자(Dependent)`: 종속되는 값
    - <mark>종속자는 결정자에 함수적으로 종속성을 가짐</mark>
    - ex. 주문-주문번호,상품번호(상품명)에서 상품명(종속자)은 상품번호(결정자)에 함수 종속성을 가짐

- <mark>부분 종속(Partial Dependency)</mark>: 주식별자 일부에만 함수 종속성을 충족 → 제2정규형 위배

  → 데이터를 변경 시 모든 중복된 데이터 변경 필요 → 변경 중간에 트랜잭션 시 일관되지 않은 데이터

  → 성능과 정합성 문제

  → 부분 종속성 제거

  → 매핑키를 활용하여 분리된 속성 조회(조인; Join)

  → 데이터 일원화로 속성값 변경시 중복값으로 인한 문제X

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-5.png)

- ‘딸기우유’와 ‘우유’와 같은 상품명 중복
- 상품명은 상품번호에는 종속되지만 주문번호에는 종속되지 않아 부분 종속
- 상품명 변경 시 해당 행 전부 변경必
</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-6.png)

- 상품 엔터티를 추가하여 부분 종속성 제거
- 상품번호를 매핑키로 활용하여 상품명을 확인
</div>
</div>

---

## 3. 제3정규형: 엔터티의 일반속성 간에는 서로 종속적X

- <mark>이행적 종속(Transitive Dependency)</mark>: 결정자가 다시 종속자로 기능함
    - ex. 주문번호 → 고객번호, 고객번호 → 고객명 이므로 주문번호 → 고객명

  → 제3정규형 위배

  → 속성값 변경시 제2정규형과 같이 성능과 정합성 문제 발생

  → 제3정규화를 통해 이행적 종속을 제거


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-7.png)

- 고객명은 고객번호에 종속되고, 고객번호는 주문번호에 종속됨 → 제3정규형 위배
- 고객명 변경 시 관련없는 주문 엔터티 변경 필요
</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-8.png)

- 고객 엔터티를 분리하여 고객 속성 변경이 주문 엔터티에 영향X
</div>
</div>

---

## 4. 반정규화와 성능

- 반정규화/역정규화: 정규화를 반대로 진행
    - <mark>성능을 위해 데이터 중복 허용</mark>
    - 주로 조회 성능↑, 입력·수정·삭제 성능↓ → 성능을 고려하여 꼭 필요한 경우만 반정규화 수행

### 가. 반정규화를 적용한 모델에서 성능이 향상될 수 있는 경우

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-9.png)

- 1234고객의 신용카드 결제수단을 미리 세팅하기 위한 SQL
- 주문 내역이 많을수록 조인 증가 → 성능 부하
</div>

<div class="col-2-item" markdown="1">

```sql
SELECT A.결제수단번호
FROM (SELECT B.결제수단번호
  FROM 주문 A, 결제 B
  WHERE A.주문번호 = B.주문번호
      AND A.고객번호 = 1234
      AND B.결제수단구분코드 = '신용카드'
  ORDER BY B.결제일시 DESC
  ) A
WHERE ROWNUM = 1;
```

</div>
</div>


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-10.png)

- 고객번호 속성 반정규화
</div>

<div class="col-2-item" markdown="1">

```sql
SELECT A.결제수단번호
FROM (SELECT A.결제수단번호
      FROM 결제 A
      WHERE A.고객번호 = 1234
        AND A.결제수단구분코드 = '신용카드'
      ORDER BY A.결제일시 DESC
     ) A
WHERE ROWNUM = 1;
```

- 고객번호 + 결제수단구분코드 + 결제일시로 인덱스 생성하여 빠른 조회 가능

</div>
</div>


### 나. 반정규화를 적용한 모델에서 성능이 저하될 수 있는 경우


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-11.png)

</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-12.png)

</div>
</div>

- 송장번호를 조회함의 성능 개선을 위해 반정규화

  → 업무 프로세스 상 송장번호은 배송 시작 이후 결정

  → 주문 엔터티의 송장번호는 NULL에서 UPDATE가 필요(갱신 로직 추가)

  → 불필요한 트랜잭션으로 인한 성능 문제 > 조회 성능 개선 이점

---