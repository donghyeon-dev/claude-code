# Claude Code 소스 기반 풀스택 개발 활용 가이드

## 문서 목적
이 문서는 Claude Code의 `src/` 구조를 바탕으로, **풀스택 개발자가 이 코드베이스를 읽고 활용할 때 어디서부터 봐야 하는지**, **어떤 방식으로 확장 포인트를 찾는지**, **어디를 건드리면 위험한지**를 빠르게 이해하도록 돕기 위한 실무용 가이드입니다.

## 이 코드베이스를 어떻게 이해하면 좋은가
이 저장소는 전형적인 웹앱 구조가 아닙니다. 라우트나 페이지 기준이 아니라 **명령(command) / 도구(tool) / 서비스(service) / UI(component) / 런타임(ink, bridge, remote)** 기준으로 나뉘어 있습니다.

즉, 기능 하나를 추가하거나 수정할 때도 보통 한 파일만 보면 끝나지 않습니다. 아래처럼 **입력 → 오케스트레이션 → 권한/도구 → 서비스 → UI 표시** 흐름으로 따라가는 습관이 중요합니다.

### 기본 멘탈 모델
1. 사용자가 명령이나 입력을 보냅니다.
2. `main.tsx`, `commands.ts`, `tools.ts`가 어떤 경로로 처리할지 결정합니다.
3. 필요한 서비스 계층(`services/api`, `services/mcp`, `services/lsp`, `services/analytics`)이 실제 동작을 수행합니다.
4. 결과는 `components/`, `screens/REPL.tsx`, `cli/print.ts`, `ink/`를 통해 사용자에게 보입니다.

## 목적별 추천 진입점

### 1) “CLI가 어떻게 시작되는지” 보고 싶을 때
- `entrypoints/cli.tsx`
- `entrypoints/init.ts`
- `setup.ts`
- `main.tsx`

이 네 파일을 순서대로 보면 초기 진입, 환경 설정, 런타임 초기화, REPL/명령 실행 흐름을 연결해서 볼 수 있습니다.

### 2) “새 명령을 어디에 붙여야 하는지” 알고 싶을 때
- `commands.ts`
- `commands/<명령명>/index.ts[x]`
- 필요하면 `main.tsx`

핵심은 `commands/` 폴더가 곧 전체 구현이 아니라는 점입니다. 많은 폴더가 **얇은 디스크립터**만 갖고 있고, 실제 로직은 더 깊은 파일이나 다른 계층에 있습니다.

### 3) “도구(tool)를 추가하거나 권한 동작을 바꾸고 싶을 때”
- `Tool.ts`
- `tools.ts`
- `tools/BashTool/`
- `components/permissions/`
- `hooks/toolPermission/`

이 영역은 단순 UI가 아니라 **정책과 안전성의 중심**입니다. 특히 Bash 계열은 `tools/BashTool/`와 `utils/bash/`를 함께 봐야 합니다.

### 4) “LLM 호출/MCP/LSP/원격 세션을 이해하고 싶을 때”
- LLM/API: `services/api/claude.ts`, `services/api/client.ts`
- MCP: `services/mcp/`, `tools/MCPTool/`, `components/mcp/`
- LSP: `services/lsp/`
- 원격 세션: `remote/`, `bridge/`, `server/`

이 네 축은 풀스택 개발자가 Claude Code를 고도화할 때 가장 자주 부딪히는 통합 지점입니다.

## 풀스택 개발자 관점에서 특히 유용한 포인트

## 1. “기능”보다 “경계(boundary)”를 먼저 찾아야 합니다
이 코드베이스는 기능명보다 **경계 파일**이 더 중요합니다.

예를 들어:
- 명령 경계: `commands.ts`
- 도구 경계: `Tool.ts`, `tools.ts`
- 권한 경계: `components/permissions/`, `hooks/toolPermission/`
- 셸 안전성 경계: `tools/BashTool/`, `utils/bash/`
- 원격 경계: `remote/`, `bridge/`
- 렌더링 경계: `components/` vs `ink/`

실무적으로는 “이 기능을 어디에 넣지?”보다 먼저 “이 기능이 어느 경계를 통과하나?”를 묻는 편이 더 잘 맞습니다.

## 2. UI를 바꿀 때 `components/`만 보면 반쪽만 보게 됩니다
겉으로는 React 컴포넌트처럼 보여도, 실제 사용자 경험은 아래 계층이 함께 만듭니다.

- `components/`: 앱 UI 표면
- `screens/REPL.tsx`: 메인 상호작용 허브
- `cli/print.ts`: 출력 파이프라인
- `ink/`: 렌더링 런타임

즉 “UI 변경”이라도 경우에 따라서는 `components/`가 아니라 `screens/REPL.tsx`, `cli/print.ts`, `ink/`를 같이 봐야 합니다.

## 3. 플러그인 구조는 한 폴더에 다 있지 않습니다
플러그인 관련 로직은 두 축으로 나뉩니다.

- `utils/plugins/`: discovery, loading, validation, marketplace, startup
- `services/plugins/`: 서비스 계층 operation/CLI 연동

풀스택 개발자가 확장 포인트를 찾을 때 흔히 `services/plugins/`만 보고 끝내거나, 반대로 `utils/plugins/`만 보고 전체라고 착각하기 쉽습니다. 이 둘을 역할 분리된 계층으로 보는 것이 중요합니다.

## 4. Bash/PowerShell/파일 권한 영역은 기능보다 정책이 먼저입니다
Claude Code는 도구 실행보다 **정책**을 우선합니다. 그래서 Bash나 파일 편집 기능을 바꿀 때는 다음을 같이 봐야 합니다.

- 도구 정의: `tools/BashTool/`, `tools/FileEditTool/`, `tools/FileWriteTool/`
- 권한 UI: `components/permissions/`
- 권한 훅: `hooks/toolPermission/`
- 셸 파서: `utils/bash/`

이 영역은 단순한 “실행 기능”이 아니라 **사용자 승인 흐름 + 안전성 검사 + 렌더링 + 정책 텍스트**가 한 묶음입니다.

## 5. 대형 파일은 “정리 대상”이기 전에 “의존성 허브”입니다
다음 파일들은 단순히 큰 파일이 아니라, 시스템의 **실질적 허브** 역할을 합니다.

- `main.tsx`
- `screens/REPL.tsx`
- `cli/print.ts`
- `utils/messages.ts`
- `utils/sessionStorage.ts`
- `utils/hooks.ts`
- `services/api/claude.ts`
- `services/mcp/client.ts`
- `bridge/bridgeMain.ts`

이 파일을 수정할 때는 “이걸 나누자”보다 먼저 “이 파일이 어떤 경계를 대신 붙잡고 있나?”를 파악해야 합니다.

## 추천 학습 순서

### A. 빠르게 전체 그림 잡기
1. `AGENTS.md`
2. `entrypoints/AGENTS.md`
3. `commands/AGENTS.md`
4. `tools/AGENTS.md`
5. `services/AGENTS.md`

### B. 실제 기능 흐름 읽기
1. `main.tsx`
2. `commands.ts`
3. `tools.ts`
4. `screens/REPL.tsx`
5. `services/api/claude.ts`

### C. 안전성과 권한 흐름 이해하기
1. `tools/BashTool/AGENTS.md`
2. `utils/bash/AGENTS.md`
3. `components/permissions/AGENTS.md`
4. `hooks/toolPermission/AGENTS.md`

### D. 확장성 이해하기
1. `utils/plugins/AGENTS.md`
2. `services/AGENTS.md`
3. `remote/AGENTS.md`
4. `bridge/AGENTS.md`

## 실제 활용 시나리오

### 시나리오 1: 새 도구를 추가하고 싶다
1. `Tool.ts`, `tools.ts`에서 도구 계약과 조립 방식을 본다.
2. 비슷한 도구 디렉터리를 하나 고른다.
3. read-only / concurrency / defer / prompt / validator가 어떤 방식으로 정의되는지 맞춘다.
4. 권한 UX가 필요한지 `components/permissions/`, `hooks/toolPermission/`에서 확인한다.

### 시나리오 2: Bash 도구의 판정이 이상하다
1. `tools/BashTool/bashPermissions.ts`
2. `tools/BashTool/bashSecurity.ts`
3. `utils/bash/bashParser.ts`
4. `utils/bash/ast.ts`
5. `components/permissions/BashPermissionRequest/`

이 순서로 보면 정책, 보안, 파싱, UI를 한 흐름으로 따라갈 수 있습니다.

### 시나리오 3: 원격 세션이나 브리지 쪽 버그를 추적한다
1. `remote/RemoteSessionManager.ts`
2. `remote/SessionsWebSocket.ts`
3. `bridge/bridgeMain.ts`
4. `bridge/replBridge.ts`
5. `server/` 관련 진입 파일

remote와 bridge를 분리해서 보되, 실제 디버깅은 둘을 같이 봐야 합니다.

## 수정할 때 특히 조심할 영역
- `tools/BashTool/`와 `utils/bash/`: 보안/정책/파싱이 함께 엮임
- `components/permissions/`와 `hooks/toolPermission/`: UI와 decision routing이 분리돼 있음
- `utils/plugins/`: 로더/검증/시작 정합화가 분리돼 있음
- `ink/`: 앱 UI가 아니라 렌더링 엔진
- `main.tsx`, `screens/REPL.tsx`: 변화의 파급 범위가 큼

## 추천 문서 습관
이 코드베이스를 읽으면서는 기능 설명 문서보다 **경계 문서**를 만드는 것이 더 효과적입니다. 그래서 AGENTS 계층을 먼저 읽고, 그다음 실제 파일로 내려가는 방식이 잘 맞습니다.

실무적으로는 다음 세 가지를 항상 같이 기록해 두는 편이 좋습니다.

1. 이 변경이 통과하는 경계는 어디인가?
2. 사용자 승인/권한 흐름에 영향이 있는가?
3. UI 변경인가, 렌더링 엔진 변경인가, 서비스 변경인가?

이 세 질문만 유지해도 이 저장소에서는 실수할 확률이 크게 줄어듭니다.
