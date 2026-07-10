# 5. 요구사항 (Requirements)

> Tizen TV AI Web Agent (AI Browser) 설계 문서

---

## 5.1 개요 (Overview)

본 장은 시스템이 **무엇을 충족해야 하는가**를 정의한다. 정의의 단위는 두 가지다.

1. **기능 요구사항 (Functional Requirements, FR)** — 시스템이 *해야 하는 일*. 입력·트리거·외부 이벤트에 대한 시스템의 관찰 가능한 행동을 기술한다.
2. **비기능 요구사항 (Non-Functional Requirements, NFR)** — 시스템이 *얼마나·언제까지·어떤 조건에서* 그 일을 해야 하는가. 성능·신뢰성·보안·접근성·유지보수성 등 품질 속성에 대한 측정 가능한 시나리오로 기술한다.

본 5장은 FR과 NFR을 별도 디렉터리·별도 문서로 분리해 작성한다. 동일 항목에 대한 FR과 NFR은 서로 양방향으로 참조하며, 추적성 표(`01-functional/99-traceability.md`)에서 통합된다.

### 5.1.1 본 사이클의 작업 범위

본 작업 사이클에서는 **기능 요구사항(FR)** 만 작성한다. NFR은 후속 사이클에서 동일 디렉터리 구조(`02-non-functional/`)로 추가한다. FR 영역 작성도 한 번에 9개를 모두 작성하지 않고, **VUI(Voice User Interface) 영역만 우선 작성**하여 작성 규약·카드 스키마의 적합성을 검증한 뒤 다른 영역으로 확장한다.

```
docs/05-requirements/
├── 00-overview.md                  # 본 문서 (5장 전체 개요)
├── 01-functional/
│   ├── 00-overview.md              # FR 작성 규약 (영역·EARS·ID·스키마)
│   ├── 01-vui.md                   # VUI 영역 FR  ← 본 사이클 작성
│   ├── 02-agt.md                   # (후속) Agent Core
│   ├── 03-brw.md                   # (후속) Browser Control
│   ├── 04-cnt.md                   # (후속) Content Pipeline
│   ├── 05-mdl.md                   # (후속) Model Abstraction
│   ├── 06-wfl.md                   # (후속) Workflow
│   ├── 07-hil.md                   # (후속) Human-in-the-Loop
│   ├── 08-int.md                   # (후속) Host Integration
│   ├── 09-ext.md                   # (후속) External Services
│   └── 99-traceability.md          # (후속) FR↔시나리오↔Stakeholder↔View 추적표
└── 02-non-functional/              # (후속 사이클)
```

---

## 5.2 작성 원칙 (Authoring Principles)

본 장 전체에 적용되는 원칙은 다음과 같다. FR/NFR 모두에 공통으로 적용된다.

### 5.2.1 What만 기술, How는 분리

요구사항은 *무엇을 충족해야 하는가*만 기술한다. *어떻게* 풀 것인가에 해당하는 알고리즘·자료구조·모듈 분할·캐시·우회 경로 등은 **8장(Solution Strategy) 이후**에서 다루며, 본 장에는 포함하지 않는다. 단, 외부와의 인터페이스 계약(예: 호스트 렌더러로의 상태 신호 발행)은 행동 스펙의 일부로서 FR에 포함될 수 있다.

### 5.2.2 추적성 (Traceability)

모든 요구사항은 다음 두 가지 출처 중 적어도 하나로부터 도출되어야 하며, 카드의 `Source` 필드에 명시한다.

1. **타겟 사용 시나리오** (2.2의 시나리오 1~5)
2. **Stakeholder Concern** (3.2~3.4)

추적은 양방향으로 이루어진다. 각 영역 파일 말미에 영역 단위 추적표를 둔다. 9개 영역 작성이 모두 끝난 시점에 `99-traceability.md`에서 통합 추적표를 생성한다.

### 5.2.3 검증 가능성 (Verifiability)

각 요구사항은 **재현 가능한 검증 절차**를 가질 수 있어야 한다. FR은 `Acceptance Criteria` 항목으로, NFR은 *Quality Attribute Scenario*의 `Response Measure`로 기술한다. "사용자가 만족해야 한다"와 같이 측정 불가능한 표현은 사용하지 않는다.

### 5.2.4 우선순위 (Priority)

본 장은 MoSCoW 표기를 사용한다.

| 표기 | 의미 |
|------|------|
| **Must** | 본 시스템의 최초 정식 출시 시점에 반드시 충족되어야 함 |
| **Should** | 출시 시점에 충족되는 것이 강하게 권장되나, 부재 시 대체·완화 수단이 존재 |
| **Could** | 일정·자원이 허용될 때 포함을 고려 |
| **Won't (this release)** | 본 릴리스에서는 명시적으로 제외, 차기 이후 재검토 |

우선순위는 본 5장 단계에서는 *제안값*이며, 12번 활동(Roadmap·라인업별 탑재 범위)에서 라인업·지역별로 재조정될 수 있다.

### 5.2.5 출처·근거의 인라인 표기

본 5장 본문에서 인용하는 외부 자료, 표준, 통계 수치는 1·2장과 동일한 각주 형식(`[^N]`)으로 인라인 표기한다. Stakeholder Concern·시나리오를 참조할 때는 절 번호(예: 3.2.4, 2.2 시나리오 4)로 표기한다. 동일 수치를 다른 절에서 재인용하지 않는다.

### 5.2.6 변경 관리

요구사항은 *Status* 필드로 상태를 관리한다. 단순 수정·문구 정정은 인플레이스 편집하되, **Approved 이후의 의미 변경**은 신규 ID 부여 또는 ADR 기록을 동반한다. ID 폐기 시 번호는 재사용하지 않는다(영구 결번).

---

## 5.3 다음 절차

본 5장의 다음 단계는 두 가지 흐름으로 진행된다.

1. **본 사이클**: `01-functional/00-overview.md`에서 FR 작성 규약(영역·EARS·ID·스키마)을 확정한 뒤, `01-functional/01-vui.md`에서 VUI 영역 FR을 카드 형태로 기술한다.
2. **후속 사이클**: VUI 영역 검토 완료 후 다른 8개 FR 영역으로 확장하고, NFR(`02-non-functional/`)을 동일 구조로 작성한다.

---

## 5.4 참고 자료 (References)

[^R1]: Mavin, A., Wilkinson, P., Harwood, A., Novak, M. (2009). *Easy Approach to Requirements Syntax (EARS)*. — 본 장의 FR 표현 패턴의 원전.
[^R2]: ISO/IEC/IEEE 29148:2018. *Systems and software engineering — Life cycle processes — Requirements engineering*. — 요구사항 명세의 일반 원칙.
[^R3]: Bass, Clements, Kazman (2021). *Software Architecture in Practice* (4th ed.) — Quality Attribute Scenario(NFR 표현)의 원전.
[^R4]: arc42 Template v8 — §10 Quality Requirements. — 본 장과 8장 이후 View 간 매핑의 근거.
[^R5]: Clegg, D. & Barker, R. (1994). *Case Method Fast-Track: A RAD Approach* — MoSCoW 우선순위 표기의 원전.
