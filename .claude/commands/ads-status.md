# /project:ads-status

현재 ADS 문서의 완성도와 추적성 상태를 분석해서 보고한다.

## 사용법
```
/project:ads-status [--area <영역코드>] [--focus <항목>]
```

### 옵션
- `--area VUI|AGT|BRW|...` — 특정 영역만 상세 분석
- `--focus traceability` — FR↔NFR↔ADR 추적성 단절 지점 집중 분석
- `--focus open-questions` — 전체 문서의 Open Question 목록 수집
- `--focus tbd` — TBD 항목 목록 수집

## 동작 방식

1. `docs/` 전체를 탐색한다.
2. CLAUDE.md의 문서 상태 표(✅/⬜)를 기준으로 완성도를 집계한다.
3. 작성된 문서의 FR/NFR 카드에서 다음 항목을 점검한다:
   - `Related NFR`가 TBD로 남아 있는 FR
   - `Related FR`가 없는 NFR
   - `Status: Draft`로 남아 있는 카드 수
   - Open Questions 절의 미해결 항목
4. 결과를 **섹션별 완성도 표** + **추적성 갭 목록** + **다음 권장 작업** 형식으로 출력한다.
5. 다음 작업 우선순위를 제안한다.
