# 3. 이해관계자 (Stakeholders)

> Tizen TV AI Web Agent (AI Browser) 설계 문서

---

## 3.1 개요 (Overview)

본 장은 **ISO/IEC/IEEE 42010:2022**(Architecture Description)와 **arc42** 템플릿의 Stakeholder 정의 방식을 하이브리드로 적용한다. 핵심 원칙은 다음 세 가지다.

1. **Stakeholder는 시스템에 대해 고유한 Concern(관심사)을 갖는 역할이다.** 동일한 사람이 여러 Stakeholder 역할을 겸할 수 있으며, 본 장은 "역할" 단위로 식별한다.
2. **각 Concern은 하나 이상의 Architectural View를 통해 해소된다.** 따라서 Stakeholder → Concern → View 매핑은 이후 본 설계 문서에서 어떤 View를 반드시 작성해야 하는지에 대한 근거를 제공한다.
3. **Stakeholder는 시스템 외부의 상호작용 주체만이 아니라, 시스템의 형성·수용·운영에 이해관계를 가진 모든 역할을 포함한다.** 엔드유저뿐 아니라 내부 개발 조직, 외부 파트너, 규제 기관은 물론, **본 시스템의 아키텍트 자신도 ISO 42010 관점에서 명시적 Stakeholder**로 기록한다.

본 장에서 식별된 Stakeholder는 크게 네 범주로 분류된다.

| 범주 | 설명 | 절 |
|------|------|----|
| Samsung 내부 조직 | 본 시스템을 설계·구축·운영·소비할 주체 | 3.2 |
| 외부 파트너 · 개발자 | 본 시스템이 동작하기 위해 연동이 필요한 제3자 | 3.3 |
| End User | 본 시스템의 최종 사용자 및 그 세부 그룹 | 3.4 |
| 규제 · 법률 기관 | 설계·운영 시 준수해야 할 법·제도·표준의 주체 | 3.5 |

본 장 마지막의 3.6 절에서는 전체 Stakeholder와 Architectural View를 통합 매핑한 표를 제공하며, 이는 이후 4장 이후의 아키텍처 View 작성 범위를 결정하는 기준이 된다.

---

## 3.2 Samsung 내부 조직

### 3.2.1 VD 사업부 경영진 (Executive Sponsor)

- **역할**: Visual Display 사업부 의사결정층. 본 과제의 전략적 투자 승인과 전사 로드맵 편입 여부를 결정한다.
- **주요 Concern**
  - AI Web Agent가 Tizen 플랫폼 경쟁력에 기여하는 방식과 크기
  - 출시 타이밍(Time-to-market)과 경쟁사 대응 포지션
  - 초기 투자 대비 플랫폼·서비스·광고 수익 구조에 미치는 영향
  - 브랜드 리스크(AI 오동작·프라이버시·접근성 이슈가 외부에 노출될 가능성)
- **관련 Architectural View**
  - Context View (시스템 경계·외부 이해관계자 전체 조망)
  - Roadmap & Capability View (단계별 출시 범위와 역량 확보 경로)

### 3.2.2 VD 상품화 담당자 (Product Commercialization)

- **역할**: VD 사업부 내 상품화 프로세스를 주관하고, 본 시스템이 실제 TV 제품으로 탑재될 때의 범위·일정·라인업·지역별 적용 여부를 결정한다. 경영진의 전략 방향과 팀의 기술 현실 사이를 잇는 의사결정자다.
- **주요 Concern**
  - 라인업별 HW 스펙(메모리·CPU·NPU) 차이에 따른 본 시스템의 탑재 가능 범위 구분
  - 기능 공개 수준(베타 / 정식, 지역별 차등)과 출시 일정 조정
  - 타 내장 앱·서비스(Smart Hub, SmartThings 등)와의 자원·진입점·UX 경쟁 정리
  - 고객 문의·CS 발생 가능성과 필요한 대응 자원 확보
  - 상품화 승인 단계별 게이트(품질·성능·법무·접근성) 통과 여부
- **관련 Architectural View**
  - Roadmap & Capability View (단계·라인업별 탑재 계획)
  - Deployment View (라인업별 구성 차이와 조건부 활성화)
  - Resource Budget View (라인업별 프로파일)
  - Context View (외부·인접 서비스와의 관계)

### 3.2.3 AI Web Agent 아키텍트 (Architect)

- **역할**: 본 시스템의 아키텍처를 정의·유지하고, 설계 결정의 근거·제약·트레이드오프를 문서로 관리하는 주체. 본 문서의 저자이자, 이해관계자 간 상충하는 Concern의 중재자.
- **주요 Concern**
  - 요구사항과 설계 결정 간의 추적성(Traceability), 결정의 기록과 근거 일관성
  - 모델·웹 엔진·UI 계층 간 **공개 인터페이스의 안정성과 진화 비용**
  - 기술 부채의 가시화와 장기 유지보수성
  - Stakeholder 간 상충하는 Concern(성능 vs 보안, 자율성 vs 확인 단계, 풍부한 UI vs 라인업 탑재 범위)의 조정 근거 명시
  - 본 설계 문서 자체의 품질(일관성·완결성·리뷰 가능성)
- **관련 Architectural View**
  - 전 View에 대한 총괄 책임
  - 특히 Logical View, Component View, Integration / Extension View, Roadmap & Capability View의 1차 저자

### 3.2.4 Tizen Platform Team (본 시스템의 내부 소비자)

- **역할**: Tizen의 Web Runtime, IPC 계층, 시스템 서비스, 리소스 관리자, 앱 수명주기 관리자를 소유함과 동시에, **Claw 계열 Main Agent와 Generative UI Renderer의 소유자**로서 본 AI Web Agent를 구성요소로 호출·사용한다. 즉, 본 시스템의 플랫폼 호스트이자 동시에 **가장 가까운 내부 소비자**다.
- **주요 Concern**
  - Main Agent 및 GenUI Renderer가 본 시스템을 호출할 때의 **공개 인터페이스 계약**의 안정성·버저닝·역호환성
  - TV 시스템의 고정된 메모리·CPU 예산 내에서 본 시스템이 Main Agent, GenUI Renderer, 기존 앱과 공존 가능한지
  - 에이전트 하네스가 Web Runtime과 샌드박스 경계를 침범하지 않는지
  - 부팅·슬립·복귀 시퀀스와 하네스 생명주기가 충돌하지 않는지
  - 본 시스템의 실패가 상위 Main Agent / GenUI Renderer로 전파되지 않는 격리·폴백 전략
- **관련 Architectural View**
  - Context View (본 시스템의 상위 소비자로서의 Main Agent · GenUI Renderer)
  - Logical View (본 시스템이 외부에 제공하는 공개 인터페이스)
  - Integration / Extension View (호출 계약·확장점)
  - Deployment View / Process View / Resource Budget View

### 3.2.5 Web Engine Team (CDP · Browser Session 지원)

- **역할**: Tizen에서 사용하는 Chromium/Blink 기반 웹 엔진 및 WebView를 유지·이식하는 팀. **본 AI Web Agent의 개발 주체가 아니며**, 에이전트가 브라우저 세션을 확보하고 DOM을 관찰·조작하기 위해 엔진이 노출해야 하는 인터페이스(**CDP: Chrome DevTools Protocol, Browser Session**)의 제공 및 **본 시스템 요구로 인한 엔진 수정 사항의 구현·상류 반영을 지원**한다.
- **주요 Concern**
  - 에이전트가 요구하는 **CDP 및 Browser Session 인터페이스의 커버리지·안정성·버전 호환성**
  - 본 시스템의 요구로 발생하는 웹 엔진 내부 수정(패치)의 범위와 유지 비용, Chromium 상류(upstream) 정합성
  - Headless 브라우징 경로에서의 성능·메모리 영향
  - 엔진 업그레이드 시 에이전트 측에 사전 공지·호환성 검증이 가능한 체계
- **관련 Architectural View**
  - Component View (Web Engine ↔ Agent Harness 경계, CDP 계약)
  - Integration / Extension View (엔진 측 확장·패치 관리)
  - Data Flow View (DOM·접근성 트리 → AI 모델로의 표현 변환)
  - Deployment View

### 3.2.6 Security & Privacy Team

- **역할**: 권한 모델, 인증·인가, 암호화, 프라이버시 정책, 위협 모델링을 주관하는 조직.
- **주요 Concern**
  - 에이전트가 제3자 사이트에 사용자 대신 로그인·결제할 때의 자격증명 보관·사용 정책
  - 음성 입력·DOM 스냅샷·스크린 콘텐츠 등 잠재적 PII가 포함된 데이터의 경로와 보존 기간
  - 하네스가 브라우저·시스템 API를 호출할 때의 최소 권한 원칙 준수
  - 악성 웹 페이지에 의한 프롬프트 인젝션, 샌드박스 이스케이프 등 에이전트 고유의 공격 표면
- **관련 Architectural View**
  - Security View (트러스트 경계, 권한 도메인)
  - Threat Model View (위협·완화 대응표)
  - Privacy & Data Flow View (데이터 분류, 보존, 전송 정책)

### 3.2.7 UX · Design Team

- **역할**: 10-feet UI 가이드라인, 리모컨·음성 혼합 인터랙션, Generative UI 디자인 시스템을 소유하는 팀.
- **주요 Concern**
  - Generative UI가 생성하는 화면의 일관성·예측가능성·브랜드 정합성
  - 리모컨 포커스 모델과 음성 명령이 충돌하지 않는 입력 조율(Input Arbitration)
  - AI 응답·행동의 설명 가능성과 사용자 취소·철회 경로
  - 실패·오해 상황에서의 폴백 UX
- **관련 Architectural View**
  - Interaction View (입력·출력 모달리티 간 조율 플로우)
  - Generative UI Component View (동적 UI 생성 규칙과 디자인 토큰)
  - Accessibility View (포커스·대체 입력)

### 3.2.8 Samsung Account · Samsung Wallet Team

- **역할**: 통합 계정, 프로필, 결제 토큰, 신원 검증을 제공하는 조직.
- **주요 Concern**
  - 음성·자동화로 진행되는 결제 승인 과정의 법적·UX적 유효성
  - 2FA, 생체 인증, 음성 화자 식별을 대체·보완 수단으로 사용할 수 있는 범위
  - 에이전트가 계정 위임 하에 제3자 사이트로 진입할 때의 위임 범위와 취소 가능성
- **관련 Architectural View**
  - Security View (Auth / Authorization Flow)
  - Sequence View (Voice → Purchase Consent → Payment)
  - Identity / Profile View (사용자·프로필·디바이스 식별 모델)

### 3.2.9 QA · Certification Team

- **역할**: 기능·성능·호환성 테스트, 회귀 검증, CTS 및 외부 인증 대응을 담당하는 조직.
- **주요 Concern**
  - 비결정적(non-deterministic) 동작을 본질로 하는 AI 에이전트에 대한 시나리오 재현성 확보
  - 실제 웹사이트 변화에 따른 에이전트 회귀(regression) 감지 전략
  - 에뮬레이터·리얼 디바이스 커버리지의 한계
  - 인증(접근성, 결제, 음성 프라이버시 등) 요건과의 정합성
- **관련 Architectural View**
  - Test Strategy View (오라클·재현성·시뮬레이션 전략)
  - Observability View (로그·트레이스·메트릭 설계)

---

## 3.3 외부 파트너 · 개발자

### 3.3.1 Tizen Web App 개발자

- **역할**: Tizen TV용 웹앱을 구축·배포·유지하는 사외 또는 사내 개발자. 본 시스템의 에이전트가 조작 또는 재구성하게 될 웹앱의 소유자이자 공급자.
- **주요 Concern**
  - 에이전트가 자기 앱의 DOM/접근성 트리를 어떻게 해석·조작하는지
  - Generative UI로 재구성될 때 브랜드·광고·결제 경로가 어떻게 보존되는지
  - 에이전트 친화적 메타데이터(접근성 속성, 시맨틱 태깅, manifest 확장)를 어느 수준까지 의무화할 것인지
  - 기존 앱 수익·트래픽·사용자 분석 지표에 미치는 영향
- **관련 Architectural View**
  - Integration / Extension View (개발자가 제공해야 할 메타데이터·매니페스트·확장 포인트)
  - Rendering Policy View (Generative UI 재구성의 적용 기준과 원본 전환 경로)

### 3.3.2 제3자 웹 서비스 운영자 (커머스 · 예약 · OTT · 포털)

- **역할**: Tizen TV 외부에서 운영되는 이커머스, 예약, OTT, 포털 등 웹 서비스의 운영 주체. 에이전트가 사용자의 대리인으로서 접속·조작하게 되는 대상.
- **주요 Concern**
  - 자동화 에이전트의 접근 허용·차단 정책과 식별 수단(User-Agent, Agent-ID, 서명 등)
  - 봇 기반 사기·어뷰즈 방지와의 양립 가능성
  - 전환율·장바구니 이탈 등 기존 분석 지표에 대한 영향
  - 계약·약관상 자동화 접근이 허용되는 범위의 불확실성
- **관련 Architectural View**
  - External Interface View (에이전트가 외부 서비스와 맺는 식별·준수 계약)
  - Rendering Policy View (외부 서비스 정책과의 정합)

### 3.3.3 AI 모델 제공사 (Cloud LLM / On-Device SLM)

- **역할**: 자체 또는 제3자가 제공하는 대규모 언어 모델, 소형 언어 모델, 멀티모달 모델의 공급자.
- **주요 Concern**
  - 본 시스템이 요구하는 모델 인터페이스·응답 스키마·컨텍스트 크기의 현실성
  - 응답 지연·가용성·비용에 대한 SLA
  - 모델 업데이트 주기 및 사전 고지 정책
  - 사용자 데이터의 학습 활용 여부와 계약상 범위
- **관련 Architectural View**
  - Logical View (Model Abstraction Layer)
  - External Interface View (SLA·스키마·버저닝)

### 3.3.4 SoC · 하드웨어 파트너

- **역할**: Tizen TV에 탑재되는 SoC·메모리·NPU 등을 공급하는 반도체·하드웨어 벤더.
- **주요 Concern**
  - 본 시스템의 메모리·CPU·NPU 풋프린트가 지원 가능한 SoC 세대 범위
  - 모델 추론을 위한 가속기(예: NPU) 활용 전략과 드라이버·SDK 요구사항
  - 열·전력 예산에 미치는 상시 동작의 영향
- **관련 Architectural View**
  - Deployment View (하드웨어 의존성 경계)
  - Resource Budget View (SoC 등급별 프로파일)

---

## 3.4 End User

End User는 단일 그룹이 아니라 인구·사용 맥락·접근성 요구가 다른 여러 세그먼트로 구성된다. 이들을 구분해 식별하는 이유는, 각 세그먼트의 Concern이 서로 다른 Architectural View로 해소되기 때문이다.

### 3.4.1 일반 시청자 (Mainstream Viewer)

- **역할**: TV를 주로 엔터테인먼트·정보 탐색·간편 쇼핑 목적으로 사용하는 대다수 사용자.
- **주요 Concern**
  - 음성 명령이 얼마나 빠르고 정확하게 의도대로 수행되는지
  - 의도와 다른 동작이 발생했을 때 이해 가능한 피드백과 쉬운 취소·재시도 경로
  - 원격조작 습관을 해치지 않는 일관된 포커스·확인 흐름
- **관련 Architectural View**
  - Interaction View (음성·리모컨·Generative UI 조율)
  - Error Recovery View (실패·모호 상황의 대응 플로우)

### 3.4.2 고령 사용자

- **역할**: 리모컨 포인팅·가상 키보드 입력에 특히 어려움을 겪는 사용자 집단으로, TV가 주요 웹 접근 수단인 경우가 많다.
- **주요 Concern**
  - 천천히·단편적으로 말해도 의도를 복원하는 인식 성능
  - 글자 크기, 대비, 음성 응답 속도의 조정 가능성
  - 결제·예약 등 되돌릴 수 없는 동작에 대한 단순하고 명확한 확인 단계
- **관련 Architectural View**
  - Accessibility View (감각·인지·운동 요구 대응)
  - Interaction View (확인·철회 패턴)

### 3.4.3 접근성 요구 사용자

- **역할**: 시각·청각·운동·인지 영역에서 접근성 지원이 필요한 사용자. 일부는 TV 외 대체 수단이 제한적이다.
- **주요 Concern**
  - 화면 낭독기, 고대비, 자막, 스위치·시선 등 대체 입력과의 통합
  - Generative UI가 접근성 속성을 일관되게 제공하는지
  - AI의 설명이 불필요하게 길거나 복잡하지 않은지
- **관련 Architectural View**
  - Accessibility View
  - Generative UI Component View (접근성 기본값)

### 3.4.4 가족 공유 사용자 (Multi-Profile Household)

- **역할**: 한 대의 TV를 복수의 가족 구성원이 공유하는 일반적인 시청 환경.
- **주요 Concern**
  - 음성 화자에 따른 프로필 식별과 개인 데이터의 격리
  - 자녀 보호·연령별 콘텐츠 경계
  - 공동 공간에서의 프라이버시(음성 기록·검색 이력 노출)
- **관련 Architectural View**
  - Identity / Profile View
  - Privacy & Data Flow View

---

## 3.5 규제 · 법률 기관

규제·법률 기관은 본 시스템과 직접 상호작용하지는 않지만, 설계·운영이 준수해야 할 규범의 주체다. 본 절에서는 법적 강제력을 가진 규제와 사실상 표준으로 기능하는 것을 모두 포함해 **단일 Stakeholder 군**으로 취급한다. 세부 영역 간 Concern은 서로 중첩되며, 공통적으로 다음의 Architectural View에 의해 해소된다.

- **포함 영역**
  - **개인정보 보호**: EU GDPR, 한국 PIPA, 미국 CCPA / CPRA 등 역내 개인정보 보호 규제
  - **접근성**: WCAG 2.2, 미국 ADA, 유럽 접근성법(EAA), 한국 장애인차별금지법
  - **전자상거래 · 소비자 보호**: 자동 결제·자동 갱신 규제, 청약 철회, 광고·표시 규제
  - **저작권 · 콘텐츠 이용**: 저작권법, DMCA, 플랫폼별 이용약관 상의 콘텐츠 이용 제한

- **주요 Concern**
  - 음성 원본·전사·DOM 스냅샷 등 잠재적 개인정보의 수집 근거와 동의 방식, 데이터 주체 권리(열람·삭제·이전) 이행 수단
  - 온디바이스 처리와 클라우드 전송 간의 선택 기준과 사용자 고지, 제3자(모델 공급사·결제 대행사)로의 데이터 이전의 적법성
  - Generative UI를 포함한 모든 출력물이 준수해야 할 대체 텍스트·대비·포커스 기준, 자동화 경로와 비자동화 경로의 동등성
  - 음성·에이전트 기반 결제 동의의 명시적·사전적 동의 요건 충족, 반복 구매·자동 구독의 고지·철회 경로
  - AI가 생성한 추천·UI가 광고성 표시 의무를 위반하지 않는지
  - 웹 콘텐츠의 요약·재가공이 공정이용·허용 범위를 벗어나지 않는지, 원본 출처·저작자 표기의 일관성

- **관련 Architectural View**
  - Privacy & Retention View (데이터 분류·보존·파기·이전)
  - Accessibility View
  - Sequence / Flow View (결제·승인·철회 플로우)
  - Consent View (동의 상태 모델)
  - Rendering Policy View / Attribution View (콘텐츠 재가공 경계와 출처 표기)

---

## 3.6 Stakeholder ↔ Architectural View 통합 매핑

본 표는 3.2~3.5에서 식별된 Stakeholder별 주요 Concern을 어느 Architectural View가 해소하는지를 통합적으로 보여준다. "●" 표식이 있는 View는 본 설계 문서 이후 장에서 **반드시 작성되어야 하는 View**이며, "○"는 해당 Stakeholder가 부차적으로 참고하는 View다. 공란은 직접 관련성이 낮음을 의미한다.

| Architectural View | VD 경영진 | VD 상품화 | 아키텍트 | Tizen Platform | Web Engine | Security&Privacy | UX/Design | Account/Wallet | QA/Cert. | 웹앱 개발자 | 제3자 서비스 | AI 모델 공급사 | SoC 파트너 | 일반 시청자 | 고령 사용자 | 접근성 사용자 | 가족 공유 | 규제·법률 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Context View | ● | ● | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Logical View | ○ | ○ | ● | ● | ● | ● | ● | ● | ○ | ● | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ |
| Component View |  |  | ● | ● | ● | ● | ● | ○ | ● | ● | ○ | ● | ○ |  |  |  |  |  |
| Deployment View | ○ | ● | ● | ● | ● | ● | ○ | ○ | ● | ○ |  | ○ | ● |  |  |  |  |  |
| Process View |  | ○ | ● | ● | ● |  |  |  | ● |  |  |  | ● |  |  |  |  |  |
| Resource Budget View | ○ | ● | ● | ● | ● |  |  |  | ● |  |  | ● | ● |  |  |  |  |  |
| Data Flow View |  | ○ | ● | ● | ● | ● | ○ | ● | ○ | ● | ● | ● |  |  |  |  | ● | ● |
| Security View | ○ | ○ | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ○ | ○ | ○ | ● | ● |
| Threat Model View |  |  | ● | ● | ● | ● |  |  | ● | ○ | ● | ● |  |  |  |  | ○ | ● |
| Privacy & Retention View | ○ | ○ | ● | ● | ○ | ● | ○ | ● | ● | ○ | ● | ● |  | ● | ● | ● | ● | ● |
| Interaction View | ○ | ○ | ● |  | ○ |  | ● | ● | ● | ● |  |  |  | ● | ● | ● | ● | ● |
| Generative UI View | ○ | ○ | ● |  | ● | ○ | ● | ○ | ● | ● | ○ | ○ |  | ● | ● | ● | ○ | ● |
| Accessibility View | ○ | ○ | ● |  | ○ | ○ | ● |  | ● | ● |  |  |  | ● | ● | ● |  | ● |
| Identity / Profile View | ○ | ○ | ● |  |  | ● | ○ | ● | ○ | ○ | ○ |  |  | ● | ● | ● | ● | ● |
| Integration / Extension View | ○ | ○ | ● | ● | ● | ● | ○ |  | ● | ● | ● | ● | ○ |  |  |  |  |  |
| External Interface View | ○ | ○ | ● | ● | ● | ● |  | ● | ● | ○ | ● | ● | ○ |  |  |  |  | ● |
| Rendering Policy View | ○ | ○ | ● |  | ● | ○ | ● |  | ○ | ● | ● |  |  | ● | ○ | ● |  | ● |
| Sequence / Flow View | ○ | ○ | ● | ○ | ○ | ● | ● | ● | ● | ● | ● | ● |  | ● | ● | ● | ● | ● |
| Consent View | ○ | ○ | ● | ○ |  | ● | ● | ● | ● | ○ | ○ |  |  | ● | ● | ● | ● | ● |
| Test Strategy View |  | ○ | ● | ○ | ● | ● |  |  | ● | ● | ○ | ● | ○ |  |  |  |  | ○ |
| Observability View | ○ | ● | ● | ● | ○ | ● | ○ |  | ● | ● | ○ | ● |  |  |  |  | ○ | ● |
| Roadmap & Capability View | ● | ● | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ● |  |  |  |  |  |

### 3.6.1 매핑 해석 원칙

1. 한 행(View)에 ● 표식이 하나 이상 있으면, 그 View는 본 설계 문서에서 **작성이 필수**이다.
2. 한 열(Stakeholder)에 ● 표식이 하나 이상 있으면, 그 Stakeholder는 본 설계 문서의 **리뷰어 후보**이며, 해당 View들로 구성된 **Stakeholder 전용 목차(Role-Based View Package)**를 제공하는 것이 바람직하다.
3. 공란이 많은 View는 과설계의 신호일 수 있으며, 추가 근거가 제시되지 않는 한 본 설계 문서에서 생략한다.
4. **아키텍트(3.2.3) 열은 정의상 거의 모든 View에 ●**가 찍히며, 이는 본 문서 전체의 품질·일관성에 대한 단일 책임자를 명시하는 기록이다.

본 통합 매핑은 4장 이후의 아키텍처 View 작성 범위를 확정하는 입력으로 사용된다.

---

## 3.7 가정과 제약 (Assumptions & Constraints)

본 장의 Stakeholder 식별은 다음 가정 위에서 이루어졌으며, 이후 장에서 이러한 가정이 변경될 경우 본 장을 재검토한다.

- **조직 구조 가정**: Samsung 내부 조직의 분류는 일반화된 역할 기반이며, 실제 조직도의 팀 명칭과 1:1 대응을 전제하지 않는다. 본 시스템을 **개발하는 주체**와 본 시스템을 **소비(호출)하는 주체**는 분리되어 있다는 전제를 따르며, 후자에는 Tizen Platform Team의 Claw 계열 Main Agent 및 GenUI Renderer가 포함된다.
- **AI 모델 소싱 가정**: AI 모델은 자사·제3자·온디바이스·클라우드 모두의 조합이 가능하다고 전제한다. 따라서 모델 공급사는 단일 벤더가 아닌 범주로 취급한다.
- **규제 범위 가정**: 본 시스템은 글로벌 출시를 전제로 하며, 가장 보수적인 규제(예: GDPR, EAA, PIPA)를 기준선으로 삼는다. 특정 지역 한정 출시의 경우 규제 Stakeholder 범위가 달라질 수 있다.
- **Out-of-Scope**: 광고 플랫폼·콘텐츠 퍼블리셔 등 본 과제의 설계 범위(1.4, 2.1 참조)에서 벗어난 주체는 본 장에서 식별하지 않는다. 또한 음성 인식(ASR) · 자연어 이해(NLU) 모델 자체를 공급·운영하는 별도 조직은 본 장에서 **AI 모델 제공사(3.3.3)**에 포괄한다.

---

## 3.8 참고 자료 (References)

1. ISO/IEC/IEEE 42010:2022 — "Systems and software engineering — Architecture description". Stakeholder 및 Concern·View 개념의 근거.
2. arc42 Template v8 — Section 1.2 "Stakeholders" 및 Section 4 "Solution Strategy"의 Stakeholder 정의 방식.
3. Rozanski & Woods, *Software Systems Architecture* (2nd ed.) — Viewpoint–Stakeholder Mapping 원칙.
4. Mendelow, A. L. (1981). *Environmental Scanning — The Impact of the Stakeholder Concept*. — Power/Interest Matrix의 원전.
5. PMI, *Project Management Body of Knowledge (PMBOK®)* — RACI(Responsibility Assignment Matrix) 정의.

---

## 부록 A. Power / Interest Matrix

본 부록은 3.2~3.5에서 식별된 Stakeholder를 **영향력(Power)**과 **관심도(Interest)** 두 축으로 분류해, 각 주체에 대한 **참여·커뮤니케이션 전략**을 명시한다. 분류 기준은 Mendelow(1981)의 원형을 따른다.

- **Power (영향력)**: 본 시스템의 설계·출시·운영에 대한 의사결정권, 거부권, 또는 자원 통제력의 크기
- **Interest (관심도)**: 본 시스템의 성공·실패가 해당 주체의 업무·KPI·경험에 직접 영향을 미치는 정도

### A.1 분류 결과

| 사분면 | 권고 전략 | 해당 Stakeholder |
|---|---|---|
| High Power / High Interest | **Manage Closely** — 주요 결정에 선제적 참여, 빈번한 합의·리뷰 | VD 경영진(3.2.1), VD 상품화 담당자(3.2.2), AI Web Agent 아키텍트(3.2.3), Tizen Platform Team(3.2.4), UX · Design Team(3.2.7) |
| High Power / Low Interest | **Keep Satisfied** — 일상 참여는 제한되나 이슈·게이트 시점에 명확한 승인·검토 요청 | Security & Privacy Team(3.2.6), Samsung Account · Wallet Team(3.2.8), SoC · 하드웨어 파트너(3.3.4), 규제 · 법률 기관(3.5) |
| Low Power / High Interest | **Keep Informed** — 결정 이유와 변경을 투명하게 공유, 피드백 채널 유지 | Web Engine Team(3.2.5), QA · Certification Team(3.2.9), Tizen Web App 개발자(3.3.1), AI 모델 제공사(3.3.3), End User — 일반 시청자(3.4.1), 고령(3.4.2), 접근성(3.4.3), 가족 공유(3.4.4) |
| Low Power / Low Interest | **Monitor** — 변화를 주기적으로 점검하되, 요청이 없으면 능동 개입 최소화 | 제3자 웹 서비스 운영자(3.3.2) |

### A.2 2×2 매트릭스 (시각화)

```
                      High Interest
                             │
   ┌─────────────────────────┼─────────────────────────┐
   │                         │                         │
   │      Manage Closely     │     Keep Informed       │
   │                         │                         │
   │  • VD 경영진            │  • Web Engine Team      │
   │  • VD 상품화 담당자     │  • QA · Certification   │
   │  • AI Web Agent 아키텍트│  • Tizen 웹앱 개발자    │
   │  • Tizen Platform Team  │  • AI 모델 제공사       │
   │  • UX · Design Team     │  • End User — 일반 시청자│
   │                         │  • End User — 고령      │
   │                         │  • End User — 접근성    │
   │                         │  • End User — 가족 공유 │
   │                         │                         │
High│                         │                         │Low
Power├─────────────────────────┼─────────────────────────┤Power
   │                         │                         │
   │      Keep Satisfied     │         Monitor         │
   │                         │                         │
   │  • Security & Privacy   │  • 제3자 웹 서비스      │
   │  • Account · Wallet     │    운영자              │
   │  • SoC · HW 파트너      │                         │
   │  • 규제 · 법률 기관     │                         │
   │                         │                         │
   └─────────────────────────┼─────────────────────────┘
                             │
                       Low Interest
```

### A.3 분류 근거 요약

- **Manage Closely** 주체들은 본 시스템의 범위·설계·출시에 직접 결정권을 행사하거나(VD 경영진·상품화, 아키텍트), 본 시스템의 호스트이자 주요 소비자(Tizen Platform Team)로서 일상 합의가 필요하다. UX · Design Team은 Generative UI라는 본 시스템 고유 산출물의 품질과 브랜드 일관성에 대한 소유권을 가지므로 High Power로 취급한다.
- **Keep Satisfied** 주체들은 보안·프라이버시·결제·HW·규제 게이트에서 강한 거부권을 가진다. 그러나 일상 설계 회의에 상시 참여하지 않으므로, 마일스톤·감사 시점의 **명확한 산출물·근거 제공**이 핵심 전략이다.
- **Keep Informed** 주체 중 Web Engine Team과 QA는 내부 팀이며 영향력은 중간 수준이나, 본 시스템의 결정이 자기 업무에 직접 흘러오므로 관심도는 높다. End User 세그먼트는 의사결정권이 없지만 경험의 최종 수신자이므로 UX · Design Team을 통한 대리 채널이 필요하다.
- **Monitor** 주체인 제3자 웹 서비스 운영자는 본 시스템의 대상(Target)이되 우리가 직접 통제하거나 상시 합의할 수 없는 외부 주체다. 정책 변화·어뷰즈 이슈 등이 발생한 시점에만 개별 대응한다.

### A.4 커뮤니케이션 케이던스

| 사분면 | 권장 케이던스 | 주요 채널·산출물 |
|---|---|---|
| Manage Closely | 주 단위 리뷰, 마일스톤별 의사결정 회의 | 아키텍처 결정 기록(ADR), 설계 문서 각 장 리뷰 |
| Keep Satisfied | 게이트·인증 시점 + 중대 이슈 시점 | 보안·프라이버시 심사 보고, 규제 준수 매핑표, HW 적용 가이드 |
| Keep Informed | 월 단위 요약 + 릴리스 노트 | 설계 문서 공개 링크, 변경 로그, 개발자 가이드 |
| Monitor | 분기 단위 점검 | 정책 모니터링 로그, 외부 변경 대응 기록 |

---

## 부록 B. RACI 매트릭스

본 부록은 본 시스템 설계·상품화 과정의 주요 활동을 정의하고, 각 활동에 대해 Stakeholder의 책임을 **R(Responsible, 실행)**, **A(Accountable, 최종 책임)**, **C(Consulted, 양방향 자문)**, **I(Informed, 일방향 통보)**로 할당한다. 활동 목록은 3.6의 Architectural View 작성 단위를 실무적 의사결정 단위로 그룹화한 것이다.

### B.1 활동 정의

| # | 활동 | 대응 Architectural View (3.6) |
|---|------|-----------------------------|
| 1 | 시스템 경계·Logical / Component Architecture 정의 | Context, Logical, Component View |
| 2 | Model Abstraction Layer 인터페이스 확정 | Logical, External Interface View |
| 3 | Web Engine 연동 계약(CDP · Browser Session) 확정 | Component, Integration / Extension, Data Flow View |
| 4 | 호스트(Main Agent · GenUI Renderer) 호출 계약 확정 | Logical, Integration / Extension View |
| 5 | 리소스 예산 · 배치 결정 | Deployment, Process, Resource Budget View |
| 6 | 보안 · 위협 모델 · 권한 모델 확정 | Security, Threat Model View |
| 7 | 프라이버시 · 데이터 보존 정책 확정 | Privacy & Retention, Data Flow View |
| 8 | 결제 · 동의 · 승인 플로우 확정 | Sequence / Flow, Consent, Identity / Profile View |
| 9 | Generative UI 디자인 시스템 확정 | Generative UI, Rendering Policy, Interaction View |
| 10 | 접근성 요건 확정 | Accessibility View |
| 11 | 테스트 전략 · 관측성 설계 | Test Strategy, Observability View |
| 12 | 로드맵 · 라인업별 탑재 범위 확정 | Roadmap & Capability, Deployment, Resource Budget View |
| 13 | 상품화 게이트 통과 · 출시 승인 | — (게이트 운영) |

### B.2 RACI 매트릭스

표기: **R** 실행, **A** 최종 책임, **C** 자문, **I** 통보. 공란은 해당 활동에 직접 관여하지 않음을 의미한다. Accountable은 활동당 1명을 원칙으로 한다.

| # | 활동 | VD 경영진 | VD 상품화 | 아키텍트 | Tizen Platform | Web Engine | Security & Privacy | UX / Design | Account / Wallet | QA / Cert. |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 시스템 경계 · Logical / Component 정의 | I | I | **A**, R |   C   |   C   |   C   |   C   |   I   |   I   |
| 2 | Model Abstraction Layer 인터페이스 | I | I | **A**, R |   C   |       |   C   |       |       |   I   |
| 3 | CDP · Browser Session 계약 |   |   | **A**, R |   C   |   R   |   C   |       |       |   I   |
| 4 | 호스트(Main Agent / GenUI Renderer) 호출 계약 |   | I | **A**, R |   R   |       |   C   |   C   |       |   I   |
| 5 | 리소스 예산 · 배치 |   C   |   C   |   R   | **A**, R |   C   |       |       |       |   I   |
| 6 | 보안 · 위협 모델 · 권한 모델 |   | I |   R   |   C   |   C   | **A**, R |       |   C   |   C   |
| 7 | 프라이버시 · 데이터 보존 |   | I |   R   |   C   |       | **A**, R |       |   C   |   I   |
| 8 | 결제 · 동의 · 승인 플로우 |   | I |   R   |   I   |       |   C   |   R   | **A**, R |   C   |
| 9 | Generative UI 디자인 시스템 |   | I |   R   |   C   |   C   |       | **A**, R |       |   I   |
| 10 | 접근성 요건 |   | I |   R   |   I   |       |   C   | **A**, R |       |   C   |
| 11 | 테스트 전략 · 관측성 |   | I |   R   |   C   |   C   |   C   |   C   |       | **A**, R |
| 12 | 로드맵 · 라인업별 탑재 범위 |   C   | **A**, R |   R   |   C   |   I   |   I   |   I   |   I   |   C   |
| 13 | 상품화 게이트 · 출시 승인 | **A** |   R   |   C   |   I   |   I   |   C   |   I   |   C   |   C   |

### B.3 외부 Stakeholder 관여

외부 주체는 RACI 열로 포함하지 않았으나, 활동별로 다음과 같이 관여한다.

| 외부 Stakeholder | 주 관여 활동 | 관여 형태 |
|---|---|---|
| Tizen Web App 개발자 (3.3.1) | 활동 9 (GenUI 디자인 시스템), 활동 1 (호환성 인터페이스) | C — 메타데이터·확장 요구사항 자문 |
| 제3자 웹 서비스 운영자 (3.3.2) | 활동 3 (에이전트 접근 정책), 활동 8 (결제 플로우) | I — 정책 모니터링 및 변경 통보 |
| AI 모델 제공사 (3.3.3) | 활동 2 (Model Abstraction Layer), 활동 7 (데이터 이전) | C — API · SLA · 데이터 이용 계약 자문 |
| SoC · 하드웨어 파트너 (3.3.4) | 활동 5 (리소스 예산), 활동 12 (라인업 범위) | C — HW 제약·가속기 드라이버 자문 |
| 규제 · 법률 기관 (3.5) | 활동 6~8, 10, 13 | I — 준수 증거 제출 및 감사 대응 |

### B.4 RACI 운영 원칙

1. **Accountable은 중복 불가**. 활동별로 "최종 책임자"는 1명이며, 본 매트릭스의 A는 해당 Stakeholder 열 내 조직장을 의미한다.
2. **Responsible 다중 허용**. 공동 실행이 필요한 활동(예: CDP 계약, 호스트 호출 계약)은 R을 복수로 둔다.
3. **Consulted 남발 금지**. 자문 대상이 과도하면 의사결정 지연이 발생하므로, 실제 결정에 유의한 영향을 주는 주체만 C로 표기한다.
4. **RACI 변경 시 ADR 기록**. 책임 할당은 아키텍처 결정의 일부이므로, 본 매트릭스의 변경은 아키텍처 결정 기록(ADR)에 남긴다.
5. **본 부록은 3.6 매핑표와 일관성 유지**. 특정 활동의 Consulted/Responsible은 3.6에서 해당 View의 "●" Stakeholder 집합과 정합해야 하며, 편차가 발생하면 두 표를 함께 수정한다.
