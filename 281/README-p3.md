# Project 3: 281Bank - EECS 281

**281Bank**는 사용자 등록 정보를 기반으로 로그인 세션을 관리하고,  
예약된 거래를 시간 기준으로 실행하며, 이후 다양한 질의(query)를 처리하는 **은행 시스템 시뮬레이터**입니다.

> ⚠️ 소스 코드는 EECS 281 수업 정책상 공개할 수 없습니다.  
> 이 문서는 구현 개요와 설계 요점, 사용 기술을 요약한 문서입니다.

---

## 프로젝트 개요

이 프로젝트는 실시간 명령어 스트림을 받아 다음을 처리합니다:

- 사용자 로그인/로그아웃 세션 관리 (IP 기반)
- 미래 시점에 실행될 거래 예약 및 실행
- 실행된 거래를 바탕으로 수수료 수익, 사용자 거래 내역 등 질의 응답

즉, 단일 실행 환경에서 **상태를 유지하는(stateful)** 시스템을 설계하는 과제입니다.

---

## 사용 기술

- **언어:** C++
- **표준 라이브러리:** `unordered_map`, `unordered_set`, `priority_queue`, `ifstream`, `stringstream`, `getopt` 등
- **자료구조 설계:**
  - 사용자 정보: `unordered_map`
  - 로그인 세션: `unordered_set`
  - 거래 예약 큐: `priority_queue` (커스텀 comparator)
- **파싱 & 명령어 처리:** `getopt_long` 사용해 CLI 인자 처리

---

## 프로그램 구조

| 기능                     | 함수                                 | 설명                                                            |
|--------------------------|--------------------------------------|-----------------------------------------------------------------|
| 인자 파싱                | `getOpts()`                          | `-f`, `-v`, `--help` 옵션 처리                                  |
| 등록 파일 로드           | `loadFile()`                         | 사용자 데이터 파일 파싱 (`timestamp|user_id|pin|balance`)       |
| 명령어 루프              | `processCommands()`                  | `login`, `place`, `balance` 등 처리                             |
| 거래 예약                | `handlePlaceTransaction()`           | 미래 거래 유효성 검증 및 큐에 저장                              |
| 거래 실행                | `processTransactions()`              | 실행 가능한 거래를 순서대로 실행                                 |
| 거래 실행 세부           | `executeTransaction()`               | 수수료 계산 및 잔액 갱신, verbose 출력 등 처리                  |
| 질의 처리                | `processQueries()` 외 헬퍼 함수들    | `list`, `revenue`, `history`, `summary` 등 쿼리 지원            |

---

## 로그인 세션 관리

- 모든 세션은 **IP 기반**으로 관리됩니다.
- 사용자가 로그인하면 해당 IP가 `active_sessions`에 추가됩니다.
- 이후의 명령(`balance`, `place` 등)은 로그인된 IP인지 확인 후 처리합니다.

```cpp
struct User {
    std::string user_id, pin;
    uint64_t reg_timestamp;
    uint32_t starting_balance;
    std::unordered_set<std::string> active_sessions; // 로그인 IP
};
```

---

## 거래 예약 및 실행

- `place` 명령으로 예약된 거래는 다음 조건을 만족해야 큐에 추가됩니다:
  - `exec_date >= timestamp`
  - 이전 거래보다 `timestamp`가 빠르지 않아야 함
- 모든 거래는 `priority_queue`에 저장되며, `exec_date`가 빠른 순 → `transaction_id` 순으로 정렬됩니다.

```cpp
struct Transaction {
    uint64_t timestamp, exec_date;
    std::string sender, recipient, ip;
    uint32_t amount;
    char fee_type; // 'f' = sender pays full, 's' = shared
    uint32_t transaction_id;
};
```

## 수수료 정책 (`calculateFee()`)

- 기본 수수료: `amount / 100`  
- 최소 \$10, 최대 \$450  
- 가입 5년 이상 고객 → 수수료 25% 할인  
- `fee_type == 's'`이면 송신자/수신자가 수수료를 절반씩 부담

---

## 명령어 요약

### 트랜잭션 이전 (`processCommands()`)

| 명령어     | 설명                                        |
|------------|---------------------------------------------|
| `login`    | 사용자 인증 후 IP 세션 추가                 |
| `out`      | IP 세션 로그아웃                            |
| `balance`  | 로그인 상태일 때 잔액 출력                  |
| `place`    | 거래 예약 (미래 실행)                       |
| `$$$`      | 현재까지 실행 가능한 거래를 모두 실행 후 쿼리 모드 진입 |

### 쿼리 모드 (`processQueries()`)

| 명령어        | 설명                                                         |
|---------------|--------------------------------------------------------------|
| `l start end` | 특정 기간 내의 거래 내역 출력 (`listTransactions`)           |
| `r start end` | 기간 내 수수료 총액 출력 (`calculateRevenue`)               |
| `h user_id`   | 특정 사용자의 거래 내역 출력 (`displayCustomerHistory`)     |
| `s timestamp` | 특정 날짜의 거래 요약 및 수수료 출력 (`summarizeDay`)       |

---

## 에러 처리

| 조건                                 | 처리 방식                                  |
|--------------------------------------|--------------------------------------------|
| `timestamp < 이전 timestamp`         | `exit(1)` + 에러 메시지 출력               |
| `exec_date < timestamp`              | `exit(1)` + 에러 메시지 출력               |
| 미등록 사용자 or 잘못된 IP           | 무시 또는 verbose 모드일 경우 경고 출력     |
| `self-transfer`, 미래 3일 이상 예약 | 거래 무시 또는 verbose 알림                 |

---

## 설계 요점

- **세션 인증 모델:** 사용자 상태를 유지하는 IP 기반 세션 모델 설계
- **우선순위 큐:** 거래 실행 시점 기준으로 자동 정렬 및 효율적인 실행
- **모듈 분리:** 명령 파싱, 거래 로직, 쿼리 응답 등 기능별 분리 구조
- **시간 일관성:** 모든 시간은 정수형 `YYMMDDHHMMSS`로 정렬 및 비교 가능

---

## 회고

이 프로젝트는 **상태 기반 시스템 설계**, **예약 이벤트 처리**, **사용자 인증 시뮬레이션**, **쿼리 처리** 등  
복합적인 시스템 구조를 직접 설계하고 구현할 수 있는 좋은 기회였습니다.


---

> 본 프로젝트는 EECS 281 수업의 개인 과제로,  
> 스펙에 맞춰 모든 기능을 C++로 직접 설계 및 구현하였습니다.