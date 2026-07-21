# 6. 품질 속성 (Quality Attributes)

> Tizen TV AI Web Agent (AI Browser) 설계 문서
> 본 장은 5장의 NFR을 입력으로, **Utility Tree**·**QA 시나리오**·**QA 간 트레이드오프**를 정리한다.
> SEI ATAM(Architecture Tradeoff Analysis Method)과 arc42 §10의 양식을 따른다.

---

## 6.1 개요 (Overview)

5장의 NFR이 "각 품질 속성이 어떤 측정값을 만족해야 하는가"를 정의했다면, 본 장은 그 위에서 다음을 정리한다.

1. **Utility Tree** — 시스템 전체 효용(Utility)을 최상위로 하고, 품질 속성(Quality Attribute) → 정련(Refinement) → 시나리오(Scenario)의 계층으로 분해한다. 각 leaf 시나리오에는 (Importance, Difficulty/Risk) 우선순위를 부여한다.
2. **QA 시나리오 카탈로그** (6.6) — leaf 시나리오를 SEI 6-part 형식(Source / Stimulus / Artifact / Environment / Response / Response Measure)으로 풀어 쓴다. NFR과 다른 점은 "환경(Environment)"과 "원천(Source)"을 명시하여 ATAM 워크숍의 직접 입력으로 사용 가능하게 한다는 점이다.
3. **QA 간 트레이드오프** (6.7) — 한 품질 속성을 강화하면 다른 속성이 약화되는 지점을 명시하고, 결정 근거를 7장(Solution Strategy)·9장(ADR)으로 위임한다.

본 장의 출력은 7장(Solution Strategy)·8장(Views)·10장(ATAM)의 입력으로 사용된다.

---

## 6.2 Utility Tree

### 6.2.1 표기법

- 각 leaf 시나리오 옆 `[I, D]` 는 **(Importance, Difficulty/Risk)** 우선순위다.
  - **Importance**: 본 시스템의 비즈니스 가치·라인업 탑재·규제 통과에 미치는 영향.
  - **Difficulty/Risk**: 해당 시나리오의 기술적 달성 난이도와 미달성 시 리스크.
  - 값: **H** (High) · **M** (Medium) · **L** (Low)
- `→ NFR-NNN` 표기는 해당 leaf 시나리오를 정량화한 NFR을 가리킨다.
- ATAM 관행에 따라 `[H, H]` (High Importance & High Difficulty)인 leaf가 **아키텍처 결정의 핵심 후보**이며, 7장·9장에서 우선 다룬다.

### 6.2.2 트리

```
Utility (Tizen TV AI Web Agent의 효용)
│
├── Performance (응답성·자원 효율)
│   │
│   ├── 응답성 (Responsiveness)
│   │   ├── 단순 명령 1초 이내 응답 (Quick Action 경로)        [H, H]  → NFR-001
│   │   ├── 복잡 태스크 1초 이내 첫 피드백                     [H, M]  → NFR-002
│   │   ├── 진행 상황 step별 갱신                              [M, L]  → NFR-003
│   │   ├── 외부 Skill 호출 라우팅 오버헤드 ≤ 500ms            [M, M]  → NFR-004
│   │   ├── 슬립 복귀 1초 이내 가용                            [H, H]  → NFR-005
│   │   ├── HITL 응답 후 재개 P95 ≤ 500ms                      [M, L]  → NFR-028
│   │   ├── 사용자 개입 시 동작 중지 P95 ≤ TBD                 [H, H]  → NFR-034
│   │   └── 복잡 태스크 카테고리별 완료 시간 한도               [H, H]  → NFR-030
│   │
│   └── 자원 효율 (Resource Efficiency)
│       ├── 메모리 peak ≤ 50MB 준수 (전 라인업 공통)            [H, H]  → NFR-006
│       ├── 백그라운드 CPU ≤ 15%, 영상 프레임 드롭 없음          [H, M]  → NFR-007
│       ├── 세션당 토큰 비용 예산 준수                           [H, H]  → NFR-008
│       └── 영구 저장소 예산 준수 (전 라인업 공통)               [M, M]  → NFR-029
│
├── Availability (가용성·복원력)
│   │
│   ├── 격리성 (Isolation)
│   │   └── 에이전트 오류의 상위 시스템 영향 격리 (≥99.95%)     [H, H]  → NFR-009
│   │
│   └── 복원력 (Resilience)
│       └── 네트워크 끊김 후 30초 내 재개 성공률 ≥90%           [H, M]  → NFR-010
│
├── Security (보안)
│   │
│   ├── 경계 통제 (Boundary Control)
│   │   ├── Web Runtime 샌드박스 경계 위반 = 0 / 권한 최소화     [H, M]  → NFR-012
│   │   └── 외부 호출자 인증·권한 검증 (무인증 거부 = 100%)     [H, M]  → NFR-015
│   │
│   └── 공격 표면 방어 (Attack Surface Defense)
│       └── 프롬프트 인젝션 탐지율 (벤치마크 기반)              [H, H]  → NFR-014
│
├── Privacy (프라이버시)
│   │
│   ├── 데이터 최소화 (Data Minimization)
│   │   └── 클라우드 전송 시 PII 탐지·마스킹 ≥95%               [H, M]  → NFR-018
│   │
│   ├── 보존 통제 (Retention Control)
│   │   └── PII 기본 보존 90일, 만료 후 24시간 내 삭제          [H, M]  → NFR-017
│   │
│   └── 데이터 주체 권리 (Data Subject Rights)
│       └── 삭제 ≤7일, 열람·이전 ≤30일                          [M, L]  → NFR-019
│
├── Functional Quality (정확성·신뢰성)
│   │
│   ├── 의도 해석 정확도
│   │   └── 표준 시나리오 의도 분류 정확도 ≥90%                 [H, H]  → NFR-020
│   │
│   └── 태스크 수행 정확도
│       └── Top-100 시나리오 성공률 ≥80%, 부분+성공 ≥90%         [H, H]  → NFR-021
│
├── Accessibility (접근성)
│   │
│   └── 표준 준수
│       └── WCAG 2.2 AA 위반 = 0 (GenUI 인터페이스 산출물 포함)  [H, M]  → NFR-022
│
├── Compatibility (라인업 호환성)
│   │
│   └── 라인업 일관성 (Consistency across SKUs)
│       └── 동일 등급 SKU 간 태스크 성공률 편차 ≤5%             [M, M]  → NFR-011
│
├── Modifiability (유지보수성·진화성)
│   │
│   ├── 인터페이스 안정성
│   │   └── 메이저 버전 변경 시 이전 버전 ≥12개월 지원          [M, L]  → NFR-023
│   │
│   ├── 모듈 결합도
│   │   └── 영역 간 비공개 심볼 참조 = 0                        [M, M]  → NFR-024
│   │
│   └── 배포성 (Deployability)
│       └── 펌웨어와 독립적 다운로드·롤백 배포                  [M, M]  → NFR-035
│
├── Testability (테스트 가능성)
│   │
│   ├── 결정적 재현
│   │   └── Top-100 시나리오 시뮬레이션 커버리지 ≥80%, 결정성 ≥95%  [H, H]  → NFR-025
│   │
│   └── 격리 테스트
│       └── 단위 테스트 외부 의존성 = 0%, Mock 커버리지 ≥95%    [M, L]  → NFR-026
│
└── Observability (관측성)
    │
    └── 추적성 (Traceability)
        └── 에이전트 트레이스 보존 ≥30일, 조회 P95 ≤5s          [M, L]  → NFR-027
```

---

## 6.3 우선순위 분석

### 6.3.1 [H, H] — 핵심 결정 후보 (Critical)

본 시스템 설계의 **아키텍처 결정 핵심 동인**이 되는 11개 leaf다. 7장(Solution Strategy)·9장(ADR)에서 우선 다뤄야 할 항목이다.

| 시나리오 | QA (정련) | NFR | 결정 영역 (예측) |
|---------|----------|-----|------|
| 단순 명령 1초 이내 응답 (Quick Action) | Performance — 응답성 | NFR-001 | 에이전트 루프 우회 경로 설계 (Quick Action 라우터) |
| 슬립 복귀 1초 이내 가용 | Performance — 응답성 | NFR-005 | Warm-start / pre-warming 전략, 하네스 생명주기 |
| 복잡 태스크 카테고리별 완료 시간 한도 | Performance — 응답성 | NFR-030 | 시나리오별 모델·도구 선택 정책, 병렬 도구 호출 |
| 메모리 peak ≤ 50MB (전 라인업 공통) | Performance — 자원 효율 | NFR-006 | 메모리 풋프린트 최적화, 컴포넌트 상주 전략 |
| 세션당 토큰 비용 예산 준수 | Performance — 자원 효율 | NFR-008 | 컨텍스트 압축, 응답 캐싱, Quick Action 경로(LLM 호출 회피) |
| 에이전트 오류의 상위 시스템 격리 (≥99.95%) | Availability — 격리성 | NFR-009 | 프로세스 격리·샌드박스 경계, 폴백 전략 |
| 프롬프트 인젝션 탐지 (벤치마크 기반) | Security — 공격 표면 방어 | NFR-014 | 입력 검증 파이프라인, 외부 콘텐츠 신뢰 모델 |
| 의도 해석 정확도 ≥90% | Functional Quality — 의도 정확도 | NFR-020 | NLU 전략 (LLM-기반 vs 하이브리드) |
| 태스크 성공률 ≥80% | Functional Quality — 수행 정확도 | NFR-021 | 에이전트 루프 설계, HITL 적용 기준, 시나리오 카탈로그 |
| 시뮬레이션 커버리지·결정성 (≥80% / ≥95%) | Testability — 결정적 재현 | NFR-025 | 테스트 인프라, 외부 의존성 모킹 전략 |
| 사용자 개입 시 동작 중지 (취소·제어권 회수) | Performance — 응답성 | NFR-034 | 인터럽트 전파·in-flight 도구 호출 중단, 제어권 이양, 외부 부작용 경계 |

### 6.3.2 [H, M] — 표준 기법 적용 영역 (High value, Established)

품질 속성 가치는 높지만 산업 표준 기법으로 달성 가능한 영역. 표준 적용 + 검증 게이트가 핵심.

- NFR-002 첫 피드백 1초 / NFR-007 CPU 점유율 / NFR-010 네트워크 재개
- NFR-012 샌드박스·권한최소화 / NFR-015 외부 인증
- NFR-017 PII 보존 / NFR-018 PII 마스킹 / NFR-022 WCAG 2.2 AA

### 6.3.3 [M, *] — 표준 운영 영역

표준 도구·관행으로 달성하며, 본 장 우선순위는 낮지만 12장(로드맵)·운영 단계에서 챙겨야 할 항목.

- NFR-003 진행 표시 / NFR-004 Skill 라우팅 / NFR-011 SKU 성공률 동등성 / NFR-019 데이터 주체 권리
- NFR-023 인터페이스 호환성 / NFR-024 모듈 결합도 / NFR-026 Mock / NFR-027 트레이스 / NFR-028 HITL 재개 / NFR-029 저장소 / NFR-035 독립 배포

---

## 6.4 카운트 요약

| 우선순위 | leaf 수 | 비중 |
|---------|--------|------|
| [H, H] | 11 | 37% |
| [H, M] | 8 | 27% |
| [M, *] | 11 | 37% |
| **총** | **30** | 100% |

[H, H] leaf가 전체의 1/3 이상을 차지하며, 이는 본 시스템이 **AI Agent 특유의 신생 영역 + TV의 리소스·UX 제약 + 라인업·규제 환경**의 교집합에서 동작하는 데 따른 결과다. 이 11개가 7~9장 핵심 결정의 입력이 된다.

---

## 6.5 후속 작업

본 절은 향후 작성될 항목의 목차다.

- **6.6 QA 시나리오 카탈로그** — 전체 30개 leaf를 SEI 6-part 형식으로 풀어 쓰고([H,H]·[H,M]·[M,*] 3그룹, QAS-001~030), ATAM 워크숍 입력으로 사용. *(작성됨, 아래 6.6 참조)*
- **6.7 QA 간 트레이드오프** — 충돌 지점 식별 및 7·9장 위임. *(작성됨, 아래 6.7 참조)*
- **6.8 Open Questions** — 본 장 내부 미결 사항. 5장 Open Questions와 별개로 관리. *(작성됨, 아래 6.8 참조)*

---

## 6.6 QA 시나리오 카탈로그

본 절은 6.2.2 Utility Tree의 **전체 30개 leaf**를 SEI 6-part 형식(Source / Stimulus / Artifact / Environment / Response / Response Measure)으로 풀어 쓴다. 각 시나리오는 5장 NFR을 정량 기준으로 인용하며, 8장(Views)·9장(ADR)·10장(ATAM)의 직접 입력으로 사용된다. 시나리오는 6.3의 우선순위 분류를 따라 세 그룹으로 배치하며, QAS 번호와 leaf(NFR)는 1:1로 대응한다.

- **6.6.1 [H, H] 핵심 시나리오** — QAS-001~011 (6.3.1의 아키텍처 결정 핵심 후보)
- **6.6.2 [H, M] 시나리오** — QAS-012~019 (6.3.2의 표준 기법 적용 영역)
- **6.6.3 [M, *] 시나리오** — QAS-020~030 (6.3.3의 표준 운영 영역)

표기:
- **Mapped NFR**: 본 시나리오를 정량화하는 NFR ID
- **Mapped QA leaf**: 6.2.2 Utility Tree 상의 위치 (Quality Attribute — 정련)

세 그룹 모두 동일한 6-part 형식으로 기술하여 ATAM 워크숍에서 우선순위와 무관하게 참조 가능하게 한다. 단, **[H, H] 그룹만 7·9장 핵심 결정의 1차 입력**이 되며, 나머지 두 그룹은 표준 기법·운영으로 달성한다(6.3 참조). TBD 수치는 해당 NFR·ADR 위치를 그대로 인용한다.

---

### 6.6.1 [H, H] 핵심 시나리오

6.3.1의 아키텍처 결정 핵심 후보 11개다.

#### QAS-001 단순 명령 1초 이내 응답 (Quick Action)

- **Source**: 사용자 (End User)
- **Stimulus**: 사용자가 단순 명령(채널 변경, 날씨 조회 등 Quick Action 후보)의 발화를 종료함
- **Artifact**: Quick Action 라우터, 의도 분류기, 외부 의존 컴포넌트(TV 채널 제어, 정보 조회 API 등)
- **Environment**: 정상 운영 상태, 정상 네트워크 조건, TV가 활성 시청 중
- **Response**: 시스템이 의도를 식별하고, 에이전트 루프를 우회하여 결과를 표시하거나 동작을 완료함
- **Response Measure**: P95 ≤ **1.0초** (정상 네트워크 조건)
- **Mapped NFR**: NFR-001
- **Mapped QA leaf**: Performance — 응답성 (Quick Action 경로) [H, H]

---

#### QAS-002 슬립 복귀 1초 이내 가용

- **Source**: TV 플랫폼 (전원 상태 변화 이벤트)
- **Stimulus**: TV가 슬립 모드에서 복귀함
- **Artifact**: 에이전트 하네스, 세션 상태 저장소
- **Environment**: 슬립 복귀 직후, 정상 네트워크, 사용자가 직후 명령 시도 가능 상태
- **Response**: 에이전트 하네스가 초기화되어 명령을 수락 가능한 상태가 됨
- **Response Measure**: ≤ **1.0초**
- **Mapped NFR**: NFR-005
- **Mapped QA leaf**: Performance — 응답성 [H, H]

---

#### QAS-003 복잡 태스크 카테고리별 End-to-end 완료 시간

- **Source**: 사용자 (End User)
- **Stimulus**: 사용자가 복잡한 태스크(검색·예약·구매 등)를 요청함
- **Artifact**: 에이전트 루프 전체, 도구 호출 파이프라인, Generative UI 렌더러
- **Environment**: 정상 네트워크, Cloud LLM API 정상 응답, HITL 대기 시간은 측정에서 제외
- **Response**: 시스템이 태스크를 완료하고 결과를 사용자에게 표시함
- **Response Measure**: 시나리오 카테고리별 P95 — **TBD** (정보 조회·예약·구매 트랜잭션 각각, NFR-030 참조)
- **Mapped NFR**: NFR-030
- **Mapped QA leaf**: Performance — 응답성 [H, H]

---

#### QAS-004 메모리 풋프린트 예산 준수 (전 라인업 공통)

- **Source**: 시스템 운영 자체 (자원 점유)
- **Stimulus**: 에이전트가 일반적인 태스크를 수행 중임
- **Artifact**: 에이전트 하네스 + Generative UI 렌더러
- **Environment**: 전 라인업 공통 메모리 예산, Main Agent·GenUI Renderer·기존 앱과 공존
- **Response**: 시스템이 메모리를 점유하며 동작
- **Response Measure**: 합산 peak 메모리 ≤ **50 MB** (전 라인업 공통, NFR-006 참조)
- **Mapped NFR**: NFR-006
- **Mapped QA leaf**: Performance — 자원 효율 [H, H]

---

#### QAS-005 세션당 토큰 비용 예산 준수

- **Source**: 사용자 (1개 태스크 세션 요청 발생)
- **Stimulus**: 사용자가 1개 태스크 세션(요청 ~ 완료)을 수행함
- **Artifact**: 클라우드 LLM 호출 경로, 컨텍스트 압축기, 응답 캐시
- **Environment**: 정상 운영, 외부 Cloud LLM API 가용, 베타 사용 데이터 수집 환경
- **Response**: 시스템이 LLM API를 호출하여 토큰을 소비함
- **Response Measure**: 세션 평균 ≤ **TBD tokens**, P95 ≤ **TBD tokens** (NFR-008 참조)
- **Mapped NFR**: NFR-008
- **Mapped QA leaf**: Performance — 자원 효율 [H, H]

---

#### QAS-006 에이전트 오류의 상위 시스템 격리

- **Source**: 에이전트 하네스 내부 결함 (crash, 무한 루프, OOM 등)
- **Stimulus**: 에이전트 하네스에 복구 불가능한 오류가 발생함
- **Artifact**: 격리 경계 (프로세스/샌드박스), 호스트 시스템(Main Agent · Generative UI Renderer), 폴백 핸들러
- **Environment**: 운영 환경, 오류 발생 빈도 측정 기간 내
- **Response**: 시스템이 오류를 격리하고, 호스트 시스템은 정상 동작을 유지함
- **Response Measure**: 호스트 시스템 가용성 ≥ **99.95%** (에이전트 오류 1000건 발생 시에도 유지, NFR-009 참조)
- **Mapped NFR**: NFR-009
- **Mapped QA leaf**: Availability — 격리성 [H, H]

---

#### QAS-007 프롬프트 인젝션 탐지

- **Source**: 외부 웹 콘텐츠 (악의적·비정상 페이지)
- **Stimulus**: 인젝션 패턴이 포함된 외부 웹 콘텐츠를 에이전트가 처리함
- **Artifact**: 입력 검증 파이프라인, 모델 호출 전처리기, 외부 콘텐츠 신뢰 모델
- **Environment**: 정상 운영 + 합의된 벤치마크 셋(OWASP LLM01 외) 평가 환경
- **Response**: 시스템이 해당 명령을 무시하고 사용자에게 통지함
- **Response Measure**: 탐지율·오탐률 — **TBD (ADR-XXX)** (NFR-014 참조)
- **Mapped NFR**: NFR-014
- **Mapped QA leaf**: Security — 공격 표면 방어 [H, H]

---

#### QAS-008 의도 해석 정확도

- **Source**: 사용자 (외부 ASR이 전사한 발화)
- **Stimulus**: 시스템이 외부 ASR로부터 전사된 사용자 발화를 입력으로 수신함
- **Artifact**: 의도 해석 컴포넌트 (NLU; LLM 기반 또는 하이브리드), 의도 분류 결과 스키마
- **Environment**: 표준 시나리오 벤치마크 환경, 본 시스템에 도착한 시점에 발화는 이미 전사 완료 상태
- **Response**: 시스템이 사용자 의도를 분류·해석함
- **Response Measure**: 표준 시나리오 벤치마크에서 의도 분류 정확도 ≥ **90%** (NFR-020 참조)
- **Mapped NFR**: NFR-020
- **Mapped QA leaf**: Functional Quality — 의도 정확도 [H, H]

---

#### QAS-009 태스크 완료 성공률

- **Source**: 사용자 (End User)
- **Stimulus**: 사용자가 표준 시나리오(검색·예약·구매 등)에 해당하는 태스크를 요청함
- **Artifact**: 에이전트 루프 전체 (계획 · 도구 호출 · HITL 게이트 · 결과 합성)
- **Environment**: Top-100 시나리오 벤치마크 환경, 정상 외부 서비스 가용
- **Response**: 시스템이 태스크를 수행하여 완료함
- **Response Measure**: 성공률 ≥ **80%**, 부분 완료 + 성공 합산 ≥ **90%** (NFR-021 참조)
- **Mapped NFR**: NFR-021
- **Mapped QA leaf**: Functional Quality — 수행 정확도 [H, H]

---

#### QAS-010 시뮬레이션 모드 커버리지·재현 결정성

- **Source**: QA 엔지니어 (또는 CI 시스템)
- **Stimulus**: 자동화 회귀 테스트 스위트를 실행함
- **Artifact**: 시뮬레이션 환경, 외부 의존 Mock/Stub, 결정적 시드·고정 컨텍스트 인프라
- **Environment**: 시뮬레이션 모드 활성, 실제 외부 서비스·사용자 데이터 차단
- **Response**: 시스템이 시뮬레이션 환경에서 표준 시나리오를 결정적으로 재현함
- **Response Measure**: Top-100 시나리오 시뮬레이션 커버리지 ≥ **80%**, 재현 결정성(동일 입력 → 동일 출력) ≥ **95%** (NFR-025 참조)
- **Mapped NFR**: NFR-025
- **Mapped QA leaf**: Testability — 결정적 재현 [H, H]

---

#### QAS-011 사용자 개입 시 에이전트 동작 중지

- **Source**: 사용자 (End User)
- **Stimulus**: 에이전트가 태스크를 실행하는 중 사용자가 취소 또는 직접 조작 개입(제어권 회수)을 입력함
- **Artifact**: 에이전트 루프(취소·인터럽트 전파), 도구 호출 파이프라인, 제어권 이양 핸들러, 외부 부작용 경계
- **Environment**: 정상 운영, 에이전트가 태스크 실행 중 (비가역 동작 직전 포함)
- **Response**: 시스템이 진행 중 동작을 중지하고, 새로운 비가역 외부 부작용을 시작하지 않으며, 제어권을 사용자에게 이양하거나 안전 상태로 복귀함
- **Response Measure**: 개입 입력 ~ 중지(추가 비가역 부작용 차단)까지 P95 ≤ **TBD ms** (NFR-034 참조)
- **Mapped NFR**: NFR-034
- **Mapped QA leaf**: Performance — 응답성 (사용자 개입 중지) [H, H]

---

### 6.6.2 [H, M] 시나리오

품질 가치는 높으나 산업 표준 기법으로 달성 가능한 8개다(6.3.2). 표준 적용 + 검증 게이트가 핵심이며, 아키텍처 결정을 새로 만들기보다 기성 패턴을 정확히 적용하는 것이 목표다.

#### QAS-012 복잡 태스크 첫 피드백 1초 이내

- **Source**: 사용자 (End User)
- **Stimulus**: 사용자가 복잡한 태스크(구매·예약 등) 명령의 발화를 종료함
- **Artifact**: 에이전트 하네스 진입 경로, 진행 상태 표시기(Generative UI), 첫 피드백 렌더러
- **Environment**: 정상 운영, 정상 네트워크, Cloud LLM API 정상 응답
- **Response**: 시스템이 첫 진행 단계(예: "검색 중...")를 사용자에게 표시하여 명령 수신을 확인시킴
- **Response Measure**: 발화 종료 ~ 최초 피드백 표시까지 ≤ **1.0초** (NFR-002 참조)
- **Mapped NFR**: NFR-002
- **Mapped QA leaf**: Performance — 응답성 [H, M]

---

#### QAS-013 백그라운드 CPU 점유·영상 프레임 드롭 없음

- **Source**: 시스템 운영 자체 (자원 점유), 동시 VOD 시청 컨텍스트
- **Stimulus**: 에이전트가 백그라운드에서 태스크를 수행 중이며, 사용자가 동시에 VOD를 시청 중임
- **Artifact**: 에이전트 하네스, 브라우저 제어 파이프라인, 미디어 재생 파이프라인(공존)
- **Environment**: 정상 운영, VOD 재생 중, 전 라인업 공통 CPU 예산
- **Response**: 시스템이 CPU를 점유하며 동작하되 동시 재생 영상 품질을 저해하지 않음
- **Response Measure**: 백그라운드 평균 CPU 점유율 ≤ **15%** (전 라인업 공통), 영상 프레임 드롭 = **0** (NFR-007 참조)
- **Mapped NFR**: NFR-007
- **Mapped QA leaf**: Performance — 자원 효율 [H, M]

---

#### QAS-014 네트워크 끊김 후 체크포인트 재개

- **Source**: 네트워크 환경 (TV Wi-Fi 불안정)
- **Stimulus**: 진행 중인 태스크 중 네트워크가 끊겼다가 30초 이내 재연결됨
- **Artifact**: 에이전트 루프(체크포인트 관리), 세션 상태 저장소, 재개 핸들러
- **Environment**: 정상 운영 중 일시적 네트워크 단절, TV Wi-Fi 환경
- **Response**: 시스템이 태스크를 처음부터 재시작하지 않고 마지막 안정 체크포인트에서 재개함
- **Response Measure**: 30초 이내 재연결 시 재개 성공률 ≥ **90%** (NFR-010 참조)
- **Mapped NFR**: NFR-010
- **Mapped QA leaf**: Availability — 복원력 [H, M]

---

#### QAS-015 샌드박스 경계 준수·권한 최소화

- **Source**: 시스템 자체 동작 (정상·비정상 입력 포함)
- **Stimulus**: 시스템이 동작 중이며 태스크가 브라우저·시스템 API 권한을 요구함
- **Artifact**: Web Runtime 샌드박스 경계, 권한 관리자(요청·해제), 정적 분석·런타임 모니터
- **Environment**: 정상 및 비정상 입력, Tizen Web Runtime CTS 평가 환경
- **Response**: 시스템이 샌드박스 경계 내에서만 자원에 접근하고, 태스크에 필요한 최소 권한만 보유하며 종료 후 불필요 권한을 해제함
- **Response Measure**: 샌드박스 위반 시도 = **0건** (정적 분석 + 런타임 모니터링), Tizen Web Runtime CTS 통과, 태스크 종료 후 불필요 권한 잔존 = **0건** (NFR-012 참조)
- **Mapped NFR**: NFR-012
- **Mapped QA leaf**: Security — 경계 통제 [H, M]

---

#### QAS-016 외부 호출자 인증·권한 검증

- **Source**: 외부 에이전트 플랫폼 (argot 등 Skill 호출자)
- **Stimulus**: 외부 에이전트 플랫폼이 Skill 호출 요청을 보냄 (인증·무인증 요청 혼재)
- **Artifact**: Skill 호출 인터페이스, 인증 핸들러(OAuth 2.0 / mTLS), 권한 스코프 검증기
- **Environment**: 정상 운영, 외부 호출 인터페이스 노출, 인증/무인증 요청 혼재
- **Response**: 시스템이 호출자를 인증하고 허가된 범위 내 요청만 처리하며, 무인증 요청은 거부함
- **Response Measure**: 인증 방식 **OAuth 2.0 또는 mTLS**, 토큰 만료 ≤ **1시간**, 무인증 요청 거부율 = **100%** (NFR-015 참조)
- **Mapped NFR**: NFR-015
- **Mapped QA leaf**: Security — 경계 통제 [H, M]

---

#### QAS-017 PII 기본 보존 기간·만료 삭제

- **Source**: 시스템 운영 자체 (데이터 수명주기), 규제 요건(GDPR/PIPA)
- **Stimulus**: 시스템이 PII 가능 데이터(음성 전사·DOM 스냅샷·화면 콘텐츠)를 저장하고 보존 기간이 만료됨
- **Artifact**: 데이터 보존 관리자, 만료 스케줄러, 삭제 실행기
- **Environment**: 정상 운영, 사용자 보존 설정 반영, GDPR Art.5(1)(e)·PIPA 제21조 파기 의무 적용
- **Response**: 시스템이 보존 기간 만료 시 해당 데이터를 자동 삭제함
- **Response Measure**: 기본 보존 기간 ≤ **90일** (사용자 단축 가능), 만료 후 자동 삭제 ≤ **24시간** (NFR-017 참조)
- **Mapped NFR**: NFR-017
- **Mapped QA leaf**: Privacy — 보존 통제 [H, M]

---

#### QAS-018 클라우드 전송 PII 탐지·마스킹

- **Source**: 시스템 자체 (LLM 호출 페이로드 생성)
- **Stimulus**: 시스템이 클라우드 LLM API 호출을 위해 요청 페이로드를 구성함
- **Artifact**: 온디바이스 PII 탐지·마스킹 전처리기, 자격증명 제거기, 컨텍스트 추출기
- **Environment**: 정상 운영, 외부 Cloud LLM API 전송 직전, 정기 페이로드 audit 병행
- **Response**: 시스템이 페이로드에서 PII를 식별·마스킹하고 자격증명을 제거한 뒤 전송함
- **Response Measure**: 자동 PII 탐지 벤치마크 커버리지 ≥ **95%** (탐지 항목 100% 마스킹), 자격증명 전송 사고 = **0건**, 정기 audit 미탐 PII ≤ **1%** (NFR-018 참조)
- **Mapped NFR**: NFR-018
- **Mapped QA leaf**: Privacy — 데이터 최소화 [H, M]

---

#### QAS-019 WCAG 2.2 AA 준수 (GenUI 산출물 포함)

- **Source**: 시스템 자체 (UI 출력), 고령 사용자·규제 요건
- **Stimulus**: 시스템이 사용자에게 UI(Generative UI 인터페이스 산출물 포함)를 출력함
- **Artifact**: 전체 UI 컴포넌트, Generative UI 인터페이스 산출물, 접근성 검사 파이프라인(axe-core 등)
- **Environment**: 정상 운영, 자동 검사 + 수동 검수 평가 환경
- **Response**: 시스템이 출력하는 UI가 WCAG 2.2 Level AA 기준을 충족함
- **Response Measure**: WCAG 2.2 AA 자동 검사 위반 = **0건** (GenUI 산출물 포함 전 컴포넌트), 수동 검수 통과 (NFR-022 참조)
- **Mapped NFR**: NFR-022
- **Mapped QA leaf**: Accessibility — 표준 준수 [H, M]

---

### 6.6.3 [M, *] 시나리오

표준 도구·관행으로 달성하며 본 장 우선순위는 낮으나, 12장(로드맵)·운영 단계에서 챙겨야 할 11개다(6.3.3). ATAM 워크숍의 완전성 확보를 위해 동일 형식으로 기술한다.

#### QAS-020 진행 상황 표시 갱신 주기

- **Source**: 시스템 자체 (진행 상태 발행)
- **Stimulus**: 에이전트가 다단계 태스크를 실행 중임
- **Artifact**: 에이전트 루프(step 이벤트 발행), 진행 상태 표시기(Generative UI)
- **Environment**: 정상 운영, 다단계 태스크 진행 중
- **Response**: 시스템이 각 step마다 현재 단계 표시를 갱신함 (heartbeat)
- **Response Measure**: 매 **step별** 진행 상황 갱신 (정지 구간 없이 지속) (NFR-003 참조)
- **Mapped NFR**: NFR-003
- **Mapped QA leaf**: Performance — 응답성 [M, L]

---

#### QAS-021 외부 Skill 호출 라우팅 오버헤드

- **Source**: 외부 에이전트 플랫폼 (Skill 호출자)
- **Stimulus**: 외부 에이전트 플랫폼이 Skill 호출 요청을 보냄
- **Artifact**: Skill 호출 인터페이스, 인증·라우팅 계층, 태스크 디스패처
- **Environment**: 정상 운영, 본 시스템이 Sub-Agent로 호출되는 시나리오
- **Response**: 시스템이 요청을 수신·인증·라우팅하여 실제 태스크 실행을 시작함
- **Response Measure**: 인증·라우팅 오버헤드 ≤ **500ms** (NFR-004 참조)
- **Mapped NFR**: NFR-004
- **Mapped QA leaf**: Performance — 응답성 [M, M]

---

#### QAS-022 HITL 응답 후 태스크 재개 시간

- **Source**: 사용자 (HITL 확인 응답)
- **Stimulus**: 사용자가 HITL 확인 화면에서 응답(승인 또는 거절)을 입력함
- **Artifact**: HITL 게이트, 에이전트 루프 재개 핸들러, 도구 호출 파이프라인
- **Environment**: 정상 운영, HITL 확인 대기 상태에서 사용자 응답 도착
- **Response**: 시스템이 입력을 처리하고 다음 단계 동작을 시작함 (또는 태스크를 안전하게 중단함)
- **Response Measure**: HITL 응답 ~ 다음 동작 시작까지 P95 ≤ **500ms** (NFR-028 참조)
- **Mapped NFR**: NFR-028
- **Mapped QA leaf**: Performance — 응답성 [M, L]

---

#### QAS-023 영구 저장소 풋프린트 예산 준수

- **Source**: 시스템 운영 자체 (영구 데이터 축적)
- **Stimulus**: 시스템이 일정 기간 운영되며 이력·캐시·트레이스 등 영구 데이터를 축적함
- **Artifact**: 영구 저장소 관리자, 이력·캐시·트레이스 저장소, 보존·정리 정책
- **Environment**: 장기 운영, 전 라인업 공통 디스크 예산
- **Response**: 시스템이 영구 저장소를 점유하되 상한을 초과하지 않음 (초과 전 정리·회전)
- **Response Measure**: 누적 영구 저장소 ≤ **TBD MB (ADR-XXX)** (전 라인업 공통, NFR-029 참조)
- **Mapped NFR**: NFR-029
- **Mapped QA leaf**: Performance — 자원 효율 [M, M]

---

#### QAS-024 SKU 간 태스크 성공률 동등성

- **Source**: 시스템 운영 (라인업 배포)
- **Stimulus**: 동일 등급 라인업의 서로 다른 SKU(개별 판매 모델)에서 동일한 태스크를 수행함
- **Artifact**: 에이전트 루프 전체, SKU별 HW 추상화 계층
- **Environment**: 동일 등급 내 복수 SKU, 표준 시나리오 벤치마크 (응답 시간은 HW 차로 자연 변동 → 측정 제외)
- **Response**: 시스템이 각 SKU에서 태스크를 실행함
- **Response Measure**: 동일 등급 내 SKU 간 **태스크 성공률** 편차 ≤ **5%** (NFR-011 참조)
- **Mapped NFR**: NFR-011
- **Mapped QA leaf**: Compatibility — 라인업 일관성 [M, M]

---

#### QAS-025 데이터 주체 권리 행사 응답 시간

- **Source**: 사용자 (데이터 주체)
- **Stimulus**: 사용자가 개인정보 열람·삭제·이전을 요청함
- **Artifact**: 데이터 주체 권리 처리기, PII 인덱스·저장소, 완료 알림 채널
- **Environment**: 정상 운영, GDPR Art.12(3)·PIPA 제35조 권리 행사, 운영팀 연계
- **Response**: 시스템이 해당 요청을 처리하고 완료 알림을 제공함
- **Response Measure**: 열람·이전 ≤ **30일**, 삭제 ≤ **7일** (NFR-019 참조)
- **Mapped NFR**: NFR-019
- **Mapped QA leaf**: Privacy — 데이터 주체 권리 [M, L]

---

#### QAS-026 공개 인터페이스 하위 호환성 유지

- **Source**: 통합 소비자 (ARGO 개발팀 / 향후 외부 에이전트 플랫폼)
- **Stimulus**: 본 시스템이 공개 인터페이스(Skill 호출 API·결과 반환 스키마)의 메이저 버전을 변경함
- **Artifact**: 공개 인터페이스(Skill API·결과 스키마), 버전 라우터, 하위호환 어댑터
- **Environment**: 메이저 버전 전환기, 이전 버전 통합 클라이언트 잔존
- **Response**: 시스템이 이전 메이저 버전 인터페이스 호출도 정상 처리함
- **Response Measure**: 이전 메이저 버전 지원 기간 ≥ **12개월** (또는 다음 메이저 릴리스 + 6개월 중 더 긴 쪽) (NFR-023 참조)
- **Mapped NFR**: NFR-023
- **Mapped QA leaf**: Modifiability — 인터페이스 안정성 [M, L]

---

#### QAS-027 영역 간 모듈 결합도

- **Source**: 개발팀 (SW 개발팀)
- **Stimulus**: 개발팀이 한 기능 영역(VUI·에이전트 코어·브라우저 제어·콘텐츠 파이프라인 등)의 내부 구현을 변경함
- **Artifact**: 영역 간 공개 인터페이스, 의존성 그래프, 정적 분석 도구
- **Environment**: 개발·CI 환경, 정적 분석 게이트
- **Response**: 변경이 다른 영역의 코드 수정을 강제하지 않음 (공개 인터페이스만을 통해 의존)
- **Response Measure**: 영역 간 비공개 심볼 참조 = **0건** (정적 분석 기준) (NFR-024 참조)
- **Mapped NFR**: NFR-024
- **Mapped QA leaf**: Modifiability — 모듈 결합도 [M, M]

---

#### QAS-028 독립 배포·롤백 가능성

- **Source**: 배포 담당자 / 운영 (결함·보안 이슈 발생)
- **Stimulus**: 운영 중 결함 또는 보안 취약점이 발견되어 수정본이 준비됨
- **Artifact**: 독립 배포 단위(에이전트 하네스), OTA/앱스토어형 업데이트 채널, 버전 관리·롤백 메커니즘
- **Environment**: 운영 중, 전체 펌웨어 릴리스 주기와 분리된 배포 채널
- **Response**: 시스템이 전체 펌웨어와 분리된 경로로 수정본을 다운로드·적용하며, 실패 시 이전 버전으로 롤백함
- **Response Measure**: 전체 펌웨어와 **독립적으로 버전 관리·다운로드·롤백** 가능 (펌웨어 재빌드 불요 = **100%**), 수정본 게시 리드타임 ≤ **TBD (ADR-XXX)** (긴급 보안 패치 별도 SLA, NFR-035 참조)
- **Mapped NFR**: NFR-035
- **Mapped QA leaf**: Modifiability — 배포성 [M, M]

---

#### QAS-029 Mock 격리 단위 테스트

- **Source**: 개발자 (단위 테스트 실행)
- **Stimulus**: 개발자가 단일 영역의 단위 테스트를 실행함
- **Artifact**: 각 기능 경계의 Mock/Stub 인터페이스, 단위 테스트 하네스, CI 러너
- **Environment**: CI·로컬 개발 환경, 외부 서비스·네트워크 차단
- **Response**: 테스트가 다른 영역·외부 서비스 의존 없이 격리되어 실행됨
- **Response Measure**: 단위 테스트 외부 의존성(네트워크·디스크·시스템 호출) = **0%**, Mock 커버리지 ≥ **95%** (NFR-026 참조)
- **Mapped NFR**: NFR-026
- **Mapped QA leaf**: Testability — 격리 테스트 [M, L]

---

#### QAS-030 에이전트 트레이스 보존·조회

- **Source**: 운영팀 (사후 분석), 시스템 (트레이스 생성)
- **Stimulus**: 시스템이 태스크를 실행하며 트레이스를 생성하고, 이후 운영팀이 사후 분석을 위해 조회함
- **Artifact**: 트레이스 수집기, 트레이스 저장소, 조회 인터페이스
- **Environment**: 정상 운영, 사용자 신고 ~ 사후 분석 기간, NFR-018 PII 마스킹 정책 적용
- **Response**: 트레이스가 저장되어 보존 기간 내 조회 가능함
- **Response Measure**: 보존 기간 ≥ **30일**, 조회 응답 P95 ≤ **5s** (NFR-027 참조)
- **Mapped NFR**: NFR-027
- **Mapped QA leaf**: Observability — 추적성 [M, L]

---

## 6.7 QA 간 트레이드오프 (Trade-offs)

본 절은 한 품질 속성을 강화하면 다른 속성이 약화되는 **트레이드오프 지점**을 식별한다. ATAM 용어로, 한 아키텍처 결정이 단일 QA에만 영향을 주면 **민감점(Sensitivity Point)**, 둘 이상의 QA에 *상반된* 영향을 주면 **트레이드오프 지점(Trade-off Point)**이다. 본 절은 후자의 *충돌 구조와 민감 파라미터*까지만 명시하고, **정량 균형점과 선택은 7장(Solution Strategy)·9장(ADR)으로 위임**한다.

대상은 주로 6.3.1의 [H, H] 핵심 결정 후보들이 서로 맞물리는 지점이다.

### 6.7.1 요약

| ID | 트레이드오프 (QA ↔ QA) | 민감 파라미터 | 해소 위치 |
|----|------------------------|---------------|-----------|
| TO-1 | 응답성 ↔ 보안 검증 깊이 | 입력 검증 파이프라인 위치·깊이 | 7장 / ADR(외부 콘텐츠 신뢰 모델) |
| TO-2 | 태스크 정확도 ↔ 프라이버시(데이터 최소화) | 클라우드 전송 컨텍스트 범위·마스킹 적극성 | 7장 / ADR(전송·PII 정책) |
| TO-3 | 자원 예산(메모리·토큰) ↔ 태스크 정확도 | 컨텍스트 압축률·캐시·Quick Action 커버리지 | 7장 / ADR(LLM 비용 전략) |
| TO-4 | 오류 격리·가용성 ↔ 응답성·메모리 | 격리 경계 입도(프로세스 vs 인프로세스) | 7장 / ADR(격리 모델) |
| TO-5 | 클라우드 모델 능력 ↔ 프라이버시·데이터 경계 | 온디바이스 vs 클라우드 처리 분담 | 7장 / ADR(데이터 경계·온디바이스 SLM) |
| TO-6 | 시뮬레이션 결정성 ↔ 실제 충실도 | 시뮬레이션 추상화 수준 | 7장 / ADR(시뮬레이션 전략) |
| TO-7 | 관측성(트레이스) ↔ 프라이버시·저장소 | 트레이스 상세도·마스킹·보존 | 7장 / ADR(트레이스 정책) |

> 6.5에서 후보로 들었던 *Accessibility(비자동화 동등 경로) ↔ Performance*는, 비자동화 동등 경로 원칙(구 FR-048)이 범위에서 제외되며 트레이드오프 대상에서 철회되었다.

### 6.7.2 트레이드오프 상세

#### TO-1 응답성 ↔ 보안 검증 깊이
- **충돌 QA**: Latency / Performance (NFR-001 Quick Action ≤ 1초, NFR-030 완료 시간) ↔ Security (NFR-014 프롬프트 인젝션 탐지)
- **긴장 관계**: 외부 웹 콘텐츠·입력을 깊게 검증할수록 안전하나 지연이 증가한다. 특히 Quick Action 1초 목표는 검증 단계를 최소화해야 달성된다.
- **민감 파라미터**: 입력 검증 파이프라인을 어디에·얼마나 깊게 두는가 (Quick Action 경로 포함 여부).
- **현재 방향(가설)**: 경로 분리 — Quick Action은 신뢰 가능한 1차 입력(외부 ASR 전사 발화)만 처리하고 외부 웹 콘텐츠에 접촉하지 않으므로 경량 처리, 외부 콘텐츠를 다루는 에이전트 루프에서만 full 인젝션 검증을 수행한다. 응답성과 보안을 *경로별로* 분리 충족.
- **관련 [H,H] leaf**: NFR-001, NFR-014
- **해소 위치**: 7장(Quick Action 라우터·입력 검증 파이프라인) / 9장 ADR(외부 콘텐츠 신뢰 모델)

#### TO-2 태스크 정확도 ↔ 프라이버시(데이터 최소화)
- **충돌 QA**: Quality / Accuracy (NFR-020 의도 정확도, NFR-021 성공률) ↔ Privacy (NFR-018 클라우드 전송 최소화·PII 마스킹)
- **긴장 관계**: 모델에 더 풍부한 컨텍스트(DOM·화면·이력)를 줄수록 성공률이 오르나, PII 마스킹·최소 전송이 컨텍스트를 깎아 정확도를 낮춘다. 과(過)마스킹은 의미 손실로 직결.
- **민감 파라미터**: 클라우드로 보내는 컨텍스트 범위 + 마스킹 적극성.
- **현재 방향(가설)**: 온디바이스 전처리로 PII만 선별 마스킹(과마스킹 회피)하고 작업 관련 최소 컨텍스트만 추출. 마스킹 토큰은 유형 정보를 보존(typed placeholder)하여 의미 손실 최소화.
- **관련 [H,H] leaf**: NFR-020, NFR-021 / 관련 NFR-018
- **해소 위치**: 7장(컨텍스트 추출·마스킹 전략) / 9장 ADR(전송·PII 처리 정책)

#### TO-3 자원 예산(메모리·토큰) ↔ 태스크 정확도
- **충돌 QA**: Resource Efficiency (NFR-006 메모리 ≤ 50MB, NFR-008 세션 토큰 예산) ↔ Quality / Accuracy (NFR-021 성공률)
- **긴장 관계**: 긴 컨텍스트·캐시는 정확도를 높이나 메모리·토큰·비용 예산을 초과한다. 컨텍스트 압축·캐싱으로 절감하면 정확도 변동 위험.
- **민감 파라미터**: 모델 선택(라우팅) 정책 + 컨텍스트 압축률 + 캐시 크기.
- **현재 방향(가설)**: 단순 명령은 Quick Action 경로로 LLM 호출 자체를 회피, 컨텍스트 압축, 응답 캐싱. 온디바이스 모델을 상주시키지 않고 추론은 클라우드 LLM 직접 호출로 일원화(TC-002와 정합).
- **관련 [H,H] leaf**: NFR-006, NFR-008, NFR-021
- **해소 위치**: 7장(컨텍스트 압축·캐싱·비용 통제) / 9장 ADR(LLM 비용 전략)

#### TO-4 오류 격리·가용성 ↔ 응답성·메모리
- **충돌 QA**: Availability (NFR-009 호스트 격리 ≥ 99.95%) · Security (NFR-012 샌드박스 경계) ↔ Latency / Performance (NFR-005/030) · Resource Efficiency (NFR-006)
- **긴장 관계**: 강한 프로세스 격리·샌드박스 경계는 IPC·직렬화·런타임 중복 오버헤드로 지연과 메모리를 증가시킨다. 반대로 약한 격리는 빠르고 가볍지만 상위 시스템 보호(거버넌스 게이트)를 위협.
- **민감 파라미터**: 격리 경계 입도 — 프로세스 격리 vs 인프로세스 샌드박스.
- **현재 방향(가설)**: 호스트(Main Agent / GenUI Renderer)와의 경계는 강하게(99.95% 게이트는 필수 조건), 본 시스템 내부 컴포넌트 간은 경량 경계로 둔다. 50MB 예산 안에서 격리 경계 수를 최소화.
- **관련 [H,H] leaf**: NFR-009 / 관련 NFR-012, NFR-005, NFR-006
- **해소 위치**: 7장(격리 경계 설계) / 9장 ADR(프로세스 격리 모델)

#### TO-5 클라우드 모델 능력 ↔ 프라이버시·데이터 경계
- **충돌 QA**: Quality / Accuracy · Latency (강력한 Cloud Frontier LLM) ↔ Privacy (NFR-017 보존·NFR-018 전송 최소화)
- **긴장 관계**: TC-002상 추론은 클라우드 LLM이 1차다. 강력하지만 사용자 데이터(발화·DOM·화면)가 외부 공급사로 전송된다. 온디바이스 처리는 프라이버시를 높이나 현재 모델 능력·메모리(50MB) 제약을 받는다.
- **민감 파라미터**: 온디바이스 vs 클라우드 처리 분담 경계.
- **현재 방향(가설)**: 민감 데이터의 전처리·마스킹은 온디바이스에서, 추론은 클라우드에서. 일부 보조 작업(의도 분류 등)의 온디바이스 SLM 가용성은 TC-002 Open Question으로 위임.
- **관련 [H,H] leaf**: NFR-020, NFR-021 / 관련 NFR-017, NFR-018, TC-002
- **해소 위치**: 7장(하이브리드 처리 경계) / 9장 ADR(데이터 경계·온디바이스 SLM)

#### TO-6 시뮬레이션 결정성 ↔ 실제 충실도
- **충돌 QA**: Testability (NFR-025 재현 결정성 ≥ 95%) ↔ Quality / Accuracy (실제 LLM·웹의 비결정성 반영)
- **긴장 관계**: 회귀 테스트의 결정성은 고정 시드·목(mock) 환경을 요구한다. 그러나 실제 에이전트는 LLM 비결정성과 실제 웹 변동에 노출되므로, 시뮬레이션이 실제를 완전히 대표하지 못한다(시뮬레이션 과적합 위험).
- **민감 파라미터**: 시뮬레이션 환경의 추상화 수준 — 목 응답 고정 vs 실제 변동 샘플링.
- **현재 방향(가설)**: 결정적 계층(도구 호출·라우팅 로직)은 목으로 고정해 결정성을 확보하고, 비결정 계층(LLM 출력)은 기록·재생(record/replay)하되 주기적으로 실측과 대조해 충실도 드리프트를 감시.
- **관련 [H,H] leaf**: NFR-025 / 관련 NFR-021
- **해소 위치**: 7장(테스트 인프라) / 9장 ADR(시뮬레이션 전략)

#### TO-7 관측성(트레이스) ↔ 프라이버시·저장소
- **충돌 QA**: Observability (NFR-027 트레이스 보존 ≥ 30일) ↔ Privacy (NFR-017 PII) · Resource Efficiency (NFR-029 저장소)
- **긴장 관계**: 디버깅을 위해 컨텍스트 스냅샷·DOM·모델 응답을 트레이스에 남기면 관측성이 오르나, 트레이스에 PII가 포함되고 저장소 예산을 잠식한다.
- **민감 파라미터**: 트레이스 상세도 + PII 마스킹 + 보존기간.
- **현재 방향(가설)**: 트레이스에도 NFR-018의 PII 마스킹 정책을 재사용하고, 보존기간·샘플링으로 저장소 예산(NFR-029)을 관리.
- **관련 leaf**: NFR-027 / 관련 NFR-017, NFR-029
- **해소 위치**: 7장(관측성 파이프라인) / 9장 ADR(트레이스 정책)

---

## 6.8 Open Questions

본 장(품질 속성) 내부의 미결 사항이다. 5장 NFR Open Questions(수치 확정)와 별개로, 트레이드오프·우선순위 관련 결정을 관리한다.

1. **트레이드오프 정량 균형점** — TO-1~7 각각의 균형 수치(예: 마스킹 적극성 대비 정확도 손실 허용폭, 컨텍스트 압축률, 격리 경계 수)는 7장·9장에서 실측·결정한다. 본 장은 충돌 구조까지만 확정.
2. **Quick Action 후보 명령 집합** — TO-1의 경로 분리는 "어떤 명령이 에이전트 루프를 우회하는 Quick Action인가"의 카탈로그를 전제로 한다(시나리오 챕터·7장 연계).
3. **전체 가용성 SLA 등급** — NFR-009의 99.95%는 호스트 격리 기준값이며, 시스템 전체 가용성 등급(99.9 / 99.95 / 99.99%)은 미확정(5장 Open Questions와 연계).
4. **QA 트리 유지보수 책임** — NFR 추가·삭제 시 Utility Tree·카운트·트레이드오프를 동기 갱신하는 책임·주기가 미정.

---

## 6.9 참고 자료 (References)

1. Bass, Clements, Kazman. *Software Architecture in Practice* (4th ed.) — Quality Attribute Scenarios, Utility Tree.
2. SEI / Kazman et al. *ATAM: Method for Architecture Evaluation* (CMU/SEI-2000-TR-004).
3. arc42 Template v8 — §10 "Quality Requirements".
4. ISO/IEC 25010:2011 — Systems and software Quality Requirements and Evaluation (SQuaRE) — Product Quality Model.
