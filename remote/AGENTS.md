# remote 지식 베이스

## 개요
`remote/`는 원격 세션 경계 계층입니다. websocket/session lifecycle, permission bridging, CCR SDK 메시지 적응이 여기서 일어납니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 원격 세션 생명주기 | `RemoteSessionManager.ts` | 메인 remote 오케스트레이션 포인트 |
| WebSocket transport | `SessionsWebSocket.ts` | 세션 이벤트/메시지 네트워크 경계 |
| 메시지 적응 | `sdkMessageAdapter.ts` | CCR SDK 메시지를 로컬 형식으로 번역 |
| 권한 브리징 | `remotePermissionBridge.ts` | synthetic assistant message와 tool/approval 브리지 |

## 규약
- 이 디렉터리는 일반 기능 모듈이 아니라 boundary adapter layer입니다.
- message-shape translation과 permission bridging이 핵심 책임입니다.
- `bridge/`, `server/`와 역할이 겹쳐 보일 수 있지만, 여기서는 remote-session 쪽 절반을 담당합니다.

## 안티패턴
- 메시지 adaptation 로직을 무관한 utility로 흩뜨리지 마세요.
- local-only UI 관심사를 remote transport/session 코드에 섞지 마세요.
- permission bridging을 바꾸면서 downstream permission UI/hook 흐름을 확인하지 않는 실수를 하지 마세요.

## 메모
- 파일 수는 적지만 전부 1급 경계 파일입니다.
- remote/direct-connect 경계 버그를 볼 때는 `bridge/AGENTS.md`와 함께 읽는 편이 좋습니다.
