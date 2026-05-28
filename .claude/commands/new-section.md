# /project:new-section

새 ADS 섹션 파일을 CLAUDE.md의 스키마에 맞춰 생성한다.

## 사용법
```
/project:new-section <섹션-타입> [추가-인자]
```

### 섹션 타입
- `fr <영역코드>` — 새 FR 영역 파일 생성 (예: `/project:new-section fr AGT`)
- `nfr <QA코드>` — NFR 카테고리 파일 생성 (예: `/project:new-section nfr LAT`)
- `adr <제목>` — ADR 파일 생성 (예: `/project:new-section adr llm-provider-selection`)
- `view <뷰타입>` — Architecture View 파일 생성 (예: `/project:new-section view component`)
- `chapter <번호> <제목>` — 최상위 챕터 파일 생성

## 동작 방식

1. `$ARGUMENTS`에서 섹션 타입과 인자를 파싱한다.
2. CLAUDE.md의 영역 목록, QA 약어, 스키마를 참조한다.
3. 연관 기존 문서(stakeholders, scenarios, 인접 섹션)를 읽어 컨텍스트를 파악한다.
4. 해당 타입의 스키마(FR 카드, NFR 카드, ADR 템플릿)를 그대로 적용한다.
5. 파일명은 CLAUDE.md의 명명 규칙을 따른다.
6. 파일 생성 후 사용자에게 커밋 여부를 묻는다.

## 주의
- 빈 템플릿이 아닌 **컨텍스트가 반영된 실질적 초안**을 생성한다.
- 모든 수치와 결정에는 근거를 반드시 추가한다.
- TBD 항목은 결정 위치를 명시한다.
