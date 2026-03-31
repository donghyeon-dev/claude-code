# components 지식 베이스

## 개요
`components/`는 앱 UI의 주 표면입니다. 규모가 크고 도메인별로 분리돼 있으며, `screens/`보다 훨씬 넓은 범위를 담당합니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 앱 셸 | `App.tsx`, `../screens/REPL.tsx`, `../interactiveHelpers.tsx` | 메인 인터랙티브 흐름은 하나의 기능 폴더 안에만 있지 않습니다 |
| 디자인 프리미티브 | `design-system/`, `ui/`, `Spinner/`, `CustomSelect/` | 재사용 UI 바닥층 |
| 메시지 렌더링 | `messages/`, `Message*.tsx` | 대화 UI 표면이 두꺼운 편 |
| 입력/프롬프트 흐름 | `PromptInput/`, `ContextSuggestions.tsx`, `QuickOpenDialog.tsx` | 입력 중심 UI |
| 권한 UX | `permissions/` | 별도 child AGENTS가 있는 정책 밀집 구역 |
| MCP/설정 다이얼로그 | `mcp/`, `Settings/`, 루트 레벨 다수의 dialog 컴포넌트 | 일회성처럼 보여도 중요도가 높음 |

## 구조
```text
components/
├── design-system/
├── permissions/
├── PromptInput/
├── messages/
├── mcp/
├── agents/
├── tasks/
├── teams/
└── 다수의 루트 레벨 dialog/status 컴포넌트
```

## 규약
- 이 트리는 presentational/container 엄격 분리보다 도메인 분할이 우선입니다.
- 중요한 다이얼로그가 기능 폴더 안이 아니라 루트 레벨 컴포넌트로 존재하는 경우가 많습니다.
- `screens/`는 작고, 재사용되거나 오래 살아남는 UI는 대부분 이 트리에 있습니다.

## 안티패턴
- 권한 흐름 전용 로직을 generic UI primitive에 넣지 마세요.
- `ink/` 런타임 내부를 앱 컴포넌트처럼 다루지 마세요.
- 파일명만 보고 ownership을 추론하지 마세요. 루트 레벨 파일 상당수는 shared primitive가 아니라 entry dialog입니다.

## 메모
- 이 트리에서 더 깊은 child 규칙이 필요하다면 우선순위 1위는 `permissions/`입니다.
- `interactiveHelpers.tsx`와 `dialogLaunchers.tsx`는 루트에 있지만 UI 오케스트레이션의 일부입니다.
