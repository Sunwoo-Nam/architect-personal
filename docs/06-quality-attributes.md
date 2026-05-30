# 6. 품질 속성 (Quality Attributes)

> Tizen TV AI Web Agent (AI Browser) 설계 문서
> 본 장은 5장의 NFR을 입력으로, **Utility Tree**·**QA 시나리오**·**QA 간 트레이드오프**를 정리한다.
> SEI ATAM(Architecture Tradeoff Analysis Method)과 arc42 §10의 양식을 따른다.

---

## 6.1 개요 (Overview)

5장의 NFR이 "각 품질 속성이 어떤 측정값을 만족해야 하는가"를 정의했다면, 본 장은 그 위에서 다음을 정리한다.

1. **Utility Tree** — 시스템 전체 효용(Utility)을 최상위로 하고, 품질 속성(Quality Attribute) → 정련(Refinement) → 시나리오(Scenario)의 계층으로 분해한다. 각 leaf 시나리오에는 (Importance, Difficulty/Risk) 우선순위를 부여한다.
2. **QA 시나리오 카탈로그** *(다음 단계)* — leaf 시나리오를 SEI 6-part 형식(Source / Stimulus / Artifact / Environment / Response / Response Measure)으로 풀어 쓴다. NFR과 다른 점은 "환경(Environment)"과 "원천(Source)"을 명시하여 ATAM 워크숍의 직접 입력으로 사용 가능하게 한다는 점이다.
3. **QA 간 트레이드오프** *(다음 단계)* — 한 품질 속성을 강화하면 다른 속성이 약화되는 지점을 명시하고, 결정 근거를 7장(Solution Strategy)·9장(ADR)으로 위임한다.

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
│   │   └── 복잡 태스크 카테고리별 완료 시간 한도               [H, H]  → NFR-030
│   │
│   └── 자원 효율 (Resource Efficiency)
│       ├── 메모리 풋프린트 라인업별 예산 준수                  [H, H]  → NFR-006
│       ├── 백그라운드 CPU ≤ 15%, 영상 프레임 드롭 없음          [H, M]  → NFR-007
│       ├── 세션당 토큰 비용 예산 준수                           [H, H]  → NFR-008
│       └── 영구 저장소 라인업별 예산 준수                       [M, M]  → NFR-029
│
├── Availability (가용성·복원력)
│   │
│   ├── 격리성 (Isolation)
│   │   └── 에이전트 오류의 상위 시스템 영향 격리 (≥99.95%)     [H, H]  → NFR-009
│   │
│   ├── 복원력 (Resilience)
│   │   └── 네트워크 끊김 후 30초 내 재개 성공률 ≥90%           [H, M]  → NFR-010
│   │
│   └── 동등성 (Consistency)
│       └── 동일 등급 라인업 SKU 간 응답·성공률 편차 ≤15%/5%    [M, M]  → NFR-011
│
├── Security (보안)
│   │
│   ├── 경계 통제 (Boundary Control)
│   │   ├── Web Runtime 샌드박스 경계 위반 = 0                  [H, M]  → NFR-012
│   │   └── 외부 호출자 인증·권한 검증 (무인증 거부 = 100%)     [H, M]  → NFR-015
│   │
│   ├── 자격증명·자산 보호 (Asset Protection)
│   │   ├── 자격증명 AES-256-GCM + TEE 저장                     [H, M]  → NFR-013
│   │   └── 프로필 간 데이터 격리 (Cross-profile leak = 0)      [H, M]  → NFR-016
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
│       └── WCAG 2.2 AA 위반 = 0 (Generative UI 포함)            [H, M]  → NFR-022
│
├── Modifiability (유지보수성·진화성)
│   │
│   ├── 인터페이스 안정성
│   │   └── 메이저 버전 변경 시 이전 버전 ≥12개월 지원          [M, L]  → NFR-023
│   │
│   └── 모듈 결합도
│       └── 영역 간 비공개 심볼 참조 = 0                        [M, M]  → NFR-024
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
| 메모리 풋프린트 라인업별 예산 준수 | Performance — 자원 효율 | NFR-006 | 모델 추상화 전략, 라인업별 기능 차등 |
| 세션당 토큰 비용 예산 준수 | Performance — 자원 효율 | NFR-008 | 컨텍스트 압축, 모델 라우팅(small/large), 캐싱 |
| 에이전트 오류의 상위 시스템 격리 (≥99.95%) | Availability — 격리성 | NFR-009 | 프로세스 격리·샌드박스 경계, 폴백 전략 |
| 프롬프트 인젝션 탐지 (벤치마크 기반) | Security — 공격 표면 방어 | NFR-014 | 입력 검증 파이프라인, 외부 콘텐츠 신뢰 모델 |
| 의도 해석 정확도 ≥90% | Functional Quality — 의도 정확도 | NFR-020 | NLU 전략 (LLM-기반 vs 하이브리드) |
| 태스크 성공률 ≥80% | Functional Quality — 수행 정확도 | NFR-021 | 에이전트 루프 설계, HITL 적용 기준, 시나리오 카탈로그 |
| 시뮬레이션 커버리지·결정성 (≥80% / ≥95%) | Testability — 결정적 재현 | NFR-025 | 테스트 인프라, 외부 의존성 모킹 전략 |

### 6.3.2 [H, M] — 표준 기법 적용 영역 (High value, Established)

품질 속성 가치는 높지만 산업 표준 기법으로 달성 가능한 영역. 표준 적용 + 검증 게이트가 핵심.

- NFR-002 첫 피드백 1초 / NFR-007 CPU 점유율 / NFR-010 네트워크 재개
- NFR-012 샌드박스 / NFR-013 자격증명 / NFR-015 외부 인증 / NFR-016 프로필 격리
- NFR-017 PII 보존 / NFR-018 PII 마스킹 / NFR-022 WCAG 2.2 AA

### 6.3.3 [M, *] — 표준 운영 영역

표준 도구·관행으로 달성하며, 본 장 우선순위는 낮지만 12장(로드맵)·운영 단계에서 챙겨야 할 항목.

- NFR-003 진행 표시 / NFR-004 Skill 라우팅 / NFR-011 라인업 동등성 / NFR-019 데이터 주체 권리
- NFR-023 인터페이스 호환성 / NFR-024 모듈 결합도 / NFR-026 Mock / NFR-027 트레이스 / NFR-028 HITL 재개 / NFR-029 저장소

---

## 6.4 카운트 요약

| 우선순위 | leaf 수 | 비중 |
|---------|--------|------|
| [H, H] | 11 | 37% |
| [H, M] | 10 | 33% |
| [M, *] | 9 | 30% |
| **총** | **30** | 100% |

[H, H] leaf가 전체의 1/3을 차지하며, 이는 본 시스템이 **AI Agent 특유의 신생 영역 + TV의 리소스·UX 제약 + 라인업·규제 환경**의 교집합에서 동작하는 데 따른 결과다. 이 11개가 7~9장 핵심 결정의 입력이 된다.

---

## 6.5 후속 작업

본 절은 향후 작성될 항목의 목차다.

- **6.7 QA 시나리오 카탈로그 (예정)** — [H, H] 11개 leaf를 SEI 6-part 형식으로 풀어 쓰고, ATAM 워크숍 입력으로 사용.
- **6.8 QA 간 트레이드오프 (예정)** — 충돌 지점 식별. 예:
  - Performance (Quick Action 1초) ↔ Security (모든 입력 검증)
  - Functional Quality (태스크 성공률) ↔ Privacy (PII 마스킹으로 인한 컨텍스트 손실)
  - Performance (메모리 예산) ↔ Modifiability (모듈화 오버헤드)
  - Accessibility (비자동화 동등 경로) ↔ Performance (단일 자동화 경로 집중 최적화)
- **6.9 Open Questions (예정)** — 본 장 내부 미결 사항. 5장 Open Questions 7개와 별개로 관리.

---

## 6.6 참고 자료 (References)

1. Bass, Clements, Kazman. *Software Architecture in Practice* (4th ed.) — Quality Attribute Scenarios, Utility Tree.
2. SEI / Kazman et al. *ATAM: Method for Architecture Evaluation* (CMU/SEI-2000-TR-004).
3. arc42 Template v8 — §10 "Quality Requirements".
4. ISO/IEC 25010:2011 — Systems and software Quality Requirements and Evaluation (SQuaRE) — Product Quality Model.
