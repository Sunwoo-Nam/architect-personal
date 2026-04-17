# 3. 이해관계자 (Stakeholders)

> Tizen TV AI Web Agent (AI Browser) 설계 문서

---

## 3.1 개요 (Overview)

본 장은 **ISO/IEC/IEEE 42010:2022**(Architecture Description)와 **arc42** 템플릿의 Stakeholder 정의 방식을 하이브리드로 적용한다. 핵심 원칙은 다음 세 가지다.

1. **Stakeholder는 시스템에 대해 고유한 Concern(관심사)을 갖는 역할이다.** 동일한 사람이 여러 Stakeholder 역할을 겸할 수 있으며, 본 장은 "역할" 단위로 식별한다.
2. **각 Concern은 하나 이상의 Architectural View를 통해 해소된다.** 따라서 Stakeholder → Concern → View 매핑은 이후 본 설계 문서에서 어떤 View를 반드시 작성해야 하는지에 대한 근거를 제공한다.
3. **Stakeholder는 시스템 외부의 상호작용 주체만이 아니라, 시스템의 형성·수용·운영에 이해관계를 가진 모든 역할을 포함한다.** 즉, 엔드유저뿐 아니라 내부 개발 조직, 외부 파트너, 규제 기관도 포함한다.

본 장에서 식별된 Stakeholder는 크게 네 범주로 분류된다.

| 범주 | 설명 | 절 |
|------|------|----|
| Samsung 내부 조직 | 본 시스템을 실제로 구축·운영·배포할 주체 | 3.2 |
| 외부 파트너 · 개발자 | 본 시스템이 동작하기 위해 연동이 필요한 제3자 | 3.3 |
| End User | 본 시스템의 최종 사용자 및 그 세부 그룹 | 3.4 |
| 규제 · 법률 기관 | 설계·운영 시 준수해야 할 법·제도·표준의 주체 | 3.5 |

본 장 마지막의 3.6 절에서는 전체 Stakeholder와 Architectural View를 통합 매핑한 표를 제공하며, 이는 이후 4장 이후의 아키텍처 View 작성 범위를 결정하는 기준이 된다.

---

## 3.2 Samsung 내부 조직

### 3.2.1 VD 사업부 경영진 (Executive Sponsor)

- **역할**: Visual Display 사업부 의사결정층. 본 과제의 전략적 투자 승인과 로드맵 확정을 담당한다.
- **주요 Concern**
  - AI Web Agent가 Tizen 플랫폼 경쟁력에 기여하는 방식과 크기
  - 출시 타이밍(Time-to-market)과 경쟁사 대응 포지션
  - 초기 투자 대비 플랫폼·서비스·광고 수익 구조에 미치는 영향
  - 브랜드 리스크(AI 오동작·프라이버시·접근성 이슈가 외부에 노출될 가능성)
- **관련 Architectural View**
  - Context View (시스템 경계·외부 이해관계자 전체 조망)
  - Roadmap & Capability View (단계별 출시 범위와 역량 확보 경로)

### 3.2.2 Tizen Platform Team

- **역할**: Tizen의 Web Runtime, IPC 계층, 시스템 서비스, 리소스 관리자, 앱 수명주기 관리자를 소유하는 조직.
- **주요 Concern**
  - 에이전트 하네스가 Web Runtime과 샌드박스 경계를 침범하지 않는지
  - TV 시스템의 고정된 메모리·CPU 예산(특히 저사양 보급형 모델) 내에서 하네스가 동작하는지
  - 에이전트 프로세스가 기존 앱·시스템 서비스와 자원 경합 없이 공존하는지
  - 부팅·슬립·복귀 시퀀스와 하네스 생명주기가 충돌하지 않는지
- **관련 Architectural View**
  - Deployment View (프로세스/서비스 배치, 권한 도메인)
  - Process View (런타임 스케줄링, 스레드·태스크 모델)
  - Resource Budget View (메모리/CPU/디스크 상한과 할당)

### 3.2.3 Web Engine Team

- **역할**: Tizen에서 사용하는 Chromium/Blink 기반 웹 엔진 및 WebView, Headless Browser 엔진을 유지·이식하는 팀.
- **주요 Concern**
  - 에이전트가 DOM에 접근·조작하기 위해 사용하는 인터페이스의 안정성과 버전 호환성
  - Renderer 프로세스 고립 원칙이 유지되는지
  - Headless 브라우징을 위한 페이지 로딩·렌더링 경로의 성능·메모리 영향
  - 기존 웹앱과 동일한 엔진 인스턴스를 공유할지, 별도 인스턴스를 쓸지의 결정이 유지보수 비용에 미치는 영향
- **관련 Architectural View**
  - Component View (Web Engine ↔ Agent Harness 컴포넌트 경계)
  - Data Flow View (DOM·접근성 트리 → AI 모델로의 표현 변환)
  - Deployment View (렌더러/브라우저 프로세스 구성)

### 3.2.4 Bixby · On-Device AI Team

- **역할**: 음성 인식(ASR), 자연어 이해(NLU), 합성(TTS), 온디바이스 SLM 및 클라우드 LLM 연동 계층을 담당하는 팀.
- **주요 Concern**
  - 에이전트 하네스가 요구하는 모델 인터페이스(모델 추상화 계층)의 계약 범위
  - 다중 턴 대화와 웹 페이지 컨텍스트의 결합 방식
  - 모델 업데이트(SLM 재배포, LLM API 스키마 변경)가 하네스 동작에 미치는 영향
  - 클라우드·온디바이스 모델 간 라우팅 기준(지연·비용·프라이버시)에 대한 공동 설계 권한
- **관련 Architectural View**
  - Logical View (Model Abstraction Layer, Intent/Plan/Act 파이프라인)
  - Sequence View (Voice → Intent → Plan → Web Action → Response 흐름)
  - External Interface View (Cloud LLM 연동)

### 3.2.5 Security & Privacy Team

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

### 3.2.6 UX · Design Team

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

### 3.2.7 Samsung Account · Samsung Wallet Team

- **역할**: 통합 계정, 프로필, 결제 토큰, 신원 검증을 제공하는 조직.
- **주요 Concern**
  - 음성·자동화로 진행되는 결제 승인 과정의 법적·UX적 유효성
  - 2FA, 생체 인증, 음성 화자 식별을 대체·보완 수단으로 사용할 수 있는 범위
  - 에이전트가 계정 위임 하에 제3자 사이트로 진입할 때의 위임 범위와 취소 가능성
- **관련 Architectural View**
  - Security View (Auth/Authorization Flow)
  - Sequence View (Voice → Purchase Consent → Payment)
  - Identity/Profile View (사용자·프로필·디바이스 식별 모델)

### 3.2.8 QA · Certification Team

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

### 3.3.2 제3자 웹 서비스 운영자 (커머스·예약·OTT·포털)

- **역할**: Tizen TV 외부에서 운영되는 이커머스, 예약, OTT, 포털 등 웹 서비스의 운영 주체. 에이전트가 사용자의 대리인으로서 접속·조작하게 되는 대상.
- **주요 Concern**
  - 자동화 에이전트의 접근 허용·차단 정책과 식별 수단(User-Agent, Agent-ID, 서명 등)
  - 봇 기반 사기·어뷰즈 방지와의 양립 가능성
  - 전환율·장바구니 이탈 등 기존 분석 지표에 대한 영향
  - 계약·약관상 자동화 접근이 허용되는 범위의 불확실성
- **관련 Architectural View**
  - External Interface View (에이전트가 외부 서비스와 맺는 식별·준수 계약)
  - Policy / Compliance View (웹 자동화 관련 규범 준수 체계)

### 3.3.3 AI 모델 제공사 (Cloud LLM / On-Device SLM)

- **역할**: 자체 또는 제3자가 제공하는 대규모 언어 모델, 소형 언어 모델, 멀티모달 모델의 공급자.
- **주요 Concern**
  - 본 시스템이 요구하는 모델 인터페이스·응답 스키마·컨텍스트 크기의 현실성
  - 응답 지연·가용성·비용에 대한 SLA
  - 모델 업데이트 주기 및 사전 고지 정책
  - 사용자 데이터의 학습 활용 여부와 계약상 범위
- **관련 Architectural View**
  - Model Abstraction Layer View (공급자 중립 인터페이스)
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

### 3.3.5 콘텐츠 퍼블리셔 · 광고 에코시스템

- **역할**: 뉴스·영상·이커머스 콘텐츠를 운영하는 퍼블리셔와 광고 네트워크·광고주.
- **주요 Concern**
  - Generative UI에 의한 페이지 재구성이 광고 노출·측정 지표에 미치는 영향
  - 콘텐츠 요약·추출이 저작권·2차적 저작물 범위에 해당하는지
  - 자사 콘텐츠의 출처·브랜드 표기 유지 여부
- **관련 Architectural View**
  - Rendering Policy View (원본 보존·요약·재구성의 규칙)
  - Attribution View (콘텐츠 출처 표기 모델)

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

규제 기관은 시스템과 직접 상호작용하지는 않지만, 설계·운영이 준수해야 할 규범의 주체이다. 본 절의 규제는 법적 강제력을 가진 것과 사실상 표준으로 기능하는 것을 모두 포함한다.

### 3.5.1 개인정보 보호 당국

- **범위**: EU GDPR, 한국 PIPA, 미국 CCPA/CPRA, 기타 역내 개인정보 보호 규제.
- **주요 Concern**
  - 음성 원본·전사·DOM 스냅샷 등 잠재적 개인정보의 수집 근거와 동의 방식
  - 온디바이스 처리와 클라우드 전송 간의 선택 기준과 사용자 고지
  - 제3자(모델 공급사, 결제 대행사)로의 데이터 이전의 적법성
  - 잊혀질 권리·열람권 등 데이터 주체 권리 이행 수단
- **관련 Architectural View**
  - Privacy & Data Flow View
  - Retention Policy View (데이터 분류별 보존·파기 규칙)

### 3.5.2 접근성 규제

- **범위**: WCAG 2.2, 미국 ADA, 유럽 접근성법(EAA), 한국 장애인차별금지법 등.
- **주요 Concern**
  - Generative UI를 포함한 모든 출력물이 준수해야 할 대체 텍스트·대비·포커스 기준
  - 자동화로 제공되는 기능이 비자동화 경로로도 동등하게 접근 가능한지
- **관련 Architectural View**
  - Accessibility View
  - Interaction View (대체 입력 경로)

### 3.5.3 전자상거래 · 소비자 보호 규제

- **범위**: 자동 결제·자동 갱신 관련 규제, 청약 철회, 광고·표시 규제.
- **주요 Concern**
  - 음성·에이전트 기반 결제 동의가 명시적·사전적 동의 요건을 충족하는지
  - AI가 생성한 추천·UI가 광고성 표시 의무를 위반하지 않는지
  - 반복 구매·자동 구독 시 고지·철회 경로
- **관련 Architectural View**
  - Sequence View (결제·승인 플로우)
  - Consent View (동의 상태 모델)

### 3.5.4 콘텐츠 · 저작권 규제

- **범위**: 저작권법, DMCA, 플랫폼별 이용약관 상의 콘텐츠 이용 제한.
- **주요 Concern**
  - 웹 콘텐츠의 요약·재가공이 공정이용·허용 범위를 벗어나지 않는지
  - 원본 출처·저작자 표기의 일관성
- **관련 Architectural View**
  - Rendering Policy View
  - Attribution View

---

## 3.6 Stakeholder ↔ Architectural View 통합 매핑

본 표는 3.2~3.5에서 식별된 Stakeholder별 주요 Concern을 어느 Architectural View가 해소하는지를 통합적으로 보여준다. 본 표의 "●" 표식이 있는 View는 본 설계 문서 이후 장에서 **반드시 작성되어야 하는 View**이며, "○"는 해당 Stakeholder가 부차적으로 참고하는 View이다.

| Architectural View | VD 경영진 | Tizen Platform | Web Engine | Bixby/AI | Security&Privacy | UX/Design | Account/Wallet | QA/Cert. | 웹앱 개발자 | 제3자 서비스 | AI 모델 공급사 | SoC 파트너 | 퍼블리셔/광고 | 일반 시청자 | 고령 사용자 | 접근성 사용자 | 가족 공유 | 개인정보 규제 | 접근성 규제 | 전자상거래 규제 | 저작권 규제 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Context View | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Logical View | ○ | ● | ● | ● | ● | ● | ● | ○ | ● | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Component View |  | ● | ● | ● | ● | ● | ○ | ● | ● | ○ | ● | ○ |  |  |  |  |  |  |  |  |  |
| Deployment View | ○ | ● | ● | ○ | ● | ○ | ○ | ● | ○ |  | ○ | ● |  |  |  |  |  |  |  |  |  |
| Process View |  | ● | ● | ● |  |  |  | ● |  |  |  | ● |  |  |  |  |  |  |  |  |  |
| Resource Budget View | ○ | ● | ● | ● |  |  |  | ● |  |  | ● | ● |  |  |  |  |  |  |  |  |  |
| Data Flow View |  | ● | ● | ● | ● | ○ | ● | ○ | ● | ● | ● |  | ● |  |  |  | ● | ● |  | ● | ● |
| Security View | ○ | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ○ | ○ | ○ | ○ | ● | ● |  | ● |  |
| Threat Model View |  | ● | ● | ● | ● |  |  | ● | ○ | ● | ● |  |  |  |  |  | ○ | ● |  |  |  |
| Privacy & Retention View | ○ | ● | ○ | ● | ● | ○ | ● | ● | ○ | ● | ● |  | ○ | ● | ● | ● | ● | ● |  | ● |  |
| Interaction View | ○ |  | ○ | ● |  | ● | ● | ● | ● |  |  |  | ○ | ● | ● | ● | ● |  | ● | ● |  |
| Generative UI View | ○ |  | ● | ● | ○ | ● | ○ | ● | ● | ○ | ○ |  | ● | ● | ● | ● | ○ |  | ● | ● | ● |
| Accessibility View | ○ |  | ○ | ○ | ○ | ● |  | ● | ● |  |  |  |  | ● | ● | ● |  |  | ● |  |  |
| Identity / Profile View | ○ |  |  | ● | ● | ○ | ● | ○ | ○ | ○ |  |  |  | ● | ● | ● | ● | ● |  | ● |  |
| Integration / Extension View | ○ | ● | ● | ○ | ● | ○ |  | ● | ● | ● | ● | ○ | ● |  |  |  |  |  |  |  |  |
| Rendering Policy View | ○ |  | ● | ● | ○ | ● |  | ○ | ● | ● |  |  | ● | ● | ○ | ● |  |  | ○ | ● | ● |
| Sequence / Flow View | ○ | ○ | ○ | ● | ● | ● | ● | ● | ● | ● | ● |  | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Test Strategy View |  | ○ | ● | ● | ● |  |  | ● | ● | ○ | ● | ○ |  |  |  |  |  | ○ | ○ | ○ |  |
| Observability View | ○ | ● | ○ | ● | ● | ○ |  | ● | ● | ○ | ● |  | ● |  |  |  | ○ | ● |  |  |  |
| Roadmap & Capability View | ● | ● | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ● | ○ |  |  |  |  |  |  |  |  |

범례: ● 주 관심, ○ 부차 관심, 공란은 직접 관련성이 낮음을 의미한다.

### 3.6.1 매핑 해석 원칙

1. 한 행(View)에 ● 표식이 하나 이상 있으면, 그 View는 본 설계 문서에서 **작성이 필수**이다.
2. 한 열(Stakeholder)에 ● 표식이 하나 이상 있으면, 그 Stakeholder는 본 설계 문서의 **리뷰어 후보**이며, 해당 View들로 구성된 **Stakeholder 전용 목차(Role-Based View Package)**를 제공하는 것이 바람직하다.
3. 공란이 많은 View는 과설계의 신호일 수 있으며, 추가 근거가 제시되지 않는 한 본 설계 문서에서 생략한다.

본 통합 매핑은 4장 이후의 아키텍처 View 작성 범위를 확정하는 입력으로 사용된다.

---

## 3.7 가정과 제약 (Assumptions & Constraints)

본 장의 Stakeholder 식별은 다음 가정 위에서 이루어졌으며, 이후 장에서 이러한 가정이 변경될 경우 본 장을 재검토한다.

- **조직 구조 가정**: Samsung 내부 조직의 분류는 일반화된 역할 기반이며, 실제 조직도의 팀 명칭과 1:1 대응을 전제하지 않는다.
- **AI 모델 소싱 가정**: AI 모델은 자사·제3자·온디바이스·클라우드 모두의 조합이 가능하다고 전제한다. 따라서 모델 공급사는 단일 벤더가 아닌 범주로 취급한다.
- **규제 범위 가정**: 본 시스템은 글로벌 출시를 전제로 하며, 가장 보수적인 규제(예: GDPR, EAA, PIPA)를 기준선으로 삼는다. 특정 지역 한정 출시의 경우 규제 Stakeholder 범위가 달라질 수 있다.
- **Out-of-Scope**: 광고 플랫폼·파트너 마케팅 등 본 과제의 설계 범위(2.1 및 1.4 참조)에서 벗어난 주체는 본 장에서 식별하지 않는다.

---

## 3.8 참고 자료 (References)

1. ISO/IEC/IEEE 42010:2022 — "Systems and software engineering — Architecture description". Stakeholder 및 Concern·View 개념의 근거.
2. arc42 Template v8 — Section 1.2 "Stakeholders" 및 Section 4 "Solution Strategy"의 Stakeholder 정의 방식.
3. Rozanski & Woods, *Software Systems Architecture* (2nd ed.) — Viewpoint-Stakeholder Mapping 원칙.
