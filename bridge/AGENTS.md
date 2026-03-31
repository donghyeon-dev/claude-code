# bridge 지식 베이스

## 개요
`bridge/`는 direct-connect/bridge 런타임입니다. 세션 bootstrap, 메시징, polling, reconnect, UI status, transport glue를 담당합니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 코어 런타임 루프 | `bridgeMain.ts` | 가장 큰 bridge 핫스팟 |
| REPL bootstrap/transport | `replBridge.ts`, `replBridgeTransport.ts`, `initReplBridge.ts` | 세션 wiring |
| API/config/types | `bridgeApi.ts`, `bridgeConfig.ts`, `types.ts` | bridge 외부 계약 |
| 메시징/권한 | `bridgeMessaging.ts`, `bridgePermissionCallbacks.ts`, `inboundMessages.ts`, `inboundAttachments.ts` | bridge 이벤트 표면 |
| 세션/제어 | `createSession.ts`, `sessionRunner.ts`, `remoteBridgeCore.ts` | 장수 transport/session 동작 |

## 규약
- 이 디렉터리는 UI 기능 코드가 아니라 런타임 transport glue 계층입니다.
- polling, reconnect, heartbeat, cleanup은 모두 1급 관심사입니다.
- session/worktree tracking은 generic utility가 아니라 bridge 동작입니다.

## 안티패턴
- bridge 상태 전이를 무관한 유틸리티에 숨기지 마세요.
- UI 전용 관심사를 transport core 파일에 섞지 마세요.
- reconnect/cleanup을 바꾸면서 disconnect/status update 경로를 안 보는 실수를 하지 마세요.

## 메모
- `bridgeMain.ts`와 `replBridge.ts`가 복잡도의 중심입니다.
- 버그가 “원격 쪽 같은데 서버는 아닌” 느낌이면 우선 이 디렉터리를 의심하세요.
