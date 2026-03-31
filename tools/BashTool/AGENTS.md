# BashTool 지식 베이스

## 개요
`tools/BashTool/`은 Bash 실행의 안전 경계입니다. 권한 분류, 파괴적 명령 처리, 경로/모드 검증, UI/결과 렌더링이 이곳에 모여 있습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 권한 판정 로직 | `bashPermissions.ts` | classifier + allow/ask/deny 흐름 |
| 명령 안전성 분석 | `bashSecurity.ts` | heredoc, substitution, wrapper stripping, 경로/read-only 검사 |
| 도구 정의 | `BashTool.tsx`, `prompt.ts` | 사용자-facing 계약과 정책 텍스트 |
| 검증 헬퍼 | `modeValidation.ts`, `pathValidation.ts`, `readOnlyValidation.ts`, `sedValidation.ts` | 정책 로컬 검사는 이 계층에 둠 |
| UI/결과 출력 | `UI.tsx`, `BashToolResultMessage.tsx` | 셸 결과의 사용자 표시 형식 |

## 규약
- 보안 로직과 권한 로직은 의도적으로 분리돼 있습니다. 섣불리 합치지 마세요.
- `../utils/bash/`는 동료 계층의 파싱/분석 의존성이지, 도구 레이어 검증을 대체하는 계층이 아닙니다.
- 프롬프트 텍스트도 정책의 일부입니다. 단순 복사 문구로 취급하면 안 됩니다.

## 안티패턴
- allowlist를 넓히거나 regex/path 체크를 완화할 때 downstream 영향을 추적하지 않고 바꾸지 마세요.
- wrapper stripping, heredoc/substitution 탐지를 우회하지 마세요.
- 권한 메시징을 무관한 UI 컴포넌트로 옮기지 마세요.

## 메모
- 이 디렉터리는 트리 전체에서 가장 정책 밀도가 높은 보안 핫스팟 중 하나입니다.
- 셸 의미론이 걸린 변경이라면 `../utils/bash/AGENTS.md`도 함께 읽어야 합니다.
