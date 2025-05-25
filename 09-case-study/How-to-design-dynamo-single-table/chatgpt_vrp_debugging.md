
# ChatGPT와 함께한 VRP 성능 디버깅 회고

## 🧭 문제 정의
- **시나리오**: Python 기반 FastAPI 서버에서 VRP (Vehicle Routing Problem) 계산 API가 CPU 8개를 할당받았음에도 불구하고 **낮은 CPU 사용률 (~15%)**과 **높은 응답 시간(187초/1000요청)**을 보임
- **목표**: 병렬 처리 구조로 CPU를 최대한 활용하여 latency와 throughput 개선

---

## 🧩 질문 & 응답 흐름 정리

### 1. 문제 현상 파악
> "VRP 계산 서버가 latency는 높은데 CPU는 거의 안 쓰고 있어요."
- ChatGPT: GIL이 병목 원인일 수 있으며, 멀티프로세스 구조 전환을 제안

### 2. 구조 변경 시도
> "worker 수를 8에서 1로 줄이고, ProcessPoolExecutor를 써봤는데 성능이 그대로예요."
- ChatGPT: executor가 실제로 병렬로 동작하는지, PID 확인 및 CPU 사용률 확인을 권장

### 3. GIL 병목 검증
> "py-spy로 GIL 100%를 봤고, dataclasses, pandas, copy가 많이 호출되고 있어요."
- ChatGPT: GIL 병목 확정, pandas groupby, deepcopy, dataclass 변환 등의 병목 가능성 제시

### 4. 구조 정비 및 uvicorn 테스트
> "spawn 방식으로 ProcessPoolExecutor 만들고, preload 끄고 uvicorn 단독으로 돌려봤는데 CPU 사용률 그대로예요."
- ChatGPT: executor가 진짜 병렬로 실행되는지 확인하기 위해 PID 추적, 반복 연산 burn 테스트 제안

### 5. Docker 환경 의심
> "같은 머신에서 Java는 잘 동작하는데 Python 컨테이너만 CPU를 안 써요."
- ChatGPT: Python multiprocessing과 spawn 모드에서의 실행 실패, Docker 제한, 이미지 문제 가능성 제기

---

## 🧨 왜 실패했다고 생각했는가?

### 1. **근본 원인이 확실히 밝혀지지 않았음**
- GIL, executor, Docker 이미지, Python 버전 등 다양한 원인을 시도했지만 명확한 원인 확정이 되지 않음
- subprocess가 떠 있는 것은 확인했지만 실제로 CPU가 사용되지 않음 → 원인 미해결

### 2. **단일 실험에 대한 결정적 증거 부족**
- `solve_vrp` 함수 내부에서 실제 CPU 연산이 수행되고 있는지, 연산 강도에 대한 충분한 증명 부족
- PID 로그만으로는 병렬 처리 여부의 확정이 어려움 → 구체적인 CPU 프로파일링 시각화까지 연결되지 않음

### 3. **Docker 환경 하에서 Python의 `multiprocessing` 신뢰성 부족**
- 동일한 구조가 독립된 `.py` 스크립트로는 동작했지만, FastAPI 서버에서는 불확실하게 작동함
- 이는 실무 적용에 불확실성을 더해줌

---

## 🔬 ChatGPT를 활용한 설계 접근의 장점

- 빠르게 문제의 **가능성 있는 원인 목록을 좁혀갈 수 있음**
- 실험 설계를 빠르게 반복하면서 구조적 개선 방향을 얻을 수 있음
- Python 런타임 내부 동작이나 GIL, spawn/fork 차이 등에 대한 설명이 빠르고 명확함

## ⚠️ 한계

- ChatGPT는 **실제 시스템 리소스 상태나 Docker 내부 환경**은 직접 알 수 없기 때문에 마지막 퍼즐 조각을 채우지 못함
- 로그, 프로세스 상태, 부하 상황 등을 종합적으로 보고 판단하는 **인간의 시스템 직관**이 필요함

---

## 📌 다음 실험 계획 예시

1. `solve_vrp()` 내부에 강한 CPU 부하 연산을 명시적으로 넣고, **PID별 CPU 점유율 추적**
2. `multiprocessing` 대신 **`concurrent.futures.ProcessPoolExecutor` + `submit` 직접 사용**으로 동작 보장 구조 실험
3. Python 3.10 → 3.11 변경 후 동일 구조 실험 (spawn 안정성 향상)
4. 같은 코드 베이스를 **Java로 포팅해서 baseline 비교**
5. Docker 컨테이너를 `python:3.10-bullseye`로 교체 후 실험 반복

---

## 🧠 요약
> 이 회고는 단지 문제 해결이 아닌, **문제에 접근하고 가설을 세우고, 실험을 구성하고, 결과를 해석하는 사고의 흐름**을 기록하는 데 의미가 있음.

ChatGPT는 이 흐름을 촉진하는 **대화형 설계 파트너**로서 매우 유용했으며, 다음엔 이 기록을 기반으로 좀 더 결정적인 실험 설계로 이어질 수 있을 것이다.
