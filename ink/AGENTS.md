# ink 런타임 지식 베이스

## 개요
`ink/`는 앱 기능 계층이 아니라 터미널 렌더링/런타임 엔진입니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 공개 wrapper/export 표면 | `../ink.ts`, `ink.tsx` | 최상위 wrapper가 theme을 주입하고 런타임 조각을 re-export 합니다 |
| 렌더러 핵심 | `renderer.ts`, `render-to-screen.ts`, `render-node-to-output.ts`, `reconciler.ts`, `root.ts` | 렌더링 파이프라인 |
| 레이아웃/이벤트 | `layout/`, `events/`, `focus.ts`, `hit-test.ts`, `dom.ts` | 앱 로직이 아니라 런타임 동작 |
| 내부 재사용 요소 | `components/`, `hooks/` | 앱 컴포넌트가 아니라 런타임 내부 구성요소 |

## 규약
- 앱 UI 관심사를 이 디렉터리에 넣지 마세요.
- `ink/` 내부와 `components/` 앱 UI는 서로 다른 계층입니다.
- 많은 파일이 저수준 렌더링/터미널 추상화이므로 파급 효과가 큽니다.

## 안티패턴
- 제품 전용 dialog 로직을 여기 넣지 마세요.
- `components/` 쪽 앱 레벨 규약을 런타임 내부에 그대로 들여오지 마세요.
- layout/event/render 파일을 영향 추적 없이 가볍게 바꾸지 마세요.

## 메모
- 이 트리는 자체적인 멘탈 모델과 실패 모드가 있어서 별도 AGENTS를 둘 가치가 있습니다.
