# Functional Requirements — Discovery Draft

> Area 코드 미배정 초안. Stakeholder Concern 기반 전수 도출.
> 모듈 배정·전체 카드 작성은 별도 단계에서 진행한다.
> 각 FR의 **Source** 필드는 직접 Concern을 제기한 Stakeholder를 가리킨다.

> **범위 조정 (2026-06-04)**: 아래 FR을 삭제·재분류했다. ID는 재배정하지 않고 **결번**으로 둔다.
> - **삭제** — FR-002(입력 중재: 상위 플랫폼 입력 레이어 책임, 본 시스템은 전사 발화를 입력으로 받음), FR-015·016·017(페이지 인식·시맨틱 역할·앱 구조정보: 외부 관찰 기능이 아닌 *내부 메커니즘* → 7장 Solution Strategy / Component View로 이관), FR-020(Gen UI 원본 요소 보존: full-page 재구성 스코프 미정 → Gen UI Open Question으로 보류), FR-023(사용자 권한 부분허용·범위설정: 현재 범위 외), FR-024(라인업·지역별 조건부 활성화: 현재 범위 외, 라인업 차등은 TC-004 제약 + NFR 예산으로 표현), FR-026·027·028(멀티프로필: 단일 시스템 다중 프로필은 현재 범위 외).
> - **NFR 재분류** — FR-032(권한 최소화/Least Privilege)는 횡단 보안 품질 속성이므로 **NFR-012**(SEC, 경계 통제)에 흡수. FR-047(Mock/Stub 인터페이스)은 제품 기능이 아닌 테스트 가능성(testability) 품질이므로 삭제 — 이미 **NFR-026**(TST, Mock 적용 범위)·**NFR-024**(MNT, 모듈 결합도)가 소유.
> - **삭제** — FR-048(비자동화 대체 경로 동등성)은 명확한 법적 구속력이 없는 원칙(C등급)이므로 완전 삭제. 관련 기능 보존(직접 조작 인수·원본 페이지 표시·실패 폴백)은 FR-009·FR-014·FR-022가 이미 담당.
> - **흡수** — FR-049(반복 구매·구독 고지·해지)는 FR-006(비가역 동작 확인)의 특수 케이스이므로 FR-006의 Acceptance Criteria로 접음. 규제 의무의 법령·구속력·실현 매핑은 [컴플라이언스 매트릭스](../04-compliance-matrix.md)가 단일 진실원으로 관리한다.
> - **제약·NFR 중복 제거** — FR-018(Gen UI 디자인 시스템 준수)·FR-019(Gen UI 접근성 메타데이터)는 *준수(conformance)* 요건으로, 각각 제약 **CN-001**·**CN-002**(+ 품질 **NFR-022** WCAG)가 이미 동일하게 규정하므로 삭제. FR-039(공개 인터페이스 버전·하위 호환)는 compatibility 품질이므로 삭제 — **NFR-023**(하위 호환 유지기간)이 소유하고, 버전 식별자 부여는 FR-038·041(versioned 인터페이스)이 포함.

---

### FR-001 사용자 의도 해석

- **Statement** *(Event-driven)*:
  - *EN*: When the system receives a transcribed user utterance from the external speech recognition system, the system shall interpret the user's intent and plan an appropriate task accordingly.
  - *KO*: 시스템이 외부 음성 인식 시스템으로부터 전사된 사용자 발화를 수신하면, 시스템은 사용자의 의도를 해석하고 그에 적합한 태스크를 계획해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-003 멀티도메인 태스크 실행

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall execute tasks across multiple domains — including search, information retrieval, reservation, purchase, content discovery, and smart home control — within a single agent session.
  - *KO*: 시스템은 검색·정보 조회·예약·구매·콘텐츠 탐색·스마트홈 제어 등 여러 도메인에 걸친 태스크를 단일 에이전트 세션으로 실행해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-004 End-to-end 태스크 자동 완료

- **Statement** *(Event-driven)*:
  - *EN*: When the user requests a task that commits an external action (e.g., purchase, reservation), the system shall execute all required steps through to completion without requiring manual page navigation.
  - *KO*: 사용자가 외부에 변경을 확정(commit)하는 태스크(구매·예약 등)를 요청하면, 시스템은 사용자가 직접 페이지를 조작하지 않고도 완료까지 모든 단계를 자동으로 수행해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-005 Human-in-the-Loop 확인 요청

- **Statement** *(Event-driven)*:
  - *EN*: When the agent encounters an ambiguous decision point or a high-impact action, the system shall pause execution and request explicit user confirmation before proceeding.
  - *KO*: 에이전트가 모호한 판단 지점 또는 영향이 큰 동작에 직면하면, 시스템은 실행을 중단하고 진행 전에 사용자에게 명시적 확인을 요청해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-006 비가역 동작 전 명시적 확인 단계

- **Statement** *(Event-driven)*:
  - *EN*: Before executing an irreversible action (e.g., payment, deletion, subscription enrollment), the system shall present a confirmation step that the user must explicitly approve.
  - *KO*: 결제·삭제·구독 등 되돌릴 수 없는 동작을 실행하기 전에, 시스템은 사용자가 명시적으로 승인해야 하는 확인 단계를 제시해야 한다.
- **Source**: 고령 사용자 (3.4.2), 규제 · 법률 기관 (3.5)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 결제·구독 등 외부 동작을 사용자 대신 확정하기 전에 확인 단계를 제시하며, 명시적 승인 없이는 실행하지 않는다.
  2. *(반복 결제·구독)* 확정 전에 반복 조건(금액·주기·다음 결제일)을 함께 고지한다. *(구 FR-049 / 컴플라이언스 매트릭스 #7)*
  3. *(반복 결제·구독)* 가입 후 해지 수단으로 사용자를 안내·연결한다 (해지 기능 자체는 해당 서비스가 소유). *(구 FR-049)*

---

### FR-007 에이전트 동작 취소 및 재시도

- **Statement** *(State-driven)*:
  - *EN*: While the agent is executing a task, the system shall allow the user to cancel the current action at any point and optionally retry with corrected input.
  - *KO*: 에이전트가 태스크를 실행하는 동안, 사용자는 언제든지 현재 동작을 취소하고 선택적으로 수정된 입력으로 재시도할 수 있어야 한다.
- **Source**: 일반 시청자 (3.4.1), UX · Design Team (3.2.6)
- **Priority**: **Must**

---

### FR-008 에이전트 진행 상황 실시간 표시

- **Statement** *(State-driven)*:
  - *EN*: While the agent is executing a task, the system shall display the current step (e.g., analyzing page, entering data, awaiting response) in real time to the user.
  - *KO*: 에이전트가 태스크를 실행하는 동안, 시스템은 현재 단계(페이지 분석 중, 데이터 입력 중, 응답 대기 중 등)를 사용자에게 실시간으로 표시해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-009 에이전트-사용자 제어권 전환 (Co-navigation)

- **Statement** *(State-driven)*:
  - *EN*: While the agent is navigating a page, the system shall allow the user to take direct manual control of page interaction at any time without terminating the agent session.
  - *KO*: 에이전트가 페이지를 탐색하는 동안, 사용자는 에이전트 세션을 종료하지 않고 언제든지 직접 페이지 조작 제어권을 가져올 수 있어야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-010 에이전트 백그라운드 실행 (멀티태스킹)

- **Statement** *(State-driven)*:
  - *EN*: While the agent is processing a long-running task, the system shall allow the user to continue using other TV features (e.g., watching content, changing channels) without interrupting the task.
  - *KO*: 에이전트가 장시간 태스크를 처리하는 동안, 사용자는 에이전트 태스크를 중단하지 않고 TV의 다른 기능(VOD 시청, 채널 변경 등)을 계속 사용할 수 있어야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-011 네트워크 끊김 시 태스크 상태 보존 및 재개

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the network connection is interrupted during agent task execution, the system shall preserve the current task state and resume from the last stable checkpoint upon reconnection.
  - *KO*: 에이전트 태스크 실행 중 네트워크 연결이 끊기면, 시스템은 현재 태스크 상태를 보존하고 재연결 후 마지막 안정 체크포인트부터 재개해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-012 Headless 실행 모드

- **Statement** *(Optional feature)*:
  - *EN*: Where the task result does not require a visible browser window (e.g., information lookup, reservation status check), the system shall complete the task and return only the result without rendering a full browser page.
  - *KO*: 정보 조회·예약 확인 등 가시적 브라우저 화면이 불필요한 태스크의 경우, 시스템은 브라우저 페이지를 렌더링하지 않고 태스크를 완료하여 결과만 반환해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-013 로그인 및 인증 장벽 처리 (자동 또는 HITL)

- **Statement** *(Event-driven)*:
  - *EN*: When the agent encounters a login wall or CAPTCHA during task execution, the system shall first attempt automatic resolution using stored credentials or platform-provided authentication; if automatic resolution is not possible, the system shall pause and hand off to the user via a Human-in-the-Loop step.
  - *KO*: 에이전트가 태스크 실행 중 로그인 페이지나 캡차를 만나면, 저장된 자격증명 또는 플랫폼 제공 인증을 사용한 자동 처리를 먼저 시도해야 하며, 자동 처리가 불가능한 경우에는 실행을 중단하고 Human-in-the-Loop 단계를 통해 사용자에게 인증을 넘겨야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Must**

---

### FR-014 Headless 처리 결과를 Headed 모드로 전환

- **Statement** *(Event-driven)*:
  - *EN*: When the agent has completed or is processing a task in headless mode and the user requests to view the underlying page, the system shall render the full browser page in headed mode without restarting the task.
  - *KO*: 에이전트가 Headless 모드로 태스크를 처리 중이거나 완료한 상태에서 사용자가 해당 페이지 보기를 요청하면, 시스템은 태스크를 재시작하지 않고 전체 브라우저 페이지를 Headed 모드로 렌더링해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-021 AI 생성 콘텐츠 표시 (출처 및 생성 여부 명시)

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall clearly indicate when displayed content is generated or summarized by AI and shall provide a reference to the original source.
  - *KO*: 시스템은 표시되는 콘텐츠가 AI에 의해 생성·요약된 경우 이를 명확히 표시하고, 원본 출처에 대한 참조를 제공해야 한다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Priority**: **Must**

---

### FR-022 에이전트 실패 시 폴백 경로 제공

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent fails to complete a task, the system shall provide a recovery path (e.g., redirect to the original page, offer step-by-step manual guidance) rather than leaving the user in an unrecoverable state.
  - *KO*: 에이전트가 태스크를 완료하지 못하면, 시스템은 사용자가 복구 불가 상태로 남지 않도록 대체 경로(원본 페이지 이동, 수동 단계 안내 등)를 제공해야 한다.
- **Source**: UX · Design Team (3.2.6), Tizen Platform Team (3.2.3)
- **Priority**: **Must**

---

### FR-025 사용자 히스토리·선호 누적 및 활용

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall accumulate the user's browsing history, past agent requests, and task outcomes, and apply this context to improve task accuracy and reduce required user input over time.
  - *KO*: 시스템은 사용자의 브라우저 이력, 과거 에이전트 요청, 태스크 결과를 누적하고, 이를 이후 태스크의 정확도 향상 및 사용자 입력 감소에 활용해야 한다.
- **Source**: 일반 시청자 (3.4.1)
- **Priority**: **Should**

---

### FR-029 UI 표시 옵션 설정 (텍스트 크기·대비·응답 속도)

- **Statement** *(Optional feature)*:
  - *EN*: Where accessibility display settings are configured, the system shall apply the user's preferred text size, contrast ratio, and agent voice response speed to all outputs.
  - *KO*: 접근성 표시 설정이 구성된 경우, 시스템은 사용자가 선택한 텍스트 크기, 대비 비율, 에이전트 음성 응답 속도를 모든 출력에 적용해야 한다.
- **Source**: 고령 사용자 (3.4.2)
- **Priority**: **Should**

---

### FR-030 자격증명 안전 저장 및 사용

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall store user credentials (e.g., login tokens, session keys) in an isolated secure credential store and shall not expose them to the AI model or transmit them beyond their intended scope.
  - *KO*: 시스템은 사용자 자격증명(로그인 토큰, 세션 키 등)을 격리된 안전한 자격증명 저장소에 보관해야 하며, AI 모델에 노출하거나 의도된 범위를 벗어나 전송해서는 안 된다.
- **Source**: Security & Privacy Team (3.2.5)
- **Priority**: **Must**

---

### FR-031 PII 데이터 처리 및 보존 기간 제어

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall classify all data that may contain PII (voice transcripts, DOM snapshots, screen content) and enforce the configured retention period, automatically deleting data upon expiry.
  - *KO*: 시스템은 잠재적 PII가 포함될 수 있는 모든 데이터(음성 전사, DOM 스냅샷, 화면 콘텐츠)를 분류하고, 설정된 보존 기간을 적용하며 만료 시 자동 삭제해야 한다.
- **Source**: Security & Privacy Team (3.2.5)
- **Priority**: **Must**

---

### FR-033 프롬프트 인젝션 탐지 및 방어

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent detects instructions embedded in external web content that attempt to hijack or redirect agent behavior, the system shall discard those instructions and notify the user.
  - *KO*: 에이전트가 외부 웹 콘텐츠에 삽입된 에이전트 동작 탈취·유도 명령을 탐지하면, 시스템은 해당 명령을 무시하고 사용자에게 알려야 한다.
- **Source**: Security & Privacy Team (3.2.5)
- **Priority**: **Must**

---

### FR-034 개인정보 동의 수집 및 데이터 주체 권리 행사

- **Statement** *(Event-driven)*:
  - *EN*: Before collecting personal data, the system shall obtain user consent; the system shall also provide a mechanism for users to exercise data subject rights (access, deletion, portability) at any time.
  - *KO*: 개인정보를 수집하기 전에, 시스템은 사용자 동의를 받아야 하며, 사용자가 데이터 주체 권리(열람·삭제·이전)를 언제든지 행사할 수 있는 수단을 제공해야 한다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Priority**: **Must**

---

### FR-035 데이터 처리 방식 사용자 고지 (온디바이스 vs 클라우드)

- **Statement** *(Event-driven)*:
  - *EN*: Before transmitting user data to an off-device cloud service, the system shall inform the user that the data will be processed outside the device and obtain acknowledgment.
  - *KO*: 사용자 데이터를 온디바이스 외부의 클라우드 서비스로 전송하기 전에, 시스템은 데이터가 디바이스 외부에서 처리됨을 사용자에게 고지하고 확인을 받아야 한다.
- **Source**: 규제 · 법률 기관 (3.5)
- **Priority**: **Must**

---

### FR-036 시스템 전원 상태 변화에 따른 에이전트 생명주기 관리

- **Statement** *(Event-driven)*:
  - *EN*: When the TV transitions to sleep mode or resumes from sleep, the system shall gracefully suspend and restore the agent harness in accordance with platform lifecycle events.
  - *KO*: TV가 슬립 모드에 진입하거나 슬립에서 복귀할 때, 시스템은 플랫폼 생명주기 이벤트에 따라 에이전트 하네스를 안전하게 일시 중단하고 복원해야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Priority**: **Must**

---

### FR-037 에이전트 오류 격리 및 상위 시스템 보호

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent harness encounters an unrecoverable error, the system shall isolate the failure and prevent it from propagating to the host Main Agent or Generative UI Renderer.
  - *KO*: 에이전트 하네스에서 복구 불가능한 오류가 발생하면, 시스템은 오류를 격리하여 상위 Main Agent 또는 Generative UI Renderer로 전파되지 않도록 해야 한다.
- **Source**: Tizen Platform Team (3.2.3)
- **Priority**: **Must**

---

### FR-038 MCP / OpenAPI 기반 Skill 등록 인터페이스 제공

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall expose a versioned, standard callable interface (MCP tool spec or OpenAPI schema) that allows external agent platforms to register and invoke this agent as a Skill or Sub-Agent.
  - *KO*: 시스템은 외부 에이전트 플랫폼이 본 시스템을 Skill 또는 Sub-Agent로 등록하고 호출할 수 있도록 버전 관리된 표준 호출 인터페이스(MCP tool spec 또는 OpenAPI 스키마)를 노출해야 한다.
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.3), Tizen Platform Team (3.2.3)
- **Priority**: **Should**

---

### FR-040 외부 호출자 인증 및 권한 검증

- **Statement** *(Event-driven)*:
  - *EN*: When an external agent platform invokes the system, the system shall authenticate the caller and verify that the requested operation falls within the caller's granted permission scope before execution.
  - *KO*: 외부 에이전트 플랫폼이 시스템을 호출하면, 시스템은 호출자를 인증하고 요청된 동작이 호출자의 허가된 권한 범위 내에 있는지 실행 전에 검증해야 한다.
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.3)
- **Priority**: **Should**

---

### FR-041 표준화된 실행 결과 반환 스키마

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall return agent task results in a structured, versioned schema that includes execution status (success, failure, partial), outcome data, and error codes.
  - *KO*: 시스템은 에이전트 태스크 결과를 실행 상태(성공·실패·부분 완료), 결과 데이터, 에러 코드를 포함한 구조화된 버전 관리 스키마로 반환해야 한다.
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.3)
- **Priority**: **Should**

---

### FR-042 에이전트 신원 식별 정보 제공 (Agent-ID)

- **Statement** *(Ubiquitous)*:
  - *EN*: When accessing external web services, the system shall identify itself via a declared Agent-ID or a custom User-Agent string that is distinguishable from regular browser traffic.
  - *KO*: 외부 웹 서비스에 접근할 때, 시스템은 일반 브라우저 트래픽과 구별 가능한 선언된 Agent-ID 또는 커스텀 User-Agent 문자열로 자신을 식별해야 한다.
- **Source**: 제3자 웹 서비스 운영자 (3.3.2)
- **Priority**: **Should**

---

### FR-043 에이전트 접근 정책 준수 (robots.txt · 이용약관)

- **Statement** *(Event-driven)*:
  - *EN*: Before performing automation on a web service, the system shall check the service's declared agent access policy (e.g., robots.txt, agent access control headers) and comply with it, including respecting declared rate limits and crawl delays.
  - *KO*: 웹 서비스에 자동화를 수행하기 전에, 시스템은 해당 서비스가 선언한 에이전트 접근 정책(robots.txt, 에이전트 접근 제어 헤더 등)을 확인하고 이를 준수해야 하며, 선언된 속도 제한(rate limit)과 크롤 지연도 포함해야 한다.
- **Source**: 제3자 웹 서비스 운영자 (3.3.2)
- **Priority**: **Should**

---

### FR-044 에이전트 루프 실행 트레이스 기록

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall record a structured trace of each agent loop iteration — including tool calls, context snapshots, model responses, and error points — and make it accessible to developers.
  - *KO*: 시스템은 에이전트 루프 각 이터레이션의 구조화된 트레이스(도구 호출, 컨텍스트 스냅샷, 모델 응답, 에러 지점)를 기록하고 개발자가 접근할 수 있도록 해야 한다.
- **Source**: AI Web Agent 개발팀 (3.2.8)
- **Priority**: **Should**

---

### FR-045 토큰 사용량·응답 지연·비용 계측 및 노출

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall measure per-request token usage, model response latency, and estimated cost, and expose these metrics in a format accessible to developers and operators.
  - *KO*: 시스템은 요청별 토큰 사용량, 모델 응답 지연 시간, 예상 비용을 측정하고, 개발자 및 운영자가 접근 가능한 형식으로 노출해야 한다.
- **Source**: AI Web Agent 개발팀 (3.2.8), VD 상품화 담당자 (3.2.2)
- **Priority**: **Should**

---

### FR-046 에이전트 실행 환경 시뮬레이션 모드

- **Statement** *(Optional feature)*:
  - *EN*: Where a simulation mode is configured, the system shall execute agent tasks against a sandboxed or mock browser environment without affecting real external services or user data.
  - *KO*: 시뮬레이션 모드가 구성된 경우, 시스템은 실제 외부 서비스나 사용자 데이터에 영향을 주지 않고 샌드박스 또는 목(mock) 브라우저 환경에서 에이전트 태스크를 실행해야 한다.
- **Source**: AI Web Agent 개발팀 (3.2.8), QA · Certification Team (3.2.7)
- **Priority**: **Should**

---

### FR-050 에이전트 행동 이유 설명 (Explainability)

- **Statement** *(Event-driven)*:
  - *EN*: When the user requests an explanation of an agent action, the system shall provide a concise, human-readable rationale describing why that action was taken.
  - *KO*: 사용자가 에이전트 행동에 대한 설명을 요청하면, 시스템은 해당 행동이 왜 취해졌는지에 대한 간결하고 이해 가능한 이유를 제공해야 한다.
- **Source**: UX · Design Team (3.2.6), 접근성 요구 사용자 (3.4.3)
- **Priority**: **Should**

---

## 요약

| 번호 | 제목 | Source | Priority |
|------|------|--------|----------|
| FR-001 | 사용자 의도 해석 | 3.4.1 | Must |
| FR-003 | 멀티도메인 태스크 실행 | 3.4.1 | Must |
| FR-004 | End-to-end 태스크 자동 완료 | 3.4.1 | Must |
| FR-005 | Human-in-the-Loop 확인 요청 | 3.4.1 | Must |
| FR-006 | 비가역 동작 전 명시적 확인 단계 | 3.4.2, 3.5 | Must |
| FR-007 | 에이전트 동작 취소 및 재시도 | 3.4.1, 3.2.6 | Must |
| FR-008 | 에이전트 진행 상황 실시간 표시 | 3.4.1 | Must |
| FR-009 | 에이전트-사용자 제어권 전환 (Co-navigation) | 3.4.1 | Should |
| FR-010 | 에이전트 백그라운드 실행 (멀티태스킹) | 3.4.1 | Should |
| FR-011 | 네트워크 끊김 시 태스크 상태 보존 및 재개 | 3.4.1 | Should |
| FR-012 | Headless 실행 모드 | 3.4.1 | Should |
| FR-013 | 로그인 및 인증 장벽 처리 (자동 또는 HITL) | 3.4.1 | Must |
| FR-014 | Headless 처리 결과를 Headed 모드로 전환 | 3.4.1 | Should |
| FR-021 | AI 생성 콘텐츠 표시 (출처 및 생성 여부 명시) | 3.5 | Must |
| FR-022 | 에이전트 실패 시 폴백 경로 제공 | 3.2.6, 3.2.3 | Must |
| FR-025 | 사용자 히스토리·선호 누적 및 활용 | 3.4.1 | Should |
| FR-029 | UI 표시 옵션 설정 (텍스트 크기·대비·응답 속도) | 3.4.2 | Should |
| FR-030 | 자격증명 안전 저장 및 사용 | 3.2.5 | Must |
| FR-031 | PII 데이터 처리 및 보존 기간 제어 | 3.2.5 | Must |
| FR-033 | 프롬프트 인젝션 탐지 및 방어 | 3.2.5 | Must |
| FR-034 | 개인정보 동의 수집 및 데이터 주체 권리 행사 | 3.5 | Must |
| FR-035 | 데이터 처리 방식 사용자 고지 (온디바이스 vs 클라우드) | 3.5 | Must |
| FR-036 | 시스템 전원 상태 변화에 따른 에이전트 생명주기 관리 | 3.2.3 | Must |
| FR-037 | 에이전트 오류 격리 및 상위 시스템 보호 | 3.2.3 | Must |
| FR-038 | MCP / OpenAPI 기반 Skill 등록 인터페이스 제공 | 3.3.3, 3.2.3 | Should |
| FR-040 | 외부 호출자 인증 및 권한 검증 | 3.3.3 | Should |
| FR-041 | 표준화된 실행 결과 반환 스키마 | 3.3.3 | Should |
| FR-042 | 에이전트 신원 식별 정보 제공 (Agent-ID) | 3.3.2 | Should |
| FR-043 | 에이전트 접근 정책 준수 (robots.txt · 이용약관) | 3.3.2 | Should |
| FR-044 | 에이전트 루프 실행 트레이스 기록 | 3.2.8 | Should |
| FR-045 | 토큰 사용량·응답 지연·비용 계측 및 노출 | 3.2.8, 3.2.2 | Should |
| FR-046 | 에이전트 실행 환경 시뮬레이션 모드 | 3.2.8, 3.2.7 | Should |
| FR-050 | 에이전트 행동 이유 설명 (Explainability) | 3.2.6, 3.4.3 | Should |
