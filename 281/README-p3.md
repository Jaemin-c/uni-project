## Project 3: 281Bank - EECS 281

> **Note:** Due to university policies, the source code is not publicly available.  
> This document provides a project overview, core design choices, and personal implementation notes.

> **안내:** EECS 281은 **모든 프로젝트를 학생 개인이 독립적으로 수행해야 하는 과목**입니다.  
> 학교 정책상 소스 코드는 외부 공개가 금지되어 있으며, 이 문서에는 프로젝트 개요 및 구현 방식, 회고를 포함합니다.

---

**281Bank**는 사용자 등록 정보와 명령어 시퀀스를 처리하여,  
로그인 세션, 거래 예약 및 실행, 거래 내역/수익 쿼리 등을 수행하는 **은행 시스템 시뮬레이터**입니다.

---

### 사용 기술

- **언어:** C++
- **표준 라이브러리:** STL (unordered_map, unordered_set, priority_queue, stringstream 등)
- **자료구조:**
  - 해시맵 기반 사용자 관리
  - 우선순위 큐 기반 거래 예약
- **파싱/입출력:** `ifstream`, `std::cin`, 명령줄 인자 처리 (getopt)

---

### 주요 기능 및 동작 흐름

#### 사용자 등록 정보 로드
- `-f [filename]` 옵션을 통해 registration 파일을 입력으로 받음
- 각 줄은 `timestamp|user_id|pin|balance` 형식이며, 사용자 정보를 해시맵(`unordered_map`)에 저장
- 각 유저는 IP 기반 로그인 세션을 추적

#### 명령어 처리 (`processCommands`)
- 명령은 stdin을 통해 순차적으로 처리되며, 다음과 같은 유형이 있음:

| 명령어      | 설명 |
|-------------|------|
| `login`     | 사용자 인증 (user_id + pin) 및 IP 로그인 |
| `out`       | 해당 IP에서 사용자 로그아웃 |
| `balance`   | 로그인 상태 확인 후 잔액 출력 |
| `place`     | 미래 실행 예정 거래를 예약 큐에 추가 |
| `$$$`       | 현재까지 실행 가능한 거래 모두 수행 후, 쿼리 모드 진입 |

#### 거래 처리 (`place`, `processTransactions`)
- 거래는 `timestamp`, `exec_date`, `sender`, `recipient`, `amount`, `fee_type`을 포함
- `exec_date`가 `timestamp`보다 빠르면 에러 종료
- 거래 큐(`priority_queue`)는 다음 기준으로 정렬:
  1. 실행 시점 (exec_date 오름차순)
  2. 등록 순서 (transaction_id 오름차순)

- 실행 가능한 거래는 `executeTransaction()`에서 수행:
  - 수수료 계산 (`calculateFee`)
  - 잔액 조건 확인 후, sender/recipient의 잔액 갱신
  - verbose 모드에서는 상세 정보 출력

#### 🔹 쿼리 처리 (`processQueries`)
- `$$$` 이후 입력되는 쿼리를 처리:

| 명령어 | 설명 |
|--------|------|
| `l start end` | 특정 구간의 거래 내역 출력 |
| `r start end` | 특정 구간 동안 은행이 수취한 총 수수료 출력 |
| `h user_id`   | 사용자 기준 거래 내역 및 요약 출력 |
| `s timestamp` | 해당 날짜의 거래 요약 및 수익 출력 |

---

### 에러 처리

#### 필수 체크 항목
- `place` 명령에서 이전보다 과거의 timestamp가 입력된 경우
- `exec_date < timestamp`인 경우

→ 이 경우에는 `cerr`로 에러 메시지를 출력하고 `exit(1)` 호출  
→ 명시된 표준 에러 메시지를 사용해야 autograder가 인식 가능

---

### 수수료 정책 (`calculateFee`)
- 기본 수수료는 `amount / 100`, 최소 $10, 최대 $450
- 장기 고객(가입 5년 이상)은 25% 할인 적용
- `fee_type == 's'`인 경우 수수료를 sender/recipient가 분담

---

### 내부 자료구조

```cpp
// 사용자 정보
struct User {
    uint64_t reg_timestamp;
    std::string user_id;
    std::string pin;
    uint32_t starting_balance;
    std::unordered_set<std::string> active_sessions;  // 현재 로그인 IP
};

// 거래 정보
struct Transaction {
    uint64_t timestamp;
    std::string ip;
    std::string sender, recipient;
    uint32_t amount;
    uint64_t exec_date;
    char fee_type;
    uint32_t transaction_id;
};
