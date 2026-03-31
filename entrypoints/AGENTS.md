# entrypoints 지식 베이스

## 개요
`entrypoints/`는 얇은 프로세스 bootstrap shim 모음입니다. CLI, MCP 서버, init 시점 환경/설정 wiring이 이 계층에서 시작됩니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 메인 CLI 프로세스 진입점 | `cli.tsx` | 전체 런타임 import 전에 fast-path flag/subcommand를 처리 |
| 시작 초기화 | `init.ts` | config/env, telemetry, proxy/mTLS, cleanup, remote/policy setup |
| MCP stdio 서버 진입점 | `mcp.ts` | 독립적인 MCP 노출 경로 |
| 공용 entrypoint 타입 | `agentSdkTypes.ts`, `sandboxTypes.ts`, `sdk/` | 경계 타입과 SDK 글루 |

## 규약
- 이 파일들은 얇게 유지해야 합니다. 장수 오케스트레이션은 `main.tsx`, `setup.ts`, 각 서브시스템 코드에 둬야 합니다.
- entrypoint는 feature logic보다 env shim, early exit, bootstrap ordering, process-specific wiring을 담당합니다.

## 안티패턴
- 시작점이라는 이유만으로 기능 로직을 entrypoint 파일 안에 밀어 넣지 마세요.
- 이미 `main.tsx`나 `entrypoints/init.ts`에 모인 setup 로직을 다시 복제하지 마세요.

## 메모
- 작지만 영향력 큰 트리입니다. startup regression은 실제 버그가 더 깊은 곳에 있어도 여기서 처음 드러나는 경우가 많습니다.
