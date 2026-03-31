---
title: "[Spring Boot] JPA N+1"
excerpt: "JPA N+1 문제의 원인 및 해결"

categories:
  - Spring Boot
tags:
  - [springboot, jpa, n+1, entitygraph, fetch, join, Batch]

permalink: /springboot/post-3/

toc: true
toc_sticky: true

date: 2026-03-31
last_modified_at: 2026-03-31
---

## 정의
- 데이터 1건을 <mark>조회</mark>하기 위한 쿼리 외에 <mark>연관된 데이터를 조회하기 위한 추가 쿼리가 발생하는 현상</mark>
- ex) 10명의 사용자를 조회(`1`) → 각 사용자가 가진 주문 내역을 조회하기 위해 10번의 추가 SELECT 쿼리(`N`)

---

## 원인
- JPQL의 독립성
  - JPA의 내부적으로 생성되는 JPQL은 연관 관계를 고려 X → 해당 엔티티만을 대상으로 SQL 생성
  - ex) `select * from Parent` → `Child`는 함께 조회 X
- Proxy와 지연 로딩(Lazy Loading)
  - 연관된 데이터는 <mark>Proxy로 대체</mark>
  - 연관 객체 사용 시, 추가 쿼리(N번)를 통해 개별 조회(`N`)

---

## 해결 방안

### JOIN
- JPQL(`(left) join fetch`) 또는 nativeQuery를 이용하여 연관된 엔티티를 조인하여 조회
- 카테시안 곱(Cartesian Product)으로 인한 데이터 중복 발생 가능성\
  → <mark>일대 다 관계에서 페이징 적용 불가</mark>

  ```java
  @Query("select m from Message m left join fetch m.attachments where m.id = :id")
  Optional<Message> findByIdWithAttachments(UUID id);
  ```

### @EntityGraph
- 특정 쿼리 실행 시에만 즉시 로딩(`Eager Fetch`)으로 전략 변경
- `Outer Join` 기반
- 카테시안 곱(Cartesian Product)으로 인한 데이터 중복 발생 가능성\
  → <mark>일대 다 관계에서 페이징 적용 불가</mark>

  ```java
  @EntityGraph(attributePaths = {"attachments"})
  Optional<Message> findById(UUID id);
  ```

### Batch Size
- 연관된 엔티티를 조회할 때, 지정한 숫자만큼 `IN` 절을 사용해 한꺼번에 조회\
  → 쿼리 횟수: `1 + (N/BatchSize)`
- <mark>일대 다 관계에서 사용 가능</mark>(Fetch Join의 한계 극복)
- <mark>페이징 사용 가능</mark>
- 별도의 코드 변경 불필요
- ex) 게시글 + 댓글: 게시글 목록 조회(1) + 댓글 IN(각 게시글 아이디) 조회(1) = 2번의 쿼리
- `application.yml` 설정
  
   ```yaml
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100 # 보통 100~1000 사이 설정
  ```