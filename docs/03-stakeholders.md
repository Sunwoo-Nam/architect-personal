# 3. 이해관계자 (Stakeholders)

> Tizen TV AI Web Agent (AI Browser) 설계 문서

---

## 3.1 개요 (Overview)

본 장은 **ISO/IEC/IEEE 42010:2022**(Architecture Description)와 **arc42** 템플릿의 Stakeholder 정의 방식을 하이브리드로 적용한다. 핵심 원칙은 다음 세 가지다.

1. **Stakeholder는 시스템에 대해 고유한 Concern(관심사)을 갖는 역할이다.** 동일한 사람이 여러 Stakeholder 역할을 겸할 수 있으며, 본 장은 "역할" 단위로 식별한다. 아키텍처 관점에서 Concern 축이 서로 다르면(예: 인터페이스를 *소비*하는 역할과 *공급*하는 역할) 조직상 인접하더라도 별도 Stakeholder로 분리한다.
2. **각 Concern은 하나 이상의 Architectural View를 통해 해소된다.** 따라서 Stakeholder → Concern → View 매핑은 이후 본 설계 문서에서 어떤 View를 반드시 작성해야 하는지에 대한 근거를 제공한다.
3. **Stakeholder는 시스템의 형성·수용·운영에 이해관계를 가진 역할을 포함한다.** 엔드유저와 내부 개발·운영 조직을 포함하며, 규제·법률 기관의 요구는 내부 컴플라이언스 역할(3.2.8)이 대변한다. 본 시스템과 직접 상호작용하지 않는 외부 주체(웹 콘텐츠 제공자, AI 모델 공급사, 외부 에이전트 플랫폼 등)는 Stakeholder가 아닌 **설계 제약(Constraint)**으로 취급한다(3.6 참조).

본 장에서 식별된 Stakeholder는 크게 두 범주로 분류된다.

| 범주 | 설명 | 절 |
|------|------|----|
| Samsung 내부 조직 | 본 시스템을 승인·개발·검증·배포·운영·소비할 주체 (규제 대응 창구 포함) | 3.2 |
| End User | 본 시스템의 최종 사용자 및 그 세부 그룹 | 3.3 |

본 장 마지막의 3.5 절에서는 전체 Stakeholder와 Architectural View를 통합 매핑한 표를 제공하며, 이는 이후 아키텍처 View 작성 범위를 결정하는 기준이 된다.

### 3.1.1 개정 이력 (v2 → v3)

v1(16개) → v2(9개, Concern 중첩 기준 병합)를 거쳐, v3에서는 **RACI 책임(Accountable)이 갈리거나 Concern 축이 반대인 역할을 재분할**하고 **직접 상호작용이 없는 외부 파트너를 제약으로 강등**하여 **10개 내부·사용자 역할**로 정리했다. 재분할은 v2 병합의 과도함을 교정한 것으로, 각 카드의 Concern은 분할 이전 Concern을 승계·분배한다.

| 변경 | v2 | v3 | 근거 |
|---|---|---|---|
| 분할 | 3.2.2 Tizen Platform · Web Engine Team | **3.2.2 ARGO 개발팀** / **3.2.3 웹 엔진 팀** | 호출 계약을 *소비*하는 Main Agent 소유자와 CDP 계약을 *공급*하는 엔진 소유자는 Concern 축이 반대 |
| 분할 | 3.2.4 개발 · QA 조직 | **3.2.4 SW 개발팀** / **3.2.6 품질 담당자** / **3.2.7 배포 담당자** | 구현·검증·릴리스는 Accountable 주체가 서로 다름 |
| 개명 | 3.2.3 UX · Design Team | **3.2.5 UX팀** | 명칭만 조정 |
| 개명 | 3.2.5 보안 · 프라이버시 · 컴플라이언스 | **3.2.8 보안 · 컴플라이언스팀** | 명칭만 조정, 규제 대변 역할 유지 |
| 개명 | 3.4.1 일반 시청자 | **3.3.1 일반 사용자** | End User 절 번호 이동(3.4→3.3) |
| 흡수 | 3.4.2 접근성 · 고령 사용자 | **3.3.2 고령 사용자** | 시각·청각·운동·인지 접근성 Concern을 고령 페르소나 카드에 유지·흡수 |
| 강등(삭제) | 3.3.1 웹 콘텐츠 · 서비스 제공자 | — | 본 시스템과 직접 상호작용 없음 → 3.6 Out-of-Scope / Constraint |
| 강등(삭제) | 3.3.2 외부 Agent 플랫폼 통합 개발자 | — | 상동. 표준 인터페이스 요구는 3.2.2 ARGO 개발팀이 참조 소비자로 대표 |

> **Note (추적성)**: 하위 문서(4장 제약, 5장 FR/NFR draft 등)의 구 절번호 참조는 본 개정에 맞춰 일괄 갱신한다. 갱신 전 참조는 본 표로 해석한다.

---

## 3.2 Samsung 내부 조직

### 3.2.1 VD 사업부 (Business Owner)

- **역할**: Visual Display 사업부의 사업 의사결정층. 본 과제의 전략적 투자 승인과 전사 로드맵 편입 여부(경영진 관점)부터, 실제 TV 제품 탑재 시의 범위·일정·라인업·지역별 적용(상품화 관점)까지를 결정한다.
- **주요 Concern**
  - AI Web Agent가 Tizen 플랫폼 경쟁력에 기여하는 방식과 크기, 출시 타이밍(Time-to-market)과 경쟁사 대응 포지션
  - 라인업별 HW 스펙(메모리·CPU·NPU) 차이에 따른 탑재 가능 범위 구분, 기능 공개 수준(베타/정식, 지역별 차등)과 출시 일정 조정
  - 초기 투자 및 Cloud LLM API 호출 비용(토큰 과금 구조) 대비 플랫폼·서비스·광고 수익 구조, 라인업·사용량 규모별 운영 비용 예측 가능성
  - 상품화 승인 단계별 게이트(품질·성능·법무·접근성) 통과 여부와 브랜드 리스크(AI 오동작·프라이버시·접근성 이슈의 외부 노출)
  - 타 내장 앱·서비스(Smart Hub, SmartThings 등)와의 자원·진입점·UX 경쟁 정리, 고객 문의·CS 대응 자원 확보
- **관련 Architectural View**
  - Context View (시스템 경계·외부 이해관계자 전체 조망)
  - Roadmap & Capability View (단계·라인업별 탑재 계획)
  - Deployment View (라인업별 구성 차이와 조건부 활성화)
  - Resource Budget View (라인업별 프로파일)

### 3.2.2 ARGO 개발팀 (Main Agent · 내부 소비자)

- **역할**: Tizen OS AI화(AIOS)의 Platform **Main Agent argot(argo-tizen)** 와 **Generative UI Renderer**를 소유·개발하는 조직. argot은 Bixby를 통해 사용자 질의를 수신하고, 브라우저 제어 태스크로 판단 시 본 시스템을 **Sub-Agent / Skill로 호출**하며 그 결과를 받아 사용자에게 표시한다. 본 시스템의 **가장 가까운 내부 소비자(호출자)**이자, 본 시스템이 노출하는 공개 인터페이스의 참조 소비자다. 본 AI Web Agent의 개발 주체는 아니다(그것은 3.2.4).
- **주요 Concern**
  - **호출 계약의 안정성·버저닝·역호환성** — Main Agent가 본 시스템을 호출하는 인터페이스(입력·출력 스키마, 실행 결과의 성공·실패·부분 완료 반환 형식과 에러 코드 체계)의 계약과 문서화 품질
  - 본 시스템의 실패가 상위 Main Agent / GenUI Renderer로 전파되지 않는 **격리·폴백 전략**과, argot의 컨텍스트·세션 모델을 침해하지 않는 경계
  - Main Agent, GenUI Renderer, 본 시스템이 고정된 TV 자원 예산 안에서 공존할 때의 자원·진입점 분배
  - 본 시스템이 전달하는 콘텐츠를 카드·UI로 렌더링하는 **GenUI 인터페이스 계약**의 일관성과 진행 상황 실시간 피드백 표현
  - 호출 계약을 **표준 인터페이스(MCP tool spec, Agent Protocol 등) 형태로 노출**할 때의 보안 경계·버저닝 정책 (외부 에이전트 플랫폼이 향후 동일 인터페이스로 호출할 가능성을 참조 소비자로서 대표)
- **관련 Architectural View**
  - Context View (본 시스템의 상위 소비자로서의 Main Agent · GenUI Renderer)
  - Logical View (본 시스템이 외부에 제공하는 공개 인터페이스)
  - Integration / Extension View (호출 계약·확장점)
  - External Interface View (인터페이스 계약·버저닝·인증)
  - Generative UI View (GenUI 렌더링 계약, 진행 피드백 컴포넌트)
  - Roadmap & Capability View (호출 가능한 능력의 단계적 공개)

### 3.2.3 웹 엔진 팀 (플랫폼 인터페이스 공급자)

- **역할**: Tizen의 Web Runtime·IPC 계층·시스템 서비스·리소스 관리자·앱 수명주기 관리자와 Chromium/Blink 기반 웹 엔진·WebView를 소유하는 플랫폼 기술 조직. 에이전트가 DOM을 관찰·조작하기 위한 **CDP(Chrome DevTools Protocol)·Browser Session 인터페이스의 공급자**이자 Headless 브라우징 경로의 소유자다.
- **주요 Concern**
  - **CDP · Browser Session 인터페이스 계약의 커버리지·버전 호환성** — 본 시스템이 엔진에 요구하는 관찰·조작 기능이 충분히·안정적으로 노출되는지
  - TV 시스템의 고정된 메모리·CPU 예산 내에서 **Headless 브라우징 경로의 성능·메모리 영향**과 기존 앱과의 공존 가능성
  - 에이전트 하네스가 Web Runtime·샌드박스 경계를 침범하지 않는지, 부팅·슬립·복귀 시퀀스와 하네스 생명주기가 충돌하지 않는지
  - 본 시스템 요구로 발생하는 **웹 엔진 내부 수정(패치)의 범위·유지 비용**과 Chromium 상류(upstream) 정합성, 엔진 업그레이드 시 사전 공지·호환성 검증 체계
  - DOM·접근성 트리를 AI 모델 입력 표현으로 변환할 때 엔진이 제공해야 할 데이터 경로
- **관련 Architectural View**
  - Component View (Web Engine ↔ Agent Harness 경계, CDP 계약)
  - External Interface View (CDP · Browser Session 계약)
  - Integration / Extension View (엔진 측 패치 관리·확장점)
  - Data Flow View (DOM · 접근성 트리 → AI 모델로의 표현 변환)
  - Deployment View / Process View / Resource Budget View
  - Threat Model View (샌드박스 경계·이스케이프)

### 3.2.4 SW 개발팀 (본 시스템 개발 주체)

- **역할**: 본 AI Web Agent(온디바이스 에이전트 하네스, VUI·AGT·BRW·CNT·WFL·HIL·GNUI·PRV 등 기능 영역)를 실제로 **구현·통합**하는 내부 엔지니어링 팀. 아키텍트가 정의한 설계를 코드로 실현하는 실행 주체이며, 본 시스템을 *소비*하는 3.2.2 ARGO 개발팀·*공급*하는 3.2.3 웹 엔진 팀과 명확히 구분되는 **개발 주체**다.
- **주요 Concern**
  - 기능 영역별 **독립 개발·테스트가 가능한 모듈 경계**와 Mock/Stub 지원, 새 도구·스킬·모델 추가의 용이성(유지보수성·확장성)
  - **비결정적(non-deterministic) AI 동작을 개발 루프 안에서 재현**할 수 있는 시뮬레이션·리플레이 환경
  - LLM API 호출 시 **토큰 사용량·응답 시간·비용**을 실시간 추적·최적화하는 개발자 도구와, 에이전트 루프 실행 트레이스(도구 호출 순서, 컨텍스트 스냅샷, 에러 지점) 가시성
  - 온디바이스 하네스와 클라우드 LLM 간 레이턴시·오류율을 개발 루프 안에서 즉시 확인 가능한지
  - CDP·클라우드 LLM API·호스트 호출 계약 등 외부 의존 인터페이스의 변경이 구현에 미치는 파급을 국소화하는 경계 설계
- **관련 Architectural View**
  - Component View (모듈 경계·개발 단위 격리)
  - Logical View (내부 구조·책임 배분)
  - Process View (에이전트 루프 실행 흐름과 계측 포인트)
  - Observability View (토큰·레이턴시·에러 메트릭, 로그·트레이스 설계 — 개발 관점)
  - Integration / Extension View (내부 확장점·모델 추가)

### 3.2.5 UX팀

- **역할**: 10-feet UI 가이드라인, 리모컨·음성 혼합 인터랙션, Generative UI 디자인 시스템을 소유하는 팀.
- **주요 Concern**
  - Generative UI가 생성하는 화면의 일관성·예측가능성·브랜드 정합성
  - 리모컨 포커스 모델과 음성 명령이 충돌하지 않는 입력 조율(Input Arbitration)
  - AI 응답·행동의 설명 가능성과 사용자 취소·철회 경로
  - 실패·오해 상황에서의 폴백 UX
- **관련 Architectural View**
  - Interaction View (입력·출력 모달리티 간 조율 플로우)
  - Generative UI View (동적 UI 생성 규칙과 디자인 토큰)
  - Accessibility View (포커스·대체 입력)

### 3.2.6 품질 담당자 (Quality Assurance)

- **역할**: 기능·성능·호환성 테스트, 회귀 검증, CTS 및 외부 인증(접근성·결제·음성 프라이버시 등) 대응을 담당하고, **품질 게이트 통과 여부를 판정**하는 조직. 3.2.4 SW 개발팀이 구현한 산출물을 독립적으로 검증한다.
- **주요 Concern**
  - **비결정적 AI 동작에 대한 재현 가능한 테스트 오라클**과 시나리오 재현성 — "무엇을 통과로 볼 것인가"의 판정 기준
  - 실제 웹사이트 변화에 따른 에이전트 **회귀(regression) 감지 전략**
  - 에뮬레이터·리얼 디바이스 **커버리지의 한계**와 라인업별 검증 범위
  - 인증(접근성·결제·음성 프라이버시·CTS) 요건과의 정합성 및 증거 산출
  - 토큰·레이턴시·에러율 등 품질 메트릭의 게이트 임계값 설정
- **관련 Architectural View**
  - Test Strategy View (오라클·재현성·시뮬레이션 전략)
  - Observability View (품질 메트릭·게이트 임계값)
  - Accessibility View (접근성 인증 요건)
  - Component View (테스트 단위 경계)

### 3.2.7 배포 담당자 (Release / Deployment)

- **역할**: 라인업별 탑재 구성, 단계적 롤아웃(베타/정식·지역별 차등), OTA 업데이트·버전 관리·롤백, 배포 후 현장 모니터링과 **출시 게이트 운영**을 담당하는 조직. VD 사업부의 사업적 go/no-go 결정을 실제 배포로 실행한다.
- **주요 Concern**
  - 라인업별 HW 스펙 차이에 따른 **탑재 가능 범위 구분과 조건부 활성화**(기능 플래그)
  - 단계적 롤아웃·지역별 차등 공개와 롤백 전략
  - **엔진 업그레이드·모델 버전 변경 시 배포 조율**과 호환성 창(compatibility window) 관리
  - 배포 후 현장 크래시·에러율·성능 지표 모니터링과 출시 게이트 판정
  - 라인업·사용량 규모별 운영 비용(토큰·인프라)의 배포 시점 예측
- **관련 Architectural View**
  - Deployment View (라인업별 구성·조건부 활성화·롤백)
  - Resource Budget View (라인업별 프로파일)
  - Roadmap & Capability View (단계적 롤아웃 계획)
  - Observability View (현장 모니터링 지표)

### 3.2.8 보안 · 컴플라이언스팀

- **역할**: 권한 모델, 인증·인가, 암호화, 프라이버시 정책, 위협 모델링을 주관하는 내부 조직이며, 동시에 **외부 규제·법률 기관의 요구를 설계에 대변하는 컴플라이언스 창구**다. 대변하는 규범의 범위: 개인정보 보호(EU GDPR, 한국 PIPA, 미국 CCPA/CPRA), 접근성(WCAG 2.2, ADA, EAA, 장애인차별금지법), 전자상거래·소비자 보호(자동 결제·갱신, 청약 철회, 광고·표시), 저작권·콘텐츠 이용(저작권법, DMCA, 플랫폼 약관). 규제 의무↔실현 수단의 세부 매핑은 `05-requirements/04-compliance-matrix.md`가 단일 진실원이다.
- **주요 Concern**
  - 에이전트가 제3자 사이트에 사용자 대신 로그인·결제할 때의 자격증명 보관·사용 정책과 최소 권한 원칙 준수
  - 악성 웹 페이지에 의한 프롬프트 인젝션, 샌드박스 이스케이프 등 에이전트 고유의 공격 표면
  - 음성 원본·전사·DOM 스냅샷 등 잠재적 PII의 수집 근거·경로·보존 기간과 제3자(모델 공급사·결제 대행사) 이전의 적법성, 데이터 주체 권리(열람·삭제·이전) 이행 수단
  - 온디바이스 처리와 클라우드 전송 간의 선택 기준과 사용자 고지
  - 음성·에이전트 기반 결제 동의의 명시적·사전적 요건 충족, 반복 구매·자동 구독의 고지·철회 경로
  - Generative UI를 포함한 모든 출력물의 접근성 기준(대체 텍스트·대비·포커스) 및 자동화·비자동화 경로의 동등성, AI 생성 추천·UI의 광고성 표시 의무
  - 웹 콘텐츠 요약·재가공의 공정이용 경계와 원본 출처·저작자 표기의 일관성
- **관련 Architectural View**
  - Security View (트러스트 경계, 권한 도메인)
  - Threat Model View (위협·완화 대응표)
  - Privacy & Retention View / Data Flow View (데이터 분류·보존·파기·이전)
  - Consent View (동의 상태 모델)
  - Sequence / Flow View (결제·승인·철회 플로우)
  - Accessibility View
  - Rendering Policy View / Attribution View (콘텐츠 재가공 경계와 출처 표기)

---

## 3.3 End User

End User는 단일 그룹이 아니라 사용 맥락·접근성 요구가 다른 세그먼트로 구성된다. 이들을 구분해 식별하는 이유는, 각 세그먼트의 Concern이 서로 다른 Architectural View로 해소되기 때문이다.

### 3.3.1 일반 사용자 (Mainstream Viewer)

- **역할**: TV를 주로 엔터테인먼트·정보 탐색·간편 쇼핑 목적으로 사용하는 대다수 사용자. 한 대의 TV를 복수의 가족 구성원이 공유하는 일반적인 시청 환경을 포함한다.
- **주요 Concern** *(테마별 그룹)*
  - **응답성·피드백**
    - 음성 명령이 얼마나 빠르고 정확하게 의도대로 수행되는지 — 간단한 명령(채널 변경, 날씨 조회)은 즉각 처리되고, 복잡한 태스크(구매, 예약)에서도 **태스크 복잡도에 비례한** 단계별 피드백이 끊임없이 제공되는지
    - **에이전트 동작 실시간 가시성**: 현재 어떤 단계를 수행 중인지(페이지 분석 중, 정보 입력 중, 결과 대기 중 등)를 실시간으로 확인할 수 있는지
    - **Headless 결과 수신**: 브라우저 화면을 띄우지 않아도 되는 태스크(정보 조회, 예약 확인 등)는 결과만 화면에 표시되는 방식 선호
  - **신뢰·통제**
    - 의도와 다른 동작이 발생했을 때 이해 가능한 피드백과 쉬운 취소·재시도 경로
    - **구매·예약 트랜잭션의 완전 자동 완료**("맥북 가장 싼 거 구매해줘" 수준)와, **판단·선택이 필요한 시점의 사용자 확인 요청**(선택지 제시·명확화 질문) 간의 균형
    - **에이전트 권한 범위 사용자 제어**: 구매 한도 금액, 특정 사이트 허용·차단, 자동 완료 허용 태스크 유형을 직접 설정할 수 있는지
  - **제어 전환·연속성**
    - **에이전트와 직접 조작 간의 자연스러운 제어 전환**(co-navigation): 에이전트 작업 중에도 사용자가 직접 페이지를 탐색·조작할 수 있고 전환이 충돌 없이 이루어지는지
    - **멀티태스킹**: 긴 태스크를 백그라운드에서 처리하는 동안 TV의 다른 기능(VOD 시청, 채널 변경 등)을 계속 사용할 수 있는지
    - **네트워크 불안정 대응**: Wi-Fi가 일시적으로 끊겨도 태스크가 graceful하게 처리되고 재연결 후 이어서 완료되는지
    - 리모컨 조작 습관을 해치지 않는 일관된 포커스·확인 흐름, **로그인·캡차 등 인증 장벽의 자동 처리**
  - **범용성·개인화**
    - **범용 태스크 처리 능력**: 단일 에이전트로 검색·조회·예약·구매·콘텐츠 탐색·스마트홈 제어 등 다양한 도메인 작업 수행
    - **히스토리·선호 기반 점진적 개선**: 과거 사용 패턴·요청 이력·수행 결과 피드백을 누적해 사용할수록 더 정확하고 빠르게 처리되는지
  - **가구 공유 환경**
    - 음성 화자에 따른 프로필 식별과 개인 데이터의 격리
    - 자녀 보호·연령별 콘텐츠 경계
    - 공동 공간에서의 프라이버시(음성 기록·검색 이력 노출)
- **관련 Architectural View**
  - Interaction View (음성·리모컨·Generative UI 조율, co-navigation 전환 플로우)
  - Error Recovery View (실패·모호 상황의 대응 플로우)
  - Process View (Headless vs. Headed 실행 경로 분기, 백그라운드 멀티태스킹, 네트워크 재연결 복구)
  - Sequence / Flow View (구매·예약 end-to-end 트랜잭션 플로우)
  - Personalization / Memory View (히스토리·선호 누적·활용 모델)
  - Generative UI View (에이전트 진행 상황 실시간 피드백 컴포넌트)
  - Consent View (사용자 정의 권한 범위·한도 설정 모델)
  - Identity / Profile View · Privacy & Data Flow View (프로필 격리·가구 공유 프라이버시)

### 3.3.2 고령 사용자 (Elderly · Accessibility)

- **역할**: 고령 사용자를 대표 페르소나로 하되, 시각·청각·운동·인지 영역에서 접근성 지원이 필요하거나 리모컨 포인팅·가상 키보드 입력에 특히 어려움을 겪는 사용자 집단을 포괄한다. TV가 주요(또는 유일한) 웹 접근 수단인 경우가 많다.
- **주요 Concern**
  - 천천히·단편적으로 말해도 의도를 복원하는 인식 성능과, 화면 낭독기·고대비·자막·스위치·시선 등 대체 입력과의 통합
  - 글자 크기, 대비, 음성 응답 속도의 조정 가능성
  - 결제·예약 등 되돌릴 수 없는 동작에 대한 단순하고 명확한 확인 단계
  - Generative UI가 접근성 속성을 일관되게 제공하는지, AI의 설명이 불필요하게 길거나 복잡하지 않은지
- **관련 Architectural View**
  - Accessibility View (감각·인지·운동 요구 대응)
  - Interaction View (확인·철회 패턴)
  - Generative UI Component View (접근성 기본값)

---

## 3.5 Stakeholder ↔ Architectural View 통합 매핑

본 표는 3.2~3.3에서 식별된 Stakeholder별 주요 Concern을 어느 Architectural View가 해소하는지를 통합적으로 보여준다. "●" 표식이 있는 View는 본 설계 문서 이후 장에서 **반드시 작성되어야 하는 View**이며, "○"는 해당 Stakeholder가 부차적으로 참고하는 View다. 공란은 직접 관련성이 낮음을 의미한다. 분할된 Stakeholder의 표식은 v2 병합 셀의 표식을 Concern 축에 따라 분배·승계했다.

| Architectural View | VD 사업부 | ARGO | 웹 엔진 | SW 개발 | UX | 품질 | 배포 | 보안·컴플 | 일반 | 고령 |
|---|---|---|---|---|---|---|---|---|---|---|
| Context View | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Logical View | ○ | ● | ● | ● | ● | ○ |  | ● | ○ | ○ |
| Component View |  | ○ | ● | ● | ● | ○ |  | ● |  |  |
| Deployment View | ● | ○ | ● | ○ | ○ | ○ | ● | ● |  |  |
| Process View | ○ | ○ | ● | ● |  | ○ | ○ |  | ○ |  |
| Resource Budget View | ● | ○ | ● | ○ |  |  | ● |  |  |  |
| Data Flow View | ○ | ○ | ● | ○ | ○ | ○ |  | ● | ● |  |
| Security View | ○ | ○ | ● | ● | ● | ○ |  | ● | ● | ○ |
| Threat Model View |  | ○ | ● | ● |  | ○ |  | ● | ○ |  |
| Privacy & Retention View | ○ | ○ | ○ | ● | ○ | ○ |  | ● | ● | ● |
| Interaction View | ○ | ○ |  | ● | ● | ○ |  | ● | ● | ● |
| Generative UI View | ○ | ● | ○ | ● | ● | ○ |  | ● | ● | ● |
| Accessibility View | ○ | ○ | ○ | ● | ● | ● |  | ● | ● | ● |
| Identity / Profile View | ○ | ○ |  | ○ | ○ | ○ |  | ● | ● | ● |
| Integration / Extension View | ○ | ● | ● | ● | ○ | ○ |  | ● |  |  |
| External Interface View | ○ | ● | ● | ● |  | ○ |  | ● |  |  |
| Rendering Policy View | ○ | ○ | ● | ○ | ● | ○ |  | ● | ● | ● |
| Sequence / Flow View | ○ | ○ | ○ | ● | ● | ○ |  | ● | ● | ● |
| Consent View | ○ | ○ |  | ● | ● | ○ |  | ● | ● | ● |
| Test Strategy View | ○ | ○ | ● | ● |  | ● | ○ | ● |  |  |
| Observability View | ● | ○ | ● | ● | ○ | ● | ● | ● | ○ |  |
| Personalization / Memory View | ○ | ○ |  | ● | ● | ○ |  | ● | ● | ● |
| Roadmap & Capability View | ● | ● | ● | ○ | ○ | ○ | ● | ○ |  |  |

### 3.5.1 매핑 해석 원칙

1. 한 행(View)에 ● 표식이 하나 이상 있으면, 그 View는 본 설계 문서에서 **작성이 필수**이다.
2. 한 열(Stakeholder)에 ● 표식이 하나 이상 있으면, 그 Stakeholder는 본 설계 문서의 **리뷰어 후보**이며, 해당 View들로 구성된 **Stakeholder 전용 목차(Role-Based View Package)**를 제공하는 것이 바람직하다.
3. 공란이 많은 View는 과설계의 신호일 수 있으며, 추가 근거가 제시되지 않는 한 본 설계 문서에서 생략한다.

본 통합 매핑은 이후 아키텍처 View 작성 범위를 확정하는 입력으로 사용된다.

---

## 3.6 가정과 제약 (Assumptions & Constraints)

본 장의 Stakeholder 식별은 다음 가정 위에서 이루어졌으며, 이후 장에서 이러한 가정이 변경될 경우 본 장을 재검토한다.

- **조직 구조 가정**: Samsung 내부 조직의 분류는 일반화된 역할 기반이며, 실제 조직도의 팀 명칭과 1:1 대응을 전제하지 않는다. 본 시스템을 **개발하는 주체(3.2.4 SW 개발팀)**, **소비(호출)하는 주체(3.2.2 ARGO 개발팀)**, **플랫폼 인터페이스를 공급하는 주체(3.2.3 웹 엔진 팀)**, **검증·배포하는 주체(3.2.6·3.2.7)**는 서로 분리된 역할이라는 전제를 따른다.
- **분할 granularity 가정**: v3의 재분할(3.1.1)은 RACI 책임(Accountable) 분리와 Concern 축 상충을 기준으로 한다. 재분할된 역할 간 이해가 실제로 수렴하는 것으로 판명되면(예: SW 개발팀과 품질 담당자가 단일 조직으로 운영) 카드를 다시 병합할 수 있다.
- **규제 대변 가정**: 규제·법률 기관은 본 시스템과 직접 상호작용하지 않으므로 별도 Stakeholder로 두지 않고, 내부 컴플라이언스 역할(3.2.8)이 그 요구를 대변한다. 본 시스템은 글로벌 출시를 전제로 하며, 가장 보수적인 규제(예: GDPR, EAA, PIPA)를 기준선으로 삼는다. 특정 지역 한정 출시의 경우 대변 범위가 달라질 수 있다.
- **AI 모델 소싱 가정**: 추론은 클라우드 LLM API 호출을 전제로 한다(TC-002, 별도 모델 추상화 계층 없음). 모델 공급사는 단일 벤더가 아닌 범주로 취급하며, 공급사 교체 가능성은 설계 제약으로 다룬다.
- **Out-of-Scope (제약으로 취급하는 외부 주체)**: 다음 주체는 본 시스템과 직접 상호작용하지 않거나 의사결정 이해관계가 없으므로 Stakeholder로 식별하지 않고, NFR · Deployment · Rendering Policy · External Interface View 등에서 **설계 제약(Constraint)**으로 다룬다.
  - **웹 콘텐츠 · 서비스 제공자** (에이전트가 조작·재구성하는 이커머스·예약·OTT·포털 등의 소유자) — 에이전트 접근 정책·식별 수단·메타데이터 요구는 External Interface / Rendering Policy View의 제약으로 반영한다.
  - **외부 Agent 플랫폼** (MCP 클라이언트, LangChain, AutoGen 등에서 본 시스템을 Skill / Sub-Agent로 호출하려는 제3자) — 표준 인터페이스 요구는 3.2.2 ARGO 개발팀이 참조 소비자로 대표하며, 향후 실질 이해관계가 구체화되면 Stakeholder로 승격을 재검토한다.
  - **AI 모델 제공사(LLM API 공급사), 음성 인식(ASR)·자연어 이해(NLU) 모델 공급 조직, 광고 플랫폼·콘텐츠 퍼블리셔, SoC · 하드웨어 벤더** — NFR · Deployment View의 제약으로 취급한다.

---

## 3.7 참고 자료 (References)

1. ISO/IEC/IEEE 42010:2022 — "Systems and software engineering — Architecture description". Stakeholder 및 Concern·View 개념의 근거.
2. arc42 Template v8 — Section 1.2 "Stakeholders" 및 Section 4 "Solution Strategy"의 Stakeholder 정의 방식.
3. Rozanski & Woods, *Software Systems Architecture* (2nd ed.) — Viewpoint–Stakeholder Mapping 원칙.
4. Mendelow, A. L. (1981). *Environmental Scanning — The Impact of the Stakeholder Concept*. — Power/Interest Matrix의 원전.
5. PMI, *Project Management Body of Knowledge (PMBOK®)* — RACI(Responsibility Assignment Matrix) 정의.

---

## 부록 A. Power / Interest Matrix

본 부록은 3.2~3.3에서 식별된 Stakeholder를 **영향력(Power)**과 **관심도(Interest)** 두 축으로 분류해, 각 주체에 대한 **참여·커뮤니케이션 전략**을 명시한다. 분류 기준은 Mendelow(1981)의 원형을 따른다.

- **Power (영향력)**: 본 시스템의 설계·출시·운영에 대한 의사결정권, 거부권, 또는 자원 통제력의 크기
- **Interest (관심도)**: 본 시스템의 성공·실패가 해당 주체의 업무·KPI·경험에 직접 영향을 미치는 정도

### A.1 분류 결과

| 사분면 | 권고 전략 | 해당 Stakeholder |
|---|---|---|
| High Power / High Interest | **Manage Closely** — 주요 결정에 선제적 참여, 빈번한 합의·리뷰 | VD 사업부(3.2.1), ARGO 개발팀(3.2.2), 웹 엔진 팀(3.2.3), UX팀(3.2.5) |
| High Power / Low Interest | **Keep Satisfied** — 일상 참여는 제한되나 게이트 시점에 명확한 승인·검토 요청 | 보안 · 컴플라이언스팀(3.2.8), 품질 담당자(3.2.6), 배포 담당자(3.2.7) |
| Low Power / High Interest | **Keep Informed** — 결정 이유와 변경을 투명하게 공유, 피드백 채널 유지 | SW 개발팀(3.2.4), 일반 사용자(3.3.1), 고령 사용자(3.3.2) |
| Low Power / Low Interest | **Monitor** | *(현재 해당 없음 — v2의 외부 파트너를 제약으로 강등하여 공란)* |

### A.2 2×2 매트릭스 (시각화)

```
                      High Interest
                             │
   ┌─────────────────────────┼─────────────────────────┐
   │                         │                         │
   │      Manage Closely     │     Keep Informed       │
   │                         │                         │
   │  • VD 사업부            │  • SW 개발팀            │
   │  • ARGO 개발팀          │  • 일반 사용자          │
   │  • 웹 엔진 팀           │  • 고령 사용자          │
   │  • UX팀                 │                         │
   │                         │                         │
High│                         │                         │Low
Power├─────────────────────────┼─────────────────────────┤Power
   │                         │                         │
   │      Keep Satisfied     │         Monitor         │
   │                         │                         │
   │  • 보안 · 컴플라이언스팀│  • (해당 없음)          │
   │  • 품질 담당자          │                         │
   │  • 배포 담당자          │                         │
   │                         │                         │
   └─────────────────────────┼─────────────────────────┘
                             │
                       Low Interest
```

### A.3 분류 근거 요약

- **Manage Closely** 주체들은 본 시스템의 범위·설계·출시에 직접 결정권을 행사하거나(VD 사업부), 본 시스템을 호출하는 소비자로서 인터페이스 계약의 거부권을 갖거나(ARGO 개발팀), CDP·엔진 인터페이스를 공급하며 성능·격리에 거부권을 갖는다(웹 엔진 팀). UX팀은 Generative UI라는 본 시스템 고유 산출물의 품질·브랜드 일관성에 대한 소유권을 가지므로 High Power로 취급한다.
- **Keep Satisfied** 주체들은 각자의 **게이트에서 강한 거부권**을 가지나 일상 설계 회의에 상시 참여하지 않는다. 보안 · 컴플라이언스팀은 보안·프라이버시·규제 게이트(외부 규제 거부권 대변), 품질 담당자는 품질 게이트, 배포 담당자는 출시 게이트를 운영한다. 마일스톤·게이트 시점의 **명확한 산출물·근거 제공**이 핵심 전략이다.
- **Keep Informed** 주체 중 SW 개발팀은 본 시스템의 개발 주체로서 관심도는 최상이나, 아키텍처 결정권은 아키텍트에게 있으므로 Power는 상대적으로 낮게 둔다(설계 결정이 자기 업무로 직접 흘러오므로 투명한 공유가 핵심). End User 세그먼트는 의사결정권이 없지만 경험의 최종 수신자이므로 UX팀을 통한 대리 채널이 필요하다.
- **Monitor** 사분면은 v2에서 외부 Agent 플랫폼 통합 개발자가 위치했으나, v3에서 외부 파트너를 제약(3.6)으로 강등하면서 비었다. 외부 표준 인터페이스가 정식 공개(GA)되어 외부 호출자의 실질 이해관계가 구체화되면 해당 주체를 Stakeholder로 승격하고 본 사분면에 배치한다.

### A.4 커뮤니케이션 케이던스

| 사분면 | 권장 케이던스 | 주요 채널·산출물 |
|---|---|---|
| Manage Closely | 주 단위 리뷰, 마일스톤별 의사결정 회의 | 아키텍처 결정 기록(ADR), 설계 문서 각 장 리뷰 |
| Keep Satisfied | 게이트·인증 시점 + 중대 이슈 시점 | 보안·프라이버시 심사 보고, 품질·출시 게이트 판정 근거, 규제 준수 매핑표 |
| Keep Informed | 월 단위 요약 + 릴리스 노트 | 설계 문서 공개 링크, 변경 로그, 개발 가이드 |

---

## 부록 B. RACI 매트릭스

본 부록은 본 시스템 설계·상품화 과정의 주요 활동을 정의하고, 각 활동에 대해 Stakeholder의 책임을 **R(Responsible, 실행)**, **A(Accountable, 최종 책임)**, **C(Consulted, 양방향 자문)**, **I(Informed, 일방향 통보)**로 할당한다. 활동 목록은 3.5의 Architectural View 작성 단위를 실무적 의사결정 단위로 그룹화한 것이다.

### B.1 활동 정의

| # | 활동 | 대응 Architectural View (3.5) |
|---|------|-----------------------------|
| 1 | 시스템 경계·Logical / Component Architecture 정의 | Context, Logical, Component View |
| 2 | 클라우드 LLM 연동 방식 확정 (API·프롬프트·비용 통제) | Logical, External Interface View |
| 3 | Web Engine 연동 계약(CDP · Browser Session) 확정 | Component, Integration / Extension, Data Flow View |
| 4 | 호스트(Main Agent · GenUI Renderer) 호출 계약 확정 | Logical, Integration / Extension, External Interface View |
| 5 | 리소스 예산 · 배치 결정 | Deployment, Process, Resource Budget View |
| 6 | 보안 · 위협 모델 · 권한 모델 확정 | Security, Threat Model View |
| 7 | 프라이버시 · 데이터 보존 정책 확정 | Privacy & Retention, Data Flow View |
| 8 | 동의 · 승인 플로우 확정 | Sequence / Flow, Consent, Identity / Profile View |
| 9 | Generative UI 디자인 시스템 확정 | Generative UI, Rendering Policy, Interaction View |
| 10 | 접근성 요건 확정 | Accessibility View |
| 11 | 테스트 전략 · 관측성 설계 | Test Strategy, Observability View |
| 12 | 로드맵 · 라인업별 탑재 범위 확정 | Roadmap & Capability, Deployment, Resource Budget View |
| 13 | 상품화 게이트 통과 · 출시 승인 · 롤아웃 | Deployment View (게이트·롤아웃 운영) |
| 14 | 개인화·사용자 히스토리 관리 설계 | Personalization / Memory, Identity / Profile, Privacy & Retention View |

### B.2 RACI 매트릭스

표기: **R** 실행, **A** 최종 책임, **C** 자문, **I** 통보. 공란은 해당 활동에 직접 관여하지 않음을 의미한다. Accountable은 활동당 1명을 원칙으로 한다. "아키텍트" 열은 3장 Stakeholder 목록에는 포함되지 않으나, 설계·문서화 활동의 실행 책임자(기능 역할)로서 RACI에 포함한다.

| # | 활동 | VD 사업부 | 아키텍트 | ARGO 개발 | 웹 엔진 | SW 개발 | UX팀 | 품질 | 배포 | 보안·컴플 |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 시스템 경계 · Logical / Component 정의 | I | **A**, R | C | C | R | C |  |  | C |
| 2 | 클라우드 LLM 연동 방식 | I | **A**, R |  |  | R |  |  |  | C |
| 3 | CDP · Browser Session 계약 |  | **A**, R |  | R | R |  | C |  | C |
| 4 | 호스트 호출 계약 확정 | I | **A**, R | R |  | R | C |  |  | C |
| 5 | 리소스 예산 · 배치 | C | R | C | R | C |  |  | **A**, R |  |
| 6 | 보안 · 위협 모델 · 권한 모델 | I | R |  | C | R |  | C |  | **A**, R |
| 7 | 프라이버시 · 데이터 보존 | I | R |  |  | R |  | C |  | **A**, R |
| 8 | 동의 · 승인 플로우 확정 | I | R | I |  | R | R | C |  | **A**, R |
| 9 | Generative UI 디자인 시스템 | I | R | C |  | R | **A**, R | C |  | C |
| 10 | 접근성 요건 | I | R |  |  | R | R | C |  | **A**, C |
| 11 | 테스트 전략 · 관측성 | I | C |  | C | R | C | **A**, R | C | C |
| 12 | 로드맵 · 라인업별 탑재 범위 | **A**, R | R | C | C | C | I | C | R | I |
| 13 | 상품화 게이트 · 출시 승인 · 롤아웃 | **A**(사업 go/no-go) | C | I | C | C | I | R(품질 게이트) | R(롤아웃) | R(보안 게이트) |
| 14 | 개인화·사용자 히스토리 관리 설계 | I | **A**, R | C |  | R | C | C |  | C |

### B.3 규제 기관 관여

외부 규제·법률 기관은 본 시스템과 직접 상호작용하지 않으므로 RACI 열로 포함하지 않으며, 보안 · 컴플라이언스팀(3.2.8)이 그 요구를 대변한다.

| 주체 | 주 관여 활동 | 관여 형태 |
|---|---|---|
| 규제 · 법률 기관 (3.2.8이 대변) | 활동 6~8, 10, 13, 14 | I — 준수 증거 제출 및 감사 대응 (창구: 보안 · 컴플라이언스팀) |

> **Note**: v2의 외부 파트너(웹 콘텐츠·서비스 제공자, 외부 Agent 플랫폼 통합 개발자)는 3.6에 따라 제약으로 강등되어 RACI 관여 주체에서 제외한다. 관련 접근 정책·인터페이스 요구는 각각 Rendering Policy / External Interface View의 제약과 3.2.2·3.2.3의 Concern으로 반영된다.

### B.4 RACI 운영 원칙

1. **Accountable은 중복 불가**. 활동별로 "최종 책임자"는 1명이며, 본 매트릭스의 A는 해당 Stakeholder 열 내 조직장을 의미한다. 활동 13은 사업 go/no-go의 최종 책임이 VD 사업부에 있고, 품질·보안·롤아웃의 실행 책임(R)이 각 게이트 소유자에게 분산된다.
2. **Responsible 다중 허용**. 공동 실행이 필요한 활동(예: CDP 계약, 호스트 호출 계약)은 R을 복수로 둔다.
3. **Consulted 남발 금지**. 자문 대상이 과도하면 의사결정 지연이 발생하므로, 실제 결정에 유의한 영향을 주는 주체만 C로 표기한다.
4. **RACI 변경 시 ADR 기록**. 책임 할당은 아키텍처 결정의 일부이므로, 본 매트릭스의 변경은 아키텍처 결정 기록(ADR)에 남긴다.
5. **본 부록은 3.5 매핑표와 일관성 유지**. 특정 활동의 Consulted/Responsible은 3.5에서 해당 View의 "●" Stakeholder 집합과 정합해야 하며, 편차가 발생하면 두 표를 함께 수정한다.
