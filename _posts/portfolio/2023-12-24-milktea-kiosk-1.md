---
title: "[포트폴리오] 밀크티 카페 키오스크"
excerpt: "자바 Swing을 활용한 밀크티 카페 키오스크"

categories:
  - 카페 키오스크
tags:
  - [tag1, tag2]

permalink: milktea-kiosk/milktea-kiosk-1/

toc: true
toc_sticky: true

date: 2023-12-21
last_modified_at: 2023-12-21
---
![키오스크 배너](https://github.com/user-attachments/assets/4d569da1-6962-405d-9b0f-b6cc15a90a84)

## 프로젝트 개요
- **프로그램명**: <mark>공차 키오스크 프로그램</mark>
- **소개**: 기존 카페 키오스크의 불편한 UI를 개선하고, 더 효율적인 메뉴 선택 기능을 제공
- **개발 기간**: 2023년 11월 ~ 2023년 12월  
- **인력 구성**: 개인
- **역할**
  - 프로젝트의 **기획, 설계, 구현 전반**을 단독 수행
  - 장바구니 옵션 수정 및 수량 관리 로직 설계
  - 메뉴 데이터 로드 및 CSV 입출력 기능 구현
- **성과**: 교수님과 동료들로부터 우수한 평가(A+)를 받음

---

## 기술 스택
- **언어**: `Java`  
- **GUI**: `Java Swing`, `AWT`  
- **데이터 처리**: CSV 파일 입출력  

---

## 주요 기능
1. **메뉴 관리**  
   - 로컬 CSV 파일에서 메뉴 데이터를 불러와 카테고리별로 정렬 
   - 밀크티, 커피, 디저트 세 가지 카테고리 지원

2. **메뉴 옵션 선택 기능**
   - 각 카테고리에 따른 옵션(사이즈, 수량, 핫/아이스, 당도 등)을 제공
   - 옵션 선택에 따른 가격 변동
   - 핫/아이스의 선택에 따라 얼음량의 옵션의 활성 여부 결정

3. **장바구니 기능**  
   - **옵션 수정 가능**: 장바구니에 추가된 메뉴의 옵션과 수량을 수정 가능
   - **중복 처리 개선**: 동일한 옵션의 메뉴 추가 시 기존 항목의 수량 증가 (장바구니 내의 옵션 수정에서도 적용)

4. **결제 및 건의사항 기능**
   - 결제 항목과 최종 금액을 영수증 형태로 화면에 출력
   - 결제 완료 후 건의사항 창을 띄워 입력가능
   - 해당 매출 데이터와 건의사항을 CSV 파일로 저장 
     - 저장 경로: `C://공차`
     - 주문 발생 시 파일 자동 생성  


---

## 결과
- 사용자들이 메뉴 옵션을 직관적으로 선택할 수 있는 UI를 구현하여, 시스템의 사용자 편의성 향상
- 건의사항 기능을 통해 고객 피드백을 수집하고, 매출 데이터를 관리

### 주요 화면
![Image](https://github.com/user-attachments/assets/bc438e99-d3e8-4c95-a4bd-90ae3ede58b2)
![Image](https://github.com/user-attachments/assets/ba80afac-ce5d-464f-bc9e-1f58452a16a0)
- CSV 파일
![Image](https://github.com/user-attachments/assets/3014b0ca-039c-4e8c-a7fd-2410d2e6c395)

### 시연 영상
<iframe width="560" height="315" 
    src="https://www.youtube.com/embed/hHEuREeWkzU"
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
</iframe>


---
## 소스코드
- [Github 바로가기](https://github.com/Minbro-Kim/Java_Kiosk)
