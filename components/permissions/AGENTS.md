# components/permissions 지식 베이스

## 개요
`components/permissions/`는 권한 요청 UI 클러스터입니다. 도구/행동별 요청 패밀리와 공용 다이얼로그 프레이밍, 규칙 설명이 이곳에 모여 있습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 공용 셸 | `PermissionDialog.tsx`, `PermissionRequest.tsx`, `PermissionPrompt.tsx`, `PermissionRequestTitle.tsx` | 모든 요청 패밀리의 공통 프레임 |
| 도구/행동별 요청 UI | `BashPermissionRequest/`, `PowerShellPermissionRequest/`, `FileEditPermissionRequest/`, `FileWritePermissionRequest/`, `NotebookEditPermissionRequest/`, `SkillPermissionRequest/`, `WebFetchPermissionRequest/` 등 | 각 패밀리가 자기 행동 전용 UI를 가집니다 |
| 규칙 설명 | `rules/`, `PermissionRuleExplanation.tsx`, `PermissionExplanation.tsx` | 사용자에게 보여주는 정책 텍스트 |
| 다이얼로그 헬퍼 | `FilePermissionDialog/`, `shellPermissionHelpers.tsx`, `useShellPermissionFeedback.ts` | 공용 affordance 모음 |

## 규약
- 시각 프리미티브 기준이 아니라 permission type 기준으로 조직돼 있습니다.
- 공용 프레임과 도구별 reasoning UI는 의도적으로 분리돼 있습니다.
- 이 UI는 `hooks/toolPermission/` 오케스트레이션 및 도구 프롬프트 정책과 강하게 결합됩니다.

## 안티패턴
- 도구별 edge case를 generic permission shell로 끌어올리지 마세요.
- 여기의 정책 문구를 바꾸면서 대응하는 tool prompt와 hook handler를 확인하지 않는 실수를 하지 마세요.
- hook 쪽 decision logic을 컴포넌트 쪽에서 다시 구현하지 마세요.

## 메모
- 이 저장소에서 가장 강한 deep-boundary 후보 중 하나입니다.
- 권한 흐름을 바꿀 때는 이 디렉터리와 `../../hooks/toolPermission/AGENTS.md`를 같이 읽으세요.
