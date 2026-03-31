# utils 지식 베이스

## 개요
`utils/`는 잡동사니 폴더가 아닙니다. 이 저장소에서 가장 큰 공용 로직 축이며, 여러 서브시스템 규모의 클러스터를 품고 있습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 설정 fan-in 핫스팟 | `settings/` | utils 안에서 의존 밀도가 가장 높음 |
| 셸 파싱/보안 | `bash/` | parser, AST, allowlist, quoting, specs |
| 플러그인 생명주기 | `plugins/` | loader, marketplace, validation, startup checks |
| 세션/대화 저장 | `sessionStorage.ts`, `memory/`, `messages.ts` | persistence + message shaping |
| 훅 런타임 | `hooks.ts`, `hooks/` | 거대한 오케스트레이션 표면 |
| 권한/설정 | `permissions/`, `config.ts`, `claudemd.ts` | 정책과 지시문 표면 |

## 구조
```text
utils/
├── bash/
├── plugins/
├── settings/
├── permissions/
├── background/
├── memory/
└── 수많은 최상위 공용 모듈
```

## 규약
- `utils/`는 여러 미니 서브시스템의 집합으로 봐야 합니다. 폴더 이름만 보고 로컬 영향이라고 단정하면 안 됩니다.
- `utils/settings/settings.ts`는 fan-in이 매우 높아서 변경 파급이 큽니다.
- `messages.ts`, `sessionStorage.ts`, `hooks.ts`, `attachments.ts` 같은 mega-file이 이 트리에 있습니다.

## 안티패턴
- 이미 큰 파일이라는 이유만으로 무관한 헬퍼를 기존 핫스팟 파일에 더 넣지 마세요.
- parser/security/plugin 코드를 건드리면서 `bash/`와 `plugins/` 하위 AGENTS를 보지 않는 실수를 하지 마세요.
- 최상위 utility 파일을 저위험 코드로 착각하지 마세요.

## 메모
- 현재 가장 좋은 하위 경계는 `bash/`와 `plugins/`입니다.
- 계층을 더 키운다면 그 다음 후보는 `settings/`입니다.
