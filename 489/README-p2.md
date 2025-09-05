# EECS 489 – Project 2: Adaptive Video Streaming via CDN (W2025)

> **Notice**  
> Source code is **not included** to respect academic integrity.  
> 본 문서는 **소스 코드 없이** 프로젝트 설명, 설계 요약, 회고만 담고 있습니다.

---

## Overview
- HTTP Proxy (**miProxy**)와 **Load Balancer**를 구현해 CDN 환경에서 **Adaptive Bitrate Streaming (ABR)**을 지원.  
- miProxy: 클라이언트의 세그먼트 수신 속도를 추정하여 비트레이트를 동적으로 선택.  
- Load Balancer: Round-Robin 또는 Geographic Shortest Path 정책으로 서버를 배정.  

---

## Implementation Summary
- **HTTP/1.1 Parsing**  
  - 헤더/바디 분리 (`\r\n\r\n` 경계, `Content-Length` 처리, case-insensitive 헤더).  

- **Manifest Handling**  
  - 브라우저에는 `*-no-list.mpd` 전달.  
  - 프록시는 원본 `.mpd`를 받아 **pugixml**로 지원 비트레이트(Kbps) 파싱 & 캐싱.  

- **Segment Request Rewriting**  
  - 패턴: `/video/vid-[BITRATE]-seg-[N].m4s` 요청 시 **[BITRATE]를 교체**.  
  - 오디오/init 파일은 수정 없이 전달.  

- **Throughput Estimation (EWMA)**  
  - `POST /on-fragment-received` 헤더(`uuid`, `size`, `start`, `end`) → `T_new = size / duration`.  
  - EWMA 공식: `T_cur = α*T_new + (1-α)*T_cur`.  
  - UUID 단위로 throughput 상태 관리.  

- **Bitrate Selection Policy**  
  - 조건: `T_cur ≥ 1.5 × bitrate`.  
  - 만족하는 최대 비트레이트 선택, 없으면 최저 비트레이트.  

- **Concurrency & Multiplexing**  
  - `select()` 기반 폴링, **요청 단위 처리**.  
  - 클라이언트 소켓 1개 ↔ 비디오 서버 소켓 1개 대응.  

- **Load Balancer**  
  - **RR 모드**: 서버 목록 순환.  
  - **Geo 모드**: 그래프 기반 최단 경로 서버 선택 (tie → 낮은 ID 우선).  
  - 프로토콜 구조체, 정수 필드는 **hton*/ntoh*** 변환 처리.  

- **Logging (spdlog)**  
  - 시작, 연결, 매니페스트, 세그먼트, POST 처리 시 **과제 지정 포맷**으로 출력.  

---


## Usage Examples

### miProxy (single server, no LB)
./miProxy -l 9000 -h 127.0.0.1 -p 8000 -a 0.5

### miProxy (with load balancer)
./miProxy -b -l 9000 -h 127.0.0.1 -p 9001 -a 0.5
Load Balancer
## Round-Robin
./loadBalancer --rr -p 9001 -s sample_round_robin.txt

## Geographic
./loadBalancer --geo -p 9001 -s sample_geography.txt


## What I Focused On
- **바이트 스트림 안전 파싱**: 멀티 요청/응답이 한 번에 들어오는 상황에서도 안정적으로 파싱할 수 있도록 버퍼 경계 관리 및 요청 단위 처리 구현.
- **UUID 단위 세션 관리**: 하나의 브라우저 탭에서 여러 소켓이 열려도 동일한 클라이언트 UUID 기준으로 throughput/EWMA 상태를 일원화.
- **이상치 필터링**: `segment_duration ≤ 0` 같은 비정상 샘플을 EWMA 업데이트에서 제외하여 안정성 확보.
- **α 값 실험**: 0.2 / 0.5 / 0.8 등 다양한 α 값을 실험하여 **반응성 vs. 부드러움** 트레이드오프 조율.
- **로그 계층화**: `info` 레벨은 과제 요구 포맷 준수, `debug` 레벨은 파서/재작성 세부 과정을 출력하도록 분리.

---

## Reflections
- **Parser 품질**이 디버깅 시간을 크게 줄였다. 작은 실수(헤더 대소문자, 경계 처리 등)도 전체 스트리밍 실패로 이어질 수 있음을 체감.
- **모듈화**: 소켓 I/O, ABR 정책, LB 로직을 분리 설계하여 블로킹·레이스 문제를 줄이고 유지보수를 쉽게 만들었다.
- **튜닝 효과**: α와 마진값 조정이 **QoE (버퍼링 빈도, 해상도 안정성)**에 직접적인 차이를 만들었다.

---

## Languages & Tools
- **C++17**, `select()` 기반 멀티플렉싱 I/O
- **spdlog** (structured logging), **cxxopts** (CLI 옵션 파싱), **pugixml** (MPD 파싱), **boost::regex** (HTTP 요청 처리) 
