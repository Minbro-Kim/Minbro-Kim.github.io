---
title: "[Spring Boot] 웹 API 종류"
excerpt: "RPC, SOAP, RESTful API, GraphQL, gRPC"

categories:
  - Spring Boot
tags:
  - [springboot, rpc, soap, rest, restful, graphql, grpc]

permalink: /springboot/post-2/

toc: true
toc_sticky: true

date: 2026-02-26
last_modified_at: 2026-02-26
---


## 1. RPC(Remote Procedure Call)
### 1) 개념
- 원격의 프로시저(함수)를 호출하는 <mark>통신 방식</mark> 
  - <mark>다른 서버의 서비스를 로컬 메서드처럼 사용하도록 추상화</mark> \
    → 분산된 서버에 요청을 보내서 하나의 서버처럼 사용 가능
  - 원격 서버에서 결과를 네트워크로 반환
  ```java
  // 내부 함수 호출처럼 보이지만, 원격 서버에 메서드 처리 요청
  User user = userService.getUserById(1);
  ```

- 모놀리식 구조에서 분산 시스템 환경에 대한 요구로 등장 → 초기 분산 시스템
- 요청-응답 과정
  - 요청 직렬화(메서드 시그니처) > 네트워크 전송 > 서버에서 실행 > 결과 직렬화 > 응답 전송 > 역직렬화

### 2) 특징

- 직관적 사용법
- 언어 추상화(객체 지향 유지)
- 빠른 적용
  - 서버와 클라이언트가 동일 스택일 때 빠르게 도입

#### ✔ 한계
- <mark>결합도(Coupling) 증가</mark>
  - 원격 서버의 메서드 시그니처 변경 시, 클라이언트도 수정 필요
- 네트워크 장애 영향
  - 원격 호출 시, 네트워크 상태에 영향을 받음
  - 에러 복원성, 유연성 및 확장성이 떨어짐
- 언어/플랫폼 의존성
  - RMI :Java 전용
  - CORBA: 높은 학습 비용과 복잡한 설정


### 3) RPC 구현 기술
- Java RMI
  - Java 환경
  - 객체를 원격에서 직렬화하여 호출
  - 자바 객체 → 바이트 스트림 → TCP 전송
- CORBA
  - `Common Object Request Broker Architecture`
  - 서로 다른 언어(C++, Java 등) 간 통신을 목표로 한 RPC 기반 미들웨어
  - IDL(Interface Definition Language) 기반으로 직렬화
    - IDL: 중립적인 인터페이스 정의 언어\
      → 실제 코드(Java, python) 생성
  - ORB(Object Request Broker)가 호출 
- gRPC
  - Google 개발 고성능 RPC 시스템

----

## 2. SOAP(Simple Object Access Protocol)

### 1) 개념
- `HTTP`, `SMTP` 등의 프로토콜에서 <mark>XML 형식으로 메시지를 주고받는 API 통신 프로토콜</mark>
  - <mark>엄격한 구조와 명세</mark>
    - `Envelope`
      - SOAP 메시지의 루트(SOAP 메시지라고 선언)
      - 모든 메시지를 포함
    - `Header`
      - 인증과 트랜잭션 등의 메타 정보를 담는 공간
      - 선택 사항
    - `Body`
      - 실제 호출하려는 메서드 정보와 파라미터
    - `Fault`
      - 에러가 발생했을 때 에러 내용을 담는 공간
    
    ```xml
    <soap:Envelope>
      <soap:Body>
        <getUser>
          <id>12</id>
        </getUser>
      </soap:Body>
    </soap:Envelope>
    ```
- API 표현: XML 문서 내의 메서드 이름
  - `getUser`
- 요청 흐름
  - WSDL로 계약 확인 > 코드 생성 > 요청(SOAP XML 메시지 전송)

### 2) 특징
- <mark>계약 기반</mark>(`Contract-fist`)
  - WSDL(Web Services Description Language, 웹 서비스 기술명세)로 서비스 정의\
    - WSDL: operation 종류 및 입출력 메시지 구조, 통신 프로토콜 종류, 서비스 엔드포인트 주소
    → 클라이언트에서 자동으로 인터페이스 생성
    
    ```xml
      <definitions>
        <types>...</types>
        <message>...</message>
        <portType>
          <operation name="getUser">
            ...
          </operation>
        </portType>
    </definitions>
    ```
- <mark>엄격한 스키마 검증</mark>
  - XSD(XML Schema Definition) 기반 스키마
  - 파라미터와 리턴 값의 명확한 구조 정의
- <mark>유연한 전송 프로토콜</mark>
  - `HTTP`, `SMTP`, `FTP` 등 다양한 프로토콜에서 동작 가능
- 높은 보안 및 신뢰성
  - WS-Security
    - 메시지 단위 암호화
    - 메시지 단위 전자서명 
    - 토큰 기반 인증
    - <mark>REST는 전송계층 보안(TLS)에 의존하지만 SOAP은 메시지 자체에 서명/암호화</mark>
    → 메시지 무결성 보장
  - WS-ReliableMessaging
    - 메시지 중복 방지 
    - 순서 보장 
    - 전달 보장 (at-least-once, exactly-once 등)
    - HTTP 기본 동작에 없는 기능
  
  → <mark>보안이 중요한 금융권과 공공기관에서 사용</mark>
- 트랜잭션 보장
  - WS-AtomicTransaction
    - SOAP 메시지 내부에서 ACID 트랜잭션 관리 가능

#### ✔ 한계
- 무겁고 복잡한 XML 구조\
  → 네트워크 트랙픽↑, 파싱 성능↓
- 개발 복잡성
  - 메시지 해석과 작성 비용↑
- 높은 WSDL 학습비용
  - 자동 코드 생성 도구 없이 개발이 어려움

### 3) 사용 환경
- 금융권
  - 보안성과 안정성 중요
- 공공기관
  - 행정망과 통신 전산망에서 XML 기반 데이터 교환 표준 사용
- 보험사
  - 보험 청구, 연계 시스템에서 WSDL 기반 인터페이스 공유 필요

---

## 3. RESTful API
### 1) 개념
- REST(`Representational State Transfer`)
  - 웹 서비스 구조의 이름
  - REST 기본 조건
    - Client-Server 구조
      - 클라이언트와 데이터를 처리하는 서버를 완전히 분리\
      → 독립적으로 진화
    - <mark>Stateless</mark>
      - 서버가 요청 정보 기억X\
        → 서버 확장(Scale-out)이 용이
      - 모든 요청은 그 자체로 완결성을 가짐(요청이 전체 정보(토큰 등) 포함)
    - Cacheable
      - 응답에 `Cache-Control` 같은 헤더를 붙여 효율적인 데이터 재사용이 가능
    - Layered System
      - 다른 계층(로드밸런서, 프록시)을 알 수 없음
    - Uniform Interface
      - URI로 자원 식별 + HTTP Method로 동작 정의 + 응답 메세지에 자신을 설명하는 정보(Self-descriptive) 포함
    - Code on Demand
- API 표현
  - <mark>URI로 자원 표현 + HTTP Method로 동작 표현</mark>
  - 사용 메서드
    - `GET` : 자원 조회(멱등성)
    - `POST` : 자원 생성
    - `PUT` : 자원 전체 갱신 또는 생성(멱등성) 
    - `PATCH` : 자원 일부 갱신
    - `DELETE`: 삭제(멱등성)
- 현재 가장 많이 사용되는 구조

### 2) 특징
- 일관성(Consistency)
  - HTTP 메서드 + <mark>자원(resource) 중심</mark> URI 구조\
    → 메서드와 URI로 기능 예측 가능
  ```markdown
    GET /users/1 → 사용자 조회
    PATCH /users/1 → 사용자 수졍
  ```
- 확장성(Scalability)
  - 자원 중심 규칙 기반으로 기능 확장 용이
  ```markdown
    GET /users/1/posts → 사용자 게시글 목록
    GET /users/1/friends → 사용자의 친구 목록
  ```

#### ✔ 한계
- Over-fetching
  - 필요한 정보보다 더 많은 데이터 요청
- Under-fetching
  - 원하는 정보보다 적은 정보롤 인해 추가 요청 필요
- API 버전 관리 문제
  - URI에 /v1,/v2

---

## 4. GraphQL
### 1) 개념
- Facebook 개발 <mark>API 쿼리 언어</mark>
- <mark>클라이언트가 필요한 필드만 요청</mark>\
  → Over-fetching 방지
- 하나의 요청으로 여러 리소스 조합 가능\
  → Under-fetching 방지
```text
  query {
    user(id: 1){
      name
      email
      posts{
        title
      }
    }
  }
```

### 2) 특징
- Over-fetching X
- Under-fetching X
- 스키마 중심
- 버전 관리 불필요

#### ✔ 한계
- HTTP 레벨의 캐싱이 어려움
  - 하나의 엔드포인트(/graphql)로 모든 요청을 POST로 보냄
  - HTTP는 URL과 HTTP 메서드의 조합으로 같은 응답을 캐싱
- 복잡한 보안 필터링 로직
- 클라이언트와 서버의 높은 학습 비용

---

## 5. gRPC
### 1) 개념
- Google 개발 RPC 프레임워크
- 마이크로서비스를 위한 고성능 RPC
  → 내부 마이크로 서비스 간 통신에 적합
- <mark>HTTP/2 기반</mark>
- <mark>양방향 스트리밍, 서버푸시, 멀티 플렉싱 등 고성능 기능 지원</mark>
- 다양한 언어(Java, Go, Python)로 자동 코드 생성
- Protocol Buffers(바이너리) 기반
```protobuf
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);  
}
```

### 2) 특징
- 고성능
- 양방향 스트리밍
- 다양한 언어 지원
- API 명세와 코드 동기화

#### ✔ 한계
- 브라우저의 HTTP/2 지원 불안정
  - 프론트엔드 도입 난이도 높음
- 디버깅 어려움
- 외부 API 제공에 부적합

## API 선택
- REST: 가장 보편, 공공 API, 일반적인 웹 서비스, 프론트엔드와 통신할 때
- gRPC: MSA(마이크로서비스) 내부에서 서버끼리 아주 빠르게 데이터를 통시할 때
- GraphQL: 클라이언트가 요청하는 데이터의 형태가 너무 다양해서 REST로는 Over-fetching이 심할 때
- SOAP: 은행, 보험사, 정부 기관처럼 강력한 보안 규격과 트랜잭션 보장이 최우선일 때