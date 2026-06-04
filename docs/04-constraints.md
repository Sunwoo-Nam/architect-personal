# 4. 제약 사항 (Architecture Constraints)

> Tizen TV AI Web Agent (AI Browser) 설계 문서
> 본 장은 본 시스템의 설계가 만들어지기 **이전에 이미 정해진 외부 조건**을 정리한다.
> arc42 §2 "Architecture Constraints"의 양식을 따른다.

---

## 4.1 개요 (Overview)

제약(Constraint)은 본 시스템의 설계 공간을 사전에 한정하는 외부 조건이다. 5장의 요구사항(FR/NFR)이 "시스템이 무엇을 해야 하는가"를 정의한다면, 본 장의 제약은 "그것을 어떤 환경·규칙 안에서 해야 하는가"를 정의한다. 제약은 일반적으로 협상·변경이 어려우며, 어겼을 때의 결과는 단순한 품질 저하가 아니라 **출시 불가·플랫폼 정책 위반**으로 직결된다.

본 장의 제약은 세 군으로 분류한다.

| 군 | 코드 | 절 | 내용 |
|---|------|----|------|
| Technical | TC | 4.2 | 본 시스템이 동작해야 하는 기술 환경·플랫폼·외부 의존 |
| Organizational | OC | 4.3 | Samsung 내부 조직 정책·프로세스·예산 |
| Conventions & Standards | CN | 4.4 | 사실상 표준·내부 규약·디자인 시스템 |

> **규제·법률 요건**(GDPR·PIPA·WCAG·EAA·전자상거래법 등)은 본 장에서 별도 군으로 두지 않는다. 본 시스템이 따라야 할 정량 요건으로서 **5장 NFR**(PRV·ACC 카테고리)과 **5장 FR**(동의 수집·접근성·결제 흐름)에서 직접 표현되며, 각 의무의 법령·구속력·실현 위치 매핑은 [5장 컴플라이언스 매트릭스](05-requirements/04-compliance-matrix.md)가 단일 진실원으로 관리한다.

각 제약 카드는 다음 필드를 따른다.

```markdown
### {ID} {제목}
- **Description**: 제약의 내용과 발생 이유
- **Source**: 제약의 원천 (조직·표준·이전 의사결정)
- **Impact**: 영향받는 설계 영역 (FR / NFR / View)
- **Status**: 확정 (Fixed) | 협상 가능 (Negotiable) | 재검토 예정 (Under Review)
```

---

## 4.2 Technical Constraints (기술 제약)

### TC-001 Chromium 기반 Web Engine 및 CDP 의존

- **Description**: 본 시스템은 Tizen의 Chromium/Blink 기반 Web Engine과 그것이 노출하는 **CDP(Chrome DevTools Protocol)** 및 Browser Session 인터페이스를 통해 페이지를 관찰·조작한다. CDP가 제공하지 않는 동작은 본 시스템에서 구현할 수 없으며, Web Engine 업그레이드 시 CDP 호환성이 사전에 검증되어야 한다.
- **Source**: Web Engine Team (3.2.4)
- **Impact**: 페이지 인식·요소 시맨틱 해석 메커니즘 (7장 Solution Strategy), Component View, Integration / Extension View, Deployment View
- **Status**: Fixed

---

### TC-002 On-device 에이전트 하네스 + Cloud LLM

- **Description**: 에이전트 루프(도구 선택·실행 흐름·HITL 흐름) 자체는 TV 디바이스 안에서 실행되며, 추론(reasoning)은 외부 클라우드 LLM API 호출로 수행한다. 본 시스템은 on-device 추론을 1차 목표로 하지 않으며, 모델 추상화 계층은 클라우드 LLM 호출을 1순위 가정으로 설계한다. (특정 보조 작업에서 on-device SLM 가용성은 별도 결정 사안.)
- **Source**: 1장 시스템 특징, Tizen Platform Team (3.2.3)
- **Impact**: NFR-008 (토큰 비용), NFR-009 (오류 격리), NFR-018 (클라우드 전송 최소화), Component View, Deployment View
- **Status**: Fixed

---

### TC-003 외부 ASR / 화자 식별 시스템 의존

- **Description**: 음성 인식(Speech-to-Text)은 본 시스템 외부의 Tizen VUI 프레임워크가 담당하며, 본 시스템은 ASR 결과(전사된 텍스트)를 입력으로 받는다. ASR 정확도·지연·웨이크워드 처리·다국어 지원 등은 외부 시스템의 책임이며 본 시스템은 그 결과에 의존한다. (화자 식별(Speaker Identification) 결과를 활용하는 멀티프로필 처리는 현재 본 시스템 범위에서 제외된다 — 5장 범위 조정 참조.)
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: FR-001 (의도 해석), Context View, External Interface View
- **Status**: Fixed

---

### TC-004 라인업별 HW 스펙 차이 (메모리·CPU·NPU)

- **Description**: Samsung TV는 동일 시점에 여러 라인업(Premium / Mid / Entry)이 동시에 출시되며, 라인업별 메모리·CPU·NPU 스펙이 서로 다르다. 본 시스템은 단일 빌드가 모든 라인업에서 동작 가능함을 1차 목표로 하며, 단일 절대 한도가 아니라 **라인업별 예산 프로파일**(메모리·CPU·저장소)을 따르는 방식이 강제된다.
- **Source**: VD 상품화 담당자 (3.2.2), Tizen Platform Team (3.2.3)
- **Impact**: NFR-006 (메모리), NFR-007 (CPU), NFR-011 (라인업 동등성), NFR-029 (저장소), Deployment View, Resource Budget View
- **Status**: Fixed

---

### TC-005 Platform Main Agent의 Sub-Agent로 동작 가능

- **Description**: 본 시스템은 독립 실행뿐 아니라, Tizen Platform Team이 소유한 Claw 계열 Platform Main Agent의 **Sub-Agent / Skill**로도 호출 가능해야 한다. 즉 본 시스템의 공개 인터페이스는 Main Agent의 호출 계약과 호환되어야 하며, Main Agent의 컨텍스트·세션 모델을 침해하지 않아야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: FR-025 (Skill 인터페이스), FR-026 (외부 인증), FR-027 (실행 결과 스키마), Logical View, Integration / Extension View
- **Status**: Fixed

---

### TC-006 외부 Agent 플랫폼 호출 표준 인터페이스

- **Description**: 본 시스템은 Claw 계열 내부 플랫폼뿐 아니라, **외부 에이전트 오케스트레이션 플랫폼**(MCP 클라이언트, Agent Protocol 등)에서도 Sub-Agent / Skill로 호출 가능하도록 표준화된 공개 인터페이스를 제공해야 한다. 인터페이스 표준은 MCP tool spec 또는 OpenAPI schema 중 하나 이상을 지원한다.
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.3)
- **Impact**: FR-025, FR-026, FR-027, NFR-023 (하위 호환성), External Interface View
- **Status**: Fixed

---

## 4.3 Organizational Constraints (조직 제약)

### OC-001 VD 사업부 상품화 게이트

- **Description**: 본 시스템은 VD 사업부의 상품화 프로세스를 통과해야 라인업에 탑재 가능하다. 게이트는 품질·성능·법무·접근성·보안·프라이버시 등 다영역에 걸치며, 각 게이트의 통과 기준은 별도 문서(상품화 가이드)에 정의되어 있다. 단일 게이트 미통과 시 해당 라인업 탑재가 보류된다.
- **Source**: VD 상품화 담당자 (3.2.2)
- **Impact**: 본 문서 전반, Roadmap & Capability View
- **Status**: Fixed

---

### OC-002 LLM API 호출 비용 예산

- **Description**: 본 시스템의 운영 비용 중 가장 큰 비중은 Cloud LLM API 호출 비용이다. VD 상품화 담당자는 사용량 규모 대비 비용 예측 가능성을 출시 승인 전제 조건으로 요구한다. 세션당 평균 토큰 사용량 상한, 비용 모니터링·알림 체계, 비용 폭주 시 강제 차단 정책이 사전에 정의되어야 한다.
- **Source**: VD 상품화 담당자 (3.2.2)
- **Impact**: NFR-008 (세션당 토큰 예산), FR-031 (토큰·비용 계측·노출), Roadmap & Capability View
- **Status**: Fixed

---

### OC-003 Tizen Platform Team의 인터페이스·격리 정책

- **Description**: Tizen Platform Team은 본 시스템의 호스트이자 가장 가까운 내부 소비자다. 본 시스템이 노출하는 공개 인터페이스의 안정성·버저닝, 그리고 본 시스템의 실패가 Main Agent / GenUI Renderer로 전파되지 않는 격리 전략은 Platform Team이 정의한 정책을 따라야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: NFR-009 (오류 격리), NFR-023 (인터페이스 호환성), FR-024 (오류 격리), Logical View, Integration / Extension View
- **Status**: Fixed

---

### OC-004 Security & Privacy Team의 보안 게이트

- **Description**: 자격증명 보관·사용, PII 데이터 흐름, 권한 모델, 위협 모델은 Security & Privacy Team의 사전 심사·승인을 받아야 한다. 본 시스템의 설계는 해당 팀이 정의한 권한 도메인·트러스트 경계·암호화 표준 안에서 이루어져야 한다.
- **Source**: Security & Privacy Team (3.2.5)
- **Impact**: NFR-012~NFR-015, NFR-017, NFR-018 (보안·프라이버시 전반), FR-018·FR-019·FR-020·FR-021·FR-022 (보안·프라이버시 전반), Security View, Threat Model View, Privacy & Retention View
- **Status**: Fixed

---

### OC-005 LLM 토큰 사용 비용 사용자 미부과

- **Description**: Cloud LLM API 호출에 따른 토큰 사용 비용은 Samsung이 부담하며, 최종 사용자에게 별도 과금하지 않는다. 본 시스템 사용에 사용량 기반 결제 모델은 존재하지 않으며, 사용 빈도가 사용자 부담으로 직결되지 않는다. 이는 본 시스템의 운영 비용 관리 부담이 전적으로 Samsung 내부 예산으로 흡수됨을 의미하며, 비용 최적화(컨텍스트 압축·모델 라우팅·캐싱 등)는 사용자 가치 보호가 아닌 **사업 지속가능성 차원의 요구**가 된다.
- **Source**: VD 사업부 경영진 (3.2.1), VD 상품화 담당자 (3.2.2)
- **Impact**: NFR-008 (세션당 토큰 예산), FR-031 (토큰·비용 계측·노출), OC-002 (비용 예산), Roadmap & Capability View
- **Status**: Fixed

---

## 4.4 Conventions & Standards (규약·표준)

### CN-001 Tizen TV 디자인 시스템 (10-feet UI)

- **Description**: 본 시스템이 출력하는 모든 UI(Generative UI 포함)는 Tizen TV 디자인 시스템의 토큰(타이포그래피·색상·포커스·모션·간격)과 10-feet UI(원거리 시청) 가이드라인을 따라야 한다. UI 라이브러리·디자인 토큰은 UX·Design Team이 소유·제공한다.
- **Source**: UX · Design Team (3.2.6)
- **Impact**: Generative UI 렌더링 전반(디자인 토큰 준수 — 본 제약이 직접 규정), Generative UI View, Interaction View
- **Status**: Fixed

---

### CN-002 ARIA / 시맨틱 마크업 표준

- **Description**: Generative UI 컴포넌트는 W3C ARIA 표준에 따라 role / label / description / state 등 접근성 속성을 완전히 포함해야 한다. 이는 보조 기술(AT, OS 레이어에서 처리)이 UI를 정확히 해석할 수 있도록 하는 기반 표준이다.
- **Source**: 접근성 요구 사용자 (3.4.3), W3C ARIA
- **Impact**: NFR-022 (WCAG — 접근성 속성 준수 측정), Generative UI View, Accessibility View
- **Status**: Fixed

---

## 4.5 요약 표

| ID | 제목 | Source | 주요 Impact |
|----|------|--------|-------------|
| TC-001 | Chromium 기반 Web Engine 및 CDP 의존 | 3.2.4 | 페이지 인식 메커니즘(7장), Component View |
| TC-002 | On-device 에이전트 하네스 + Cloud LLM | 1장, 3.2.3 | NFR-008, NFR-018 |
| TC-003 | 외부 ASR / 화자 식별 시스템 의존 | 3.2.3 | FR-001 |
| TC-004 | 라인업별 HW 스펙 차이 | 3.2.2, 3.2.3 | NFR-006, NFR-007, NFR-011 |
| TC-005 | Platform Main Agent의 Sub-Agent 동작 | 3.2.3 | FR-025, FR-026, FR-027 |
| TC-006 | 외부 Agent 플랫폼 호출 표준 인터페이스 | 3.3.3 | FR-025, FR-026, FR-027, NFR-023 |
| OC-001 | VD 사업부 상품화 게이트 | 3.2.2 | 전반 |
| OC-002 | LLM API 호출 비용 예산 | 3.2.2 | NFR-008, FR-031 |
| OC-003 | Tizen Platform Team 인터페이스·격리 정책 | 3.2.3 | NFR-009, NFR-023 |
| OC-004 | Security & Privacy Team 보안 게이트 | 3.2.5 | NFR-012~NFR-015, NFR-017, NFR-018 |
| OC-005 | LLM 토큰 사용 비용 사용자 미부과 | 3.2.1, 3.2.2 | NFR-008, FR-031 |
| CN-001 | Tizen TV 디자인 시스템 (10-feet UI) | 3.2.6 | Generative UI View |
| CN-002 | ARIA / 시맨틱 마크업 표준 | 3.4.3, W3C | NFR-022 |

총 13건.

---

## 4.6 Open Questions

본 장의 제약 식별에서 추후 확인·재검토가 필요한 항목.

1. **TC-002 — On-device SLM 보조 사용 범위** — 일부 보조 작업(예: 의도 분류, 캐싱 응답)에서 on-device SLM 활용 가능성. 라인업별 NPU 가용성과 함께 별도 ADR로 결정.

---

## 4.7 참고 자료 (References)

1. arc42 Template v8 — §2 "Architecture Constraints".
2. ISO/IEC/IEEE 42010:2022 — "Systems and software engineering — Architecture description".
3. W3C — ARIA in HTML.
