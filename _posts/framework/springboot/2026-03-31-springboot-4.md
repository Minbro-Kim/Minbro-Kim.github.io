---
title: "[Spring Boot] JPA 대량 저장/수정/삭제"
excerpt: "JPA 대량 저장/수정/삭제 시 발생하는 성능 결함"

categories:
  - Spring Boot
tags:
  - [springboot, jpa, bulk]

permalink: /springboot/post-4/

toc: true
toc_sticky: true

date: 2026-03-31
last_modified_at: 2026-03-31
---

## 정의
- 여러 데이터를 한 번에 저장/수정/삭제하는 과정에서 <mark>각 대상에 대해 별개의 쿼리가 나가는 현상</mark>

---

## 원인

- JPA의 개별 엔티티 관리 방식
  - 영속성 컨텍스트는 변경 감지(Dirty Checking)를 위해 데이터를 하나씩 메모리에 올리고 상태 확인
  - 수정/삭제 시 개별 대상에 대한 쿼리 발생(N)
  - ex) `deleteByParentId`, 여러 상품의 가격을 일괄 인상하는 경우
- JPA 표준 벌크 저장(Insert)의 부재
  - JPA의 `@Query`는 `INSERT INTO ... VALUES` 형태의 벌크 저장은 지원 X
  - ex) `saveAll()`

---

## 해결 방안

### 벌크 연산(`Bulk Operation`)
- 여러 데이터의 수정과 삭제를 한꺼번에 처리
- <mark>영속성 컨텍스트를 무시하고 직접 DB에 쿼리</mark>\
  → 벌크 연산 이후에 같은 트랜잭션 내에서 조회하는 경우, 영속성 컨택스트를 비우는 옵션 필요
- `@Modifying`과 함께 작성
- ex) 기존: 전체 조회(1) + 개별 삭제(N) → 한 번에 삭제(1)
- 저장로직과 개별적인 수정 로직에 대해 적용 불가

  ```java
  @Modifying
  @Query("delete from Child c where c.parent.id = :parentId")
  void deleteChildrenByParentId(Long parentId);
  ```
  ```java
  // 주로 수정된 값을 반환하기 때문에 DB와 영속성 컨텍스트 간의 데이터 불일치를 방지 필요
  @Modifying(clearAutomatically = true)
  @Query("update Product p set p.price = p.price + :increment where p.id in :ids")
  int bulkUpdatePriceByIds(@Param("ids") List<Long> ids, @Param("increment") int increment);
  ```

### JDBC Batch Size
- `Write-behind 배치`
- 여러 데이터를 저장/수정할 때, <mark>지정한 숫자만큼 쿼리를 패킷에 담아 보냄</mark>
- <mark>쿼리 개수는 변경 X</mark> → 네트워크 통신 횟수(Round-trip)를 줄여 성능을 높임
- `order_updates/inserts: true` 설정으로 같은 쿼리끼리 하나로 묶음
  - `false`: UPDATE product (ID 1) → UPDATE member (ID 5) → UPDATE product (ID 2)\
    → 쿼리 종류가 바뀌면 배치가 끊김 → 3번 통신
  - `true`: UPDATE product (ID 1, 2) → UPDATE member (ID 5)\
    → 같은 테이블끼리 통합 → 2번 통신
- ex) `saveAll`, 여러 엔티티를 개별적으로 수정하는 경우
- <mark>MySQL의 IDENTITY 전략 사용 시 Insert 배치는 작동 X</mark>
  - 객체를 영속성 컨텍스트에 저장하려면 반드시 **식별자(ID)**가 필요함.
  - `GenerationType.IDENTITY`는 DB에 데이터를 실제로 넣을 때 ID 생성
- `application.yml` 설정

  ```yaml
  spring:
    jpa:
      properties:
        hibernate:
          # 1. 한 번에 보낼 쿼리 묶음 크기 (보통 100~500)
          jdbc.batch_size: 100 
        
          # 2. 같은 종류의 쿼리끼리 모아서 순서대로 정렬 (Batch 효율 극대화)
          order_inserts: true
          order_updates: true
  ```

#### 배치 작동 확인
- 하이버네이트 로그는 준비된 쿼리를 모두 보여줌
  → **하이버네이트 통계** 설정 필요 → `[... nanoseconds spent executing 1 JDBC batches]`
- `application.yml`
```yaml
spring:
  jpa:
    properties:
      hibernate:
        generate_statistics: true # 통계 로그 활성화
```