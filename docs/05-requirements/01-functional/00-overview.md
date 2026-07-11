# 5.A 기능 요구사항 (Functional Requirements) — 작성 규약

> Tizen TV AI Web Agent (AI Browser) 설계 문서

본 문서는 5장 하위 기능 요구사항(FR)의 **분류 체계, 표현 패턴(EARS), ID 체계, 카드 스키마**를 정의한다. 본 문서는 영역별 FR 파일(`01-vui.md` 등)의 작성·리뷰 기준이며, 본 문서를 변경할 경우 영역 파일 전체와 정합성을 재확인한다.

---

## 5.A.1 분류 체계 — FR 영역 (Areas)

본 시스템의 FR은 8개 영역으로 분류한다. 분류 기준은 **시스템 외부 또는 인접 영역과의 인터페이스 단위**다. 한 FR이 두 영역에 걸치는 경우, *주된 책임을 가진 영역*에 귀속시키고 다른 영역의 FR로 양방향 참조를 둔다.

| 코드 | 영역 | 책임 범위 | 주된 출처 (시나리오 / Concern) |
|------|------|-----------|------------------------------|
| **VUI** | Voice User Interface | 음성 입력 수신, ASR/NLU 결과 수신, 발화 세션 관리, 음성 응답 요청, 입력 모달리티 조율 | 2.2 시1~4 / 3.2.5 UX팀, 3.3 End User |
| **AGT** | Agent Core | 의도→계획→실행 루프, 상태/컨텍스트 관리, 추론 호출 조율, 결과 합성 | 2.2 시1~4 / 아키텍트(기능 역할) |
| **BRW** | Browser Control | 웹 엔진 세션 확보, CDP·Browser Session 호출, Headless 라이프사이클, 직접 조작 명령 매핑 | 2.2 시1~4 / 3.2.3 웹 엔진 팀 |
| **CNT** | Content Pipeline | DOM·접근성 트리 분석, 핵심 콘텐츠 추출, 모델 입력용 표현 변환, 가공 결과 전달 | 2.2 시1·3 / 3.2.3 웹 엔진 팀, 3.2.5 UX팀 |
| **WFL** | Workflow | 자동 구매·예약·반복 작업의 단계 정의·진행·재개, 사용자 의도 위임 범위 관리 | 2.2 시2·3 / 3.3 End User, 3.2.8 컴플라이언스 |
| **HIL** | Human-in-the-Loop | 결제·민감 작업의 확인·승인·취소·철회 게이트, 동의 상태 모델 운용 | 2.2 시2·3 / 3.2.8 보안 · 컴플라이언스팀 |
| **INT** | Host Integration | Main Agent · GenUI Renderer 호출 계약, 라이프사이클 신호, 진입점 관리 | 3.2.2 ARGO 개발팀 |
| **EXT** | External Services | Account/Wallet 위임, 제3자 사이트 식별·준수 계약, 외부 모델 SLA 핸들링 | 3.6 웹 콘텐츠 · 서비스 제공자(제약), AI 모델 공급사(제약, 3.6) |

영역 간 의존도가 낮은 쪽부터 작성한다. 작성 순서는 **VUI → AGT → BRW → CNT → WFL → HIL → INT → EXT**.

> **참고**: 구 MDL(Model Abstraction Layer) 영역은 모델 추상화 계층을 두지 않기로 결정하여 삭제 — 추론은 클라우드 LLM API 직접 호출(TC-002)이며, 추론 호출 조율은 AGT(Agent Core)가 담당한다.

---

## 5.A.2 표현 패턴 — EARS (Easy Approach to Requirements Syntax)

본 문서는 FR의 단일 문장 표현으로 EARS 5패턴을 채택한다 [^R1]. 모호성을 줄이고 검증 절차로의 변환을 단순하게 만들기 위함이며, 본 시스템이 임베디드·실시간 성격이 섞인 Tizen TV 환경에서 동작한다는 점에서 일반 자유서술형보다 적합하다.

### 5.A.2.1 5가지 패턴

| 패턴 | 형식 | 사용 맥락 |
|------|------|-----------|
| **Ubiquitous** | The `<system>` shall `<response>`. | 상시 충족돼야 하는 기본 능력 |
| **Event-driven** | When `<trigger>`, the `<system>` shall `<response>`. | 외부 이벤트에 대한 반응 |
| **State-driven** | While `<state>`, the `<system>` shall `<response>`. | 특정 상태 동안 유지되어야 하는 행동 |
| **Optional feature** | Where `<feature included>`, the `<system>` shall `<response>`. | 라인업·지역·옵션별 차등 기능 |
| **Unwanted behavior** | If `<unwanted condition>`, then the `<system>` shall `<response>`. | 오류·예외·이상 상황 대응 |

### 5.A.2.2 작성 규칙

- 패턴명을 Statement 끝에 *(Event-driven)* 등으로 병기한다.
- 두 패턴이 결합되어야 표현 가능한 경우 *(Hybrid: Event-driven + Unwanted behavior)* 식으로 명시한다.
- `<system>` 자리에는 본 시스템 약칭으로 **the AI Web Agent**를 사용한다. 하위 모듈명(예: VUI Module)은 사용하지 않는다 — 모듈 분할은 9장 이후의 책임이다.
- Statement는 영문을 1차 표현으로 두고, 바로 아래 줄에 국문 번역을 병기한다 (Statement 영문 + 국문 병기).

### 5.A.2.3 형용사·부사 사용 제한

"적절히, 빠르게, 효율적으로, 충분히, 사용자 친화적으로"와 같은 측정 불가능한 표현은 Statement에 사용하지 않는다. 정량 임계가 필요한 경우 *NFR-XXX-NNN에서 정의한다*로 위임 표기한다(`Related NFR` 필드 참조).

---

## 5.A.3 ID 체계

### 5.A.3.1 형식

```
FR-<영역코드>-<일련번호 3자리>

예: FR-VUI-001, FR-AGT-007, FR-BRW-013
```

### 5.A.3.2 부여 규칙

- 영역별로 `001`부터 연속 부여한다.
- 결번을 두지 않는다. 단, **폐기 시 번호 재사용 금지**(영구 결번).
- ID 변경은 의미 변경에 해당하므로 ADR 기록을 동반한다.
- ID는 본 5장 외부의 8장(Solution Strategy), 9장 이후 View, ADR, 테스트 케이스에서 모두 동일한 형태로 인용된다.

### 5.A.3.3 폐기·대체

| 사유 | 표기 | 처리 |
|------|------|------|
| 의미 무효화 (요구가 더 이상 유효하지 않음) | Status: *Deprecated* | 본문 유지, 폐기 사유와 폐기 ADR 링크를 본문 말미에 추가 |
| 분할 (한 FR이 둘 이상으로 나뉨) | 신규 ID 부여 후 본 ID는 *Superseded by FR-XXX-NNN* | 본문에 *Superseded* 표기, 신규 ID에 본 ID를 *Supersedes* 표기 |
| 통합 (둘 이상이 하나로 합쳐짐) | 동일 | 동일 |

---

## 5.A.4 카드 스키마

각 FR은 다음 필드를 갖는 한 섹션으로 작성한다. 헤더 깊이는 영역 파일 내에서 `###`(즉, 5.A.5.x.y 수준).

```markdown
### FR-<영역>-<번호> <한국어 제목>

- **Statement** *(EARS 패턴명)*:
  - *EN*: <영문 한 문장>
  - *KO*: <국문 한 문장>
- **Description**: <배경·의도·주변 맥락 1~3문장>
- **Source**:
  - 시나리오: <2.2 시 N> (해당 없으면 "—")
  - Stakeholder Concern: <3.x.y 절번 + 짧은 인용>
- **Priority**: <Must / Should / Could / Won't>
- **Acceptance Criteria**:
  1. <검증 가능한 조건 1>
  2. <검증 가능한 조건 2>
- **Related Views**: <3.5 매핑표의 View 명 (콤마 구분)>
- **Related NFR**: <NFR-...-NNN | TBD>
- **Status**: <Draft / Reviewed / Approved | Deprecated / Superseded>
```

### 5.A.4.1 필드별 작성 가이드

| 필드 | 가이드 |
|------|--------|
| Statement | EARS 패턴 1개를 사용. 두 문장 이상 금지. 영문 + 국문 병기. |
| Description | 왜 이 FR이 필요한가, 어떤 시스템 경계에서 의미를 갖는가. *해법은 쓰지 않는다*. |
| Source | 도출 출처. 시나리오·Stakeholder Concern 중 하나 이상. 둘 다 있으면 모두 명시. |
| Priority | 본 5장 단계의 *제안값*. 12번 활동에서 라인업별 재조정 가능. |
| Acceptance Criteria | 정성/정량 모두 허용. 정량 임계가 필요하면 NFR로 위임. 외부 의존(예: 화자 ID 모델)은 해당 외부 인터페이스 가정으로 명시. |
| Related Views | 3.5의 View 명을 그대로 사용. 본 FR이 직접 해소되는 View만 기재. |
| Related NFR | NFR 작성 전이면 *TBD*, 작성 후에는 NFR ID 콤마 구분. |
| Status | Draft → Reviewed → Approved. 본 사이클 산출물의 기본 상태는 Draft. |

### 5.A.4.2 Acceptance Criteria 표현

다음 두 형식을 허용한다.

1. **조건 목록**: 번호 매김 목록으로 검증 조건 나열 (대다수 FR에 적합)
2. **Given–When–Then**: 동작 흐름이 중요할 때 (시2·3의 워크플로 류)

### 5.A.4.3 영역 파일 구조 템플릿

각 영역 파일(`01-vui.md` 등)은 다음 순서를 갖는다.

```markdown
# 5.A.<순번> <영역 풀네임> (<영역코드>)

## 5.A.<순번>.1 영역 정의 및 경계
- 책임 범위
- 영역 외(out-of-area)와의 경계
- 외부 의존 가정

## 5.A.<순번>.2 FR 목록
### FR-<영역>-001 ...
### FR-<영역>-002 ...
...

## 5.A.<순번>.3 영역 단위 추적성 표
| FR ID | 시나리오 | Stakeholder Concern | Priority | Related Views |

## 5.A.<순번>.4 미해결 항목 (Open Questions)
- (있다면 기술)
```

---

## 5.A.5 본 사이클 작업 범위

본 사이클에서는 **VUI(5.A.5)** 만 작성한다. 다른 영역(AGT~EXT)은 후속 사이클로 미루며, 본 문서의 분류·EARS·ID·스키마가 VUI 작업 검증을 통과하면 그대로 확장한다.

---

## 5.A.6 참고 자료 (References)

본 문서의 표현·표기 표준에 대한 출처는 `00-overview.md`의 5.4 참고 자료를 따른다 (EARS [^R1], ISO 29148 [^R2], MoSCoW [^R5]).
