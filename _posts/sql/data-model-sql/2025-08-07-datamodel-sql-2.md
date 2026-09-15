---
title: "[데이터모델과 SQL] 관계와 조인(Join)의 이해"
excerpt: "데이터 관계와 조인의 이해"

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, 정규화, Join, 조인, 계층형, 배타적]

permalink: /sql/data-model-sql-2/

toc: true
toc_sticky: true

date: 2025-08-07
last_modified_at: 2025-08-07
---

## 1. 조인(Join)

- <mark>상속된 속성을 매핑키로 활용</mark>하여 데이터를 결합 → 데이터를 연결해서 봄
  - 매핑키 → 조인키(Join Key)


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-13.png)

</div>

<div class="col-2-item" markdown="1">

```sql
SELECT B.고객명
FROM 주문 A, 고객 B
WHERE A.주문번호 = 1
	AND A.고객번호 = B.고객번호; 
```

- `고객번호`를 조인키로 하여 `고객명`을 찾음
</div>
</div>

---

## 2. 계층형 데이터 모델(Hierarchical Data Model)

- 계층형 데이터 모델
  - 계층 구조를 가진 데이터 → 계층형 쿼리(Connect By절)
  - `부모_id` 같은 컬럼으로 트리처럼 표현하는 방식 → 자신의 상위 카테고리를 재귀적으로 참조
  - ex. 카테고리, 사원-관리자 관계

  ⇒ **<mark>셀프 조인(Self Join)</mark>**


<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-14.png)

- 관리자번호는 같은 테이블의 사원번호를 참조
</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-15.png)

</div>
</div>

![img.png](/assets/images/posts_img/sql/data-model-sql/sql-16.png)

```sql
SELECT B.ENAME
FROM EMP A, EMP B
WHERE A.ENAME = '히나타'
	AND A.MGR = B.EMPNO;
```

---

## 3. 상호배타적 관계(Exclusive-OR)

- <mark>자식엔터티의 한 속성이 복수의 부모앤터티로부터 식별자를 상속받을 수 있을 때, 하나의 부모엔터티로부터만 상속을 받을 수 있음</mark> → 하나의 테이블만 조인
- 부모엔터티끼리 식별자가 겹치지 않도록 주의 → 중복되는 경우 구분코드 필요

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-17.png)

- 주문데이터에서 번호 속성에서 개인 또는 법인 고객 식별자 중 하나만 상속
- 고객 구분코드를 통해 부모 엔터티를 구분
</div>

<div class="col-2-item" markdown="1">
![img.png](/assets/images/posts_img/sql/data-model-sql/sql-18.png)

</div>
</div>


### 가. UNION ALL 이용

```sql
SELECT B.개인고객명
FROM 주문 A, 개인고객 B
WHERE A.주문번호 = 1001
	AND A.고객구분코드 = '01'
	AND A.개인/법인번호 = B.개인번호
UNION ALL
SELECT B.법인명
FROM 주문 A, 법인고객 B
WHERE A.주문번호 = 1001
	AND A.고객구분코드 = '02'
	AND A.개인/법인번호 = B.법인번호;
```

- 개인번호와 법인번호의 중복 우려

  → 구분코드 이용

- `UNION ALL` 을 이용하여 합집합 중 주문번호가 일치하는 행 출력
- 결과X 경우 → `공집합(NO ROWS)` 출력


### 나. LEFT OUTER JOIN 이용

```sql
SELECT COALESCE(B.개인고객명, C.법인명) 고객명
FROM 주문 A LEFT OUTER JOIN 개인고객 B
	ON (A.개인/법인번호 = B.개인번호) LEFT OUTER JOIN 법인고객 C
	ON (A.개인/법인번호 = C.법인번호)
WHERE A.주문번호 = 1001;
```

- 중복X → 아우터 조인
- `LEFT OUTER JOIN` 을 두 테이블에 대해 진행하여 주문번호가 일치하면 값 O, 없으면 NULL
  - 왼쪽 테이블의 모든 행을 가져옴
  - `ON` 조건절을 통해 오른쪽 테이블의 값을 가져옴
    - 조건 만족 - 값
    - 조건 불만족 - NULL로 채움
- `COALESCE()` 을 사용하여 왼쪽부터 NULL이 아닌 값을 반환(둘 다 NULL이면 NULL 반환)
- 결과X 경우 → 한 건의 `NULL ROWS` 출력

---