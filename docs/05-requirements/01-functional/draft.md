# Functional Requirements — Discovery Draft

> Area 코드 미배정 초안. Stakeholder Concern 기반 전수 도출.
> 모듈 배정·전체 카드 작성은 별도 단계에서 진행한다.
> 각 FR의 **Source** 필드는 직접 Concern을 제기한 Stakeholder를 가리킨다.

> **정비 이력**: 초안의 전수 도출 목록에서 범위 외·중복·품질성 항목을 정리하고 **FR-001~033으로 재번호**했다. 범위 제외(입력 중재·페이지 인식·멀티프로필·조건부 활성화·Gen UI 원본 보존 등), NFR·제약으로 표현(권한 최소화·Mock/Stub·디자인/접근성 준수·인터페이스 하위 호환), Acceptance Criteria로 흡수(반복 구매·구독 고지 → 비가역 동작 확인 FR). 상세 이력은 git 및 [컴플라이언스 매트릭스](../04-compliance-matrix.md) 참조.
>
> **범위 제외 (자격증명 저장)**: 자격증명의 *저장·암호화*는 플랫폼/브라우저 소관이므로 본 시스템 범위에서 제외했다(구 FR-018 삭제, NFR-013 삭제 — FR-018은 현재 결번). 에이전트는 플랫폼/브라우저가 보관한 자격증명을 *사용*(FR-012)하며, 모델 컨텍스트로의 비유출은 NFR-018이 보증한다.
>
> **범위 제외 (AI 생성·출처 표시)**: 구 FR-014(AI 생성 콘텐츠 표시) 삭제 — FR-014는 현재 결번. 사유: ① "AI 생성 표시"는 본 draft가 다루는 에이전트 하네스의 동작이 아니라 Generative UI(GNUI) 렌더러의 표시 책임이며, ② Source로 인용된 3.2.8(구 3.2.5)의 "광고성 표시 의무" Concern은 *광고/협찬 고지* 의무로 "AI 생성 고지"와 다른 규제 도메인이라 VoC 매핑이 잘못되어 있었음. 표시·라벨링 FR은 GNUI area에서 별도로 다룬다.
>
> **범위 제외 (Generative Web Page, GWP)**: 구 FR-034~038(GWP 적용 판정·트리거, 콘텐츠 추출·재구성, D-pad 포커스 네비게이션, 원본 토글, 의도 힌트 전달) 삭제 — FR-034~038은 현재 결번. GWP(페이지 자체 재가공 서비스)가 과제 범위에서 제외됨에 따라 관련 FR 전체를 제거했다(1장 Out-of-Scope 참조). 연동 삭제: NFR-031~033, 구 UC-08·28·29(현재 UC는 2026-07 재번호됨), 2장 시나리오 5~7.

---

### FR-001 사용자 의도 해석

- **Statement** *(Event-driven)*:
  - *EN*: When the system receives a transcribed user utterance from the external speech recognition system, the system shall interpret the user's intent and plan an appropriate task accordingly.
  - *KO*: 시스템이 외부 음성 인식 시스템으로부터 전사된 사용자 발화를 수신하면, 시스템은 사용자의 의도를 해석하고 그에 적합한 태스크를 계획해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**

---

### FR-002 멀티도메인 태스크 실행

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall execute tasks across multiple domains — including search, information retrieval, reservation, purchase, content discovery, and smart home control — within a single agent session.
  - *KO*: 시스템은 검색·정보 조회·예약·구매·콘텐츠 탐색·스마트홈 제어 등 여러 도메인에 걸친 태스크를 단일 에이전트 세션으로 실행해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**

---

### FR-003 End-to-end 태스크 자동 완료

- **Statement** *(Event-driven)*:
  - *EN*: When the user requests a task that commits an external action (e.g., purchase, reservation), the system shall execute all required steps through to completion without requiring manual page navigation, **except where a Human-in-the-Loop step is mandated** (ambiguous decision per FR-004, irreversible-action confirmation per FR-005, or authentication hand-off per FR-012); in such cases the system shall pause, obtain user input, and resume automated execution from the pause point upon confirmation.
  - *KO*: 사용자가 외부에 변경을 확정(commit)하는 태스크(구매·예약 등)를 요청하면, 시스템은 사용자가 직접 페이지를 조작하지 않고도 완료까지 모든 단계를 자동으로 수행해야 한다. **단, Human-in-the-Loop 단계가 요구되는 경우(FR-004의 모호한 판단, FR-005의 비가역 동작 승인, FR-012의 인증 인계)는 예외이며**, 이 경우 시스템은 실행을 일시 중단하고 사용자 입력을 받은 뒤 중단 지점부터 자동 실행을 재개해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. HITL 사유 외의 단계는 사용자의 추가 조작 없이 자동으로 진행한다.
  2. FR-004 / FR-005 / FR-012 트리거가 발생하면 즉시 자동 진행을 중단하고 사용자 입력을 요청한다.
  3. 사용자 확인을 받으면 중단 지점에서 자동 실행을 재개하며, 태스크를 처음부터 다시 수행하지 않는다.

---

### FR-003a Quick Action — 에이전트 루프 우회 빠른 경로

- **Statement** *(Event-driven)*:
  - *EN*: When the user issues a command that matches a predefined Quick Action (e.g., app launch, channel change, weather lookup, direct device control), the system shall execute it deterministically through a dedicated fast path that bypasses the LLM-based agent reasoning loop.
  - *KO*: 사용자가 사전 정의된 Quick Action(앱 실행, 채널 변경, 날씨 조회, 직접 디바이스 제어 등)에 해당하는 명령을 발화하면, 시스템은 LLM 기반 에이전트 추론 루프를 우회하는 전용 빠른 경로로 결정론적으로 실행해야 한다.
- **Source**: 일반 사용자 (3.3.1) — "태스크 복잡도에 비례한 응답 속도" Concern
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. Quick Action 카탈로그에 일치하는 입력은 에이전트 추론 단계를 호출하지 않는다.
  2. Quick Action 분류 결과가 모호하면 일반 에이전트 경로(FR-001)로 폴백한다.
  3. Quick Action 실행 실패 시에도 FR-015 폴백 경로가 적용된다.
- **Related NFR**: `NFR-001` (간단한 명령의 End-to-end 응답 시간 — Quick Action 경로 대상)

---

### FR-004 Human-in-the-Loop 확인 요청

- **Statement** *(Event-driven)*:
  - *EN*: When the agent encounters an ambiguous decision point or a high-impact action, the system shall pause execution and request explicit user confirmation before proceeding.
  - *KO*: 에이전트가 모호한 판단 지점 또는 영향이 큰 동작에 직면하면, 시스템은 실행을 중단하고 진행 전에 사용자에게 명시적 확인을 요청해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**

---

### FR-005 비가역 동작 전 명시적 확인 단계

- **Statement** *(Event-driven)*:
  - *EN*: Before executing an irreversible action (e.g., payment, deletion, subscription enrollment), the system shall present a confirmation step that the user must explicitly approve.
  - *KO*: 결제·삭제·구독 등 되돌릴 수 없는 동작을 실행하기 전에, 시스템은 사용자가 명시적으로 승인해야 하는 확인 단계를 제시해야 한다.
- **Source**: 고령 사용자 (3.3.2), 보안 · 컴플라이언스팀 (3.2.8)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 결제·구독 등 외부 동작을 사용자 대신 확정하기 전에 확인 단계를 제시하며, 명시적 승인 없이는 실행하지 않는다.
  2. *(반복 결제·구독)* 확정 전에 반복 조건(금액·주기·다음 결제일)을 함께 고지한다. *(컴플라이언스 매트릭스 #7)*
  3. *(반복 결제·구독)* 가입 후 해지 수단으로 사용자를 안내·연결한다 (해지 기능 자체는 해당 서비스가 소유).

---

### FR-006 에이전트 동작 취소 및 재시도

- **Statement** *(State-driven)*:
  - *EN*: While the agent is executing a task, the system shall allow the user to cancel the current action at any point and optionally retry with corrected input.
  - *KO*: 에이전트가 태스크를 실행하는 동안, 사용자는 언제든지 현재 동작을 취소하고 선택적으로 수정된 입력으로 재시도할 수 있어야 한다.
- **Source**: 일반 사용자 (3.3.1), UX팀 (3.2.5)
- **Priority**: **Must**
- **Related NFR**: `NFR-034` (개입 시 동작 중지 응답성)

---

### FR-007 에이전트 진행 상황 실시간 표시

- **Statement** *(State-driven)*:
  - *EN*: While the agent is executing a task, the system shall display the current step (e.g., analyzing page, entering data, awaiting response) in real time to the user.
  - *KO*: 에이전트가 태스크를 실행하는 동안, 시스템은 현재 단계(페이지 분석 중, 데이터 입력 중, 응답 대기 중 등)를 사용자에게 실시간으로 표시해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**

---

### FR-008 에이전트-사용자 제어권 전환 (Co-navigation)

- **Statement** *(State-driven)*:
  - *EN*: While the agent is navigating a page, the system shall allow the user to take direct manual control of page interaction at any time without terminating the agent session.
  - *KO*: 에이전트가 페이지를 탐색하는 동안, 사용자는 에이전트 세션을 종료하지 않고 언제든지 직접 페이지 조작 제어권을 가져올 수 있어야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**
- **Related NFR**: `NFR-034` (제어권 회수 시 동작 중지 응답성)

---

### FR-009 에이전트 백그라운드 실행 (멀티태스킹)

- **Statement** *(State-driven)*:
  - *EN*: While the agent is processing a long-running task, the system shall allow the user to continue using other TV features (e.g., watching content, changing channels) without interrupting the task.
  - *KO*: 에이전트가 장시간 태스크를 처리하는 동안, 사용자는 에이전트 태스크를 중단하지 않고 TV의 다른 기능(VOD 시청, 채널 변경 등)을 계속 사용할 수 있어야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**

---

### FR-010 네트워크 끊김 시 태스크 상태 보존 및 재개

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the network connection is interrupted during agent task execution, the system shall preserve the current task state and resume from the last stable checkpoint upon reconnection.
  - *KO*: 에이전트 태스크 실행 중 네트워크 연결이 끊기면, 시스템은 현재 태스크 상태를 보존하고 재연결 후 마지막 안정 체크포인트부터 재개해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**

---

### FR-011 Headless 실행 모드

- **Statement** *(Optional feature)*:
  - *EN*: Where the task result does not require a visible browser window (e.g., information lookup, reservation status check), the system shall complete the task and return only the result without rendering a full browser page.
  - *KO*: 정보 조회·예약 확인 등 가시적 브라우저 화면이 불필요한 태스크의 경우, 시스템은 브라우저 페이지를 렌더링하지 않고 태스크를 완료하여 결과만 반환해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**

---

### FR-012 로그인 및 인증 장벽 처리 (자동 또는 HITL)

- **Statement** *(Event-driven)*:
  - *EN*: When the agent encounters a login wall or CAPTCHA during task execution, the system shall first attempt automatic resolution using platform/browser-stored credentials or platform-provided authentication; if automatic resolution is not possible, the system shall pause and hand off to the user via a Human-in-the-Loop step.
  - *KO*: 에이전트가 태스크 실행 중 로그인 페이지나 캡차를 만나면, **플랫폼·브라우저가 보관한 자격증명** 또는 플랫폼 제공 인증을 사용한 자동 처리를 먼저 시도해야 하며, 자동 처리가 불가능한 경우에는 실행을 중단하고 Human-in-the-Loop 단계를 통해 사용자에게 인증을 넘겨야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Must**

---

### FR-013 Headless 처리 결과를 Headed 모드로 전환

- **Statement** *(Event-driven)*:
  - *EN*: When the agent has completed or is processing a task in headless mode and the user requests to view the underlying page, the system shall render the full browser page in headed mode without restarting the task.
  - *KO*: 에이전트가 Headless 모드로 태스크를 처리 중이거나 완료한 상태에서 사용자가 해당 페이지 보기를 요청하면, 시스템은 태스크를 재시작하지 않고 전체 브라우저 페이지를 Headed 모드로 렌더링해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**

---

### FR-015 에이전트 실패 시 폴백 경로 제공

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent fails to complete a task, the system shall provide a recovery path (e.g., redirect to the original page, offer step-by-step manual guidance) rather than leaving the user in an unrecoverable state.
  - *KO*: 에이전트가 태스크를 완료하지 못하면, 시스템은 사용자가 복구 불가 상태로 남지 않도록 대체 경로(원본 페이지 이동, 수동 단계 안내 등)를 제공해야 한다.
- **Source**: UX팀 (3.2.5), ARGO 개발팀 (3.2.2)
- **Priority**: **Must**

---

### FR-016 사용자 히스토리·선호 누적 및 활용

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall accumulate the user's browsing history, past agent requests, and task outcomes, and apply this context to improve task accuracy and reduce required user input over time.
  - *KO*: 시스템은 사용자의 브라우저 이력, 과거 에이전트 요청, 태스크 결과를 누적하고, 이를 이후 태스크의 정확도 향상 및 사용자 입력 감소에 활용해야 한다.
- **Source**: 일반 사용자 (3.3.1)
- **Priority**: **Should**

---

### FR-017 UI 표시 옵션 설정 (텍스트 크기·대비·응답 속도)

- **Statement** *(Optional feature)*:
  - *EN*: Where accessibility display settings are configured, the system shall apply the user's preferred text size, contrast ratio, and agent voice response speed to all outputs.
  - *KO*: 접근성 표시 설정이 구성된 경우, 시스템은 사용자가 선택한 텍스트 크기, 대비 비율, 에이전트 음성 응답 속도를 모든 출력에 적용해야 한다.
- **Source**: 고령 사용자 (3.3.2)
- **Priority**: **Should**

---

### FR-019 PII 데이터 처리 및 보존 기간 제어

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall classify all data that may contain PII (voice transcripts, DOM snapshots, screen content) and enforce the configured retention period, automatically deleting data upon expiry.
  - *KO*: 시스템은 잠재적 PII가 포함될 수 있는 모든 데이터(음성 전사, DOM 스냅샷, 화면 콘텐츠)를 분류하고, 설정된 보존 기간을 적용하며 만료 시 자동 삭제해야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8)
- **Priority**: **Must**

---

### FR-020 프롬프트 인젝션 탐지 및 방어

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent detects instructions embedded in external web content that attempt to hijack or redirect agent behavior, the system shall discard those instructions and notify the user.
  - *KO*: 에이전트가 외부 웹 콘텐츠에 삽입된 에이전트 동작 탈취·유도 명령을 탐지하면, 시스템은 해당 명령을 무시하고 사용자에게 알려야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8)
- **Priority**: **Must**

---

### FR-021 개인정보 동의 수집 및 데이터 주체 권리 행사

- **Statement** *(Event-driven)*:
  - *EN*: Before collecting personal data, the system shall obtain user consent; the system shall also provide a mechanism for users to exercise data subject rights (access, deletion, portability) at any time.
  - *KO*: 개인정보를 수집하기 전에, 시스템은 사용자 동의를 받아야 하며, 사용자가 데이터 주체 권리(열람·삭제·이전)를 언제든지 행사할 수 있는 수단을 제공해야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8)
- **Priority**: **Must**

---

### FR-022 데이터 처리 방식 사용자 고지 (온디바이스 vs 클라우드)

- **Statement** *(Event-driven)*:
  - *EN*: Before transmitting user data to an off-device cloud service, the system shall inform the user that the data will be processed outside the device and obtain acknowledgment.
  - *KO*: 사용자 데이터를 온디바이스 외부의 클라우드 서비스로 전송하기 전에, 시스템은 데이터가 디바이스 외부에서 처리됨을 사용자에게 고지하고 확인을 받아야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8)
- **Priority**: **Must**

---

### FR-023 시스템 전원 상태 변화에 따른 에이전트 생명주기 관리

- **Statement** *(Event-driven)*:
  - *EN*: When the TV transitions to sleep mode or resumes from sleep, the system shall gracefully suspend and restore the agent harness in accordance with platform lifecycle events.
  - *KO*: TV가 슬립 모드에 진입하거나 슬립에서 복귀할 때, 시스템은 플랫폼 생명주기 이벤트에 따라 에이전트 하네스를 안전하게 일시 중단하고 복원해야 한다.
- **Source**: 웹 엔진 팀 (3.2.3)
- **Priority**: **Must**

---

### FR-024 에이전트 오류 격리 및 상위 시스템 보호

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the agent harness encounters an unrecoverable error, the system shall isolate the failure and prevent it from propagating to the host Main Agent or Generative UI Renderer.
  - *KO*: 에이전트 하네스에서 복구 불가능한 오류가 발생하면, 시스템은 오류를 격리하여 상위 Main Agent 또는 Generative UI Renderer로 전파되지 않도록 해야 한다.
- **Source**: ARGO 개발팀 (3.2.2)
- **Priority**: **Must**

---

### FR-025 표준 인터페이스 기반 Skill / Sub-Agent 등록·호출

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall expose a versioned, standard callable interface that allows external agent platforms to register and invoke this agent as a Skill or Sub-Agent.
  - *KO*: 시스템은 외부 에이전트 플랫폼이 본 시스템을 Skill 또는 Sub-Agent로 등록·호출할 수 있도록 버전 관리된 표준 호출 인터페이스를 노출해야 한다.
- **Source**: ARGO 개발팀 (3.2.2, 참조 소비자) — cf. 3.6 외부 Agent 플랫폼(제약)
- **Priority**: **Should**

---

### FR-026 외부 호출자 인증 및 권한 검증

- **Statement** *(Event-driven)*:
  - *EN*: When an external agent platform invokes the system, the system shall authenticate the caller and verify that the requested operation falls within the caller's granted permission scope before execution.
  - *KO*: 외부 에이전트 플랫폼이 시스템을 호출하면, 시스템은 호출자를 인증하고 요청된 동작이 호출자의 허가된 권한 범위 내에 있는지 실행 전에 검증해야 한다.
- **Source**: ARGO 개발팀 (3.2.2, 참조 소비자) — cf. 3.6 외부 Agent 플랫폼(제약)
- **Priority**: **Should**

---

### FR-027 표준화된 실행 결과 반환 스키마

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall return agent task results in a structured, versioned schema that includes execution status (success, failure, partial), outcome data, and error codes.
  - *KO*: 시스템은 에이전트 태스크 결과를 실행 상태(성공·실패·부분 완료), 결과 데이터, 에러 코드를 포함한 구조화된 버전 관리 스키마로 반환해야 한다.
- **Source**: ARGO 개발팀 (3.2.2, 참조 소비자) — cf. 3.6 외부 Agent 플랫폼(제약)
- **Priority**: **Should**

---

### FR-028 에이전트 신원 식별 정보 제공 (Agent-ID)

- **Statement** *(Ubiquitous)*:
  - *EN*: When accessing external web services, the system shall identify itself via a declared Agent-ID or a custom User-Agent string that is distinguishable from regular browser traffic.
  - *KO*: 외부 웹 서비스에 접근할 때, 시스템은 일반 브라우저 트래픽과 구별 가능한 선언된 Agent-ID 또는 커스텀 User-Agent 문자열로 자신을 식별해야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8) — cf. 3.6 웹 콘텐츠·서비스 제공자(제약)
- **Priority**: **Should**

---

### FR-029 에이전트 접근 정책 준수 (robots.txt · 이용약관)

- **Statement** *(Event-driven)*:
  - *EN*: Before performing automation on a web service, the system shall check the service's declared agent access policy (e.g., robots.txt, agent access control headers) and comply with it, including respecting declared rate limits and crawl delays.
  - *KO*: 웹 서비스에 자동화를 수행하기 전에, 시스템은 해당 서비스가 선언한 에이전트 접근 정책(robots.txt, 에이전트 접근 제어 헤더 등)을 확인하고 이를 준수해야 하며, 선언된 속도 제한(rate limit)과 크롤 지연도 포함해야 한다.
- **Source**: 보안 · 컴플라이언스팀 (3.2.8) — cf. 3.6 웹 콘텐츠·서비스 제공자(제약)
- **Priority**: **Should**

---

### FR-030 에이전트 루프 실행 트레이스 기록

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall record a structured trace of each agent loop iteration — including tool calls, context snapshots, model responses, and error points — and make it accessible to developers.
  - *KO*: 시스템은 에이전트 루프 각 이터레이션의 구조화된 트레이스(도구 호출, 컨텍스트 스냅샷, 모델 응답, 에러 지점)를 기록하고 개발자가 접근할 수 있도록 해야 한다.
- **Source**: SW 개발팀 (3.2.4)
- **Priority**: **Should**

---

### FR-031 토큰 사용량·응답 지연·비용 계측 및 노출

- **Statement** *(Ubiquitous)*:
  - *EN*: The system shall measure per-request token usage, model response latency, and estimated cost, and expose these metrics in a format accessible to developers and operators.
  - *KO*: 시스템은 요청별 토큰 사용량, 모델 응답 지연 시간, 예상 비용을 측정하고, 개발자 및 운영자가 접근 가능한 형식으로 노출해야 한다.
- **Source**: SW 개발팀 (3.2.4), VD 사업부 (3.2.1)
- **Priority**: **Should**

---

### FR-032 에이전트 실행 환경 시뮬레이션 모드

- **Statement** *(Optional feature)*:
  - *EN*: Where a simulation mode is configured, the system shall execute agent tasks against a sandboxed or mock browser environment without affecting real external services or user data.
  - *KO*: 시뮬레이션 모드가 구성된 경우, 시스템은 실제 외부 서비스나 사용자 데이터에 영향을 주지 않고 샌드박스 또는 목(mock) 브라우저 환경에서 에이전트 태스크를 실행해야 한다.
- **Source**: SW 개발팀 (3.2.4)
- **Priority**: **Should**

---

### FR-033 에이전트 행동 이유 설명 (Explainability)

- **Statement** *(Event-driven)*:
  - *EN*: When the user requests an explanation of an agent action, the system shall provide a concise, human-readable rationale describing why that action was taken.
  - *KO*: 사용자가 에이전트 행동에 대한 설명을 요청하면, 시스템은 해당 행동이 왜 취해졌는지에 대한 간결하고 이해 가능한 이유를 제공해야 한다.
- **Source**: UX팀 (3.2.5), 고령 사용자 (3.3.2)
- **Priority**: **Should**

---

## 요약

| 번호 | 제목 | Source | Priority |
|------|------|--------|----------|
| FR-001 | 사용자 의도 해석 | 3.3.1 | Must |
| FR-002 | 멀티도메인 태스크 실행 | 3.3.1 | Must |
| FR-003 | End-to-end 태스크 자동 완료 | 3.3.1 | Must |
| FR-003a | Quick Action — 에이전트 루프 우회 빠른 경로 | 3.3.1 | Must |
| FR-004 | Human-in-the-Loop 확인 요청 | 3.3.1 | Must |
| FR-005 | 비가역 동작 전 명시적 확인 단계 | 3.3.2, 3.2.8 | Must |
| FR-006 | 에이전트 동작 취소 및 재시도 | 3.3.1, 3.2.5 | Must |
| FR-007 | 에이전트 진행 상황 실시간 표시 | 3.3.1 | Must |
| FR-008 | 에이전트-사용자 제어권 전환 (Co-navigation) | 3.3.1 | Should |
| FR-009 | 에이전트 백그라운드 실행 (멀티태스킹) | 3.3.1 | Should |
| FR-010 | 네트워크 끊김 시 태스크 상태 보존 및 재개 | 3.3.1 | Should |
| FR-011 | Headless 실행 모드 | 3.3.1 | Should |
| FR-012 | 로그인 및 인증 장벽 처리 (자동 또는 HITL) | 3.3.1 | Must |
| FR-013 | Headless 처리 결과를 Headed 모드로 전환 | 3.3.1 | Should |
| FR-015 | 에이전트 실패 시 폴백 경로 제공 | 3.2.5, 3.2.2 | Must |
| FR-016 | 사용자 히스토리·선호 누적 및 활용 | 3.3.1 | Should |
| FR-017 | UI 표시 옵션 설정 (텍스트 크기·대비·응답 속도) | 3.3.2 | Should |
| FR-019 | PII 데이터 처리 및 보존 기간 제어 | 3.2.8 | Must |
| FR-020 | 프롬프트 인젝션 탐지 및 방어 | 3.2.8 | Must |
| FR-021 | 개인정보 동의 수집 및 데이터 주체 권리 행사 | 3.2.8 | Must |
| FR-022 | 데이터 처리 방식 사용자 고지 (온디바이스 vs 클라우드) | 3.2.8 | Must |
| FR-023 | 시스템 전원 상태 변화에 따른 에이전트 생명주기 관리 | 3.2.3 | Must |
| FR-024 | 에이전트 오류 격리 및 상위 시스템 보호 | 3.2.2 | Must |
| FR-025 | 표준 인터페이스 기반 Skill / Sub-Agent 등록·호출 | 3.2.2 | Should |
| FR-026 | 외부 호출자 인증 및 권한 검증 | 3.2.2 | Should |
| FR-027 | 표준화된 실행 결과 반환 스키마 | 3.2.2 | Should |
| FR-028 | 에이전트 신원 식별 정보 제공 (Agent-ID) | 3.2.8 | Should |
| FR-029 | 에이전트 접근 정책 준수 (robots.txt · 이용약관) | 3.2.8 | Should |
| FR-030 | 에이전트 루프 실행 트레이스 기록 | 3.2.4 | Should |
| FR-031 | 토큰 사용량·응답 지연·비용 계측 및 노출 | 3.2.4, 3.2.1 | Should |
| FR-032 | 에이전트 실행 환경 시뮬레이션 모드 | 3.2.4 | Should |
| FR-033 | 에이전트 행동 이유 설명 (Explainability) | 3.2.5, 3.3.2 | Should |
