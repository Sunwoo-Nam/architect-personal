# 4.A.5 Voice User Interface (VUI)

> Tizen TV AI Web Agent (AI Browser) 설계 문서 — 기능 요구사항: VUI 영역

---

## 4.A.5.1 영역 정의 및 경계

### 4.A.5.1.1 책임 범위

VUI(Voice User Interface) 영역은 본 시스템에서 **음성 모달리티의 입출력 경계**를 책임진다. 구체적으로 다음 기능을 다룬다.

- 음성 입력 트리거(리모컨 음성 키, far-field hotword, 외부 시그널)의 수신과 발화 세션 라이프사이클 관리
- 외부 ASR(Automatic Speech Recognition) 모듈로부터의 전사 결과 수신 및 본 시스템 내부 표현으로의 정규화
- 외부 NLU(Natural Language Understanding) 모듈로부터의 의도·슬롯 결과 수신 및 정규화
- 발화 단위·세션 단위 컨텍스트 유지, 화자 식별 결과 수용
- 입력 모달리티(음성·리모컨) 간 조율
- 음성 응답(TTS) 합성 요청 및 결과 스트림의 호스트 렌더러 위임
- VUI 상태 신호의 호스트 렌더러 발행
- 음성 데이터에 대한 보존 정책의 적용 책임 (정책 *내용*은 본 영역에서 정의하지 않음)

### 4.A.5.1.2 영역 외 (Out-of-Area)

다음은 VUI 영역의 책임이 아니며, 별도 영역 또는 본 설계 범위 밖이다.

| 항목 | 책임 영역 / 분류 |
|------|---------------|
| ASR · NLU · TTS 모델 자체의 설계·학습·운영 | 1.4 Out-of-Scope, 3.3.3 AI 모델 공급사 |
| ASR/NLU 모델 호출의 라우팅·선택 (클라우드 vs 온디바이스) | MDL 영역 (4.A.x) |
| 발화 의도→실행 계획 수립 및 실행 루프 | AGT 영역 (4.A.x) |
| 음성 명령으로 발생한 브라우저 직접 조작의 *세부* 액션 변환 | BRW 영역 (4.A.x) |
| 결제·민감 작업의 확인·승인 게이트 | HIL 영역 (4.A.x) |
| TTS 합성 결과의 *시각적* 렌더링 (자막·립싱크 등) | GenUI Renderer 책임, 본 시스템 외부 |

### 4.A.5.1.3 외부 의존 가정

VUI 영역의 FR은 다음 외부 인터페이스가 존재한다고 가정한다. 가정이 변경되면 본 영역 FR을 재검토한다.

1. Tizen 플랫폼은 통일된 채널로 음성 활성화 이벤트(리모컨 음성 키, hotword, 외부 시그널)를 발행한다 (3.2.4 Tizen Platform Concern).
2. ASR 모듈은 부분(partial) 결과와 최종(final) 결과를 구분 가능한 형태로 반환하며, 신뢰도(confidence)를 동반할 수 있다 (3.3.3).
3. NLU 모듈은 의도(intent) 분류 결과와 0개 이상의 슬롯(slot)을 반환하며, 신뢰도를 동반할 수 있다 (3.3.3).
4. 화자 식별(speaker identification)은 별도 컴포넌트가 신뢰도와 함께 결과를 제공한다 (3.2.8 Account/Wallet, 3.4.4 가족 공유).
5. 본 시스템은 TTS 모델을 보유하지 않으며, MDL을 통해 외부 TTS 모듈을 호출한다 (3.3.3).
6. 호스트 측 GenUI Renderer는 상태 신호를 구독해 시각 피드백을 책임진다 (3.2.4).

---

## 4.A.5.2 FR 목록

### FR-VUI-001 음성 입력 트리거 수신

- **Statement** *(Event-driven)*:
  - *EN*: When the host platform delivers a voice activation event of any supported type, the AI Web Agent shall transition the active voice session into the *Listening* state and acknowledge readiness to the host renderer.
  - *KO*: 호스트 플랫폼이 지원되는 음성 활성화 이벤트를 전달하면, AI Web Agent는 활성 음성 세션을 *Listening* 상태로 전이시키고 호스트 렌더러에 수신 준비 완료를 통지해야 한다.
- **Description**: VUI 진입의 시작점. 트리거 종류(리모컨 음성 키, far-field hotword, 외부 시그널)의 식별·정합은 4.A.5.1.3의 가정 1에 따른다. 본 FR은 *수신과 상태 전이*만 책임지며, 트리거 자체의 검출 알고리즘은 외부 책임이다.
- **Source**:
  - 시나리오: 2.2 시1, 시2, 시3, 시4 (모든 음성 시나리오의 진입)
  - Stakeholder Concern: 3.2.4 (Tizen Platform — 호출 계약 안정성), 3.2.7 (UX·Design — 입력 조율)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 가정 1에서 정의한 트리거 종류 모두에 대해 활성화 이벤트 수신이 가능하다.
  2. 수신 시 GenUI Renderer로 *Listening* 상태 전이 신호가 발행된다 (FR-VUI-011).
  3. 동일 활성화 이벤트가 중복 수신되어도 세션이 추가로 생성되지 않는다(idempotent).
- **Related Views**: Logical View, Component View, Interaction View, Sequence / Flow View, Integration / Extension View
- **Related NFR**: *TBD (NFR-LAT-*, NFR-AVL-*)*
- **Status**: Draft

---

### FR-VUI-002 ASR 전사 결과 수신 및 정규화

- **Statement** *(Event-driven)*:
  - *EN*: When the ASR module returns a transcription result for the active voice session, the AI Web Agent shall associate it with that session, preserve the partial-vs-final distinction, and forward the normalized representation to the NLU stage.
  - *KO*: ASR 모듈이 활성 음성 세션에 대한 전사 결과를 반환하면, AI Web Agent는 해당 결과를 세션과 연결하고 부분(partial)/최종(final) 구분을 보존한 정규화 표현으로 NLU 단계에 전달해야 한다.
- **Description**: ASR 모듈은 외부 공급(3.3.3)이며 본 시스템은 결과의 *수신과 정규화*만 책임진다. 부분 결과는 사용자에게 즉시 시각 피드백을 제공하기 위해 GenUI 측으로도 흘릴 수 있도록 보존한다.
- **Source**:
  - 시나리오: 2.2 시1~4
  - Stakeholder Concern: 3.3.3 (AI 모델 공급사 — 응답 스키마), 3.2.3 (아키텍트 — 인터페이스 안정성)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. partial / final 플래그가 결과 객체에 보존된다.
  2. 동일 세션의 다중 세그먼트가 도착 순서대로 결합 가능하다.
  3. 신뢰도(confidence)가 동반될 경우 정규화 표현에 포함된다.
- **Related Views**: Logical View, Component View, Data Flow View, External Interface View
- **Related NFR**: *TBD (NFR-LAT-*)*
- **Status**: Draft

---

### FR-VUI-003 NLU 의도·슬롯 결과 수신 및 정규화

- **Statement** *(Event-driven)*:
  - *EN*: When the NLU module returns an intent classification with optional slots for the active voice session, the AI Web Agent shall validate the result against the system's intent schema and forward the normalized representation to the Agent Core.
  - *KO*: NLU 모듈이 활성 음성 세션에 대한 의도 분류 결과(슬롯 포함 가능)를 반환하면, AI Web Agent는 해당 결과를 본 시스템의 의도 스키마에 대해 검증하고 정규화 표현으로 Agent Core에 전달해야 한다.
- **Description**: NLU 모델별로 출력 형식이 다를 수 있으므로 본 시스템 내부 의도 스키마(별도 정의 예정)로의 정규화는 VUI의 책임이다. 의도 스키마 외 결과는 *Unknown* 의도로 표시되며, 후속 단계에서 명확화 흐름(FR-VUI-008)으로 분기될 수 있다.
- **Source**:
  - 시나리오: 2.2 시1~4
  - Stakeholder Concern: 3.3.3 (AI 모델 공급사), 3.2.3 (아키텍트 — 인터페이스 안정성)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 의도 스키마에 정의된 의도 집합 외 결과는 *Unknown* 의도로 표시된다.
  2. 필수 슬롯이 누락된 경우 명확화 분기(FR-VUI-008)로 연결될 수 있도록 누락 슬롯 목록이 정규화 표현에 포함된다.
  3. 신뢰도가 동반될 경우 정규화 표현에 포함된다.
- **Related Views**: Logical View, External Interface View, Data Flow View
- **Related NFR**: *TBD*
- **Status**: Draft

---

### FR-VUI-004 발화 종료 및 세션 상태 전이

- **Statement** *(Event-driven)*:
  - *EN*: When endpoint detection signals end-of-utterance, when an explicit user end-of-input occurs, or when an external session-end signal is received, the AI Web Agent shall close the *Listening* state and transition the session to *Processing* or *Idle*.
  - *KO*: 종단점 검출이 발화 종료를 알리거나, 사용자가 명시적으로 입력 종료를 지시하거나, 외부에서 세션 종료 신호가 수신되면, AI Web Agent는 *Listening* 상태를 종료하고 세션을 *Processing* 또는 *Idle* 상태로 전이시켜야 한다.
- **Description**: 종료 트리거는 ① 무음 타임아웃, ② 사용자 명시적 종료(예: "그만"), ③ 외부 종료 신호 3가지로 정의한다. 본 FR은 *정상 종료*에 한정하며, 처리 단계의 중도 취소는 FR-VUI-005에서 별도 다룬다.
- **Source**:
  - 시나리오: 2.2 시1~4
  - Stakeholder Concern: 3.2.7 (UX·Design — 취소·철회 경로)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 정의된 세 종료 트리거 모두에 대해 상태 전이가 발생한다.
  2. 종료 직후 GenUI Renderer로 다음 상태 신호가 발행된다 (FR-VUI-011).
  3. 후속 처리 결과가 없는 종료(예: 사용자 명시적 취소)는 *Idle*로 직접 전이한다.
- **Related Views**: Sequence / Flow View, Interaction View, Component View
- **Related NFR**: *TBD*
- **Status**: Draft

---

### FR-VUI-005 처리 중 사용자 중단 명령 수용

- **Statement** *(Event-driven)*:
  - *EN*: When the user issues a stop or cancel command during an active voice session or its processing pipeline, the AI Web Agent shall abort in-flight ASR/NLU/TTS calls associated with the session and propagate an abort signal to the Agent Core.
  - *KO*: 활성 음성 세션 또는 그 처리 파이프라인이 진행 중일 때 사용자가 중지·취소 명령을 발화하면, AI Web Agent는 해당 세션과 연관된 진행 중 ASR/NLU/TTS 호출을 중단하고 Agent Core에 중단 신호를 전달해야 한다.
- **Description**: 시2(자동 구매), 시3(예약)의 사용자 신뢰에 직결되는 안전 장치. 본 FR은 *VUI 단계*의 중단 처리에 한정하며, 결제·예약 등 도메인 단계의 철회는 HIL 영역(4.A.x)에서 다룬다.
- **Source**:
  - 시나리오: 2.2 시2, 시3, 시4
  - Stakeholder Concern: 3.2.6 (Security — 위임된 작업의 취소), 3.4.1 (일반 시청자 — 쉬운 취소·재시도 경로)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 중단 명령 발화 후 진행 중 ASR/NLU/TTS 호출이 종결된다(외부 API 응답 폐기 포함).
  2. Agent Core에 중단 신호가 전달되어 후속 액션이 실행되지 않는다.
  3. GenUI Renderer로 *Idle* 또는 *Error* 상태 전이 신호가 발행된다.
- **Related Views**: Sequence / Flow View, Security View, Component View
- **Related NFR**: *TBD (NFR-LAT-*)*
- **Status**: Draft

---

### FR-VUI-006 다중 턴 컨텍스트 유지

- **Statement** *(Ubiquitous)*:
  - *EN*: The AI Web Agent shall preserve voice-session context across consecutive utterances within a configurable session window, so that follow-up references resolve to the prior turn(s).
  - *KO*: AI Web Agent는 설정 가능한 세션 윈도우 내에서 연속 발화 간 음성 세션 컨텍스트를 보존해, 후속 발화의 참조 표현이 직전 턴(들)을 참조하도록 해야 한다.
- **Description**: 시1의 follow-up 발화("내일은?", "부산은?")가 가장 명확한 사례. 보존 단위는 *대화 세션*이며, 종료 조건은 명시적 종료 또는 윈도우 만료다. 보존 기간·턴 수의 정량 임계는 NFR-PRV-* 및 NFR-LAT-*에서 정의한다.
- **Source**:
  - 시나리오: 2.2 시1
  - Stakeholder Concern: 3.4.1 (일반 시청자 — 의도대로 빠르게 수행), 3.4.2 (고령 사용자 — 단편 발화의 의도 복원)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 직전 N턴(N은 NFR로 정의)의 의도·슬롯·전사 결과가 후속 턴 처리 시점에 조회 가능하다.
  2. 세션 종료(명시적·만료) 시 컨텍스트가 폐기된다 (FR-VUI-012의 보존 정책 적용).
  3. 화자 식별 결과(FR-VUI-007)가 변경되면 컨텍스트가 격리되어 교차 참조가 발생하지 않는다.
- **Related Views**: Logical View, Privacy & Retention View, Identity / Profile View
- **Related NFR**: *TBD (NFR-PRV-*, NFR-LAT-*)*
- **Status**: Draft

---

### FR-VUI-007 화자 식별 결과 수신 및 프로필 연결

- **Statement** *(Event-driven)*:
  - *EN*: When the platform provides a speaker identification result with confidence for the active voice session, the AI Web Agent shall associate the session with the corresponding profile and propagate the association to downstream stages.
  - *KO*: 플랫폼이 활성 음성 세션에 대한 신뢰도 동반 화자 식별 결과를 제공하면, AI Web Agent는 해당 세션을 대응 프로필에 연결하고 그 연결을 후속 단계에 전달해야 한다.
- **Description**: 가족 공유(3.4.4) 시나리오에서 프로필 격리의 첫 단계. 식별 실패 또는 저신뢰 시의 폴백(예: *Guest* 또는 활성 프로필)은 본 FR이 정의하지 않으며, FR-VUI-013의 일반 외부 의존 손실 처리와 분리해 별도 ADR에서 결정한다.
- **Source**:
  - 시나리오: 2.2 시1~3 (계정·이력 참조 시)
  - Stakeholder Concern: 3.4.4 (가족 공유 — 화자 식별·개인 데이터 격리), 3.2.8 (Account/Wallet — 음성 화자 식별의 보완 수단)
- **Priority**: **Should**
- **Acceptance Criteria**:
  1. 화자 ID와 신뢰도가 동반되어 수신된다.
  2. 프로필 매핑 결과가 Agent Core, HIL, Privacy 적용 단계에서 조회 가능하다.
  3. 식별 결과 부재 시에도 본 FR이 후속 처리를 차단하지 않는다(FR-VUI-013과의 결합으로 폴백).
- **Related Views**: Identity / Profile View, Privacy & Retention View, Security View
- **Related NFR**: *TBD*
- **Status**: Draft

---

### FR-VUI-008 모호 의도 명확화 요청

- **Statement** *(Event-driven)*:
  - *EN*: When the NLU result has confidence below a defined threshold or required slots are missing, the AI Web Agent shall request a disambiguation prompt from the host renderer instead of forwarding the intent for execution.
  - *KO*: NLU 결과의 신뢰도가 정의된 임계치 미만이거나 필수 슬롯이 누락된 경우, AI Web Agent는 의도를 실행 단계로 전달하는 대신 호스트 렌더러에 명확화 질의를 요청해야 한다.
- **Description**: 시2~3에서 사용자 의도와 다른 자동 실행을 막기 위한 안전 게이트. 임계치(신뢰도 기준)는 NFR-QLT-*에서 정의한다. 명확화 *질문 자체의 표현*(문구·UI)은 GenUI Renderer 책임이며, 본 FR은 누락 슬롯 목록·신뢰도 등의 메타데이터 전달까지 책임진다.
- **Source**:
  - 시나리오: 2.2 시2, 시3
  - Stakeholder Concern: 3.4.2 (고령 사용자 — 의도 복원), 3.4.1 (일반 시청자 — 이해 가능한 피드백)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 신뢰도 임계치 미만 또는 필수 슬롯 누락이 검출되면 의도가 Agent Core 실행 단계로 전달되지 않는다.
  2. 누락 슬롯 목록·신뢰도 등 메타데이터가 GenUI Renderer로 전달된다.
  3. 명확화 응답(사용자 후속 발화)은 동일 세션의 후속 턴(FR-VUI-006)으로 처리된다.
- **Related Views**: Interaction View, Sequence / Flow View, Generative UI View
- **Related NFR**: *TBD (NFR-QLT-*)*
- **Status**: Draft

---

### FR-VUI-009 입력 모달리티 조율 (음성 ↔ 리모컨)

- **Statement** *(Ubiquitous)*:
  - *EN*: The AI Web Agent shall coordinate concurrent voice input and remote-control input such that the most recent user-initiated input takes precedence in modality arbitration, while the underlying browser session focus state is preserved.
  - *KO*: AI Web Agent는 음성 입력과 리모컨 입력이 동시에 발생할 때 가장 최근 사용자 입력이 모달리티 조정에서 우선되도록 조율하고, 그 과정에서 하위 브라우저 세션의 포커스 상태가 보존되도록 해야 한다.
- **Description**: 3.2.7 UX·Design 영역의 핵심 Concern인 *Input Arbitration*. 본 FR은 우선순위 정책을 *행동 수준*에서 정의하고, 충돌 검출·해결 알고리즘은 8장 이후 View에서 다룬다.
- **Source**:
  - 시나리오: 2.2 시4 (음성·리모컨 혼용 가능성), 시1~3 (음성 입력 도중 리모컨 인터랙션)
  - Stakeholder Concern: 3.2.7 (UX·Design — 리모컨 포커스와 음성 명령 충돌 방지), 3.4.1 (일관된 포커스·확인 흐름)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 음성 입력 활성 상태에서 사용자가 리모컨 입력을 수행하면 리모컨 입력의 결과가 즉시 적용된다.
  2. 리모컨 입력 활성 상태에서 사용자가 음성 입력을 시작하면 음성 입력 세션이 정상 진입한다.
  3. 모달리티 전환 시 브라우저 세션의 포커스 상태가 손실되지 않는다.
- **Related Views**: Interaction View, Component View
- **Related NFR**: *TBD*
- **Status**: Draft

---

### FR-VUI-010 음성 응답(TTS) 합성 요청 및 위임

- **Statement** *(Event-driven)*:
  - *EN*: When the Agent Core indicates that a verbal response is required, the AI Web Agent shall request TTS synthesis through the Model Abstraction Layer and forward the resulting audio stream to the host renderer for playback.
  - *KO*: Agent Core가 음성 응답이 필요함을 지시하면, AI Web Agent는 Model Abstraction Layer를 경유해 TTS 합성을 요청하고 결과 오디오 스트림을 호스트 렌더러로 전달해 재생하도록 해야 한다.
- **Description**: 본 시스템은 TTS 모델을 보유하지 않으며 MDL을 경유한다. *재생*은 호스트 렌더러 책임이고, VUI는 합성 요청·결과 전달·재생 중 사용자 발화 발생 시 중단 신호 발행까지 책임진다.
- **Source**:
  - 시나리오: 2.2 시1~3 (응답 단계)
  - Stakeholder Concern: 3.4.3 (접근성 사용자 — 음성 응답 길이 조정), 3.4.1 (일반 시청자 — 이해 가능한 피드백)
- **Priority**: **Should**
- **Acceptance Criteria**:
  1. TTS 합성 요청과 결과 스트림 수신·전달이 가능하다.
  2. 응답 재생 중 사용자 발화가 시작되면 재생 중단 신호가 호스트 렌더러로 발행된다 (FR-VUI-005와 결합).
  3. 합성 실패 시 *Error* 상태 신호가 발행된다 (FR-VUI-013과의 결합).
- **Related Views**: External Interface View, Sequence / Flow View, Accessibility View
- **Related NFR**: *TBD (NFR-LAT-*, NFR-AVL-*)*
- **Status**: Draft

---

### FR-VUI-011 VUI 상태 신호 발행

- **Statement** *(Ubiquitous)*:
  - *EN*: The AI Web Agent shall publish VUI state transitions among Idle, Listening, Processing, Responding, and Error to the host renderer so that visual feedback can be presented consistently.
  - *KO*: AI Web Agent는 VUI 상태(Idle, Listening, Processing, Responding, Error) 간 전이를 호스트 렌더러에 발행하여, 시각 피드백이 일관되게 제공될 수 있도록 해야 한다.
- **Description**: GenUI Renderer 측의 listening dot, processing 표시, 응답 중 표시 등의 시각화는 호스트 책임이며, 본 FR은 *상태 모델·전이 이벤트의 발행*까지를 책임진다.
- **Source**:
  - 시나리오: 2.2 시1~4 (모든 음성 시나리오의 사용자 피드백)
  - Stakeholder Concern: 3.2.4 (Tizen Platform — 호출 계약), 3.2.7 (UX·Design — 입력·출력 조율), 3.4.1 (일관된 포커스·확인 흐름)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 5개 상태(Idle, Listening, Processing, Responding, Error) 간 전이가 모두 발행된다.
  2. 동일 상태가 연속 발행되어도 호스트 측에서 idempotent로 처리될 수 있도록 직전 상태와의 차이를 식별 가능한 형태로 발행한다.
  3. 본 발행 채널은 FR-VUI-001, FR-VUI-004, FR-VUI-005, FR-VUI-010, FR-VUI-013과 동일한 채널을 사용한다.
- **Related Views**: Component View, Interaction View, Generative UI View, Integration / Extension View
- **Related NFR**: *TBD*
- **Status**: Draft

---

### FR-VUI-012 음성 데이터 보존 정책 적용

- **Statement** *(Ubiquitous)*:
  - *EN*: The AI Web Agent shall apply the configured retention policy to raw audio, transcriptions, NLU outputs, and session contexts originating from voice interactions so that any data exceeding the policy is discarded.
  - *KO*: AI Web Agent는 음성 인터랙션에서 발생한 원본 오디오, 전사, NLU 출력, 세션 컨텍스트에 대해 설정된 보존 정책을 적용하여, 정책을 초과한 데이터가 폐기되도록 해야 한다.
- **Description**: 정책 *내용*(보존 기간·범주별 처리)은 본 영역에서 정의하지 않는다. 7장 이후의 Privacy & Retention View와 Consent View에서 결정되며, 본 FR은 *정책 적용의 책임 소재*만 명시한다.
- **Source**:
  - 시나리오: 2.2 시 전반 (음성 데이터가 발생하는 모든 시나리오)
  - Stakeholder Concern: 3.2.6 (Security & Privacy — PII 데이터 경로·보존), 3.4.4 (가족 공유 — 음성 기록 노출 방지), 3.5 (규제 — GDPR/PIPA의 데이터 주체 권리)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. 보존 정책 변경이 후속 음성 세션부터 즉시 반영된다.
  2. 정책에 따라 폐기 대상이 되는 데이터는 후속 조회 경로에서 더 이상 노출되지 않는다.
  3. 사용자 또는 규제 요청에 따른 데이터 주체 권리(예: 삭제 요청)가 본 채널을 통해 적용 가능하다.
- **Related Views**: Privacy & Retention View, Data Flow View, Consent View
- **Related NFR**: *TBD (NFR-PRV-*)*
- **Status**: Draft

---

### FR-VUI-013 외부 ASR/NLU/TTS 가용성 손실 시 차단

- **Statement** *(Unwanted behavior)*:
  - *EN*: If the ASR, NLU, or TTS service becomes unavailable beyond a defined threshold, then the AI Web Agent shall surface an *Error* state to the host renderer and refuse to start new voice sessions or accept new voice intents until recovery is observed.
  - *KO*: ASR, NLU 또는 TTS 서비스가 정의된 임계치를 초과해 가용 불가 상태가 되면, AI Web Agent는 호스트 렌더러에 *Error* 상태를 노출하고, 복구가 관측될 때까지 신규 음성 세션 시작 또는 신규 음성 의도 수용을 거부해야 한다.
- **Description**: 클라우드 모델 의존 시 가용성 변동의 영향이 크며, 음성 모달리티의 부분적 동작은 사용자 혼란을 유발한다. 본 FR은 *검출과 차단*까지 책임지며, 폴백 정책(예: 온디바이스 백업, 다른 모델로의 라우팅)은 MDL 영역과 ADR에서 결정한다.
- **Source**:
  - 시나리오: 2.2 시 전반
  - Stakeholder Concern: 3.3.3 (AI 모델 공급사 — SLA), 3.2.4 (Tizen Platform — 본 시스템 실패의 격리), 3.4.1 (일반 시청자 — 이해 가능한 피드백)
- **Priority**: **Must**
- **Acceptance Criteria**:
  1. ASR/NLU/TTS 호출 실패율이 임계치(NFR-AVL-*)를 초과한 시점이 검출된다.
  2. *Error* 상태 신호가 호스트 렌더러로 발행된다 (FR-VUI-011).
  3. 복구 관측 시점까지 신규 음성 세션 시작 또는 신규 의도 수용이 차단된다.
- **Related Views**: Component View, Process View, Observability View
- **Related NFR**: *TBD (NFR-AVL-*)*
- **Status**: Draft

---

## 4.A.5.3 영역 단위 추적성 표

| FR ID | 시나리오 | Stakeholder Concern | Priority | Related Views (요약) |
|-------|---------|--------------------|----------|--------------------|
| FR-VUI-001 | 시1, 시2, 시3, 시4 | 3.2.4, 3.2.7 | Must | Logical, Component, Interaction, Sequence, Integration |
| FR-VUI-002 | 시1~4 | 3.3.3, 3.2.3 | Must | Logical, Component, Data Flow, External Interface |
| FR-VUI-003 | 시1~4 | 3.3.3, 3.2.3 | Must | Logical, External Interface, Data Flow |
| FR-VUI-004 | 시1~4 | 3.2.7 | Must | Sequence, Interaction, Component |
| FR-VUI-005 | 시2, 시3, 시4 | 3.2.6, 3.4.1 | Must | Sequence, Security, Component |
| FR-VUI-006 | 시1 | 3.4.1, 3.4.2 | Must | Logical, Privacy & Retention, Identity / Profile |
| FR-VUI-007 | 시1~3 | 3.4.4, 3.2.8 | Should | Identity / Profile, Privacy & Retention, Security |
| FR-VUI-008 | 시2, 시3 | 3.4.2, 3.4.1 | Must | Interaction, Sequence, Generative UI |
| FR-VUI-009 | 시1~4 | 3.2.7, 3.4.1 | Must | Interaction, Component |
| FR-VUI-010 | 시1~3 | 3.4.3, 3.4.1 | Should | External Interface, Sequence, Accessibility |
| FR-VUI-011 | 시1~4 | 3.2.4, 3.2.7, 3.4.1 | Must | Component, Interaction, Generative UI, Integration |
| FR-VUI-012 | 시 전반 | 3.2.6, 3.4.4, 3.5 | Must | Privacy & Retention, Data Flow, Consent |
| FR-VUI-013 | 시 전반 | 3.3.3, 3.2.4, 3.4.1 | Must | Component, Process, Observability |

---

## 4.A.5.4 미해결 항목 (Open Questions)

본 영역 작성 과정에서 식별된, **다른 영역 또는 후속 장에서 결정해야 할 항목**을 기록한다.

| # | 항목 | 결정 위치 (예정) |
|---|------|----------------|
| Q1 | NLU 의도 스키마(본 시스템 내부 의도 집합)의 정의 | AGT 영역 또는 8장 Logical View |
| Q2 | 화자 식별 저신뢰 시 폴백 프로필(*Guest* 등) 정책 | HIL 영역 또는 7장 Solution Strategy ADR |
| Q3 | NLU 신뢰도 임계치(FR-VUI-008), 보존 기간(FR-VUI-012), 가용성 임계치(FR-VUI-013)의 수치 | 본 4장 NFR 사이클 |
| Q4 | 시4 직접 조작 명령(스크롤·뒤로가기 등)의 처리 경로 결정 | 7장 Solution Strategy ADR (별도 추적) |
| Q5 | 다국어/지역별 발화 처리 범위와 라인업·지역 차등 | 12번 활동(Roadmap·라인업별 탑재 범위), Optional feature 패턴 도입 검토 |

---

## 4.A.5.5 참고 자료 (References)

본 영역의 표현·표기 표준은 `00-overview.md`(4장 전체 개요)의 4.4와 `01-functional/00-overview.md`(FR 작성 규약)의 4.A.6을 따른다.
