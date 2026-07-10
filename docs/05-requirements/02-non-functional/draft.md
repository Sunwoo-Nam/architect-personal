# Non-Functional Requirements — Discovery Draft

> Area 코드 미배정 초안. Stakeholder Concern 기반 전수 도출.
> 각 NFR은 SEI 스타일(Stimulus / Response / Measure)로 정량화한다.
> 수치 근거가 불충분한 항목은 `TBD (ADR-XXX)`로 표시한다.
> **Category** 필드는 Quality Attribute 전체 명칭(Latency / Performance, Resource Efficiency, Availability, Security, Privacy, Quality / Accuracy, Compatibility, Accessibility, Maintainability, Testability, Observability)을 사용한다.
> **Measure Basis** 필드는 각 수치의 도출 근거를 별도로 기록한다(작성 예정).

> **정비 이력**: FR draft 정비에 연동하여 NFR을 정리했다 — NFR-016(프로필 격리)은 멀티프로필 범위 제외로 삭제(결번 유지), NFR-012는 권한 최소화 품질을 흡수, NFR-006·011·029는 제약 TC-004(라인업별 HW)에 앵커링, NFR-022·023·024·026은 대응 FR 없이 자립(접근성·호환성·테스트 가능성 품질). **NFR-013(자격증명 저장 암호화) 삭제** — 자격증명 저장·암호화는 플랫폼/브라우저 소관(에이전트 범위 외)이며, 모델 비유출은 NFR-018이 보증(결번 유지). 상세 이력은 git 참조.

---

### NFR-001 간단한 명령의 End-to-end 응답 시간

- **Category**: Latency / Performance
- **Statement**: 채널 변경·날씨 조회 등 단순 명령은 발화 종료 후 결과 표시까지의 시간이 일정 이내여야 한다. 단순 명령은 가능한 경우 에이전트 루프를 우회하는 Quick Action 경로로 처리되며, 본 NFR은 해당 경로 대상이다.
- **Stimulus**: 사용자가 단순 명령(Quick Action 후보) 발화를 종료함
- **Response**: 시스템이 결과를 표시하거나 동작을 완료함
- **Measure**: 1초 이내 (정상 네트워크 조건)
- **Measure Basis**: VD 사업부(상품화) 요구사항
- **Rationale**: 단순 명령은 기존 TV 음성 어시스턴트의 사용성을 후퇴시키지 않아야 한다. Quick Action 경로는 에이전트 루프 비용 없이 단순 의도를 처리하므로 동등 이상 응답성을 보장하고, 에이전트 루프 경로는 LLM 호출 비용을 감안해 별도 완화 한도를 부여.
- **Related FR**: `FR-001`, `FR-007`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-002 복잡한 태스크 첫 피드백 시간

- **Category**: Latency / Performance
- **Statement**: 구매·예약 등 복잡한 태스크의 경우, 발화 종료 후 최초의 진행 상황 피드백이 표시되기까지의 시간이 일정 이내여야 한다.
- **Stimulus**: 사용자가 복잡한 태스크 명령 발화를 종료함
- **Response**: 시스템이 첫 단계(예: "검색 중...")를 사용자에게 표시함
- **Measure**: 1초 이내
- **Measure Basis**: *(작성 예정)* (launching time 측정 필요)
- **Rationale**: 복잡한 태스크는 완료까지 수 초~수 분이 걸릴 수 있으므로, 첫 피드백 지연 시 사용자가 시스템이 명령을 받지 못했다고 오해. 첫 피드백은 "받았다"는 확인 신호로서 가능한 한 빨라야 함.
- **Related FR**: `FR-007`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-003 진행 상황 표시 업데이트 주기

- **Category**: Latency / Performance
- **Statement**: 에이전트 태스크 실행 중, 진행 상황 표시는 일정 주기 이내로 갱신되어야 한다.
- **Stimulus**: 에이전트가 태스크를 실행 중임
- **Response**: 시스템이 현재 단계 표시를 갱신함
- **Measure**: 매 step별 진행상황 업데이트
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 일정 시간 화면이 정지되면 사용자는 시스템이 멈췄다고 오해하여 취소·재시도 시도. heartbeat 표시로 "여전히 진행 중"임을 지속 전달해야 함.
- **Related FR**: `FR-007`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-004 외부 Skill 호출 인터페이스 응답 시간

- **Category**: Latency / Performance
- **Statement**: 외부 에이전트 플랫폼이 본 시스템을 Skill로 호출했을 때, 인증 및 라우팅 오버헤드는 일정 이내여야 한다.
- **Stimulus**: 외부 에이전트 플랫폼이 Skill 호출 요청을 보냄
- **Response**: 시스템이 요청을 수신·인증·라우팅하여 실제 태스크 실행을 시작함
- **Measure**: 500ms 이내
- **Measure Basis**: *(작성 예정)* (실제 동작 확인 필요)
- **Rationale**: Sub-Agent로 호출되는 시나리오에서 본 시스템의 통합 오버헤드가 호출자 측 사용자 경험을 저해해서는 안 됨.
- **Related FR**: `FR-025`, `FR-026`
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.2)
- **Status**: Draft

---

### NFR-005 슬립 복귀 후 에이전트 가용 시간

- **Category**: Latency / Performance
- **Statement**: TV가 슬립 모드에서 복귀한 후, 에이전트가 명령을 수락할 수 있는 상태에 도달하기까지의 시간이 일정 이내여야 한다.
- **Stimulus**: TV가 슬립 모드에서 복귀함
- **Response**: 에이전트 하네스가 초기화되어 명령을 수락 가능한 상태가 됨
- **Measure**: ≤ 1초 이내
- **Measure Basis**: *(작성 예정)* (launching time 측정 필요)
- **Rationale**: 사용자가 TV를 켠 직후 음성 명령을 시도하는 경우가 빈번. 복귀가 느리면 첫 명령이 무시되어 사용자에게 "동작하지 않는다"는 인식을 남김.
- **Related FR**: `FR-023`
- **Source**: Tizen Platform · Web Engine Team (3.2.2), 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-006 메모리 풋프린트 한도

- **Category**: Resource Efficiency
- **Statement**: 에이전트 하네스 + 모델 추상화 계층 + Generative UI 렌더러의 합산 **peak 메모리** 사용량이 고정 상한 이내여야 하며, 이 한도는 전 라인업(Premium/Mid/Entry)에 공통 적용된다.
- **Stimulus**: 에이전트가 일반적인 태스크를 수행 중임
- **Response**: 시스템이 메모리를 점유함
- **Measure**: 합산 peak 메모리 ≤ **50 MB** (전 라인업 공통)
- **Measure Basis**: VD 사업부(상품화) 요구사항.
- **Rationale**: 본 시스템이 Main Agent·GenUI Renderer·기존 앱과 한정된 메모리 예산 안에서 공존해야 라인업 탑재가 가능. 가장 제약이 큰 라인업에서도 동작하도록 단일 상한을 전 라인업 공통으로 적용한다.
- **Related Constraint**: `TC-004` (라인업별 HW 스펙 차이)
- **Source**: VD 사업부 (3.2.1), Tizen Platform · Web Engine Team (3.2.2)
- **Status**: Draft

---

### NFR-007 CPU 점유율 한도

- **Category**: Resource Efficiency
- **Statement**: 에이전트 동작 중 CPU 점유율은 동시 재생 중인 비디오·UI 렌더링 품질에 영향을 주지 않는 수준을 유지해야 한다.
- **Stimulus**: 에이전트가 백그라운드에서 태스크를 수행 중이며, 사용자가 동시에 VOD를 시청 중임
- **Response**: 시스템이 CPU를 점유함
- **Measure**: 백그라운드 평균 CPU 점유율 ≤ **15%** (전 라인업 공통), 영상 프레임 드롭 없음
- **Measure Basis**: *(작성 예정)* (실측 및 VD 사업부(상품화) 요구사항 확인)
- **Rationale**: 멀티태스킹 보장의 정량적 근거. 영상 프레임 드롭은 사용자가 즉각 인지하는 품질 후퇴 지표이며, 에이전트가 그 원인이 되어서는 안 됨.
- **Related FR**: `FR-009`
- **Source**: Tizen Platform · Web Engine Team (3.2.2), 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-008 세션당 토큰 비용 예산

- **Category**: Resource Efficiency
- **Statement**: 평균 사용자 세션의 LLM 토큰 사용량은 운영 비용 예측 가능성을 위해 상한선 내에 있어야 한다.
- **Stimulus**: 사용자가 1개 태스크 세션(요청 ~ 완료)을 수행함
- **Response**: 시스템이 LLM API를 호출하여 토큰을 소비함
- **Measure**: 세션 평균 토큰 사용량 ≤ **TBD tokens (ADR-XXX)**, P95 ≤ **TBD tokens**
- **Measure Basis**: TBD — 베타 테스트 사용 데이터 수집 후 비용 모델과 함께 확정.
- **Rationale**: VD 사업부(상품화)가 라인업·지역별 운영 비용을 예측 가능해야 출시 의사결정 가능. 실측 데이터 없이 추정 불가하므로 베타 단계로 위임.
- **Related FR**: `FR-031`
- **Source**: VD 사업부 (3.2.1), 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-009 에이전트 오류의 상위 시스템 영향 격리

- **Category**: Availability
- **Statement**: 에이전트 하네스에서 발생한 복구 불가능한 오류가 상위 Main Agent / GenUI Renderer의 가용성에 영향을 주지 않아야 한다.
- **Stimulus**: 에이전트 하네스에 복구 불가능한 오류(crash, 무한 루프, OOM)가 발생함
- **Response**: 시스템이 오류를 격리하고 호스트 시스템은 정상 동작을 유지함
- **Measure**: 호스트 시스템 가용성 ≥ **99.95%** (에이전트 오류 1000건 발생 시에도 유지)
- **Measure Basis**: *(작성 예정)*
- **Rationale**: Tizen Platform · Web Engine Team의 핵심 거버넌스 조건. 본 시스템 실패가 플랫폼 전체 안정성을 흔들면 라인업 탑재 자체가 불가.
- **Related FR**: `FR-024`
- **Source**: Tizen Platform · Web Engine Team (3.2.2)
- **Status**: Draft

---

### NFR-010 네트워크 끊김 후 재개 성공률

- **Category**: Availability
- **Statement**: 태스크 실행 중 네트워크가 끊겼다가 일정 시간 내 재연결되는 경우, 시스템은 태스크를 처음부터 재시작하지 않고 마지막 체크포인트에서 재개해야 한다.
- **Stimulus**: 진행 중인 태스크 중 네트워크가 끊겼다가 N초 이내 재연결됨
- **Response**: 시스템이 마지막 안정 체크포인트에서 태스크를 재개함
- **Measure**: 30초 이내 재연결 시 재개 성공률 ≥ **90%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: TV 환경의 Wi-Fi 불안정성은 일상적이며, 처음부터 다시 시도해야 하는 경험은 트랜잭션 완성도와 신뢰를 동시에 무너뜨림.
- **Related FR**: `FR-010`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-011 라인업 내 SKU 간 태스크 성공률 동등성

- **Category**: Compatibility
- **Statement**: 동일 라인업 등급 내의 서로 다른 **SKU**(Stock Keeping Unit — 같은 등급 안에서 화면 크기·세부 사양만 다른 개별 판매 모델)에서, 활성화된 에이전트 기능은 동등한 **태스크 성공률**을 보여야 한다.
- **Stimulus**: 동일 등급 라인업의 서로 다른 SKU(개별 판매 모델)에서 동일한 태스크를 수행함
- **Response**: 시스템이 태스크를 실행함
- **Measure**: 동일 등급 내 SKU 간 태스크 성공률 편차 ≤ **5%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 상품화 담당자의 라인업 관리 일관성 요구. 응답 시간은 SKU별 HW 차이로 자연히 달라질 수 있으므로 동등성 대상에서 제외하고, HW와 무관하게 동일해야 하는 **태스크 성공률**만 보증 대상으로 한다. 동일 등급 내 SKU별 성공률 분산은 CS 비용과 브랜드 리스크를 증가시킴.
- **Related Constraint**: `TC-004` (라인업별 HW 스펙 차이)
- **Source**: VD 사업부 (3.2.1)
- **Status**: Draft

---

### NFR-012 Web Runtime 샌드박스 경계 준수 및 권한 최소화 (Least Privilege)

- **Category**: Security
- **Statement**: 시스템은 Web Runtime 샌드박스 경계를 위반하지 않으며, 허가된 권한 범위를 벗어난 시스템 자원에 접근하지 않아야 한다. 또한 현재 태스크에 필요한 **최소한의 브라우저·시스템 API 권한만** 요청·행사하고, 더 이상 필요하지 않은 권한은 즉시 해제해야 한다 (Least Privilege).
- **Stimulus**: 시스템이 동작 중임 (정상 및 비정상 입력 포함), 태스크가 권한을 요구함
- **Response**: 시스템이 시스템 자원에 접근을 시도하고, 태스크에 필요한 최소 권한만 보유함
- **Measure**: 샌드박스 위반 시도 = **0건** (정적 분석 + 런타임 모니터링), Tizen Web Runtime CTS 통과, **태스크 종료 후 불필요 권한 잔존 = 0건**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 사용자가 직접 관찰할 수 없는 횡단 보안 제약. 위반 시 Tizen Platform · Web Engine Team의 거버넌스 게이트 통과 불가. 권한 최소화는 공격 표면과 오용 시 피해 범위를 동시에 축소하므로 같은 "경계 통제" 품질로 묶어 관리한다. *(권한 최소화 원칙 포함)*
- **Related Constraint**: `OC-004` (보안 게이트)
- **Source**: Tizen Platform · Web Engine Team (3.2.2), 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-014 프롬프트 인젝션 탐지율

- **Category**: Security
- **Statement**: 외부 웹 콘텐츠에 삽입된 알려진 프롬프트 인젝션 패턴에 대해 시스템은 합의된 벤치마크 기준의 탐지율 목표를 달성해야 한다.
- **Stimulus**: 에이전트가 프롬프트 인젝션 패턴이 포함된 외부 웹 콘텐츠를 처리함
- **Response**: 시스템이 해당 명령을 무시하고 사용자에게 통지함
- **Measure**: 탐지율·오탐률 수치 **TBD (ADR-XXX)**
- **Measure Basis**: TBD — 벤치마크 셋(OWASP LLM01 외) 확정 및 현실적 목표 수치 합의 후 결정.
- **Rationale**: 에이전트 특유의 공격 표면이며 Security 팀의 핵심 Concern. 현재 SOTA가 일관된 고탐지율을 달성하지 못하는 단계이므로 벤치마크 합의가 선행되어야 함.
- **Related FR**: `FR-020`
- **Source**: 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-015 외부 호출자 인증 강도

- **Category**: Security
- **Statement**: 외부 에이전트 플랫폼이 본 시스템을 호출할 때, 인증되지 않은 요청은 거부되어야 한다.
- **Stimulus**: 외부 에이전트 플랫폼이 Skill 호출 요청을 보냄
- **Response**: 시스템이 호출자를 인증하고 허가된 범위 내 요청만 처리함
- **Measure**: 인증: **OAuth 2.0 또는 mTLS**, 토큰 만료: **1시간 이내**, 무인증 요청 거부율 = **100%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 외부 호출 인터페이스가 노출되는 순간 공격 표면이 됨. 인증 없는 호출 허용은 사용자 데이터·자격증명 노출로 직결.
- **Related FR**: `FR-026`
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.2), 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-017 PII 데이터 기본 보존 기간

- **Category**: Privacy
- **Statement**: 음성 전사, DOM 스냅샷, 화면 콘텐츠 등 PII 가능성이 있는 데이터의 기본 보존 기간은 규제 요건 및 사용자 선택을 따라야 한다.
- **Stimulus**: 시스템이 PII 가능 데이터를 수집·저장함
- **Response**: 시스템이 보존 기간 만료 시 자동 삭제함
- **Measure**: 기본 보존 기간 ≤ **90일** (사용자가 단축 가능), 만료 후 자동 삭제 SLA = **24시간 이내**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 디버깅·개선 목적의 데이터 활용과 규제(GDPR Article 5(1)(e), PIPA 제21조) 파기 의무 사이의 균형 지점이 명문화되어야 함.
- **Related FR**: `FR-019`, `FR-021`
- **Source**: 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-018 클라우드 전송 데이터의 최소화

- **Category**: Privacy
- **Statement**: 클라우드 LLM으로 전송되는 사용자 데이터는 태스크 수행에 필요한 최소 정보로 한정되어야 하며, 식별된 PII는 전송 전 모두 마스킹되어야 한다.
- **Stimulus**: 시스템이 LLM API를 호출함
- **Response**: 시스템이 요청 페이로드에서 PII를 식별·마스킹하고, 자격증명을 제거한 뒤 전송함
- **Measure**: 자동 PII 탐지 도구의 표준 벤치마크 커버리지 ≥ **95%** (탐지된 항목은 100% 마스킹), 자격증명 전송 사고 = **0건**, 정기 페이로드 audit 시 미탐 PII 발견율 ≤ **1%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: GDPR 데이터 최소화 원칙. 마스킹 정책은 "탐지되면 100% 차단"이며, 잔여 리스크는 탐지 도구 커버리지와 정기 audit으로 관리. 자격증명의 모델 컨텍스트 비유출도 본 NFR이 보증한다(저장 자체는 플랫폼/브라우저 소관).
- **Related FR**: `FR-022`
- **Source**: 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-019 데이터 주체 권리 행사 응답 시간

- **Category**: Privacy
- **Statement**: 사용자가 개인정보 열람·삭제·이전을 요청한 시점부터 시스템이 처리 완료하기까지의 시간이 일정 이내여야 한다.
- **Stimulus**: 사용자가 데이터 주체 권리(열람·삭제·이전)를 행사함
- **Response**: 시스템이 해당 요청을 처리하고 완료 알림을 제공함
- **Measure**: 열람·이전 ≤ **30일**, 삭제 ≤ **7일**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 데이터 주체 권리는 규제(GDPR Article 12(3), PIPA 제35조)의 명문 요건. 시스템이 이를 지원하지 않으면 운영팀이 수동 대응으로 SLA를 맞춰야 함.
- **Related FR**: `FR-021`
- **Source**: 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-020 의도 파악 정확도

- **Category**: Quality / Accuracy
- **Statement**: 인식된 발화로부터 사용자 의도를 정확히 파악하는 비율이 일정 수준 이상이어야 한다.
- **Stimulus**: 시스템이 발화 전사를 받음
- **Response**: 시스템이 사용자 의도를 분류·해석함
- **Measure**: 표준 시나리오 벤치마크에서 의도 분류 정확도 ≥ **90%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 의도 파악 실패는 잘못된 태스크 실행으로 직결되어 사용자 신뢰의 가장 큰 변수. 음성 인식 정확도(외부 ASR 책임)와 분리해 본 시스템 책임 영역만 측정.
- **Related FR**: `FR-001`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-021 태스크 완료 성공률

- **Category**: Quality / Accuracy
- **Statement**: 표준 시나리오(검색·예약·구매 등) 벤치마크에서 에이전트의 태스크 완료 성공률이 일정 수준 이상이어야 한다.
- **Stimulus**: 사용자가 표준 시나리오에 해당하는 태스크를 요청함
- **Response**: 시스템이 태스크를 수행하여 완료함
- **Measure**: Top-100 시나리오 기준 성공률 ≥ **80%**, 부분 완료 + 성공 합산 ≥ **90%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 태스크 성공률은 본 시스템의 핵심 가치 명제(자동화)의 측정 지표. 일정 수준 미달이면 사용자가 에이전트를 우회하여 직접 조작하므로 시스템 가치가 실현되지 않음.
- **Related FR**: `FR-002`, `FR-003`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-022 WCAG 2.2 AA 준수

- **Category**: Accessibility
- **Statement**: 시스템이 출력하는 모든 UI(Generative UI 인터페이스 산출물 및 Generative Web Page 변환 결과 포함)는 WCAG 2.2 Level AA 기준을 충족해야 한다.
- **Stimulus**: 시스템이 사용자에게 UI를 출력함
- **Response**: 시스템이 출력하는 UI가 접근성 표준에 부합함
- **Measure**: WCAG 2.2 AA 자동 검사 도구(axe-core 등) 위반 = **0건** (Generative UI 인터페이스 산출물 + GWP 변환된 페이지 DOM 포함 전체 컴포넌트), 수동 검수 통과
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 글로벌 출시의 hard requirement이자 EU 접근성법·한국 장차법·ADA의 명문 요건. 자동 검사만으로는 부족하므로 수동 검수 병행.
- **Related FR**: *(설계 품질 — 직접 대응 FR 없음; 제약 CN-002 / 컴플라이언스 매트릭스 #9)*
- **Source**: 접근성 · 고령 사용자 (3.4.2), 보안 · 프라이버시 · 컴플라이언스 (3.2.5)
- **Status**: Draft

---

### NFR-023 공개 인터페이스 하위 호환성 유지 기간

- **Category**: Maintainability
- **Statement**: 본 시스템이 공개한 인터페이스(Skill 호출 API, 결과 반환 스키마 등)는 메이저 버전 변경 후에도 일정 기간 이전 버전을 지원해야 한다.
- **Stimulus**: 본 시스템이 인터페이스의 메이저 버전을 변경함
- **Response**: 시스템이 이전 메이저 버전 인터페이스 호출도 정상 처리함
- **Measure**: 이전 메이저 버전 지원 기간 ≥ **12개월** (또는 다음 메이저 릴리스 + 6개월 중 더 긴 쪽)
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 외부 Agent 플랫폼 통합 개발자가 본 시스템의 버전 변경에 따라 자기 통합 코드를 마이그레이션할 시간이 필요. 너무 짧으면 통합 자체를 포기.
- **Related FR**: *(설계 품질 — 직접 대응 FR 없음; 버전 식별자 부여는 인터페이스 FR(Skill 등록·결과 스키마)이 포함)*
- **Source**: 외부 Agent 플랫폼 통합 개발자 (3.3.2), Tizen Platform · Web Engine Team (3.2.2)
- **Status**: Draft

---

### NFR-024 모듈 결합도

- **Category**: Maintainability
- **Statement**: 주요 기능 영역(VUI, 에이전트 코어, 브라우저 제어, 모델 추상화) 간 의존성은 명시된 공개 인터페이스만을 통해 이루어져야 한다.
- **Stimulus**: 개발팀이 한 영역의 내부 구현을 변경함
- **Response**: 변경이 다른 영역의 코드를 수정하도록 강제하지 않음
- **Measure**: 정적 분석 도구(예: dependency graph) 기준 영역 간 비공개 심볼 참조 = **0건**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 개발팀의 영역별 독립 개발·테스트 요구. 결합도 높으면 한 영역 변경이 전체 회귀 테스트를 강제하여 개발 속도가 저하됨.
- **Related FR**: *(설계 품질 — 직접 대응 FR 없음; 개발 · QA 조직 Concern 3.2.4)*
- **Source**: 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-025 시뮬레이션 모드 시나리오 커버리지

- **Category**: Testability
- **Statement**: 시뮬레이션 모드에서 실행 가능한 표준 시나리오의 비율이 일정 수준 이상이어야 한다.
- **Stimulus**: QA가 자동화 회귀 테스트를 실행함
- **Response**: 시스템이 시뮬레이션 환경에서 시나리오를 결정적으로 재현함
- **Measure**: Top-100 표준 시나리오 시뮬레이션 커버리지 ≥ **80%**, 재현 결정성(동일 입력 → 동일 출력) ≥ **95%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: AI 비결정성에도 불구하고 회귀 감지를 가능하게 하려면 시뮬레이션 환경의 커버리지와 결정성이 핵심. 둘 다 갖춰져야 자동 회귀가 의미를 가짐.
- **Related FR**: `FR-032`
- **Source**: 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-026 Mock 인터페이스 적용 범위

- **Category**: Testability
- **Statement**: 각 기능 경계에 Mock/Stub 인터페이스가 제공되어 단위 테스트가 외부 의존성 없이 실행 가능해야 한다.
- **Stimulus**: 개발자가 단일 영역의 단위 테스트를 실행함
- **Response**: 테스트가 다른 영역·외부 서비스 의존 없이 격리되어 실행됨
- **Measure**: 단위 테스트 외부 의존성(네트워크·디스크·시스템 호출) 비율 = **0%**, Mock 커버리지 ≥ **95%**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 단위 테스트의 격리성은 CI 안정성과 빌드 속도에 직결. 외부 의존이 남으면 flaky test가 발생하여 신뢰가 무너짐.
- **Related FR**: *(설계 품질 — 직접 대응 FR 없음; 개발 · QA 조직 Concern 3.2.4)*
- **Source**: 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-027 에이전트 트레이스 보존 기간

- **Category**: Observability
- **Statement**: 에이전트 루프 실행 트레이스는 운영팀의 사후 분석이 가능한 기간 동안 보존되어야 한다.
- **Stimulus**: 시스템이 태스크를 실행하며 트레이스를 생성함
- **Response**: 트레이스가 저장되어 조회 가능함
- **Measure**: 보존 기간 ≥ **30일**, 조회 응답 P95 ≤ **5 s**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: 사용자 신고에서 사후 분석까지 일정 시간이 소요되므로, 트레이스가 그 기간 안에 사라지면 디버깅이 불가능. 조회 속도도 운영 효율성에 직결.
- **Related FR**: `FR-030`
- **Source**: 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-028 HITL 응답 후 태스크 재개 시간

- **Category**: Latency / Performance
- **Statement**: HITL 확인을 요청받은 사용자가 응답(승인 또는 거절)을 입력한 시점부터, 시스템이 다음 동작을 시작하기까지의 시간이 일정 이내여야 한다.
- **Stimulus**: 사용자가 HITL 확인 화면에서 응답을 입력함
- **Response**: 시스템이 입력을 처리하고 다음 단계 동작을 시작함 (또는 태스크를 안전하게 중단함)
- **Measure**: P95 ≤ **500 ms**
- **Measure Basis**: *(작성 예정)*
- **Rationale**: HITL 흐름은 본질적으로 사용자 대기를 강제하는 단계이므로 응답 후 재개는 즉각적이어야 함. 지연이 길면 사용자는 자기 입력이 무시되었다고 오해.
- **Related FR**: `FR-004`, `FR-005`
- **Source**: 일반 시청자 (3.4.1), UX · Design Team (3.2.3)
- **Status**: Draft

---

### NFR-029 영구 저장소 풋프린트 한도

- **Category**: Resource Efficiency
- **Statement**: 본 시스템이 사용하는 영구 저장소(사용자 이력, 캐시, 로컬 모델 가중치, 트레이스 등)의 누적 사용량이 고정 상한 이내여야 하며, 이 한도는 전 라인업에 공통 적용된다.
- **Stimulus**: 시스템이 일정 기간 운영되며 영구 데이터를 축적함
- **Response**: 시스템이 영구 저장소를 점유함
- **Measure**: 누적 ≤ **TBD MB (ADR-XXX)** (전 라인업 공통)
- **Measure Basis**: TBD — VD 사업부(상품화) + Tizen Platform · Web Engine Team의 전 라인업 공통 디스크 예산 합의 후 확정.
- **Rationale**: 메모리(NFR-006)와 별개로 영구 저장소도 라인업의 고정 자원. 이력·트레이스가 무제한 누적되면 다른 앱 가용성을 침해.
- **Related FR**: `FR-016`, `FR-019`, `FR-030`
- **Related Constraint**: `TC-004` (라인업별 HW 스펙 차이)
- **Source**: VD 사업부 (3.2.1), Tizen Platform · Web Engine Team (3.2.2)
- **Status**: Draft

---

### NFR-030 복잡한 태스크 End-to-end 완료 시간

- **Category**: Latency / Performance
- **Statement**: 표준 시나리오 카테고리별로 에이전트 태스크 시작부터 완료까지의 시간이 일정 이내여야 한다 (HITL 대기 시간 제외).
- **Stimulus**: 사용자가 복잡한 태스크(검색·예약·구매 등)를 요청함
- **Response**: 시스템이 태스크를 완료하고 결과를 사용자에게 표시함
- **Measure**: 시나리오 카테고리별 P95 — **TBD (ADR-XXX)**
  - 정보 조회·검색: TBD초
  - 예약: TBD초
  - 구매 트랜잭션: TBD초
- **Measure Basis**: TBD — 시나리오 카테고리 정의(NFR-021 Top-100과 연계) 후 카테고리별 한도 합의.
- **Rationale**: 첫 피드백(NFR-002) 이후 완료까지의 총 시간이 합리적 범위를 넘으면 사용자가 태스크를 포기하고 직접 조작으로 회귀. 본 시스템의 핵심 가치 명제(자동화)가 성립하려면 "기다릴 만한 시간" 안에 완료되어야 함. HITL 대기 시간은 사용자 응답에 의존하므로 측정에서 제외.
- **Related FR**: `FR-002`, `FR-003`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-031 GWP 변환 추가 지연 시간

- **Category**: Latency / Performance
- **Statement**: 페이지 로드 시 GWP 적용 판정과 (적용 시) 변환·렌더링이 추가로 야기하는 지연이 일정 이내여야 한다.
- **Stimulus**: 사용자가 URL 접근 또는 링크 이동으로 페이지를 로드함
- **Response**: 시스템이 GWP 적합성 판정을 수행하고, 적용 시 변환된 DOM을 렌더링함
- **Measure**:
  - 적합성 판정 자체의 추가 지연 P95 ≤ **300ms** (변환 미적용 시 사용자 체감 영향 최소)
  - GWP 적용 시 원본 표시 대비 첫 픽셀까지의 추가 지연 P95 ≤ **1.5초**
- **Measure Basis**: 일반적인 웹 페이지 로드 인지 지연 허용 한계(2-3초) 내에서 GWP가 유발하는 증분이 사용자가 "느려졌다"고 체감하지 않을 수준. ADR 확정 시 사용자 테스트로 보정 예정.
- **Rationale**: GWP가 페이지 로드를 눈에 띄게 지연시키면 사용자는 GWP를 끄거나 우회하려 하며, 본 기능의 가치 명제가 무너짐. 자율 트리거가 일반화될수록 이 지연이 누적 체감에 미치는 영향이 큼.
- **Related FR**: `FR-034`, `FR-035`
- **Source**: 일반 시청자 (3.4.1)
- **Status**: Draft

---

### NFR-032 GWP D-pad 포커스 응답 시간

- **Category**: Latency / Performance
- **Statement**: GWP 렌더링된 페이지에서 D-pad 방향키 입력에 대한 포커스 이동·시각 피드백이 즉각적이어야 한다.
- **Stimulus**: 사용자가 리모컨 방향키를 누름
- **Response**: 시스템이 포커스 인디케이터를 인접 요소로 이동·표시함
- **Measure**: 키 입력 ~ 포커스 인디케이터 화면 갱신까지의 시간 P95 ≤ **100ms**
- **Measure Basis**: OTT 앱 표준 포커스 응답 체감 기준. 100ms를 초과하면 사용자가 "버벅인다"고 인식.
- **Rationale**: GWP가 "OTT 앱 같은 경험"을 약속하므로, 포커스 응답성이 OTT 앱 수준에 미치지 못하면 핵심 차별점이 무너짐. 포커스 응답은 사용자 신뢰의 일차 지표.
- **Related FR**: `FR-036`
- **Source**: 일반 시청자 (3.4.1), 접근성 · 고령 사용자 (3.4.2), UX · Design Team (3.2.3)
- **Status**: Draft

---

### NFR-033 GWP 변환 콘텐츠의 시맨틱 보존 정확도

- **Category**: Quality / Accuracy
- **Statement**: GWP 변환된 페이지에서 원본 페이지의 핵심 콘텐츠·액션이 누락·왜곡 없이 보존되어야 한다.
- **Stimulus**: GWP 서비스가 페이지를 변환함
- **Response**: 변환된 DOM이 원본의 핵심 정보·액션 가능 요소를 포함함
- **Measure**: 표준 페이지 코퍼스(쇼핑·뉴스·예약 도메인 각 N개)에 대해
  - 핵심 콘텐츠 보존율 ≥ **TBD%** (수동 평가)
  - 핵심 액션 가능 요소(폼·결제 버튼·필수 링크) 보존율 = **100%** (누락 시 회귀로 간주)
- **Measure Basis**: 액션 가능 요소 누락은 사용자 태스크 실패로 직결되므로 무관용. 콘텐츠 보존율 임계값은 코퍼스 정의 후 결정.
- **Rationale**: 변환 과정에서 결제 버튼이 사라지거나 핵심 정보가 누락되면 GWP는 가치 마이너스 기능이 됨. 시맨틱 보존은 GWP의 안전성 baseline.
- **Related FR**: `FR-035`
- **Source**: 일반 시청자 (3.4.1), 개발 · QA 조직 (3.2.4)
- **Status**: Draft

---

### NFR-034 사용자 개입 시 에이전트 동작 중지 시간

- **Category**: Latency / Performance
- **Statement**: 사용자가 실행 중인 에이전트 동작에 대해 취소 또는 제어권 회수(직접 조작 개입)를 입력한 시점부터, 시스템이 진행 중 동작을 중지하고 추가적인 비가역 외부 부작용을 시작하지 않기까지의 시간이 일정 이내여야 한다.
- **Stimulus**: 에이전트가 태스크를 실행하는 중 사용자가 취소 또는 직접 조작 개입(제어권 회수)을 입력함
- **Response**: 시스템이 진행 중 동작을 즉시 중지하고, 새로운 비가역 외부 부작용을 시작하지 않으며, 제어권을 사용자에게 이양하거나 안전 상태로 복귀함
- **Measure**: 개입 입력 ~ 동작 중지(추가 비가역 부작용 차단)까지 P95 ≤ **TBD ms (ADR-XXX)**
- **Measure Basis**: TBD — barge-in 지각 한계 기준 사용자 테스트로 확정. (참고: 인지적 즉시성 한계는 일반적으로 100~300ms)
- **Rationale**: 사용자가 멈추라고 했는데 에이전트가 계속 진행하면 통제감·신뢰가 무너지며, 특히 비가역 동작(FR-005) 직전 개입에서는 안전 문제로 직결된다. 이미 전송된 외부 요청은 되돌릴 수 없을 수 있으므로, 본 NFR은 "새로운 비가역 부작용을 더 시작하지 않을 때까지"를 중지 경계로 정의한다.
- **Related FR**: `FR-006`, `FR-008` (FR-005 비가역 동작과 연계)
- **Source**: 일반 시청자 (3.4.1), UX · Design Team (3.2.3)
- **Status**: Draft

---

## 요약

| 번호 | 제목 | Category | Related FR | Source |
|------|------|----------|------------|--------|
| NFR-001 | 간단한 명령의 End-to-end 응답 시간 | Latency / Performance | FR-001, FR-007 | 3.4.1 |
| NFR-002 | 복잡한 태스크 첫 피드백 시간 | Latency / Performance | FR-007 | 3.4.1 |
| NFR-003 | 진행 상황 표시 업데이트 주기 | Latency / Performance | FR-007 | 3.4.1 |
| NFR-004 | 외부 Skill 호출 인터페이스 응답 시간 | Latency / Performance | FR-025, FR-026 | 3.3.2 |
| NFR-005 | 슬립 복귀 후 에이전트 가용 시간 | Latency / Performance | FR-023 | 3.2.2, 3.4.1 |
| NFR-006 | 메모리 풋프린트 한도 (peak ≤ 50MB, 전 라인업 공통) | Resource Efficiency | TC-004 | 3.2.1, 3.2.2 |
| NFR-007 | CPU 점유율 한도 | Resource Efficiency | FR-009 | 3.2.2, 3.4.1 |
| NFR-008 | 세션당 토큰 비용 예산 | Resource Efficiency | FR-031 | 3.2.1, 3.2.4 |
| NFR-009 | 에이전트 오류의 상위 시스템 영향 격리 | Availability | FR-024 | 3.2.2 |
| NFR-010 | 네트워크 끊김 후 재개 성공률 | Availability | FR-010 | 3.4.1 |
| NFR-011 | 라인업 내 SKU 간 태스크 성공률 동등성 | Compatibility | TC-004 | 3.2.1 |
| NFR-012 | Web Runtime 샌드박스 경계 준수 및 권한 최소화 | Security | OC-004 | 3.2.2, 3.2.5 |
| NFR-014 | 프롬프트 인젝션 탐지율 | Security | FR-020 | 3.2.5 |
| NFR-015 | 외부 호출자 인증 강도 | Security | FR-026 | 3.3.2, 3.2.5 |
| NFR-017 | PII 데이터 기본 보존 기간 | Privacy | FR-019, FR-021 | 3.2.5 |
| NFR-018 | 클라우드 전송 데이터의 최소화 | Privacy | FR-022 | 3.2.5 |
| NFR-019 | 데이터 주체 권리 행사 응답 시간 | Privacy | FR-021 | 3.2.5 |
| NFR-020 | 의도 파악 정확도 | Quality / Accuracy | FR-001 | 3.4.1 |
| NFR-021 | 태스크 완료 성공률 | Quality / Accuracy | FR-002, FR-003 | 3.4.1 |
| NFR-022 | WCAG 2.2 AA 준수 | Accessibility | — (CN-002) | 3.4.2, 3.2.5 |
| NFR-023 | 공개 인터페이스 하위 호환성 유지 기간 | Maintainability | — | 3.3.2, 3.2.2 |
| NFR-024 | 모듈 결합도 | Maintainability | — | 3.2.4 |
| NFR-025 | 시뮬레이션 모드 시나리오 커버리지 | Testability | FR-032 | 3.2.4 |
| NFR-026 | Mock 인터페이스 적용 범위 | Testability | — | 3.2.4 |
| NFR-027 | 에이전트 트레이스 보존 기간 | Observability | FR-030 | 3.2.4 |
| NFR-028 | HITL 응답 후 태스크 재개 시간 | Latency / Performance | FR-004, FR-005 | 3.4.1, 3.2.3 |
| NFR-029 | 영구 저장소 풋프린트 한도 | Resource Efficiency | FR-016, FR-019, FR-030, TC-004 | 3.2.1, 3.2.2 |
| NFR-030 | 복잡한 태스크 End-to-end 완료 시간 | Latency / Performance | FR-002, FR-003 | 3.4.1 |
| NFR-031 | GWP 변환 추가 지연 시간 | Latency / Performance | FR-034, FR-035 | 3.4.1 |
| NFR-032 | GWP D-pad 포커스 응답 시간 | Latency / Performance | FR-036 | 3.4.1, 3.4.2, 3.2.3 |
| NFR-033 | GWP 변환 콘텐츠의 시맨틱 보존 정확도 | Quality / Accuracy | FR-035 | 3.4.1, 3.2.4 |
| NFR-034 | 사용자 개입 시 에이전트 동작 중지 시간 | Latency / Performance | FR-006, FR-008 | 3.4.1, 3.2.3 |

---

## Open Questions

다음 항목은 수치 확정을 위해 별도 결정이 필요하며, ADR로 기록되어야 한다.

1. **NFR-008 토큰 예산** — 평균/P95 세션 토큰 한도. 베타 테스트 데이터 기반 확정 필요.
2. **NFR-014 프롬프트 인젝션 탐지율** — 벤치마크 셋(OWASP LLM01 외) 확정 및 현실적 목표 수치 합의 필요.
3. **NFR-021 표준 시나리오 Top-100** — 시나리오 카탈로그 정의 필요 (시나리오 챕터에서 작성).
4. **NFR-029 영구 저장소 한도** — 전 라인업 공통 디스크 예산(MB) 값 미확정. VD 사업부(상품화) + Tizen Platform · Web Engine Team 합의 필요.
5. **NFR-030 카테고리별 완료 시간 한도** — 시나리오 카테고리 정의 후 카테고리별 P95 한도 합의 필요.
6. **AVL 등급** — 시스템 전체 가용성 SLA 등급 (99.9% vs 99.95% vs 99.99%) 미확정.
7. **NFR-034 동작 중지 시간** — 사용자 개입(취소·제어권 회수) ~ 중지까지의 P95 한도(ms). barge-in 지각 한계 기준 사용자 테스트로 확정 필요.
