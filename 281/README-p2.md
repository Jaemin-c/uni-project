## Project 2A: Stock - EECS 281

> **Note:** Due to university policies, the source code is not publicly available. This document provides a project overview and my personal reflections.

> **안내:** EECS 281 수업은 **모든 프로젝트를 개인이 단독으로 수행**하는 수업입니다.  

> 학교 정책상 소스 코드는 비공개입니다. 이 문서에는 프로젝트 개요, 구현 방식, 그리고 개인적인 회고가 담겨 있습니다.

---


**언어/기술:** C++, Priority Queue, STL, Heap  

### 🔍 프로젝트 개요
다수의 주식 종목에 대해 시간 순으로 매수/매도 주문을 처리하고, 매칭하는 전자 주식 시장 시뮬레이터입니다.  
우선순위 큐(priority queue)를 활용해 매수자는 가장 낮은 가격에, 매도자는 가장 높은 가격에 거래할 수 있도록 설계되었습니다.  
여러 출력 옵션을 명령줄 인자로 받아 다양한 정보를 출력할 수 있습니다.

### 주요 기능
- **주문 매칭:** `std::priority_queue`와 커스텀 comparator를 사용해 시장 논리에 따라 주문 매칭
- **중앙값 계산:** max-heap, min-heap 두 개의 heap을 사용해 실시간 거래 중앙값을 효율적으로 계산
- **트레이더 통계:** 트레이더 별 매수/매도 수량과 순이익(transfer net value) 통계 출력
- **타임트래블러 분석:** 주식별로 역사적 데이터를 기반으로 최대 이익을 낼 수 있는 최적의 매수/매도 타이밍 탐색 (O(n) 시간, O(1) 메모리)
- **명령줄 옵션:**
  - `-v` / `--verbose`: 거래 발생 시 상세 내역 출력
  - `-m` / `--median`: 타임스탬프별 주식의 거래 중앙값 출력
  - `-i` / `--trader_info`: 하루 종료 시 트레이더 별 통계 출력
  - `-t` / `--time_travelers`: 타임트래블러가 얻을 수 있는 최대 이익과 시점 출력

### 입력 모드
- **Trade List (TL):** 수동으로 입력한 거래 목록 처리
- **Pseudorandom (PR):** `P2random.h`를 통한 난수 기반 주문 자동 생성

### 🗓 설계 및 구현
- 전체 로직(입력 파싱, 매칭, 통계, 중앙값, 타임트래블러 분석) 직접 설계 및 개발
- 매수/매도 별로 다른 우선순위를 갖는 priority queue와 comparator 설계
- heap을 이용한 중앙값 계산 최적화
- 명령줄 옵션 조합에 따라 다양한 출력 지원

<br><br><br>

## ⚡ 추가 구현: EECS 281 Project 2b - Priority Queues

**Project 2b**는 C++로 **Priority Queue(우선순위 큐)**의 다양한 구현 방식을 학습하는 프로젝트입니다.  
주어진 인터페이스(`Eecs281PQ.hpp`)를 상속하여 다음 세 가지 자료구조 기반의 Priority Queue를 직접 구현했습니다.

#### 구현한 Priority Queue
| 자료구조  | 설명 | 시간 복잡도 |
| --- | --- | --- |
| **SortedPQ** | 삽입 시 정렬된 벡터를 유지 | `push` O(n), `pop` O(1) |
| **BinaryPQ** | **Binary Heap** 기반, 배열로 완전 이진트리 표현 | `push` / `pop` O(log n) |
| **PairingPQ** | **Pairing Heap** 기반, 트리 구조 사용 | `push` O(1), `pop` Amortized O(log n) |

#### 공통 사항
- **모든 PQ는 max-heap이 기본:** 비교 함수(`compare`)를 통해 최대값이 top에 오도록 구성
- `updatePriorities`: 내부 자료의 순서를 PQ invariant에 맞게 재정렬하는 기능 포함
- **`updateElt`:** PairingPQ에만 존재하며, 특정 노드의 값을 변경할 때 힙 속성을 보장하도록 재배치

#### 개발 환경
- 언어: C++
- 사용한 STL: `vector`, `deque`, `algorithm` 제한적 사용
- 금지된 함수: `std::priority_queue`, `std::sort_heap` 등 PQ 구현을 단순화하는 함수 일체

### 🛠 개인적인 회고
- 자료구조의 시간복잡도와 메모리 구조를 깊이 이해할 수 있었음
- 특히 **Pairing Heap**은 개념적으로 어렵지만 직접 구현하며 pointer 조작과 트리 구조의 중요성을 체감
- **Binary Heap**을 직접 구현하며 힙의 기본 원리를 재확인하고, `fixUp`, `fixDown` 로직을 명확히 이해
- 추상화된 인터페이스 위에 다양한 자료구조를 올리는 설계가 유익했음

---
