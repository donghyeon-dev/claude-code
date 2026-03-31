# 프로젝트 지식 베이스

**생성일:** 2026-03-31 Asia/Seoul
**커밋:** 현재 소스 서브트리에서는 확인 불가
**브랜치:** 현재 소스 서브트리에서는 확인 불가

## 개요
Claude Code의 `src/` 소스 서브트리입니다. 큰 규모의 TypeScript 기반 CLI/터미널 앱이며, 명령/도구 레지스트리, 두꺼운 공용 런타임, React/Ink UI, remote/bridge/MCP/LSP 서브시스템이 함께 얽혀 있습니다.

## 구조
```text
src/
├── entrypoints/      # CLI, MCP, 초기화용 프로세스 진입점
├── commands/         # 슬래시/로컬 명령 디스크립터와 로더
├── tools/            # 도구 정의와 권한 인지 런타임 도구
├── services/         # API, MCP, LSP, 분석, 메모리, 자동화
├── components/       # 앱 UI 표면, 도메인별로 분리됨
├── ink/              # 터미널 렌더링/런타임 계층
├── utils/            # 공용 로직의 중심축, 코드 밀도가 가장 높음
├── bridge/           # 브리지/다이렉트 커넥트 런타임
├── hooks/            # UI/런타임 오케스트레이션 훅
├── remote/           # 원격 세션 및 웹소켓 경계
├── types/            # 공용 타입과 생성된 payload
└── .claude/          # 계층형 지시문/설정 표면
```

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| CLI 시작 흐름 | `entrypoints/cli.tsx`, `main.tsx`, `entrypoints/init.ts`, `setup.ts` | `main.tsx`가 전체 오케스트레이션 허브이고, `entrypoints/cli.tsx`는 얇은 프로세스 진입점입니다 |
| 명령 등록/변경 | `commands.ts`, `commands/` | 병합 순서와 사용 가능 여부 필터링은 `commands.ts`에 있습니다 |
| 도구 등록/변경 | `Tool.ts`, `tools.ts`, `tools/` | `buildTool(...)` 패턴과 런타임 필터링/중복 제거는 `tools.ts`에 있습니다 |
| Bash 안전성 | `tools/BashTool/`, `utils/bash/`, `constants/tools.ts` | 권한 흐름과 파서/허용목록 로직이 두 트리에 나뉘어 있습니다 |
| MCP 동작 | `services/mcp/`, `components/mcp/`, `tools/MCPTool/` | 설정/인증/클라이언트/UI가 분리돼 있습니다 |
| API 요청 흐름 | `services/api/claude.ts`, `services/api/client.ts`, `services/api/bootstrap.ts` | 요청 조립, 재시도, 스트리밍, 사용량 계산 담당 |
| 터미널 UI/런타임 | `screens/REPL.tsx`, `components/`, `ink/` | 앱 UI와 렌더링 엔진은 별도 계층입니다 |
| 세션/대화 저장 | `utils/sessionStorage.ts`, `services/SessionMemory/`, `history.ts` | 저장소와 마크다운 메모리 시스템이 다릅니다 |
| 브리지/다이렉트 커넥트 | `bridge/`, `server/`, `remote/` | 런타임 전송 경계가 분리돼 있습니다 |
| 권한 UX | `components/permissions/`, `hooks/toolPermission/` | 권한 요청 UI와 오케스트레이션 핸들러 |
| 기타 중요한 최상위 영역 | `cli/`, `server/`, `skills/`, `tasks/`, `plugins/`, `query/` | 출력 파이프라인, 직접 연결 세션, 스킬 레지스트리, 태스크 런타임, 플러그인 루트, 쿼리 엔진이 별도 축으로 존재합니다 |

## 규약
- 이 디렉터리는 패키지 루트가 아니라 소스 트리입니다. `src/` 아래에는 로컬 `package.json`, `tsconfig`, 린트 설정, CI 워크플로, 일반적인 테스트 하네스가 보이지 않았습니다.
- 아키텍처는 레지스트리 중심입니다. 명령/도구/스킬이 번들, 플러그인, 워크플로, 사용자 로드 소스에서 조립됩니다.
- 구조는 라우트/프레임워크 기준이 아니라 도메인 폴더 기준입니다.
- 권한 모델은 fail-closed입니다. read/search 도구와 write/exec 도구를 다르게 취급하며 allow/deny 필터링이 명시적입니다.
- `.claude/`는 단순 설정 폴더가 아니라 동작 표면의 일부입니다. 계층형 지시문을 항상 염두에 두세요.

## 이 프로젝트의 안티패턴
- 사용자가 명시적으로 요청하지 않았다면 커밋/푸시/훅 우회/깃 설정 변경을 하지 마세요.
- plan 모드에서는 계획 산출물 외의 쓰기를 하지 마세요.
- 기존 파일은 Edit로 충분한데 Write를 쓰지 마세요. 먼저 읽고 수정하세요.
- 공개용/undercover 출력에 Anthropic 내부 정보나 AI 정체성 정보를 노출하지 마세요.
- 공유 메모리나 생성 메모리에 비밀값을 저장하지 마세요.
- 셸 허용목록을 가볍게 넓히지 마세요. Bash/PowerShell 보안 로직은 정책 밀도가 높습니다.

## 이 코드베이스만의 스타일
- 많은 명령 리프는 lazy `load()`를 가진 작은 디스크립터입니다. 실제 동작은 `index.ts[x]` 아래 더 깊은 곳에 있는 경우가 많습니다.
- `tools/`는 도구 하나당 디렉터리 하나 구조이며, Zod 스키마 + 프롬프트 + 동시성/읽기 전용 메타데이터를 함께 둡니다.
- `components/`는 도메인 분할이고, `ink/`는 앱 기능 UI가 아니라 인프라 계층입니다.
- `utils/`는 단순 잡동사니 폴더가 아니라 여러 서브시스템(`bash`, `plugins`, `settings`, `permissions`, `background`)의 집합입니다.

## 명령 실행 메모
```bash
# src/ 아래에는 로컬 build/test/lint 진입점이 없습니다.
# 실제 프로젝트 루트로 올라가서 명령을 실행해야 합니다.
```

## 메모
- 대형 핫스팟 파일: `cli/print.ts`, `utils/messages.ts`, `utils/sessionStorage.ts`, `utils/hooks.ts`, `screens/REPL.tsx`, `main.tsx`, `services/api/claude.ts`, `services/mcp/client.ts`, `bridge/bridgeMain.ts`.
- 생성 코드가 `types/generated/` 아래에 있습니다. 생성 규칙을 건드리는 작업이 아니라면 부모 지침만 따르는 편이 안전합니다.
- 우선순위가 높은 하위 지식 베이스: `commands/`, `tools/`, `services/`, `components/`, `components/permissions/`, `utils/`, `utils/bash/`, `utils/plugins/`, `bridge/`, `ink/`, `hooks/toolPermission/`, `entrypoints/`, `remote/`.
