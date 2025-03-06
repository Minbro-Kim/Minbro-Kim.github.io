---
title: "[포트폴리오] 기업 내 프로젝트 팀빌딩 시스템"
excerpt: "사내 데이터를 활용한 프로젝트 팀원 추천 시스템"

categories:
  - 팀빌딩 시스템
tags:
  - [tag1, tag2]

permalink: /team-building/team-building-1

toc: true
toc_sticky: true

date: 2024-12-20
last_modified_at: 2024-12-20
---


![팀빌딩 배너](https://github.com/user-attachments/assets/96668b08-542e-4944-9714-b74ce0894a80)

## 프로젝트 개요
- **프로그램명**: <mark>사내 데이터를 활용한 팀원 추천 시스템 (`Team-Sync`)</mark>
- **소개**:
  - 프로젝트의 특징을 고려한 팀 매칭 알고리즘을 통해 프로젝트에 적합한 팀원을 추천해주는 기업 관리자를 위한 웹 애플리케이션
  - 사원들의 성향과 기술 스택, 역량, 과거 프로젝트 내역, KPI 평가 점수, 할당된 Man/Month 등을 고려하여 진행하려는 프로젝트에 적합한 팀원을 포지션별로 추천해주는 시스템
- **개발 기간**: 2024년 9월 ~ 2024년 12월
- **인력 구성**: 팀(4인)
  - **<mark>백엔드 2인 (본인)</mark>**
  - 프론트엔드 1인
  - 모델 개발 1인
- **수행업무 (백엔드 담당)**
  - <mark>기본 서버 개발</mark>:
    - `Spring Boot`를 기반으로 RESTful API 설계 및 구현
    - Spring Security와 jwt 토큰 인증을 통한 로그인 기능 구현
    - 사원 정보 기능 구현 (엑셀 등록, 템플릿 다운, 사원 정보 조회, 스킬 목록 조회, 과거 프로젝트 등록 등)
    - 팀 기능 구현(프로젝트에 사원 등록, 조회, 삭제 등)
    - 팀 추천 요청 구현(`Webclient`를 사용하여 모델 서버에 팀 추천 요청)
  
  - <mark>모델 서버 개발</mark>:
    - `FastAPI` 기반으로 RESTful API 설계 및 구현
    - 팀원 추천 알고리즘을 서버 형태로 변환
    - 기존 통합 모델 형태에서 사원 점수 스케일링, 사원 성향 임베딩, 프로젝트 적합도 산출, 프로젝트 팀 추천으로 모델을 나누어 추천 소요 시간을 줄임
    - 모델 서버와 기본 서버를 연결하고 데이터베이스 연결
    - 모델 변환 과정에서 최적화 진행

  - <mark>데이터베이스 설계</mark>
    - 기본 서버: `MySQL`을 사용하여 사원 정보 및 프로젝트 정보 저장
    - 모델 서버: `MySQL`을 사용하여 기본 서버와 연결된 스키마를 활용하여 공유팀원 추천 알고리즘에 필요한 형태로 가공된 정보를 별도의 스키마로 저장
  - <mark>배포</mark>
    - 각 서버를 `Docker` 이미지를 생성하여 컨테이너 기반 배포 환경 구축
    - 두 개의 백엔드 서버와 한 개의 프론트엔드 서버 Docker image를 docker-compose 파일로 `AWS Elastic Beanstalk`로 서버에 배포
    - 데이터베이스를 AWS의 RDS로 배포

- **성과**: `특허 출원`

---

## 기술 스택 (Backend)
- **기본 서버**
  - `Spring Boot`
  - `MySQL`
  - `Spring Security (JWT)`
  - `WebClient` (모델서버 연동)
- **모델 서버**
  - `FastAPI`
  - `MySQL`
  
- **배포**
  - `Docker`, `Docker Compose`
  - `AWS Elastic Beanstalk`
- **외부 API**
  - `Upstage Solar API`

---
## 시스템 구조도
- 시스템 아키텍처
  ![Image](https://github.com/user-attachments/assets/7e29240e-f7af-4fc6-a29e-081276f95122)
- 블록 다이어그램
  ![Image](https://github.com/user-attachments/assets/dd3a706f-3e45-42cf-9faa-c9ff0beff438)
- 유스케이스
  ![Image](https://github.com/user-attachments/assets/e4fa93fd-fd1f-4582-a3f0-fa5a508d1db0)
- 팀원 추천 기능 시퀀스 다이어그램
  ![Image](https://github.com/user-attachments/assets/8b6281e1-d41e-4ed5-a7a0-226c9a38998f)
- 팀원 추천 알고리즘 변수
  ![Image](https://github.com/user-attachments/assets/e1e52af0-aa9a-4bbf-bdda-be6a5d0022d7)
- ERD
  ![Image](https://github.com/user-attachments/assets/4165e3a8-5595-407b-825d-a85a367e748f)


## 주요 기능
1. **회원가입 및 로그인**
   - JWT 기반 사용자 인증 및 보안 강화
2. **사원 정보 관리**
   - 기업별 사원 정보(이름, 스킬 등)를 엑셀파일로 업로드
   - 사원이 해당 시스템 이전에 진행했던 프로젝트 목록을 엑셀파일로 업로드
3. **프로젝트 관리**
   - 새로운 프로젝트를 등록하고 프로젝트 상세 정보(타임라인, 필요 인원, 스프린터, Man/Month 등)를 등록
   - 프로젝트 기간에 따라 진행, 예정, 종료로 표시
   - 종료된 프로젝트의 이력 차트를 제공
4. **프로젝트 팀원 관리**
   - 프로젝트 팀원을 사용자가 수동으로 등록(Man/Month를 초과하는 경우 불가)
   - 팀원 추천 기능
     - 사용자가 네 가지 가중치 타입(기본, 기술 중심, 프로젝트 유사도 중심, 개인 성향 중심) 중 하나를 선택하여 팀원 추천 요청
     - 종합 점수가 높은 1~3순위의 팀원 조합을 `Radar Chart` 형태로 제공


---

## 결과
1. 실무 프로젝트 성공률 향상
   - 다양한 변수로 고려한 팀원배치로 협업 원활
2. 자원 활용의 효율성 증가
     - 팀구성 시간을 절감하여 기업의 핵심 업무에 집중
3. 기업 맞춤 관리 시스템
     - 지속적 인사&프로젝트 데이터 축적을 통해 기업 맞춤형 관리 시스템으로 발전 가능

### 주요 화면
1. **기본 화면**
  ![Image](https://github.com/user-attachments/assets/06a6c12f-d543-4afd-9e17-10bdc0e7bcf4)
2. **인원 관리 화면**
   ![Image](https://github.com/user-attachments/assets/1af8654b-cf17-48e6-9c12-495f2592bd60)
3. **프로젝트 관리 화면**
   ![Image](https://github.com/user-attachments/assets/9bf134af-6768-48ef-bf39-55b4c0b5cadd)
4. **팀원 관리 화면**
   ![Image](https://github.com/user-attachments/assets/c533e9ba-7a1b-4687-b2ee-936d949f08c2)

### 시연 영상
<iframe width="560" height="315" 
    src="https://www.youtube.com/embed/PymX1aP7JQw"
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
</iframe>


---
## 소스코드
- [Github 바로가기](https://github.com/Minbro-Kim/PocketStone-Minbro)