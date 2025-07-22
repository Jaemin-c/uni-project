## Project 2b: Priority Queues - EECS 281

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

### 개인적인 회고
- 자료구조의 시간복잡도와 메모리 구조를 깊이 이해할 수 있었음
- 특히 **Pairing Heap**은 개념적으로 어렵지만 직접 구현하며 pointer 조작과 트리 구조의 중요성을 체감
- **Binary Heap**을 직접 구현하며 힙의 기본 원리를 재확인하고, `fixUp`, `fixDown` 로직을 명확히 이해
- 추상화된 인터페이스 위에 다양한 자료구조를 올리는 설계가 유익했음

---
