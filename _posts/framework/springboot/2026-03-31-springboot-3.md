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
- JPQL(`(left) join fetch`) 또는 NativeQuery, DTO 프로젝션을 이용하여 연관된 엔티티를 조인하여 조회
- 카테시안 곱(Cartesian Product)으로 인한 데이터 중복 발생 가능성\
  → <mark>일대 다 관계에서 페이징 적용 불가</mark>

#### Fetch Join 
- `fetch`를 활용하여 연관된 객체를 즉시 로딩(`Eager Loading`)

  ```java
  //fetch join
  @Query("select m from Message m left join fetch m.attachments where m.id = :id")
  Optional<Message> findByIdWithAttachments(UUID id);
  ```
#### Native Query
- `JOIN`을 활용한 SQL문 + <mark>조립(Mapping) 필요</mark>
- 연관 테이블을 가져오지만, 연관 객체로 인식 X\
  → `@SqlResultSetMapping` 설정 등 복잡한 설정이 필수


#### DTO Projection
- <mark>DTO를 활용하여 하나의 객체로 조회</mark> → N+1 개념 삭제
- 필요한 필드만 조회 → 성능 우수
- 엔티티 형태로 재사용 불가

  ```java
  //DTO Projection
  @Query("select new com.example.dto.MessageDto(m.id, m.content, a.fileName) " +
       "from Message m left join m.attachments a")
  List<MessageDto> findAllMessageDtos();
  ```
  
### @EntityGraph
- 특정 쿼리 실행 시에만 즉시 로딩(`Eager Loading`)으로 전략 변경
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
- ex) 게시글 + 댓글: 게시글 목록 조회(1) + 댓글 IN(각 게시글 아이디) 조회(1) = <mark>2번의 쿼리</mark>
- 어노테이션 또는 yaml 파일 설정 활용
  - `@BatchSize`
    - <mark>전역 설정 보다 우선 적용</mark>
    - 컬렉션 필드 위나 클래스(엔티티) 위에 붙임
      - 특정 엔티티를 조회할 때 그 안에 딸린 리스트(1:N)만 배치 로딩하고 싶을 때 적용(권장)
        ```java
        @Entity
        public class Message {
          @Id @GeneratedValue
          private UUID id;

          @BatchSize(size = 200) // 이 필드를 조회할 때만 200개씩 IN 절로 묶음
          @OneToMany(mappedBy = "message")
          private List<Attachment> attachments = new ArrayList<>();
        }
        ```
      - 엔티티가 `N`의 입장에서 조회될 때(부모를 조회했는데 자식이 여러 명일 때) 적용
        ```java
        @BatchSize(size = 200)
        @Entity
        public class Attachment { ... }
        ```
        
  - `application.yml` 설정
    - 프로젝트 전체에 적용

      ```yaml
      jpa:
        properties:
          hibernate:
            default_batch_fetch_size: 100 # 보통 100~1000 사이 설정
      ```