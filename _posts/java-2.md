---
title: "[JAVA] 제네릭, 컬렉션, 오브젝트 차이"
excerpt: ""

categories:
  - JAVA
tags:
  - [tag1, tag2]

permalink: /java/2/

toc: true
toc_sticky: true

date: 2025-05-15
last_modified_at: 2025-05-15
---


## 비교
| 개념           | 설명                      | 관련 예시                  |
| ------------ | ----------------------- | ---------------------- |
| `Object`     | 모든 타입을 담을 수 있는 상위 클래스   | `Object o = "hi";`     |
| `Generic`    | 타입을 컴파일 타임에 고정할 수 있는 **문법** | `List<String>`         |
| `Collection` | 데이터를 여러 개 담는 자료구조 인터페이스 | `List`, `Set`, `Map` 등 |

