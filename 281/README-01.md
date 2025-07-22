# Project 1: Back to the Ship! - EECS 281

> **Note:** Due to university policies, the source code is not publicly available. This document provides a project overview and my personal reflections.

> **안내:** EECS 281 수업은 **모든 프로젝트를 개인이 단독으로 수행**하는 수업입니다.  
> Starter code 없이 알고리즘과 코드 구조를 직접 설계하고 구현했습니다

> **주의:** 학교 정책상 소스 코드는 비공개입니다. 이 문서에는 프로젝트 개요, 구현 방식, 그리고 개인적인 회고가 담겨 있습니다.

---
## 📌 프로젝트 개요
이 프로젝트는 3차원 우주선 미로에서 시작점(S)에서 우주선 행거(H)까지의 경로를 탐색하는 알고리즘을 구현하는 과제입니다.  
탐색 알고리즘으로는 **BFS (Breadth-First Search)**와 **DFS (Depth-First Search)**를 사용하며,  
입력 데이터의 형태와 출력 형식을 명령어 옵션으로 조절할 수 있도록 요구되었습니다.

특히, 각 층에서 특정 좌표에 위치한 **엘리베이터(E)**를 통해 같은 좌표의 다른 층으로 이동할 수 있는 로직이 스펙의 핵심 중 하나였습니다.

---
## Features
- BFS (Breadth-First Search)를 통한 최단 경로 탐색
- DFS (Depth-First Search)를 통한 깊이 우선 경로 탐색
- 3D Maze에서 층간 이동 처리 (엘리베이터)
- 두 가지 입력 모드 지원: 
  - Map 모드 (M)
  - Coordinate List 모드 (L)
- 두 가지 출력 모드 지원:
  - Map 모드 (M)
  - Coordinate List 모드 (L)
- Robust한 에러 핸들링
- 빠른 입출력을 위한 최적화
## 사용한 알고리즘 및 데이터 구조
- **BFS (너비 우선 탐색):** Queue 기반, 최단 경로를 보장
- **DFS (깊이 우선 탐색):** Stack 기반, 깊게 파고드는 방식
- **Deque:** Stack과 Queue의 동작을 하나의 공용 컨테이너로 통일
- **Backtrace:** 경로를 거꾸로 추적하여 경로 출력 (Map 모드, List 모드)
- **3차원 벡터:** 미로의 층, 행, 열을 표현하는 자료구조

---

## 핵심 구성 및 함수 설명

### `getOpts()`
- 명령줄에서 전달된 옵션을 읽어
  - `--stack`: DFS 모드
  - `--queue`: BFS 모드
  - `--output`: M 또는 L로 출력 모드 결정
  - `--help`: 도움말 출력
- stack/queue가 동시에 주어지거나 둘 다 없는 경우 에러 종료
- ✅ **의도:** 사용자의 잘못된 입력 방지 및 명확한 탐색/출력 모드 지정

### `input()`
- 첫 글자를 읽어 입력 모드('M' 또는 'L') 판별
- **Map 모드(M):** 한 줄씩 맵을 읽어 3차원 map 벡터에 저장
- **Coordinate List 모드(L):** (level, row, col, character) 형태의 좌표별 데이터를 읽어 저장
- ✅ **의도:** 다양한 입력 포맷을 유연하게 처리

### `storeData()`
- 좌표와 문자를 받아:
  - `S`: 시작점 저장
  - `H`: 행거 저장
  - 문자 유효성 검사 후 3D map에 저장
- ✅ **의도:** 입력 저장과 동시에 필요한 포인트 기록

### `search()`
- 시작점에서 search container(`deque`) 초기화
- BFS/DFS 모드에 따라 pop 위치 조절
- `investigate()`로 인접 노드 및 엘리베이터 탐색
- ✅ **의도:** BFS/DFS 통합 구현

### `investigate()`
- 현재 위치에서:
  - 북, 동, 남, 서 방향으로 `addToSC()` 호출
  - 'E'면 `elevator()` 호출
- ✅ **의도:** 각 방향과 엘리베이터 탐색 가능성 점검

### `addToSC()`
- 인접 좌표가:
  - 벽이 아니고
  - 방문하지 않았고
  - 행거(H)에 도달했는지 확인
- 통과 시:
  - `sc`에 push
  - 방향 기록
  - `discovered` 표시
- ✅ **의도:** 이동 가능 노드의 탐색 확장

### `elevator()`
- 같은 (row, col)의 모든 층에서:
  - 'E'이면서 미방문한 층 탐색
  - originLevel과 층 번호 기록 후 이동
- ✅ **의도:** 층간 이동 및 원래 층 정보 기록

### `mapOutput()`
- backtrace로 경로 추적
- 방향(n, e, s, w) 및 숫자(층수)를 통해 맵에 표시
- ✅ **의도:** 경로의 시각적 표현

### `listOutput()`
- backtrace 후 경로를 stack에 저장
- 역순으로 좌표와 이동 방향 출력
- ✅ **의도:** 좌표 기반 경로 명시

### `checkInvalidCharacter()`, `checkCoordinateError()`
- 문자와 좌표 유효성 검증
- 위반 시 에러 출력 후 종료
- ✅ **의도:** 프로그램 안정성 확보

---

## **내 코드의 전체적인 흐름**
1. **명령어 옵션 파싱:** `getOpts()`에서 탐색 방법, 출력 형식, 도움말 처리
2. **입력 데이터 처리:** `input()`으로 입력 모드에 따라 데이터 파싱
3. **탐색 수행:** `search()`에서 BFS/DFS로 경로 탐색
4. **결과 출력:** `output()`에서 출력 모드에 맞게 결과 출력

---
- **탐색 알고리즘:** BFS, DFS 구현 및 옵션 반영
- **입출력 포맷:** Map 모드, Coordinate List 모드 지원
- **엘리베이터 처리:** 층간 이동 및 숫자 기록
- **에러 체크:** 문자, 좌표, 옵션 검증

---
- **Starter Code 없이 처음부터 직접 구조를 설계:**  
  처음부터 3D 미로를 어떻게 표현할지, 좌표를 어떤 자료구조로 관리할지 고민이 많았습니다.  
  → 좌표를 위한 `pos` 구조체와, 셀의 상태를 담는 `coordinate` 구조체를 직접 정의해 복잡한 탐색 상태를 관리할 수 있었습니다.

- **BFS/DFS 공용 컨테이너 추상화:**  
  중복 코드를 줄이기 위해 deque를 활용해 BFS/DFS를 하나의 로직으로 통일

- **엘리베이터 이동 로직:**  
  중복 이동 방지와 originLevel 기록 문제를 수정하여 정확한 backtrace 보장

- **backtrace 로직:**  
  방향 매핑 오류와 층 이동 로직 디버깅을 통해 정확한 경로 추적 완성

---

## 💡 내가 배운 점
- BFS/DFS의 차이와 추상화
- 3D 배열과 층간 이동 상태 설계
- backtrace를 통한 경로 재구성
- 에러 검증 및 입출력 최적화의 중요성

---



## Project 2a: Stock - EECS 281

> **Note:** Due to university policies, the source code is not publicly available. This document provides a project overview and my personal reflections.

> **안내:** EECS 281 수업은 **모든 프로젝트를 개인이 단독으로 수행**하는 수업입니다.  

> 학교 정책상 소스 코드는 비공개입니다. 이 문서에는 프로젝트 개요, 구현 방식, 그리고 개인적인 회고가 담겨 있습니다.

---

<br><br><br>

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
