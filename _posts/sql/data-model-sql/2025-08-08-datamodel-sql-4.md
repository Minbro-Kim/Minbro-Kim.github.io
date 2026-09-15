---
title: "[데이터모델과 SQL] Null 속성"
excerpt: "Null 속성의 이해"

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, Null]

permalink: /sql/data-model-sql-4/

toc: true
toc_sticky: true

date: 2025-08-08
last_modified_at: 2025-08-08
---

## 1. Null 표기법
- 바커표기법에서 앞 동그라미가 Null 허용
- IE 표기법은 알 수 없음

## 2. Null 값의 연산은 언제나 Null

- Null 값: <mark>공백X, 0 X → 아직 정의되지 않은 미지의 값</mark>, 현재 데이터를 입력하지 못하는 경우 → 값 존재X
- `lS NULL`, `IS NOT NULL` 외의 <mark>연산은 언제나 Null</mark>

  → `NVL(a,b)` : 첫 번째 인자 값이 Null이면 두 번째 인자 값 반환(cf. `COALESCE` 는 여러 인자 가능)

<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">

| 주문번호 | 주문금액 | 주문취소금액 |
| --- | --- | --- |
| 10001 | 100000 | 20000 |
| 10002 | 15000 |  |


</div>

<div class="col-2-item" markdown="1">

|  COL1 | COL2 | COL3 |
| --- | --- | --- |
| 80000 | 80000 | 80000 |
|  | 0 | 15000 |

</div>
</div>


```plsql
SELECT 주문금액 - 주문취소금액 COL1,
  NVL(주문금액 - 주문취소금액, 0) COL2,
  NVL(주문금액, 0)-NVL(주문취소금액,0) COL3
FROM 주문
```

- Null의 연산은 Null이기 때문에 연산이 필요한 경우 값을 `NVL()`로 처리해야 함

---

## 3. 집계함수는 Null 값을 제외하고 처리

- <mark>집계함수는 Null을 제외하고 처리</mark> → 아무것도 없는 경우 Null
  - `AVG` `SUM` `COUNT` `MAX` `MIN`
    - `COUNT(*)` → 전체 행
    - `COUNT(컬럼)` → 해당 컬럼이 NULL 아닌 행만

  
<div class="col-2-container align-center" markdown="1">
<div class="col-2-item" markdown="1">

| 주문번호 | 주문금액 | 주문취소금액 |
| --- | --- | --- |
| 10001 | 100000 | 20000 |
| 10002 | 15000 |  |
| 10003 | 40000 | 10000 |
| 10004 | 45000 | 10000 |
| 10005 | 100000 | 10000 |

</div>

<div class="col-2-item" markdown="1">

|  COL1 | COL2 |
| --- | --- |
| 10000 | 12500 |

</div>
</div>

```plsql
SELECT SUM(주문취소금액)/COUNT(*) COL1,
       AVG(주문취소금액) COL2
FROM 주문
```

- `AVG` 로 계산 시, 모수에 Null이 들어간 행이 포함 되지 않아 잘못된 값 도출

---