# commands 지식 베이스

## 개요
`commands/`는 명령 표면 전체를 담습니다. 대부분은 얇은 디스크립터이지만, 일부 다중 파일 명령군은 자체 규칙을 가질 만큼 무겁습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 레지스트리 동작 변경 | `../commands.ts` | 병합 순서, 필터링, hidden/enabled 로직은 이 디렉터리 밖에 있습니다 |
| 작은 명령 추가 | `commands/<name>/index.ts[x]` | 보통 lazy `load()`를 가진 디스크립터만 export 합니다 |
| 여러 파일에 걸친 명령 흐름 수정 | `context/`, `plugin/`, `session/`, `tasks/`, `skills/`, `help/`, `review/`, `fast/` | 로컬 하위 트리 중 가장 무거운 편입니다 |
| alias/visibility 수정 | 개별 디스크립터 + `../types/command.ts` | `aliases`, `immediate`, `isHidden`, `supportsNonInteractive` 확인 |
| 내장 명령 이름 확인 | `../commands.ts` | 외부에 노출되는 이름의 기준점 |

## 구조
```text
commands/
├── <여러 명령 폴더>/          # 대부분 디스크립터 중심
├── mcp/ permissions/ plan/   # 로컬 흐름이 비교적 무거운 명령군
├── plugin/                   # 플러그인 명령군
├── session/                  # 세션 관련 흐름
├── tasks/                    # 태스크 상호작용 명령
├── skills/                   # 스킬 관련 명령군
└── help/ review/ fast/ ...   # 기타 무거운 기능군
```

## 규약
- 많은 리프 폴더는 전체 구현이 아니라 래퍼입니다.
- 명령 객체는 `prompt`, `local`, `local-jsx` 중 하나일 수 있으며, 같은 논리 명령이 여러 모드를 노출할 수 있습니다.
- availability 필터링이 `isEnabled()`보다 먼저 적용됩니다.
- `loadedFrom` 같은 플러그인 출처 정보는 모델의 일부이므로 없애면 안 됩니다.

## 안티패턴
- 리프 명령 안에 레지스트리 로직을 다시 구현하지 마세요.
- 모든 명령이 인터랙티브하다고 가정하지 마세요. 빠른 경로와 noninteractive 지원이 많습니다.
- 얇은 디스크립터 폴더에 실제 로직이 없는데도 무거운 지역 규칙을 붙이지 마세요.

## 메모
- 이 트리 안에서 별도 AGENTS가 더 필요해질 후보는 단일 export 래퍼보다 다중 파일 명령군입니다.
- 현재 변경이 잦아 보이는 군집은 `mcp/`, `permissions/`, `plan/`, `session/`, `plugin/`, `skills/`, `tasks/`입니다.
- 명령 구현이 안 보인다고 느껴지면 먼저 `main.tsx`와 `commands.ts`를 읽으세요.
