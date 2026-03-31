# tools 지식 베이스

## 개요
`tools/`는 런타임 도구 카탈로그입니다. 의미 있는 도구는 대체로 디렉터리 단위로 분리되어 있고, `Tool.ts`와 `tools.ts`를 통해 조립됩니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 도구 계약 변경 | `../Tool.ts` | `buildTool(...)`, capability 플래그, validation hook 정의 |
| 런타임 조립/필터링 | `../tools.ts` | feature/env gate, 권한 deny 필터링, dedupe, 정렬 |
| Bash 실행 규칙 | `BashTool/` | 보안/정책 밀도가 가장 높음 |
| 에이전트/태스크 생명주기 도구 | `AgentTool/`, `Task*Tool/`, `TodoWriteTool/`, `SkillTool/` | 오케스트레이션 성격이 강한 도구 |
| 검색/읽기 도구 | `FileReadTool/`, `GlobTool/`, `GrepTool/`, `LSPTool/`, `WebFetchTool/`, `WebSearchTool/` | 상대적으로 안전하며 읽기 전용인 경우가 많음 |

## 구조
```text
tools/
├── <ToolName>Tool/   # 도구 패밀리 하나당 디렉터리 하나
├── shared/           # 공용 헬퍼
├── testing/          # 도구별 테스트/지원 헬퍼
└── utils.ts          # 로컬 유틸리티 글루
```

## 규약
- 리프 도구는 `buildTool({ ... })` 기반 모듈이며, 스키마 + 프롬프트 + 런타임 메타데이터를 함께 가집니다.
- read-only/concurrency/defer 여부는 이름으로 추정하지 말고 메타데이터를 보세요.
- 런타임 dedupe에서는 내장 도구가 MCP 중복보다 우선합니다.
- 도구 UI/결과 렌더링은 `components/`가 아니라 도구 디렉터리 안에 같이 있을 수 있습니다.

## 안티패턴
- 동작을 바꾸면서 `buildTool(...)` 메타데이터를 우회하지 마세요.
- 모든 도구가 concurrency-safe라고 가정하지 마세요.
- 이미 도구 프롬프트/validator가 책임지는 정책을 임의 유틸리티 코드로 옮기지 마세요.

## 메모
- 가장 자주 건드릴 가능성이 높은 하위 트리는 `BashTool/`입니다.
- `TaskOutputTool`은 프로젝트 텍스트에서 deprecated로 언급됩니다. 가능하면 출력 파일 직접 읽기 경로를 우선하세요.
