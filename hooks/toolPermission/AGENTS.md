# hooks/toolPermission 지식 베이스

## 개요
`hooks/toolPermission/`은 권한 UI 뒤쪽 오케스트레이션 계층입니다. context, logging, interactive/coordinator/swarm worker 경로별 handler 선택을 담당합니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 중앙 context/오케스트레이션 | `PermissionContext.ts` | approval update, analytics, hook execution, abort flow |
| 로깅 | `permissionLogging.ts` | 권한 이벤트 진단 정보 |
| 모드별 handler | `handlers/interactiveHandler.ts`, `handlers/coordinatorHandler.ts`, `handlers/swarmWorkerHandler.ts` | 실행 모드별 분기 |

## 규약
- UI는 `../../components/permissions/`에 있고, decision orchestration은 여기 있습니다.
- execution-mode 분리는 의도된 구조입니다. worker/coordinator/interactive를 함부로 합치지 마세요.
- analytics/logging도 approval path의 일부입니다.

## 안티패턴
- permission routing을 컴포넌트 안에서 다시 구현하지 마세요.
- handler에 있어야 할 분기를 `PermissionContext.ts`에 one-off로 추가하지 마세요.
- approval flow를 바꾸면서 abort/update/logging 부작용을 확인하지 않는 실수를 하지 마세요.

## 메모
- 디렉터리는 작지만 영향력은 큽니다.
- permission component와 관련 tool prompt 파일을 함께 보는 습관이 필요합니다.
