# utils/bash 지식 베이스

## 개요
`utils/bash/`는 Bash 실행 아래쪽에 깔린 셸 분석/파싱 계층입니다. 명령 안전성 판단의 기반이 되는 파서와 AST 분석이 여기 있습니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 파서 핵심 | `bashParser.ts`, `parser.ts`, `ParsedCommand.ts` | 주 파싱 파이프라인 |
| AST/보안 분석 | `ast.ts`, `treeSitterAnalysis.ts` | 의미 분석, allowlisting |
| 명령 분해/레지스트리 | `commands.ts`, `registry.ts`, `bashPipeCommand.ts` | 실행 형태 분석 헬퍼 |
| prefix/quoting | `prefix.ts`, `shellPrefix.ts`, `shellQuote.ts`, `shellQuoting.ts` | 명령 정규화 |
| specs 데이터 | `specs/` | 파싱/검증에 쓰이는 셸 사양 입력 |

## 규약
- 이 계층은 parser-first입니다. AST 경로가 있는데 ad hoc regex로 대체하지 마세요.
- 도구 레벨 안전성은 `tools/BashTool/`에 있습니다. 파서와 정책 책임을 섞지 마세요.
- `specs/`는 데이터처럼 보여도 동작상 중요합니다.

## 안티패턴
- 하나의 edge case를 고치려고 셸 파싱 규칙을 완화하면서 downstream 권한/보안 영향을 보지 않는 변경을 하지 마세요.
- quoting/prefix 로직을 호출자 쪽에서 중복 구현하지 마세요.
- `specs/`를 버려도 되는 fixture처럼 취급하지 마세요.

## 메모
- 이 트리의 가장 큰 파일은 `bashParser.ts`와 `ast.ts`입니다.
- 여기서의 변경은 상위 계층 권한 판정 결과를 조용히 바꿀 수 있습니다.
