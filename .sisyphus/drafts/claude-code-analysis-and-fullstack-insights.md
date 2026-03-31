# 초안: Claude Code 분석과 풀스택 활용 인사이트

## 확인된 요구사항
- [먼저 분석 후 문서화]: ["일단 먼저 분석을 해서 문서화 합니다"]
- [실무에 도움이 되는 포인트 도출]: ["풀스택개발자로서 클로드코드를 활용할때 도움이 될만한 것들을 도출"]
- [대상 프로젝트]: ["현재 프로젝트는 claude code의 소스코드입니다."]

## 기술적 결정
- [산출물 형태]: [저장소 분석 + 문서화 + 풀스택 개발자용 활용 가이드를 함께 제공]
- [작업 방식]: [소스 구조를 먼저 탐색하고, 고밀도 경계를 중심으로 계층형 문서를 만든 뒤 실전 가이드를 추가]

## 조사 결과
- [워크스페이스 루트]: [`/Users/parkdonghyeon/Dev/src` 아래에 `cli/`, `commands/`, `components/`, `entrypoints/`, `server/`, `services/`, `tools/`, `types/`, `skills/` 같은 대형 디렉터리가 존재]
- [주요 런타임 축]: [`remote/`는 메시징/세션 경계, `services/lsp/`는 LSP 생명주기, `services/mcp/`는 MCP transport/auth/UI glue, `services/analytics/`는 분석 로깅, `services/MagicDocs/`와 `services/PromptSuggestion/`는 문서/보조 기능을 담당]
- [핵심 앵커 파일]: [`remote/RemoteSessionManager.ts`, `remote/SessionsWebSocket.ts`, `services/lsp/LSPServerManager.ts`, `services/mcp/InProcessTransport.ts`, `services/mcp/MCPConnectionManager.tsx`, `services/analytics/index.ts`, `projectOnboardingState.ts`]
- [구조 특성]: [패키지 루트보다 소스 서브트리에 가까우며, 실제 build/test/lint 명령은 상위 프로젝트 루트에서 찾아야 함]
- [복잡도 핫스팟]: [`cli/print.ts`, `utils/messages.ts`, `utils/sessionStorage.ts`, `utils/hooks.ts`, `screens/REPL.tsx`, `main.tsx`, `services/api/claude.ts`, `services/mcp/client.ts`, `bridge/bridgeMain.ts`]

## 남아 있던 질문과 정리 결과
- [문서 표면]: [기존 전용 아키텍처 문서는 없었고, 이번에 `AGENTS.md` 계층을 새로 생성함]
- [검증 인프라]: [`src/` 바로 아래에는 로컬 manifest/CI/test harness가 보이지 않았음]
- [문서 언어]: [최종 사용자가 한국어 개발자이므로 한국어 기준으로 정리]

## 범위 경계
- INCLUDE: [코드베이스 분석, 계층형 AGENTS 문서화, 소스 기반 실전 활용 가이드]
- EXCLUDE: [문서 외의 소스 코드 동작 변경]
