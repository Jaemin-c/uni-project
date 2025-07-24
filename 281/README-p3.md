## Project 3: 281Bank - EECS 281

> **Note:** Due to university policies, the source code is not publicly available. This document provides a project overview and my personal reflections.  
> **안내:** EECS 281 수업은 **모든 프로젝트를 개인이 단독으로 수행**하는 수업입니다.  
> 학교 정책상 소스 코드는 비공개입니다. 이 문서에는 프로젝트 개요, 구현 방식, 그리고 개인적인 회고가 담겨 있습니다.

---

**언어/기술:** C++, STL, Priority Queue, Custom Comparator, File I/O, Parsing  

### 🔍 프로젝트 개요
파일로 주어진 사용자 등록 정보와 다양한 명령어를 처리해, 은행 시스템을 시뮬레이션하는 프로젝트입니다.  
사용자의 로그인/로그아웃, 거래 요청(place), 거래 실행, 잔고 조회 및 거래 내역 조회 등의 기능을 구현했습니다.  
정해진 명세에 따라 오류 처리를 하고, 다양한 쿼리 명령을 통해 거래 내역 및 수익 요약을 출력합니다.

### 주요 기능
- **사용자 등록 및 로그인 관리:** 등록 파일에서 사용자 정보 로드, 로그인 상태 추적(IP 기반)
- **거래 요청 처리 (`place`):** 타임스탬프 기반 실행 예약, 실행일 도달 시 거래 실행
- **거래 수수료 계산:** 거래 금액, 수수료 유형, 장기 고객 여부에 따라 수수료 차등 부과
- **명령어 처리:**
  - `login`, `out`, `balance`: 사용자 인증 및 로그인 상태 기반 행동 처리
  - `place`: 미래 실행 시점을 가진 거래 요청을 큐에 저장
  - `$$$`: 현재 시점까지 실행 가능한 거래들을 모두 실행 후 쿼리 처리 단계로 진입
- **쿼리 처리:**
  - `l start end`: 특정 기간의 거래 출력
  - `r start end`: 특정 기간 동안 은행의 수익 출력
  - `h user`: 특정 사용자 거래 내역 출력
  - `s timestamp`: 하루 동안의 거래 요약 출력
- **에러 처리:**
  - 이전 `place`보다 빠른 timestamp
  - 실행일이 timestamp보다 이전인 경우
- **명령줄 옵션:**
  - `-f [filename]`: 등록 파일 입력
  - `-v` / `--verbose`: 디버깅을 위한 상세 출력

### 설계 및 구현
- `Bank` 클래스를 중심으로 전체 로직 구현
- 사용자 정보는 `unordered_map`으로 저장, 로그인 세션은 `unordered_set`으로 관리
- 실행 대기 거래는 커스텀 comparator가 적용된 `priority_queue`로 관리 (정렬 기준: 실행 시점 + 등록 순서)
- 날짜 처리를 위해 타임스탬프를 `uint64_t`로 직접 파싱 및 비교
- 수수료 정책은 최소 $10, 최대 $450, 장기 고객(5년 이상) 할인 등 명세에 따라 계산

---
