# 4. 제약 사항 (Architecture Constraints)

> Tizen TV AI Web Agent (AI Browser) 설계 문서
> 본 장은 본 시스템의 설계가 만들어지기 **이전에 이미 정해진 외부 조건**을 정리한다.
> arc42 §2 "Architecture Constraints"의 양식을 따른다.

---

## 4.1 개요 (Overview)

제약(Constraint)은 본 시스템의 설계 공간을 사전에 한정하는 외부 조건이다. 5장의 요구사항(FR/NFR)이 "시스템이 무엇을 해야 하는가"를 정의한다면, 본 장의 제약은 "그것을 어떤 환경·규칙 안에서 해야 하는가"를 정의한다. 제약은 일반적으로 협상·변경이 어려우며, 어겼을 때의 결과는 단순한 품질 저하가 아니라 **출시 불가·법적 책임·플랫폼 거버넌스 위반**으로 직결된다.

본 장의 제약은 네 군으로 분류한다.

| 군 | 코드 | 절 | 내용 |
|---|------|----|------|
| Technical | TC | 4.2 | 본 시스템이 동작해야 하는 기술 환경·플랫폼·외부 의존 |
| Organizational | OC | 4.3 | Samsung 내부 조직 정책·프로세스·예산 |
| Conventions & Standards | CN | 4.4 | 사실상 표준·내부 규약·디자인 시스템 |
| Regulatory & Legal | RC | 4.5 | 법·규제·강제력을 가진 외부 규범 |

각 제약 카드는 다음 필드를 따른다.

```markdown
### {ID} {제목}
- **Description**: 제약의 내용과 발생 이유
- **Source**: 제약의 원천 (조직·표준·법규·이전 의사결정)
- **Impact**: 영향받는 설계 영역 (FR / NFR / View)
- **Status**: 확정 (Fixed) | 협상 가능 (Negotiable) | 재검토 예정 (Under Review)
```

---

## 4.2 Technical Constraints (기술 제약)

### TC-001 Tizen Web Runtime 기반 동작

- **Description**: 본 시스템은 Tizen TV의 Web Runtime 위에서 웹 애플리케이션 형태로 동작한다. Native 앱·별도 OS 프로세스 등은 선택지가 아니다. 이는 Tizen 플랫폼 표준 배포·생명주기 관리 모델을 따른다는 의미이며, 동시에 Web Runtime이 노출하는 API와 샌드박스 경계 안에서만 시스템이 동작 가능함을 의미한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: NFR-012 (샌드박스 경계), NFR-009 (오류 격리), Deployment View, Component View
- **Status**: Fixed

---

### TC-002 Chromium 기반 Web Engine 및 CDP 의존

- **Description**: 본 시스템은 Tizen의 Chromium/Blink 기반 Web Engine과 그것이 노출하는 **CDP(Chrome DevTools Protocol)** 및 Browser Session 인터페이스를 통해 페이지를 관찰·조작한다. CDP가 제공하지 않는 동작은 본 시스템에서 구현할 수 없으며, Web Engine 업그레이드 시 CDP 호환성이 사전에 검증되어야 한다.
- **Source**: Web Engine Team (3.2.4)
- **Impact**: FR-015 (페이지 상태 인식), FR-016 (요소 시맨틱 인식), Component View, Integration / Extension View, Deployment View
- **Status**: Fixed

---

### TC-003 On-device 에이전트 하네스 + Cloud LLM

- **Description**: 에이전트 루프(도구 선택·실행 흐름·HITL 흐름) 자체는 TV 디바이스 안에서 실행되며, 추론(reasoning)은 외부 클라우드 LLM API 호출로 수행한다. 본 시스템은 on-device 추론을 1차 목표로 하지 않으며, 모델 추상화 계층은 클라우드 LLM 호출을 1순위 가정으로 설계한다. (특정 보조 작업에서 on-device SLM 가용성은 별도 결정 사안.)
- **Source**: 1장 시스템 특징, Tizen Platform Team (3.2.3)
- **Impact**: NFR-008 (토큰 비용), NFR-009 (오류 격리), NFR-018 (클라우드 전송 최소화), Component View, Deployment View
- **Status**: Fixed

---

### TC-004 외부 ASR / 화자 식별 시스템 의존

- **Description**: 음성 인식(Speech-to-Text)·화자 식별(Speaker Identification)은 본 시스템 외부의 Tizen VUI 프레임워크가 담당한다. 본 시스템은 ASR 결과(전사된 텍스트)와 화자 식별 결과(프로필 식별자)를 입력으로 받는다. ASR 정확도·지연·웨이크워드 처리·다국어 지원 등은 외부 시스템의 책임이며 본 시스템은 그 결과에 의존한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: FR-001 (의도 해석), FR-026 (활성 프로필 컨텍스트 전환), Context View, External Interface View
- **Status**: Fixed

---

### TC-005 라인업별 HW 스펙 차이 (메모리·CPU·NPU)

- **Description**: Samsung TV는 동일 시점에 여러 라인업(Premium / Mid / Entry)이 동시에 출시되며, 라인업별 메모리·CPU·NPU 스펙이 서로 다르다. 본 시스템은 단일 빌드가 모든 라인업에서 동작 가능해야 하거나, 라인업별 기능 차등 활성화 정책을 따라야 한다. 단일 절대 한도가 아니라 **라인업별 예산 프로파일**을 따르는 방식이 강제된다.
- **Source**: VD 상품화 담당자 (3.2.2), Tizen Platform Team (3.2.3)
- **Impact**: FR-024 (기능 조건부 활성화), NFR-006 (메모리), NFR-007 (CPU), NFR-011 (라인업 동등성), NFR-029 (저장소), Deployment View, Resource Budget View
- **Status**: Fixed

---

### TC-006 Platform Main Agent의 Sub-Agent로 동작 가능

- **Description**: 본 시스템은 독립 실행뿐 아니라, Tizen Platform Team이 소유한 Claw 계열 Platform Main Agent의 **Sub-Agent / Skill**로도 호출 가능해야 한다. 즉 본 시스템의 공개 인터페이스는 Main Agent의 호출 계약과 호환되어야 하며, Main Agent의 컨텍스트·세션 모델을 침해하지 않아야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: FR-038 (Skill 인터페이스), FR-040 (외부 인증), FR-041 (실행 결과 스키마), Logical View, Integration / Extension View
- **Status**: Fixed

---

### TC-007 외부 Agent 플랫폼 호출 표준 인터페이스

- **Description**: 본 시스템은 Claw 계열 내부 플랫폼뿐 아니라, **외부 에이전트 오케스트레이션 플랫폼**(MCP 클라이언트, Agent Protocol 등)에서도 Sub-Agent / Skill로 호출 가능하도록 표준화된 공개 인터페이스를 제공해야 한다. 인터페이스 표준은 MCP tool spec 또는 OpenAPI schema 중 하나 이상을 지원한다.
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.3)
- **Impact**: FR-038, FR-039, FR-040, FR-041, External Interface View
- **Status**: Fixed

---

## 4.3 Organizational Constraints (조직 제약)

### OC-001 VD 사업부 상품화 게이트

- **Description**: 본 시스템은 VD 사업부의 상품화 프로세스를 통과해야 라인업에 탑재 가능하다. 게이트는 품질·성능·법무·접근성·보안·프라이버시 등 다영역에 걸치며, 각 게이트의 통과 기준은 별도 문서(상품화 가이드)에 정의되어 있다. 단일 게이트 미통과 시 해당 라인업 탑재가 보류된다.
- **Source**: VD 상품화 담당자 (3.2.2)
- **Impact**: 본 문서 전반, Roadmap & Capability View
- **Status**: Fixed

---

### OC-002 라인업·지역별 단계적 출시

- **Description**: 본 시스템은 전 라인업·전 지역에 동시 출시되지 않는다. 베타 → 상위 라인업 우선 → 점진 확대 모델을 따르며, 지역별로 규제 환경·LLM API 가용성·언어 지원 수준 차이에 따라 출시 시점이 다르다. 본 시스템의 기능 활성화는 라인업·지역 구성에 의해 조건부로 결정된다.
- **Source**: VD 사업부 경영진 (3.2.1), VD 상품화 담당자 (3.2.2)
- **Impact**: FR-024 (기능 조건부 활성화), Roadmap & Capability View, Deployment View
- **Status**: Fixed

---

### OC-003 LLM API 호출 비용 예산

- **Description**: 본 시스템의 운영 비용 중 가장 큰 비중은 Cloud LLM API 호출 비용이다. VD 상품화 담당자는 사용량 규모 대비 비용 예측 가능성을 출시 승인 전제 조건으로 요구한다. 세션당 평균 토큰 사용량 상한, 비용 모니터링·알림 체계, 비용 폭주 시 강제 차단 정책이 사전에 정의되어야 한다.
- **Source**: VD 상품화 담당자 (3.2.2)
- **Impact**: NFR-008 (세션당 토큰 예산), FR-045 (토큰·비용 계측·노출), Roadmap & Capability View
- **Status**: Fixed

---

### OC-004 Tizen Platform Team의 인터페이스·격리 거버넌스

- **Description**: Tizen Platform Team은 본 시스템의 호스트이자 가장 가까운 내부 소비자다. 본 시스템이 노출하는 공개 인터페이스의 안정성·버저닝, 그리고 본 시스템의 실패가 Main Agent / GenUI Renderer로 전파되지 않는 격리 전략은 Platform Team의 거버넌스를 통과해야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Impact**: NFR-009 (오류 격리), NFR-023 (인터페이스 호환성), FR-037 (오류 격리), Logical View, Integration / Extension View
- **Status**: Fixed

---

### OC-005 Security & Privacy Team의 보안 게이트

- **Description**: 자격증명 보관·사용, PII 데이터 흐름, 권한 모델, 위협 모델은 Security & Privacy Team의 사전 심사·승인을 받아야 한다. 본 시스템의 설계는 해당 팀이 정의한 권한 도메인·트러스트 경계·암호화 표준 안에서 이루어져야 한다.
- **Source**: Security & Privacy Team (3.2.5)
- **Impact**: NFR-012~NFR-018 (보안·프라이버시 전반), FR-030~FR-035 (보안·프라이버시 전반), Security View, Threat Model View, Privacy & Retention View
- **Status**: Fixed

---

### OC-006 글로벌 출시 전제 — 가장 보수적 규제 기준선

- **Description**: 본 시스템은 글로벌 출시를 전제로 설계되며, 지역별 분기 설계 대신 **가장 보수적인 규제를 기준선으로 통일된 설계**를 따른다. 예: GDPR(EU), EAA(EU), PIPA(한국)가 ADA(미국)·CCPA(미국)보다 엄격할 경우 EU·한국 기준이 글로벌 기준이 된다.
- **Source**: 3.7 가정과 제약, 규제 · 법률 기관 (3.5)
- **Impact**: 4.5 (규제 제약 전반), NFR-017~NFR-022, FR-021, FR-034, FR-048~FR-049, Privacy & Retention View, Consent View
- **Status**: Fixed

---

## 4.4 Conventions & Standards (규약·표준)

### CN-001 Tizen TV 디자인 시스템 (10-feet UI)

- **Description**: 본 시스템이 출력하는 모든 UI(Generative UI 포함)는 Tizen TV 디자인 시스템의 토큰(타이포그래피·색상·포커스·모션·간격)과 10-feet UI(원거리 시청) 가이드라인을 따라야 한다. UI 라이브러리·디자인 토큰은 UX·Design Team이 소유·제공한다.
- **Source**: UX · Design Team (3.2.6)
- **Impact**: FR-018 (디자인 시스템 준수), FR-019 (접근성 메타데이터), FR-020 (원본 보존), Generative UI View, Interaction View
- **Status**: Fixed

---

### CN-002 ARIA / 시맨틱 마크업 표준

- **Description**: Generative UI 컴포넌트는 W3C ARIA 표준에 따라 role / label / description / state 등 접근성 속성을 완전히 포함해야 한다. 이는 보조 기술(AT, OS 레이어에서 처리)이 UI를 정확히 해석할 수 있도록 하는 기반 표준이다.
- **Source**: 접근성 요구 사용자 (3.4.3), W3C ARIA
- **Impact**: FR-019, NFR-022 (WCAG), Generative UI View, Accessibility View
- **Status**: Fixed

---

### CN-003 Tizen Web Runtime CTS 통과

- **Description**: 본 시스템은 Tizen Web Runtime의 Compatibility Test Suite(CTS)를 통과해야 라인업 탑재가 승인된다. CTS는 샌드박스 경계 준수, 표준 API 사용, 성능·메모리 프로파일 등을 포함한다.
- **Source**: Tizen Platform Team (3.2.3), QA · Certification Team (3.2.7)
- **Impact**: NFR-012 (샌드박스), Test Strategy View
- **Status**: Fixed

---

### CN-004 ADR(Architectural Decision Record) 기록 의무

- **Description**: 본 시스템의 모든 아키텍처 결정은 ADR로 기록되어야 한다. 결정의 배경·고려된 옵션·선택 근거·결과·트레이드오프가 9장에 누적되며, 후속 결정은 기존 ADR을 참조한다. 결정이 변경될 경우 새 ADR로 기록(Superseded 표기)한다.
- **Source**: 본 프로젝트 CLAUDE.md, arc42 §9
- **Impact**: 9장 전체, 모든 설계 결정 흐름
- **Status**: Fixed

---

## 4.5 Regulatory & Legal Constraints (규제·법률 제약)

### RC-001 EU GDPR (General Data Protection Regulation)

- **Description**: EU 거주자의 개인정보 수집·처리·이전은 GDPR의 적법성·최소화·보존 제한·데이터 주체 권리 요건을 충족해야 한다. 위반 시 글로벌 매출 4% 또는 €20M(둘 중 큰 금액)의 과징금이 부과될 수 있다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-017 (PII 보존), NFR-018 (전송 최소화), NFR-019 (데이터 주체 권리), FR-034 (동의 수집), FR-035 (클라우드 고지), Privacy & Retention View, Consent View
- **Status**: Fixed

---

### RC-002 한국 PIPA (개인정보 보호법)

- **Description**: 한국 사용자에 대해서는 PIPA가 적용되며, 개인정보 수집 시 명시적 동의·목적 한정·파기 의무를 강제한다. 데이터 주체의 열람·삭제·정정 권리 행사를 시스템 차원에서 지원해야 한다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-017, NFR-019, FR-034, Privacy & Retention View
- **Status**: Fixed

---

### RC-003 미국 CCPA / CPRA

- **Description**: 캘리포니아 거주자에 대해서는 CCPA/CPRA가 적용되며, 데이터 판매·공유 거부 권리(Right to Opt-Out), 민감 정보 사용 제한, 데이터 주체 권리 등을 보장해야 한다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-019, FR-034, Privacy & Retention View
- **Status**: Fixed

---

### RC-004 EU 접근성법 (EAA, European Accessibility Act)

- **Description**: 2025년 6월 28일부터 EU에서 판매되는 디지털 제품·서비스(TV·SmartTV 포함)에 강제 적용된다. 본 시스템의 모든 UI·인터랙션은 WCAG 2.2 Level AA 또는 동등 수준의 접근성을 보장해야 하며, 미준수 시 EU 시장 출시가 불가하다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-022 (WCAG 준수), FR-019 (Generative UI 접근성), FR-048 (비자동화 동등 경로), Accessibility View, Generative UI View
- **Status**: Fixed

---

### RC-005 한국 장애인차별금지법 (장차법)

- **Description**: 한국에서 출시되는 정보통신 서비스는 장차법에 따라 장애인 접근성을 보장해야 한다. WCAG 등 표준이 사실상 기준선으로 적용되며, 미준수 시 차별 행위로 간주될 수 있다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-022, FR-019, FR-048, Accessibility View
- **Status**: Fixed

---

### RC-006 미국 ADA (Americans with Disabilities Act)

- **Description**: 미국에서 디지털 서비스의 접근성 의무에 대한 법적 근거. 판례·DOJ 가이드라인을 통해 WCAG 2.x 수준의 준수가 사실상 강제된다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: NFR-022, FR-019, FR-048, Accessibility View
- **Status**: Fixed

---

### RC-007 WCAG 2.2 (Web Content Accessibility Guidelines)

- **Description**: 본 시스템의 접근성 기준선. EAA·장차법·ADA가 사실상 본 표준의 준수를 요구한다. 본 시스템은 Level AA를 최소 기준으로 한다.
- **Source**: W3C, 규제 · 법률 기관 (3.5)
- **Impact**: NFR-022, FR-019, Generative UI View, Accessibility View
- **Status**: Fixed

---

### RC-008 전자상거래·소비자 보호 규제

- **Description**: 결제·구매·구독 흐름은 각 지역의 전자상거래·소비자 보호 규제를 따라야 한다. 자동 결제·자동 갱신 사전 고지, 청약 철회 권리, 광고성 표시 의무, AI 생성 콘텐츠 표시 의무가 포함된다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Impact**: FR-006 (비가역 동작 확인), FR-021 (AI 생성 콘텐츠 표시), FR-049 (반복 구매·구독 고지·철회), Sequence / Flow View, Consent View
- **Status**: Fixed

---

### RC-009 저작권법 · DMCA · 콘텐츠 이용 약관

- **Description**: 본 시스템의 웹 콘텐츠 요약·재가공은 각 콘텐츠 소유자의 저작권 및 플랫폼별 이용약관의 자동화 접근 허용 범위를 벗어나서는 안 된다. 원본 출처·저작자 표기, 공정이용 한계, 자동화 접근 정책(robots.txt 등) 준수가 요구된다.
- **Source**: 규제 · 법률 기관 (3.5), 제3자 웹 서비스 운영자 (3.3.2)
- **Impact**: FR-021 (AI 생성 콘텐츠 표시), FR-043 (접근 정책 준수), Rendering Policy View
- **Status**: Fixed

---

## 4.6 요약 표

| ID | 제목 | Source | 주요 Impact |
|----|------|--------|-------------|
| TC-001 | Tizen Web Runtime 기반 동작 | 3.2.3 | NFR-012, NFR-009 |
| TC-002 | Chromium 기반 Web Engine 및 CDP 의존 | 3.2.4 | FR-015, FR-016 |
| TC-003 | On-device 에이전트 하네스 + Cloud LLM | 1장, 3.2.3 | NFR-008, NFR-018 |
| TC-004 | 외부 ASR / 화자 식별 시스템 의존 | 3.2.3 | FR-001, FR-026 |
| TC-005 | 라인업별 HW 스펙 차이 | 3.2.2, 3.2.3 | NFR-006, NFR-007, NFR-011 |
| TC-006 | Platform Main Agent의 Sub-Agent 동작 | 3.2.3 | FR-038, FR-040, FR-041 |
| TC-007 | 외부 Agent 플랫폼 호출 표준 인터페이스 | 3.3.3 | FR-038~FR-041 |
| OC-001 | VD 사업부 상품화 게이트 | 3.2.2 | 전반 |
| OC-002 | 라인업·지역별 단계적 출시 | 3.2.1, 3.2.2 | FR-024 |
| OC-003 | LLM API 호출 비용 예산 | 3.2.2 | NFR-008, FR-045 |
| OC-004 | Tizen Platform Team 거버넌스 | 3.2.3 | NFR-009, NFR-023 |
| OC-005 | Security & Privacy Team 게이트 | 3.2.5 | NFR-012~NFR-018 |
| OC-006 | 글로벌 출시 — 가장 보수적 기준선 | 3.5 | 4.5 전반 |
| CN-001 | Tizen TV 디자인 시스템 (10-feet UI) | 3.2.6 | FR-018~FR-020 |
| CN-002 | ARIA / 시맨틱 마크업 표준 | 3.4.3, W3C | FR-019, NFR-022 |
| CN-003 | Tizen Web Runtime CTS 통과 | 3.2.3, 3.2.7 | NFR-012 |
| CN-004 | ADR 기록 의무 | CLAUDE.md, arc42 | 9장 전체 |
| RC-001 | EU GDPR | 3.5 | NFR-017~NFR-019, FR-034~FR-035 |
| RC-002 | 한국 PIPA | 3.5 | NFR-017, NFR-019, FR-034 |
| RC-003 | 미국 CCPA / CPRA | 3.5 | NFR-019, FR-034 |
| RC-004 | EU 접근성법 (EAA, 2025-06) | 3.5 | NFR-022, FR-019, FR-048 |
| RC-005 | 한국 장차법 | 3.5 | NFR-022, FR-019, FR-048 |
| RC-006 | 미국 ADA | 3.5 | NFR-022, FR-019, FR-048 |
| RC-007 | WCAG 2.2 (기준선) | W3C, 3.5 | NFR-022, FR-019 |
| RC-008 | 전자상거래·소비자 보호 규제 | 3.5 | FR-006, FR-021, FR-049 |
| RC-009 | 저작권법 · DMCA · 이용약관 | 3.5, 3.3.2 | FR-021, FR-043 |

---

## 4.7 Open Questions

본 장의 제약 식별에서 추후 확인·재검토가 필요한 항목.

1. **TC-003 — On-device SLM 보조 사용 범위** — 일부 보조 작업(예: 의도 분류, 캐싱 응답)에서 on-device SLM 활용 가능성. 라인업별 NPU 가용성과 함께 별도 ADR로 결정.
2. **CN-003 — Tizen Web Runtime CTS 항목 매핑** — 본 시스템 특유의 비결정성(AI)을 CTS가 어떻게 검증하는지 매핑 필요. QA · Certification Team과 합의.
3. **RC-008 — 지역별 결제·구독 규제 세부 매핑** — EU·한국·미국 외 추가 출시 지역에 따라 지역별 강제 요건 차이 정리 필요. OC-002 단계적 출시 계획과 연계.

---

## 4.8 참고 자료 (References)

1. arc42 Template v8 — §2 "Architecture Constraints".
2. ISO/IEC/IEEE 42010:2022 — "Systems and software engineering — Architecture description".
3. EU Regulation 2016/679 (GDPR).
4. EU Directive 2019/882 (European Accessibility Act, EAA).
5. 한국 개인정보 보호법(PIPA), 장애인차별금지법.
6. W3C — Web Content Accessibility Guidelines (WCAG) 2.2.
7. W3C — ARIA in HTML.
