# 1. 배경 및 목적 (Background & Purpose)

> Tizen TV On-Device AI Web Agent 설계 문서

---

## 1.1 배경 (Background)

### 1.1.1 AI Web Agent 기술 트렌드

#### 산업 전반의 Agentic AI 부상

2025년을 기점으로 AI 산업은 단순한 대화형 챗봇에서 **자율적으로 작업을 수행하는 Agent 패러다임**으로 급격히 전환되고 있다. 글로벌 AI Agent 시장 규모는 2025년 $8.29B에서 2026년 $12.06B으로 성장했으며, 2030년까지 $52.62B에 도달할 것으로 전망된다 (CAGR 46.3%). 2026년 기준 기업의 51%가 AI Agent를 프로덕션에 배포 중이며, 추가 23%가 확장 단계에 있다.

#### 주요 빅테크의 AI Browser Agent 경쟁

AI Web Agent 분야는 빅테크 기업 간 핵심 경쟁 영역으로 부상했다.

| 기업 | 제품/기술 | 출시 시점 | 접근 방식 | WebVoyager 벤치마크 |
|------|-----------|-----------|-----------|---------------------|
| OpenAI | Operator (CUA) | 2025.01 | 소비자 제품 → ChatGPT Agent Mode로 통합 | 87% |
| Google | Project Mariner / Gemini 2.5 CU | 2024.12 ~ 2025.10 | 클라우드 VM 기반, 개발자 API/SDK | 83.5% |
| Anthropic | Claude Computer Use | 2024.10 | 개발자 API 중심, 타사가 제품화 | 56% (초기) → 개선 중 |

OpenAI Operator는 브라우저 기반 작업에서 87%의 성공률을 달성했고, Google의 Gemini 2.5 Computer Use 모델은 70% 이상의 정확도를 보인다. 이러한 수치는 AI가 웹 브라우저를 통해 사용자를 대신해 실질적인 작업을 수행할 수 있는 수준에 도달했음을 의미한다.

#### On-Device AI로의 전환

클라우드 중심에서 온디바이스(Edge) AI로의 전환이 가속화되고 있다. On-Device AI 시장은 2026~2033년 CAGR 27.8%로 성장해 $75.5B 규모에 도달할 전망이며, Edge AI 시장은 동기간 CAGR 21.7%로 $118.69B 규모로 확대될 것으로 예상된다. Gartner는 2027년까지 조직의 소규모 태스크 특화 AI 모델 사용량이 범용 LLM 대비 3배 증가할 것으로 예측했다.

이러한 전환의 핵심 동인은 다음과 같다.

- **프라이버시**: 사용자 데이터가 디바이스를 벗어나지 않아 개인정보 보호 강화
- **응답 지연(Latency)**: 네트워크 왕복 없이 즉각적인 응답 가능
- **오프라인 동작**: 네트워크 불안정 환경에서도 기본 기능 유지
- **비용 효율**: 클라우드 API 호출 비용 절감

### 1.1.2 TV 브라우저 및 웹 에코시스템 현황

#### Smart TV 시장 현황

글로벌 Smart TV 앱 시장은 2025년 기준 약 $50B 규모이며, 2033년까지 CAGR 15%로 성장이 전망된다. Smart TV 플랫폼 시장은 2024년 $449.74B에서 2033년 $966.87B으로 확대될 것으로 예상된다 (CAGR 10.04%).

OS별 점유율을 보면, Android TV가 약 43%로 1위를 차지하고 있으며, Samsung Tizen은 약 20~23%의 점유율을 유지하고 있다. Tizen은 2020년 약 34%에서 점유율이 하락 추세에 있으나, 여전히 전 세계 190개국 2억 대 이상의 Smart TV에서 구동되는 주요 플랫폼이다.

#### TV 웹 브라우저의 구조적 한계

TV 환경에서의 웹 브라우징은 다음과 같은 구조적 한계에 직면해 있다.

**입력 장치의 제약**: 리서치에 따르면 Smart TV 사용자의 91%가 기존 UI에 불만족하고 있으며, 가장 빈번한 불만 요인은 리모컨을 통한 포인팅 조작의 어려움이다. 커서 키로 화면을 탐색하는 것은 마우스나 터치스크린에 비해 현저히 불편하며, 특히 텍스트 입력은 TV 플랫폼에서의 혁신을 저해하는 핵심 장벽으로 지목되고 있다.

**웹 콘텐츠 호환성**: 대부분의 웹사이트는 데스크톱/모바일 환경에 최적화되어 있어, TV의 대화면(10-feet UI)과 리모컨 기반 네비게이션에 적합하지 않다. 반응형 웹 디자인(RWD)이 보편화되었음에도 TV 해상도와 입력 방식을 고려한 설계는 극히 드물다.

**인지 부하(Cognitive Overload)**: TV의 복잡한 네비게이션 구조는 사용자에게 인지적 과부하를 유발하며, 특히 고령 사용자층에서 포커스 조작의 어려움이 가장 빈번한 유저빌리티 문제로 보고되고 있다.

#### 웹앱 에코시스템의 중요성

TV 플랫폼에서 웹 기술은 핵심 인프라 역할을 수행한다. Tizen의 웹앱은 HTML/CSS/JavaScript 기반으로 구축되며, 플랫폼 컨테이너 내에서 웹 런타임(Web Runtime)을 통해 실행된다. 이는 다음과 같은 이점을 제공한다.

- **개발 비용 절감**: 웹 표준 기술 기반으로 크로스 플랫폼 개발 가능
- **빠른 업데이트**: 앱스토어 배포 없이 서버사이드 업데이트 가능
- **풍부한 생태계**: 기존 웹 기술 자산과 개발자 풀 활용 가능

그러나 TV 웹앱의 품질은 디바이스 성능, 브라우저 엔진 호환성, 리모컨 기반 UX 등 복합적 요인에 의해 모바일/데스크톱 대비 현저히 낮은 수준에 머물러 있다.

---

## 1.2 과제의 필요성 (Why This Project Is Essential)

### 1.2.1 TV 웹 경험의 근본적 전환 필요

TV 브라우저의 91% 사용자 불만족률은 단순한 UI 개선으로 해결될 수 있는 문제가 아니다. 리모컨이라는 입력 장치의 근본적 제약 위에 구축된 현재의 인터랙션 모델은 **패러다임 수준의 전환**이 필요하다. AI Web Agent는 사용자의 의도를 이해하고 웹 브라우저를 자율적으로 조작함으로써, 입력 장치의 제약을 우회하는 새로운 인터랙션 패러다임을 제시한다.

### 1.2.2 경쟁사 대비 차별화 시급

Android TV(43% 점유율)가 Google의 Gemini 기반 AI 역량을 빠르게 통합하고 있는 상황에서, Tizen(~20% 점유율)이 AI 기반 웹 경험에서 뒤처질 경우 플랫폼 경쟁력의 추가 하락이 불가피하다. 2020년 34%에서 현재 20~23%로 하락한 점유율 추세를 반전시키기 위해서는, 경쟁사가 제공하지 못하는 차별화된 AI 웹 경험이 필수적이다.

### 1.2.3 On-Device 실행의 전략적 가치

| 비교 항목 | 클라우드 기반 | On-Device |
|-----------|-------------|-----------|
| 응답 지연 | 200~500ms (네트워크 RTT 포함) | <100ms (로컬 추론) |
| 프라이버시 | 사용자 행동 데이터 서버 전송 | 디바이스 내 처리, 외부 전송 없음 |
| 운영 비용 | API 호출당 과금 (대규모 TV fleet 시 막대한 비용) | 초기 모델 최적화 비용 후 추가 비용 없음 |
| 가용성 | 네트워크 의존 | 오프라인에서도 기본 동작 |

2억 대 이상의 Tizen TV가 배포된 상황에서, 클라우드 기반 AI Agent를 전 디바이스에 적용할 경우 API 호출 비용이 기하급수적으로 증가한다. On-Device 방식은 이러한 스케일 문제를 근본적으로 해결하며, 동시에 사용자 브라우징 데이터라는 민감한 정보의 프라이버시를 보장한다.

### 1.2.4 시장 기회와 타이밍

AI Agent 시장이 연평균 46.3%로 급성장하고 있는 현 시점은, TV 플랫폼에 AI Web Agent를 도입하기 위한 최적의 타이밍이다. 2026년 기준 기업의 85%가 AI Agent 도입을 완료했거나 계획 중인 상황에서, TV 분야는 아직 본격적인 AI Agent 적용이 이루어지지 않은 **블루오션 영역**이다.

CES 2026에서 Displace가 최초의 AI-Native TV를 발표하는 등 TV에서의 On-Device AI 처리가 본격화되고 있으며, 이러한 시장 흐름에 선제적으로 대응하는 것이 전략적으로 중요하다.

---

## 1.3 목적 (Purpose)

본 과제의 목적은 Tizen TV 플랫폼에서 **On-Device로 동작하는 AI Web Agent(AI Browser)**를 설계하는 것이다. 구체적인 목적은 다음과 같다.

### 1.3.1 핵심 목적

**TV 웹 인터랙션 패러다임의 혁신**: 리모컨 기반의 수동적 웹 브라우징에서, 음성 및 자연어를 통한 AI 에이전트 기반의 자율적 웹 인터랙션으로의 전환

### 1.3.2 세부 목적

1. **음성 기반 웹 인터랙션 체계 구축**: 사용자가 음성 명령으로 웹 브라우징, 검색, 폼 입력, 구매/예약 등의 작업을 AI Agent에게 위임할 수 있는 시스템 설계

2. **TV 웹앱 품질 자동 향상**: AI Agent가 웹 콘텐츠를 분석하고 TV 환경(대화면, 리모컨, 시청 거리)에 최적화된 형태로 자동 변환하는 메커니즘 설계

3. **Headless Browser 및 Generative UI 활용**: 웹 콘텐츠를 Headless Browser로 사전 처리하고, AI가 사용자 의도에 맞는 UI를 동적으로 생성하는 Generative UI 아키텍처 설계

4. **On-Device 추론 아키텍처 설계**: TV SoC의 NPU/GPU 자원을 활용하여 SLM(Small Language Model) 기반의 온디바이스 추론 파이프라인 설계, 필요 시 클라우드와의 하이브리드 구조 포함

5. **자동화 시나리오 지원**: 웹 기반 자동 구매, 예약, 정보 수집 등 사용자의 반복적 웹 작업을 자율적으로 수행하는 에이전트 워크플로우 설계

### 1.3.3 기대 효과

| 지표 | 현재 (TV 브라우저) | 목표 (AI Web Agent) |
|------|-------------------|-------------------|
| 웹 작업 완료 시간 | 리모컨 조작 기준 60~120초 | 음성 명령 기준 10~20초 |
| 텍스트 입력 속도 | 리모컨 가상키보드 ~5 WPM | 음성 입력 ~100 WPM |
| 사용자 만족도 | 9% (91% 불만족) | 70% 이상 목표 |
| 웹 콘텐츠 TV 최적화율 | 수동 대응 (극히 제한적) | AI 자동 최적화 80% 이상 |

---

## 1.4 과제 범위 (Scope)

### In-Scope

- On-Device AI 추론 엔진 및 모델 아키텍처 설계
- AI Web Agent 코어 시스템 아키텍처 설계
- 음성 인터페이스와 웹 브라우저 간 통합 설계
- Headless Browser 기반 웹 콘텐츠 처리 파이프라인 설계
- Generative UI 렌더링 시스템 설계
- 주요 사용 시나리오별 에이전트 워크플로우 설계
- TV SoC 자원 제약 하의 성능 최적화 전략

### Out-of-Scope

- 구체적인 SLM 모델 학습 및 파인튜닝 방법론
- 서버 사이드 인프라 운영 세부 사항
- 특정 웹사이트별 자동화 스크립트 구현
- DRM 콘텐츠 처리

---

## 참고 자료 (References)

1. GM Insights, "AI Agents Market Size & Share, Growth Opportunity 2025-2034" — AI Agent 시장 규모 ($8.29B → $52.62B)
2. Master of Code, "150+ AI Agent Statistics [2026]" — 기업 AI Agent 도입률 (51% 프로덕션)
3. AIMultiple, "AI Agents: Operator vs Browser Use vs Project Mariner" — 주요 AI Browser Agent 벤치마크 비교
4. Grand View Research, "On-Device AI Market Report, 2033" — On-Device AI 시장 전망 (CAGR 27.8%)
5. Grand View Research, "Edge AI Market Report, 2033" — Edge AI 시장 전망 ($118.69B by 2033)
6. Electroiq, "Smart TV Statistics 2025" — Smart TV OS 점유율 통계
7. Media Play News, "Samsung's Tizen Tops Global Smart-TV OS Market Share" — Tizen 설치 기반 (2억 대+)
8. Khan et al., "Perspectives on the Design, Challenges, and Evaluation of Smart TV User Interfaces" (2022) — TV UI 사용자 불만족률 91%
9. ResearchGate, "Smart Enough for the Web? A Responsive Web Design Approach for Smart TVs" — TV 웹 브라우징 UX 연구
10. AndroidGuys, "Displace Pro TV 2: First AI-Native TV at CES 2026" — CES 2026 AI-Native TV
11. Gartner — 2027년 태스크 특화 SLM 사용량 LLM 대비 3배 증가 예측
12. Coherent Market Insights, "On-Device AI Market Trends, 2026-2033" — On-Device AI 시장 동인 분석
