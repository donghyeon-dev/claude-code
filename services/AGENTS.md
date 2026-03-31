# services 지식 베이스

## 개요
`services/`는 서브시스템 규모의 백엔드를 담습니다. API, MCP, LSP, analytics, memory, automation, plugins, policy 지원이 이곳에 있습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| Claude/API 요청 흐름 | `api/claude.ts`, `api/client.ts`, `api/bootstrap.ts` | 요청 조립, 재시도, 스트리밍, 사용량 계산 |
| MCP 클라이언트/인증/설정 | `mcp/` | 가장 큰 서비스 클러스터 중 하나 |
| Language Server 생명주기 | `lsp/` | client/server instance + diagnostic tracking |
| Analytics | `analytics/` | 공개 로깅 API, 메타데이터 보강, sink |
| 메모리/상태 마크다운 | `SessionMemory/`, `extractMemories/` | 프롬프트 기반 마크다운 유지 |
| 자동화/문서 지원 | `autoDream/`, `MagicDocs/`, `PromptSuggestion/` | 계획/문서/프롬프트 지원 기능 |

## 구조
```text
services/
├── api/
├── mcp/
├── lsp/
├── analytics/
├── SessionMemory/
├── autoDream/
├── MagicDocs/
└── PromptSuggestion/
```

추가로 눈여겨볼 하위 트리: `oauth/`, `plugins/`, `settingsSync/`, `teamMemorySync/`, `remoteManagedSettings/`, `tools/`, `toolUseSummary/`, `tips/`.

## 규약
- 이 트리는 공용 프레임워크가 아니라 서브시스템 단위로 분리돼 있습니다.
- `api/`와 `mcp/`는 대형 오케스트레이션 파일이 모인 핫스팟입니다.
- 몇몇 서비스는 마크다운/상태 산출물을 직접 유지하므로 텍스트 형식 자체가 동작의 일부입니다.

## 안티패턴
- API, MCP, LSP, analytics, memory 책임을 쉽게 섞지 마세요.
- owning prompt가 금지한 마크다운 메모리 헤더/템플릿을 임의로 수정하지 마세요.
- 공유 메모리나 팀 메모리 흐름에 비밀 데이터를 넣지 마세요.

## 메모
- `services/api/claude.ts`, `services/mcp/client.ts`, `services/mcp/auth.ts`는 대표적인 복잡도 핫스팟입니다.
- `services/plugins/`는 메인 플러그인 로더 경로가 아닙니다. 서비스 계층의 plugin operation/CLI 연동을 맡고, discovery/load/validation/startup은 `utils/plugins/`가 담당합니다.
- 이 트리에서 향후 더 세분화할 하위 문서를 하나만 고른다면 `mcp/`가 우선입니다.
