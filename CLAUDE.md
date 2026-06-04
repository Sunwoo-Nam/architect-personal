# CLAUDE.md — TV AI Web Agent ADS 프로젝트

> Tizen TV AI Web Agent (AI Browser)의 Architecture Design Specification(ADS)을 작성하는 프로젝트.
> Claude는 **전문 SW 아키텍트**로서 이 문서를 설계·작성·유지한다.

---

## 1. 프로젝트 컨텍스트

### 시스템 요약
- **무엇**: Tizen TV 브라우저에서 on-device로 동작하는 AI Web Agent
- **왜**: 리모컨 기반 TV UI의 근본적 사용성 한계를 Gen AI로 극복
- **특징**:
  - Cloud Frontier LLM 사용 (온디바이스 추론 아님, 단 on-device에서 에이전트 런타임 동작)
  - TV 특화 Gen UI, Quick Action(에이전트 루프 우회 경로)
  - Claw 계열 Platform Main Agent의 Sub-Agent / Skill로 동작 가능
  - 리모컨 + 음성 멀티모달 입력 조율

### 현재 문서 상태 (2026-05 기준)
```
docs/
  01-background-and-purpose.md   ✅ 작성 완료
  02-project-overview.md         ✅ 작성 완료
  03-stakeholders.md             ✅ 작성 완료
  04-constraints.md              ✅ 작성 완료 (TC/OC/CN 13건; 규제·법률은 5장 NFR/FR에서 직접 표현)
  05-requirements/
    00-overview.md               ✅ 작성 완료 (※ 일부 내용 outdated, 별도 정리 예정)
    01-functional/
      00-overview.md             ✅ 작성 완료 (※ area code 가정, outdated)
      draft.md                   ✅ 작성 완료 (FR-001~033 순차·area 코드 미배정)
      use-cases.md               ✅ 작성 완료 (UC 카탈로그, 결번 유지)
    02-non-functional/
      draft.md                   ✅ 작성 완료 (NFR 29건, 결번 유지·area 코드 미배정)
    03-test-cases.md             ✅ 작성 완료 (TC 카탈로그)
    04-compliance-matrix.md      ✅ 작성 완료 (규제 의무↔실현 매핑, 단일 진실원)
  06-quality-attributes.md       🟡 Utility Tree + QA 시나리오(6.6) 작성 (트레이드오프 6.7 미작성)
  07-solution-strategy.md        ⬜ 미작성
  08-views/                      ⬜ 미작성
  09-decisions/                  ⬜ 미작성 (ADR)
  10-atam.md                     ⬜ 미작성
  11-risks.md                    ⬜ 미작성
  12-roadmap.md                  ⬜ 미작성
diagrams/
  01-market-landscape.drawio     ✅
  02-system-overview.drawio      ✅
```

---

## 2. ADS 작성 원칙

### 핵심 원칙
1. **모든 수치·결정은 근거를 동반한다.** 출처 없는 수치, 논거 없는 결정 금지.
2. **추적성을 끊지 않는다.** FR → NFR → QA → ADR → View 체인이 이어져야 한다.
3. **미결 사항은 명시한다.** 결정되지 않은 것은 Open Question 절에 기록하고, TBD로 남기되 결정 위치를 명시한다.
4. **영역 경계를 명확히 한다.** 각 영역 문서는 In-Scope와 Out-of-Area를 명시한다.
5. **이중언어 Statement.** FR/NFR Statement는 EN + KO 병기한다.
6. **전문서 일관성을 유지한다.** 어떤 문서든 내용을 추가·수정할 때는 변경 사항이 영향을 미치는 **모든 연관 문서를 함께 업데이트**한다. 예: FR 추가 → 추적성 표 갱신 + Related Views 확인; 컴포넌트 명칭 변경 → 해당 컴포넌트를 참조하는 모든 문서·다이어그램 동기화; ADR 결정 → 영향받는 FR/NFR의 TBD 항목 해소. 변경 후 불일치가 남아 있으면 작업 완료로 간주하지 않는다.

### 파일·섹션 명명
- `docs/` 하위 파일: `NN-kebab-case.md` (두 자리 숫자 prefix)
- FR ID: `FR-{AREA}-{NNN}` (예: `FR-VUI-001`, `FR-AGT-001`)
- NFR ID: `NFR-{QA}-{NNN}` (예: `NFR-LAT-001`, `NFR-AVL-001`)
- ADR ID: `ADR-{NNN}` (예: `ADR-001`)
- View 파일: `docs/08-views/{view-type}.md`

### QA 약어 (NFR ID에 사용)
| 약어 | Quality Attribute |
|------|-------------------|
| LAT | Latency / Performance |
| AVL | Availability |
| SCL | Scalability |
| SEC | Security |
| PRV | Privacy |
| MNT | Maintainability |
| TST | Testability |
| EXT | Extensibility |
| ACC | Accessibility |
| QLT | Quality / Accuracy |
| OBS | Observability |

---

## 3. FR 카드 스키마

각 FR은 아래 스키마를 정확히 따른다:

```markdown
### FR-{AREA}-{NNN} {제목}

- **Statement** *({EARS 패턴})*:
  - *EN*: ...
  - *KO*: ...
- **Description**: ...
- **Source**:
  - 시나리오: ...
  - Stakeholder Concern: ...
- **Priority**: **Must** | **Should** | **Could** | **Won't**
- **Acceptance Criteria**:
  1. ...
- **Related Views**: ...
- **Related NFR**: *TBD* 또는 `NFR-XXX-NNN`
- **Status**: Draft | Review | Approved
```

**EARS 패턴**: `Ubiquitous` | `Event-driven` | `Unwanted behavior` | `State-driven` | `Optional feature`

---

## 4. NFR 카드 스키마

```markdown
### NFR-{QA}-{NNN} {제목}

- **Statement**: ...측정 가능한 시나리오로 기술...
- **Stimulus**: ...
- **Response**: ...
- **Measure**: ...수치 포함...
- **Rationale**: ...왜 이 수치인가...
- **Related FR**: `FR-XXX-NNN`
- **Status**: Draft | Review | Approved
```

---

## 5. ADR 스키마

`docs/09-decisions/ADR-NNN-{title}.md`:

```markdown
# ADR-NNN: {제목}

- **Status**: Proposed | Accepted | Deprecated | Superseded by ADR-NNN
- **Date**: YYYY-MM-DD
- **Deciders**: ...

## Context
...결정이 필요한 배경...

## Decision Drivers
- ...

## Considered Options
1. Option A — ...
2. Option B — ...

## Decision
Option X를 선택한다.

## Consequences
- Good: ...
- Bad: ...

## ATAM Sensitivity / Tradeoff Points
...
```

---

## 6. 다이어그램 규약

- 모든 다이어그램은 `diagrams/` 디렉터리에 `.drawio` XML 형식으로 저장한다.
- 파일명: `NN-{purpose}.drawio` (예: `03-component-view.drawio`)
- `.drawio` 파일은 draw.io / diagrams.net 앱에서 직접 편집 가능한 XML이어야 한다.
- 문서에서 다이어그램 참조 시 `![설명](../diagrams/NN-file.drawio)` 형식 사용.
- 다이어그램 작성 요청 시: draw.io XML 전체를 생성하고 파일로 저장한다.

---

## 7. 버전 관리 워크플로우

- **브랜치 전략**: 논리적 단위로 `feature/` 브랜치 사용.
  - 예: `feature/fr-agent-core`, `feature/nfr-performance`, `feature/atam`
- **커밋**: 작업 완료 후 반드시 커밋 여부를 사용자에게 물어본다. **자동 커밋 금지.**
- **커밋 메시지**: 글로벌 CLAUDE.md의 규약을 따른다.
- **PR**: 각 기능 영역 완료 시 PR 생성 여부 확인.

---

## 8. 작업 흐름 (How to work)

새 섹션 작성 시:
1. 해당 섹션과 연관된 기존 문서를 먼저 읽는다 (stakeholders, scenarios, 이미 작성된 인접 섹션).
2. 작성 전 설계 방향을 제안하고 승인을 받는다.
3. 작성 후 추적성 표(`Related FR`, `Related NFR`, `Source` 등) 항목이 모두 채워져 있는지 확인한다.
4. Open Question이 발생하면 즉시 해당 문서의 Open Questions 절에 기록한다.
5. 커밋 여부를 사용자에게 물어본다.

---

## 9. 영역 목록 (FR 영역 코드)

| 코드 | 영역명 | 파일 |
|------|--------|------|
| VUI | Voice User Interface | `01-vui.md` |
| AGT | Agent Core | `02-agt.md` |
| BRW | Browser Control | `03-brw.md` |
| CNT | Content Pipeline | `04-cnt.md` |
| MDL | Model Abstraction Layer | `05-mdl.md` |
| WFL | Workflow / Quick Action | `06-wfl.md` |
| HIL | Human-in-the-Loop | `07-hil.md` |
| GNUI | Generative UI | `08-gnui.md` |
| PRV | Privacy & Retention | `09-prv.md` |
