---
title: "[데이터모델링의 이해] 관계(Relationship)"
excerpt: "Entity Relationship 개념, 분류, 표기법 "

categories:
  - SQL
tags:
  - [SQL, 데이터모델링, Oracle, Entity, 엔터티, 관계, Relationship]

permalink: /sql/data-modeling-4/

toc: true
toc_sticky: true

date: 2025-08-04
last_modified_at: 2025-08-04
---

## 1. 관계의 개념

### 가. 관계의 정의

- 엔터티의 인스턴스 사이의 논리적인 연관성으로서 존재 또는 행위로서 서로에게 연관성이 부여된 상태
- 엔터티와 엔터티 간 연관성 표현
- ex. 강사-<mark>가르친다</mark>-수강생

### 나. 관계 페어링(Relationship Paring)

- 엔터티 내에 인스턴스와 인스턴스 사이에 관계가 설정된 실제 연결 현상(Occurrence)
- 관계는 <mark>관계 페어링의 집합</mark>을 논리적으로 표현한 것

  → 각각의 연관성(Paring)의 집합이 `강의`라는 관계 도출

![img.png](/assets/images/posts_img/sql/datamodeling/sql-14.png)

---

## 2. 관계의 분류

- 연결의 목적으로 분류

1. `존재에 의한 관계`

    ![img.png](/assets/images/posts_img/sql/datamodeling/sql-15.png)
    
    - 사신은 항상 부서에 속함
    - <mark>존재의 형태에 의한 관계</mark>
    - UML(Unified Modeling Language)의 클래스다이어그램의 연관관계(Association)
        - <mark>실선(→)</mark>으로 표기
    - 소스코드에서 멤버변수로 선언

2. `행위에 의한 관계`

   ![img.png](/assets/images/posts_img/sql/datamodeling/sql-16.png)
    
    - ‘주문한다’는 행위를 통해 주문번호 생성
    - <mark>행위에 의해 발생한 관계</mark>
    - UML(Unified Modeling Language)의 클래스다이어그램의 의존관계(Dependency)
        - <mark>점선(⇢)</mark>으로 표기
    - Operation(Method)에서 파라미터로 이용


---

## 3. 관계의 표기법

### 가. 관계명(Membership)

- 관계명은 관계의 이름으로 엔터티가 관계에 참여하는 형태
- 관계시작점(The Beginning) - 관계끝점(The End)
- 각각의 관계는 두 개의 관계명을 가짐
    - 관계시작점과 관계끝점 모두 관계이름을 가짐
    - 참여자 관점에 따라 관계이름이 능동적(Active)/수동적(Passive)으로 명명 → 포함한다-소속된다
- 애매한 동사X→ 관계된다X
- 현재형 표현 → 신청했다X

### 나. 관계 차수(`Cardinality`)

- <mark>두 개의 엔터티간 관계에서 참여자의 수</mark>
- 한 개의 관계 or 두 개 이상의 멤버십
- 표현 방법 → 1:M, 1:1, M:N
  - 한 개 - 실선, 다수 - 까마귀발

1. `1:1 (ONE TO ONE)`
    - 각각의 엔터티는 관계를 맺는 다른 엔터티에 대해 하나의 관계만 가짐
2. `1:M (ONE TO MANY)`
    - 각각의 엔터티는 관계를 맺는 다른 엔터티에 대해 하나 이상의 관계를 가지고 반대 방향은 하나만의 관계를 가짐
3. `M:M (MANY TO MANY)`
    - 각각의 엔터티는 관계를 맺는 다른 엔터티에 대해 하나 이상의 관계를 가지고 반대 방향도 동일
    - M:N 관계는 이후에 두 개의 주식별자를 상속받은 관계 엔터티를 이용하여 3개의 엔터티로 구분

### 다. 관계 선택사양(Optionality)

- `필수참여관계(Mandatory Membership)`
    - 모든 참여자가 반드시 관계를 가짐
- `선택참여관계(Optional Membership)`
    - Foreign Key로 연결될 경우, Null 허용
    - ERD에서 <mark>선택참여 쪽을 원 표시</mark>
    - 양쪽 엔터티 모두 선택참여인 경우(0:0, Zero To Zero)는, 잘못된 관계일 확률이 높음

---

## 4. 관계의 정의 및 읽는 방법

### 가. 관계 체크사항

- 두 엔터티 사이에 관심 있는 연관규칙이 존재
- 두 엔터티 사이에 정보의 조합 발생
- 업무기술서, 장표에  관계연결에대한 규칙 서술 존재
- 업무기술서, 장표에 관계연결을 가능하게하는 동사 존재

### 나. 관계 읽기

- 기준 엔터티 - 한 개/각
- 대상 엔터티- 관계참여도(하나, 하나 이상)
- 관계선택사양
- 관계명

![img.png](/assets/images/posts_img/sql/datamodeling/sql-17.png)

---