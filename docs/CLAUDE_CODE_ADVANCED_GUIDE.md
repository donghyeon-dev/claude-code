# Claude Code 풀스택 개발자를 위한 심화 가이드

> 소스코드 분석 기반의 실전 활용 전략

---

## 목차

1. [CLAUDE.md 계층 활용 전략](#1-claudemd-계층-활용-전략)
2. [settings.json 권한 규칙 실전 패턴](#2-settingsjson-권한-규칙-실전-패턴)
3. [Hooks 시스템 활용 레시피](#3-hooks-시스템-활용-레시피)
4. [컨텍스트 관리 전략](#4-컨텍스트-관리-전략)
5. [Agent Tool/서브에이전트 활용 패턴](#5-agent-tool서브에이전트-활용-패턴)
6. [쿼리 루프 이해](#6-쿼리-루프-이해)
7. [Permission 시스템 마스터](#7-permission-시스템-마스터)
8. [성능 최적화](#8-성능-최적화)
9. [MCP 서버 통합 실전](#9-mcp-서버-통합-실전)
10. [커스텀 스킬 고급 작성법](#10-커스텀-스킬-고급-작성법)

---

## 1. CLAUDE.md 계층 활용 전략

### 계층 구조 개요

Claude Code는 CLAUDE.md 파일을 다중 계층으로 로드한다. 소스코드 분석에 따르면 로딩 우선순위는 다음과 같다:

```
~/.claude/CLAUDE.md          ← 전역 (모든 프로젝트에 적용)
{project}/.claude/CLAUDE.md  ← 프로젝트 루트 로컬
{project}/src/CLAUDE.md      ← 서브디렉토리 (해당 디렉토리 작업 시)
{project}/src/api/CLAUDE.md  ← 더 깊은 서브디렉토리
```

Claude는 현재 작업 디렉토리에서 상위로 순회하며 CLAUDE.md를 수집하고, 하위 디렉토리의 내용도 동적으로 포함한다. 이 계층 메커니즘을 활용하면 팀 전체 규칙, 프로젝트 규칙, 모듈별 규칙을 분리해서 관리할 수 있다.

---

### 프론트엔드 전용 CLAUDE.md 예시

```markdown
# Frontend CLAUDE.md

## 기술 스택
- React 18 + TypeScript 5.x
- Tailwind CSS v3
- Vite 5.x 빌드 도구
- Vitest + Testing Library

## 컴포넌트 작성 규칙
- 컴포넌트는 항상 named export 사용 (default export 금지)
- props 타입은 컴포넌트 파일 내 `interface Props` 로 정의
- 상태 관리: useState → useReducer → Zustand 순으로 복잡도에 따라 선택
- 사이드이펙트는 항상 useEffect 의존성 배열을 명시적으로 지정

## 파일 구조 규칙
- 컴포넌트: `src/components/{Feature}/{ComponentName}.tsx`
- 훅: `src/hooks/use{Name}.ts`
- 유틸: `src/utils/{name}.ts`
- 타입: `src/types/{domain}.ts`

## 금지 패턴
- `any` 타입 사용 금지 (unknown 사용)
- inline style 금지 (Tailwind 클래스 사용)
- console.log 커밋 금지

## 테스트 규칙
- 모든 UI 컴포넌트는 스냅샷 테스트 필수
- 사용자 인터랙션 테스트는 userEvent 사용
- `describe` 블록으로 기능 단위 그룹화

## 빌드/검증 명령어
- 타입 체크: `pnpm tsc --noEmit`
- 린트: `pnpm eslint src --ext .ts,.tsx`
- 테스트: `pnpm vitest run`
```

---

### 백엔드 전용 CLAUDE.md 예시

```markdown
# Backend CLAUDE.md

## 기술 스택
- Node.js 20 LTS + TypeScript
- Fastify 4.x (Express 사용 금지)
- Prisma ORM + PostgreSQL 15
- Zod 스키마 검증

## API 설계 규칙
- RESTful: `GET /resources`, `POST /resources`, `PATCH /resources/:id`
- 에러 응답 형식: `{ error: { code: string, message: string, details?: unknown } }`
- 성공 응답 형식: `{ data: T, meta?: { page, total } }`
- 모든 입력값은 Zod 스키마로 검증 후 사용

## 데이터베이스 규칙
- 직접 SQL 금지: 반드시 Prisma Client 사용
- N+1 쿼리 방지: include/select 명시적 지정
- 트랜잭션: `prisma.$transaction()` 사용
- 마이그레이션: `prisma migrate dev` 실행 후 커밋

## 보안 규칙
- 환경변수: 모든 시크릿은 `.env` 파일에서 로드 (하드코딩 절대 금지)
- SQL Injection 방지: Prisma 파라미터화 쿼리만 사용
- 인증: JWT 토큰은 httpOnly 쿠키로 전달

## 성능 규칙
- 데이터베이스 조회 결과는 Redis 캐싱 고려 (TTL 명시)
- 무거운 작업은 BullMQ 큐로 오프로드

## 검증 명령어
- 타입 체크: `pnpm tsc -p tsconfig.server.json --noEmit`
- 테스트: `pnpm jest --testPathPattern=server`
- DB 마이그레이션 상태: `pnpm prisma migrate status`
```

---

### 풀스택 프로젝트 루트 CLAUDE.md 예시

```markdown
# 프로젝트 루트 CLAUDE.md

## 프로젝트 개요
- 모노레포 구조: `packages/web`, `packages/api`, `packages/shared`
- 패키지 매니저: pnpm workspaces
- CI/CD: GitHub Actions → Vercel(FE) + Railway(BE)

## 작업 전 필수 확인사항
1. 변경 전 `git status` 확인
2. 기능 브랜치에서 작업 (`feat/`, `fix/`, `chore/` 접두사)
3. 커밋 전 `pnpm run check` 실행 (타입 + 린트 + 테스트)

## 전체 프로젝트 공통 규칙
- 언어: 코드는 영어, 주석/문서는 한국어
- 로깅: `console.log` 대신 `logger.info()` (packages/shared/logger 사용)
- 에러 처리: 모든 async 함수는 try-catch 또는 Result 타입 사용

## 공유 타입
- `packages/shared/types/` 의 타입을 웹/API에서 공유
- API 응답 타입 변경 시 shared 패키지 먼저 수정

## 서브 CLAUDE.md 위치
- `packages/web/.claude/CLAUDE.md` - 프론트엔드 세부 규칙
- `packages/api/.claude/CLAUDE.md` - 백엔드 세부 규칙

## 전체 검증 명령어
- 전체 빌드: `pnpm run build --recursive`
- 전체 테스트: `pnpm run test --recursive`
- 타입 체크: `pnpm run typecheck --recursive`
```

---

### 실전 팁

**팁 1: `@파일명` 임포트 활용**

CLAUDE.md 내에서 `@docs/api-spec.md` 처럼 다른 문서를 참조할 수 있다. 대규모 프로젝트에서는 CLAUDE.md를 간결하게 유지하고 세부 내용은 별도 파일로 분리한 뒤 임포트하는 패턴이 효과적이다.

```markdown
# CLAUDE.md

## API 스펙
@docs/api-specification.md 참조

## 코딩 컨벤션
@docs/coding-conventions.md 참조
```

**팁 2: 팀 vs 개인 규칙 분리**

- `.claude/CLAUDE.md` → 팀 공유 (git에 커밋)
- `~/.claude/CLAUDE.md` → 개인 선호 설정 (git 제외)

개인 전역 CLAUDE.md에서는 선호하는 코딩 스타일, 자주 쓰는 도구 경로, 개인 workflow 설정을 관리하면 된다.

---

## 2. settings.json 권한 규칙 실전 패턴

### 기본 구조

settings.json은 `~/.claude/settings.json` (전역) 또는 `.claude/settings.json` (프로젝트)에 위치한다.

```json
{
  "permissions": {
    "allow": [],
    "deny": []
  },
  "env": {},
  "hooks": {}
}
```

소스코드 분석에 따르면 권한 규칙은 glob 패턴 매칭을 사용하며, `deny` 규칙이 `allow` 규칙보다 우선 적용된다.

---

### 패턴 1: Bash 명령어 세밀한 제어

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(npm run *)",
      "Bash(pnpm *)",
      "Bash(ls *)",
      "Bash(cat *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(curl * | bash*)",
      "Bash(wget * | sh*)",
      "Bash(chmod 777 *)"
    ]
  }
}
```

**기술적 근거**: Bash 허용 규칙은 `Bash(패턴)` 형식으로 지정하며, 내부적으로 glob 매칭을 수행한다. `*`는 임의의 문자열에 매칭되며, 공백을 포함한 전체 명령어 문자열에 적용된다.

---

### 패턴 2: 파일 편집 경로 제한

```json
{
  "permissions": {
    "allow": [
      "Edit(src/**)",
      "Edit(tests/**)",
      "Edit(*.md)",
      "Write(src/**)",
      "Write(tests/**)"
    ],
    "deny": [
      "Edit(.env*)",
      "Edit(secrets/**)",
      "Edit(*.pem)",
      "Edit(*.key)",
      "Write(.env*)",
      "Write(secrets/**)"
    ]
  }
}
```

**주의사항**: 경로 패턴은 프로젝트 루트 기준 상대경로다. `.env` 파일 수정을 차단하면 실수로 시크릿을 노출하는 사고를 예방할 수 있다.

---

### 패턴 3: MCP 서버 차단

```json
{
  "permissions": {
    "deny": [
      "mcp__filesystem__write_file",
      "mcp__filesystem__delete_file",
      "mcp__database__execute_query",
      "mcp__browser__navigate"
    ],
    "allow": [
      "mcp__filesystem__read_file",
      "mcp__filesystem__list_directory",
      "mcp__database__query"
    ]
  }
}
```

**기술적 근거**: MCP 도구는 `mcp__{서버명}__{도구명}` 형식으로 참조된다. 서버 전체를 차단하려면 `mcp__{서버명}__*` 패턴을 사용한다.

```json
{
  "permissions": {
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

### 패턴 4: 도구별 완전 허용/거부 (fullAuto 환경)

CI/CD 파이프라인이나 자동화 환경에서 사용하는 패턴:

```json
{
  "permissions": {
    "allow": [
      "Bash(*)",
      "Edit(*)",
      "Write(*)",
      "Read(*)",
      "Glob(*)",
      "Grep(*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(:(){ :|:& };:)",
      "Bash(dd if=*)",
      "Bash(mkfs*)"
    ]
  }
}
```

**주의사항**: fullAuto 모드는 `--dangerously-skip-permissions` 플래그와 함께 사용하거나, PermissionMode를 `bypassPermissions`로 설정할 때 적용된다. 프로덕션 환경에서는 절대 사용하지 않는다.

---

### 패턴 5: 프로젝트별 환경 분리

개발 환경용 `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(docker *)",
      "Bash(kubectl *)",
      "Bash(terraform *)"
    ]
  },
  "env": {
    "NODE_ENV": "development",
    "DATABASE_URL": "postgresql://localhost:5432/dev_db",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "8192"
  }
}
```

프로덕션 배포용 CI 환경 `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm run build)",
      "Bash(pnpm run test)",
      "Bash(pnpm run lint)"
    ],
    "deny": [
      "Bash(rm *)",
      "Bash(git push *)",
      "Edit(*)"
    ]
  }
}
```

---

### 패턴 6: 네트워크 접근 제어

```json
{
  "permissions": {
    "allow": [
      "Bash(curl https://api.github.com/*)",
      "Bash(curl https://registry.npmjs.org/*)",
      "WebFetch(https://docs.anthropic.com/*)"
    ],
    "deny": [
      "Bash(curl http://*)",
      "WebFetch(http://*)"
    ]
  }
}
```

**실전 팁**: `settings.local.json`은 `.gitignore`에 추가해 개인 설정을 팀 설정에서 분리한다.

```gitignore
.claude/settings.local.json
```

---

## 3. Hooks 시스템 활용 레시피

### 훅 실행 흐름 이해

소스코드 분석에 따르면 훅은 다음 시점에 실행된다:

```
SessionStart    → 세션 시작 시 한 번
PreToolUse      → 도구 실행 직전 (차단 가능)
PostToolUse     → 도구 실행 직후
FileChanged     → 파일 변경 감지 시
Notification    → Claude가 알림을 보낼 때
Stop            → Claude가 응답 완료 시
```

훅 스크립트는 `stdin`으로 JSON 이벤트 데이터를 받고, `exit 2`로 실행 차단, `exit 0`으로 정상 통과를 신호한다.

---

### 레시피 1: 위험 명령어 PreToolUse 차단

`.claude/hooks/pre-bash-check.sh`:

```bash
#!/bin/bash
# PreToolUse 훅: 위험한 Bash 명령어 차단

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name // ""')
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

if [ "$TOOL" != "Bash" ]; then
  exit 0
fi

# 위험 패턴 목록
DANGEROUS_PATTERNS=(
  "rm -rf /"
  "rm -rf ~"
  ":(){ :|:& };"
  "dd if=/dev/zero"
  "mkfs"
  "> /dev/sda"
  "chmod -R 777 /"
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qF "$pattern"; then
    echo "차단됨: 위험한 명령어 패턴 감지 - '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

settings.json에 등록:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-bash-check.sh"
          }
        ]
      }
    ]
  }
}
```

---

### 레시피 2: PostToolUse 자동 포맷팅

파일 편집 후 자동으로 prettier를 실행하는 훅:

`.claude/hooks/post-edit-format.sh`:

```bash
#!/bin/bash
# PostToolUse 훅: 편집된 파일 자동 포맷팅

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name // ""')

if [ "$TOOL" != "Edit" ] && [ "$TOOL" != "Write" ]; then
  exit 0
fi

FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // ""')

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

# 지원 파일 타입만 포맷팅
case "$FILE_PATH" in
  *.ts|*.tsx|*.js|*.jsx|*.json|*.css|*.md)
    if command -v prettier &>/dev/null; then
      prettier --write "$FILE_PATH" 2>/dev/null
      echo "포맷팅 완료: $FILE_PATH" >&2
    fi
    ;;
  *.py)
    if command -v black &>/dev/null; then
      black "$FILE_PATH" 2>/dev/null
      echo "Black 포맷팅 완료: $FILE_PATH" >&2
    fi
    ;;
esac

exit 0
```

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/post-edit-format.sh"
          }
        ]
      }
    ]
  }
}
```

---

### 레시피 3: SessionStart 환경 검증

세션 시작 시 필요한 도구와 환경변수를 검증:

`.claude/hooks/session-start.sh`:

```bash
#!/bin/bash
# SessionStart 훅: 개발 환경 사전 검증

ERRORS=()

# 필수 도구 확인
REQUIRED_TOOLS=("git" "node" "pnpm" "docker")
for tool in "${REQUIRED_TOOLS[@]}"; do
  if ! command -v "$tool" &>/dev/null; then
    ERRORS+=("필수 도구 없음: $tool")
  fi
done

# 필수 환경변수 확인
REQUIRED_ENVS=("DATABASE_URL" "REDIS_URL")
for env in "${REQUIRED_ENVS[@]}"; do
  if [ -z "${!env}" ]; then
    ERRORS+=("환경변수 미설정: $env")
  fi
done

# .env 파일 존재 확인
if [ ! -f ".env" ] && [ ! -f ".env.local" ]; then
  ERRORS+=(".env 또는 .env.local 파일이 없습니다")
fi

# 오류 보고
if [ ${#ERRORS[@]} -gt 0 ]; then
  echo "⚠️  환경 검증 실패:" >&2
  for err in "${ERRORS[@]}"; do
    echo "  - $err" >&2
  done
  echo "위 문제를 해결한 후 작업을 시작하세요." >&2
  # exit 2로 세션 차단하거나 exit 0으로 경고만 표시
  exit 0
fi

echo "✅ 환경 검증 완료" >&2
exit 0
```

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

---

### 레시피 4: Notification 훅으로 데스크탑 알림

`.claude/hooks/notify.sh`:

```bash
#!/bin/bash
# Notification 훅: 시스템 알림 전송

INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message // "Claude 알림"')

# macOS
if command -v osascript &>/dev/null; then
  osascript -e "display notification \"$MESSAGE\" with title \"Claude Code\""
fi

# Linux (libnotify)
if command -v notify-send &>/dev/null; then
  notify-send "Claude Code" "$MESSAGE"
fi

exit 0
```

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/notify.sh"
          }
        ]
      }
    ]
  }
}
```

---

### 레시피 5: FileChanged 훅으로 자동 테스트 실행

```bash
#!/bin/bash
# FileChanged 훅: 변경된 파일에 따라 관련 테스트 자동 실행

INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.file // ""')

if [ -z "$FILE" ]; then
  exit 0
fi

# TypeScript 파일 변경 시 해당 테스트 실행
if [[ "$FILE" == *.ts ]] || [[ "$FILE" == *.tsx ]]; then
  TEST_FILE="${FILE%.ts}.test.ts"
  TEST_FILE_ALT="${FILE%.tsx}.test.tsx"

  if [ -f "$TEST_FILE" ]; then
    echo "관련 테스트 실행 중: $TEST_FILE" >&2
    pnpm vitest run "$TEST_FILE" 2>&1 | tail -5 >&2
  elif [ -f "$TEST_FILE_ALT" ]; then
    echo "관련 테스트 실행 중: $TEST_FILE_ALT" >&2
    pnpm vitest run "$TEST_FILE_ALT" 2>&1 | tail -5 >&2
  fi
fi

exit 0
```

---

### 훅 디버깅 팁

훅이 예상대로 동작하지 않을 때:

```bash
# 훅 스크립트 직접 테스트
echo '{"tool_name":"Bash","tool_input":{"command":"rm -rf /"}}' | \
  bash .claude/hooks/pre-bash-check.sh

# 훅 로그 확인 (stderr로 출력된 내용)
CLAUDE_CODE_HOOK_DEBUG=1 claude

# 권한 확인
chmod +x .claude/hooks/*.sh
```

---

## 4. 컨텍스트 관리 전략

### auto compact 메커니즘

소스코드 분석에서 확인된 핵심 임계치:

```
자동 압축 트리거 = contextWindow - 13000 토큰
```

예를 들어 Claude Sonnet의 contextWindow가 200,000 토큰이라면, 사용된 토큰이 **187,000**을 초과하면 자동 압축이 실행된다. 이 `13000` 값은 하드코딩된 버퍼로, 다음 응답을 위한 최소 공간을 확보하기 위함이다.

자동 압축 시 Claude는 대화 이력을 요약하여 컨텍스트를 축소하고, 핵심 정보를 유지한 채 재시작한다.

---

### CLAUDE_CODE_AUTO_COMPACT_WINDOW 환경변수

자동 압축 임계치를 커스터마이즈할 수 있다:

```bash
# 더 일찍 압축 (20,000 토큰 여유 확보)
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=20000

# ~/.zshrc 또는 ~/.bashrc에 추가
echo 'export CLAUDE_CODE_AUTO_COMPACT_WINDOW=20000' >> ~/.zshrc
```

또는 settings.json의 env 섹션에:

```json
{
  "env": {
    "CLAUDE_CODE_AUTO_COMPACT_WINDOW": "20000"
  }
}
```

**설정 가이드라인:**
- 복잡한 멀티파일 작업: `20000` ~ `30000` (더 많은 여유 확보)
- 단순 질문/답변: `8000` ~ `10000` (컨텍스트 최대 활용)
- 기본값: `13000`

---

### /compact 수동 압축

컨텍스트가 많이 쌓였지만 자동 압축 임계치에 도달하지 않았을 때 수동으로 압축:

```
/compact

# 특정 내용을 보존하며 압축
/compact 현재까지 구현한 인증 모듈과 API 설계 결정사항을 보존해주세요
```

**수동 압축 사용 시점:**
- 긴 탐색 단계가 끝나고 구현 단계로 전환할 때
- 디버깅 세션이 해결되고 새 기능 개발을 시작할 때
- 컨텍스트의 70% 이상이 이미 완료된 작업 이력일 때

---

### 토큰 버짓 관리 전략

**전략 1: 작업 단위 분리**

큰 작업을 여러 세션으로 분리하고 각 세션의 결과를 파일로 저장:

```
세션 1: 설계 및 계획 → docs/design.md 저장
세션 2: 구현 → 코드 저장
세션 3: 테스트 및 검증 → test-results.md 저장
```

**전략 2: CLAUDE.md로 컨텍스트 오프로드**

반복적으로 설명해야 하는 정보는 CLAUDE.md에 영구적으로 저장:

```markdown
# CLAUDE.md

## 아키텍처 결정사항 (ADR)
- ADR-001: 모노레포 선택 이유 → docs/adr/001-monorepo.md
- ADR-002: Fastify vs Express → docs/adr/002-fastify.md
```

**전략 3: 요약 파일 활용**

```bash
# 이전 세션 요약을 파일로 저장
cat > .claude/session-summary.md << 'EOF'
## 이전 세션 요약 (2024-01-15)
- 구현 완료: 사용자 인증 모듈 (src/auth/)
- 미완료: 이메일 인증 플로우
- 결정사항: JWT 토큰 만료 시간 24시간으로 설정
- 알려진 버그: 로그아웃 시 refresh token 삭제 안 됨
EOF
```

새 세션 시작 시 CLAUDE.md에서 참조:

```markdown
## 현재 작업 상태
@.claude/session-summary.md
```

---

### 컨텍스트 사용량 모니터링

```bash
# 현재 세션 컨텍스트 사용량 확인
# Claude Code UI에서 우측 하단 토큰 카운터 확인

# 환경변수로 최대 출력 토큰 제한 (응답이 너무 길어지는 것 방지)
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096
```

---

## 5. Agent Tool/서브에이전트 활용 패턴

### 서브에이전트 격리 모드

소스코드 분석에서 확인된 `isolation` 옵션:

```typescript
// isolation: "worktree" - Git worktree를 새로 생성해 완전히 격리된 환경에서 실행
// isolation: "none" - 현재 작업 디렉토리 공유 (기본값)
```

`worktree` 격리는 병렬 에이전트가 동일 파일을 동시에 수정해 충돌이 발생하는 문제를 방지한다:

```
Task(
  subagent_type="oh-my-claudecode:executor",
  isolation="worktree",
  prompt="..."
)
```

---

### TaskType별 특성

소스코드에서 확인된 세 가지 TaskType:

| TaskType | 설명 | 사용 사례 |
|----------|------|-----------|
| `local_bash` | 로컬 Bash 명령어 실행 | 빌드, 테스트, 스크립트 |
| `local_agent` | 독립적인 Claude 인스턴스 | 복잡한 코드 분석, 구현 |
| `in_process_teammate` | 동일 프로세스 내 협업 에이전트 | 빠른 응답, 컨텍스트 공유 |

---

### run_in_background 활용

장시간 실행 작업을 백그라운드로 처리:

```python
# 병렬 빌드 실행 (백그라운드)
Task(
  type="local_bash",
  command="pnpm run build",
  run_in_background=True,
  description="전체 프로젝트 빌드"
)

# 테스트 실행 (백그라운드)
Task(
  type="local_bash",
  command="pnpm run test:coverage",
  run_in_background=True,
  description="커버리지 포함 테스트"
)

# 두 작업이 완료될 때까지 다른 작업 진행 가능
```

---

### 병렬 에이전트 실행 패턴

**패턴 1: 독립 분석 병렬화**

```
# 동시에 실행
에이전트 A: "src/auth 디렉토리의 보안 취약점 분석"
에이전트 B: "src/api 디렉토리의 성능 병목 분석"
에이전트 C: "tests 디렉토리의 커버리지 미달 영역 분석"

# 결과 수집 후 통합 보고서 작성
```

**패턴 2: 파이프라인 패턴**

```
에이전트 1 (탐색): 코드베이스 구조 파악 → 결과를 파일로 저장
에이전트 2 (분석): 결과 파일 읽어서 문제점 식별 → 계획 수립
에이전트 3 (실행): 계획에 따라 코드 수정
에이전트 4 (검증): 변경사항 테스트 실행
```

**패턴 3: 스웜 패턴 (대규모 리팩토링)**

```bash
# 100개의 TypeScript 오류를 5개 에이전트가 병렬로 수정
# 각 에이전트는 서로 다른 파일 담당

에이전트 1: src/components/A.tsx ~ src/components/F.tsx
에이전트 2: src/components/G.tsx ~ src/components/L.tsx
에이전트 3: src/api/controllers/*.ts
에이전트 4: src/api/services/*.ts
에이전트 5: src/utils/*.ts
```

---

### 서브에이전트 결과 통합

```markdown
# 서브에이전트 실행 후 결과 수집 패턴

## 1. 결과 파일로 저장하게 지시
"분석 결과를 .claude/analysis-result.json 에 저장해주세요"

## 2. 메인 에이전트에서 결과 읽기
각 에이전트 완료 후 결과 파일들을 읽어서 통합

## 3. 충돌 해결
같은 파일을 수정한 에이전트가 있다면 diff 비교 후 수동 또는 자동 병합
```

---

### 실전 주의사항

1. **파일 소유권 충돌 방지**: 병렬 에이전트에게 수정할 파일 목록을 명시적으로 지정한다.
2. **백그라운드 태스크 한도**: 최대 5개의 백그라운드 태스크를 동시 실행할 수 있다.
3. **worktree 정리**: 작업 완료 후 `git worktree remove` 로 worktree를 정리한다.
4. **컨텍스트 격리**: 서브에이전트는 부모 세션의 대화 이력을 상속하지 않는다.

---

## 6. 쿼리 루프 이해

### 전체 흐름

소스코드 분석으로 확인된 쿼리 실행 흐름:

```
사용자 입력
    ↓
query()
    ↓
messages 구성 (시스템 프롬프트 + 대화 이력 + 현재 입력)
    ↓
Anthropic API 호출 (streaming)
    ↓
StreamingToolExecutor
    ↓
스트림에서 tool_use 블록 감지
    ↓
runTools()
    ↓ (병렬 실행 가능한 도구들)
도구 실행 결과 수집
    ↓
tool_result를 messages에 추가
    ↓
Claude 응답 계속 (다음 반복)
    ↓
stop_reason == "end_turn" 또는 도구 없음
    ↓
최종 응답 반환
```

---

### isConcurrencySafe: 병렬 도구 실행

소스코드에서 확인된 병렬 실행 안전성 분류:

```typescript
// 동시 실행 안전한 도구 (읽기 전용)
isConcurrencySafe = true:
  - Read (파일 읽기)
  - Glob (파일 검색)
  - Grep (내용 검색)
  - WebFetch (웹 요청)
  - Bash (읽기 전용 명령어로 분류된 경우)

// 동시 실행 위험한 도구 (쓰기/상태 변경)
isConcurrencySafe = false:
  - Edit (파일 편집)
  - Write (파일 생성)
  - Bash (일반 명령어)
```

Claude가 여러 도구를 한 번에 요청하면(`parallel tool calls`), 안전한 도구들은 동시에 실행되고 안전하지 않은 도구들은 순차 실행된다.

---

### 자동 재시도 메커니즘

소스코드에서 확인된 자동 재시도 트리거:

```
재시도 조건:
1. API 오류 (5xx, 네트워크 오류)
2. max_tokens 도달 → max_output_tokens 자동 증가 후 재시도
3. overloaded_error → 지수 백오프 후 재시도

재시도 불가 조건:
1. 권한 거부 (permission denied)
2. 사용자 취소 (Ctrl+C)
3. 잘못된 요청 (400 오류)
```

**max_output_tokens 복구 메커니즘:**

응답이 `max_tokens`로 중단되면 Claude Code는 자동으로 `max_output_tokens`를 증가시키고 이어서 생성을 시도한다. 이를 비활성화하거나 제한하려면:

```bash
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096  # 최대 출력 토큰 고정
```

---

### 도구 실행 최적화 팁

**팁 1: 병렬 읽기 요청 유도**

Claude에게 여러 파일을 동시에 읽도록 명시적으로 요청:

```
"src/auth/index.ts, src/auth/jwt.ts, src/auth/middleware.ts 세 파일을
동시에 읽고 분석해주세요"
```

**팁 2: 쓰기 작업 배치 최소화**

파일 수정은 최대한 한 번에 완결하도록 지시:

```
"auth.ts 파일을 한 번만 수정해서 JWT 검증 로직과 refresh token 처리를
모두 추가해주세요. 여러 번 나눠서 수정하지 마세요."
```

**팁 3: 스트리밍 출력 처리**

긴 응답이 예상될 때 중간 결과를 파일로 저장하도록 요청:

```
"분석 결과가 길 것 같으니 섹션별로 analysis.md 파일에 저장하면서
진행해주세요"
```

---

## 7. Permission 시스템 마스터

### PermissionMode 5종

소스코드에서 확인된 5가지 Permission 모드:

| 모드 | 설명 | 활성화 방법 |
|------|------|-------------|
| `default` | 기본 모드, 위험한 작업에 승인 요청 | 기본값 |
| `plan` | 읽기 전용, 실행 불가 (계획만 수립) | `--plan` 플래그 |
| `autoEdit` | 파일 편집 자동 승인, bash 등은 물어봄 | `--auto-edit` 플래그 |
| `fullAuto` | 모든 작업 자동 승인, 위험 패턴만 차단 | `--full-auto` 플래그 |
| `bypassPermissions` | 모든 권한 검사 우회 | `--dangerously-skip-permissions` |

---

### 모드별 실전 활용

**plan 모드: 작업 사전 검토**

```bash
# 무엇을 할지만 계획하고 실행은 안 함
claude --plan "인증 시스템 리팩토링"

# 결과: 변경 계획 목록만 출력
# 실제 파일 수정 없음
```

**autoEdit 모드: 코드 작업 효율화**

```bash
# 파일 편집은 자동 승인, 빌드/테스트는 수동 확인
claude --auto-edit "TypeScript 타입 오류 모두 수정"

# 적합한 상황:
# - 타입 수정, 린트 픽스 같은 안전한 코드 변경
# - 파일 편집만 필요하고 외부 명령 실행이 적은 경우
```

**fullAuto 모드: CI/CD 자동화**

```bash
# GitHub Actions 워크플로우 예시
- name: Claude Code 자동 수정
  run: |
    claude --full-auto "빌드 오류 수정 및 테스트 통과"
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

### yoloClassifier 자동 모드 감지

소스코드에서 확인된 `yoloClassifier` 동작:

Claude Code는 사용 패턴을 분석해 자동으로 적절한 권한 모드를 추론한다. 특정 키워드나 패턴이 감지되면 더 관대한 권한을 자동 적용할 수 있다.

```
감지 패턴 예시:
- "자동으로 처리해줘", "알아서 해줘" → fullAuto 경향
- "확인하면서 진행해줘", "물어봐줘" → default 유지
- "계획만 세워줘", "어떻게 할지만 알려줘" → plan 모드
```

---

### 패턴 매칭 규칙 상세

permissions의 allow/deny 규칙에서 사용되는 매칭 로직:

```
1. 정확 매칭: "Bash(git status)" → "git status" 명령어에만 매칭
2. 와일드카드: "Bash(git *)" → "git"으로 시작하는 모든 명령어
3. 접두사 매칭: "Edit(src/*)" → src/ 하위 모든 파일
4. 재귀 와일드카드: "Edit(src/**)" → src/ 전체 경로 재귀 매칭
5. 도구 전체: "Edit(*)" → 모든 편집 작업 허용/거부

우선순위:
- deny 규칙 > allow 규칙
- 더 구체적인 규칙 > 더 일반적인 규칙
- 프로젝트 settings > 전역 settings
```

---

### 권한 관련 실전 FAQ

**Q: 특정 도구를 완전히 비활성화하려면?**

```json
{
  "permissions": {
    "deny": ["WebFetch(*)", "mcp__browser__*"]
  }
}
```

**Q: 특정 디렉토리만 읽기 전용으로 설정하려면?**

```json
{
  "permissions": {
    "allow": ["Read(config/**)", "Read(.env*)"],
    "deny": ["Edit(config/**)", "Write(config/**)", "Edit(.env*)"]
  }
}
```

**Q: 개발 서버 시작 명령어는 허용하되 종료 명령어는 차단하려면?**

```json
{
  "permissions": {
    "allow": ["Bash(pnpm run dev)", "Bash(pnpm run start)"],
    "deny": ["Bash(kill *)", "Bash(pkill *)"]
  }
}
```

---

## 8. 성능 최적화

### 시스템 프롬프트 캐싱 메커니즘

소스코드 분석에서 확인된 두 가지 시스템 프롬프트 섹션 타입:

```typescript
// 캐시 가능한 섹션 (변경이 없으면 캐시 히트)
systemPromptSection: {
  type: "text",
  text: "...",
  cache_control: { type: "ephemeral" }
}

// 캐시 불가 섹션 (매번 새로 전송)
DANGEROUS_uncachedSystemPromptSection: {
  type: "text",
  text: "..."
  // cache_control 없음
}
```

**캐시 히트 효과:**
- 캐시된 토큰은 입력 비용의 약 10%만 청구
- 응답 지연 시간 감소
- 대용량 CLAUDE.md나 긴 시스템 프롬프트에서 효과적

---

### SYSTEM_PROMPT_DYNAMIC_BOUNDARY

소스코드에서 확인된 동적 경계 설정:

```
시스템 프롬프트 구조:
[정적 섹션 - 캐시 가능]
  - 기본 지시사항
  - CLAUDE.md 내용
  - 도구 설명

SYSTEM_PROMPT_DYNAMIC_BOUNDARY  ← 캐시 경계선

[동적 섹션 - 캐시 불가]
  - 현재 작업 디렉토리
  - 현재 날짜/시간
  - 세션별 변수
```

경계선 이전의 모든 내용은 캐시 후보가 되고, 이후의 동적 내용은 매번 새로 전송된다.

---

### 캐시 안정성 최적화

캐시 히트율을 높이기 위한 실전 전략:

**전략 1: CLAUDE.md를 안정적으로 유지**

```markdown
# 좋은 예 (캐시 친화적)
## 코딩 규칙
- TypeScript strict 모드 사용
- 함수형 컴포넌트 선호

# 나쁜 예 (캐시 미스 유발)
## 오늘의 작업 (2024-01-15 업데이트됨)
- 현재 작업 중인 기능: 로그인 페이지
```

**전략 2: 동적 내용은 CLAUDE.md 밖으로**

```bash
# 세션마다 변하는 내용은 별도 파일로
cat > .claude/current-task.md << EOF
현재 작업: 로그인 페이지 구현
담당자: 홍길동
EOF

# CLAUDE.md에서는 안정적으로 참조
# @.claude/current-task.md  ← 이렇게 동적 참조
```

**전략 3: 시스템 프롬프트 크기 최소화**

```markdown
# 비효율적: 모든 내용을 CLAUDE.md에 직접 작성
# 효율적: 핵심 규칙만 CLAUDE.md에, 세부사항은 별도 파일 참조

# CLAUDE.md
코딩 규칙: @docs/coding-rules.md
API 스펙: @docs/api-spec.md
데이터베이스 스키마: @docs/schema.md
```

---

### API 비용 최적화

```bash
# 모델별 토큰 비용 고려 (저비용 → 고비용)
Haiku  → 탐색, 단순 검색, 빠른 확인
Sonnet → 일반 구현, 표준 작업 (기본값)
Opus   → 복잡한 설계, 아키텍처 결정, 어려운 디버깅

# 에코모드 활성화로 토큰 효율화
claude eco "오류 수정"
# 또는
export CLAUDE_CODE_MODE=ecomode
```

---

### 토큰 사용량 추적

```bash
# API 사용량 확인 (Anthropic Console)
open https://console.anthropic.com/usage

# 세션별 사용량 로그 (환경변수로 활성화)
export CLAUDE_CODE_LOG_USAGE=true
export CLAUDE_CODE_LOG_DIR=~/.claude/logs

# 사용량 분석
ls -la ~/.claude/logs/
```

---

## 9. MCP 서버 통합 실전

### MCP 도구 네이밍 규칙

소스코드 분석에서 확인된 MCP 도구 네이밍 규칙:

```
형식: mcp__{서버명}__{도구명}

예시:
- mcp__filesystem__read_file
- mcp__filesystem__write_file
- mcp__database__execute_query
- mcp__github__create_pull_request
- mcp__slack__send_message

규칙:
- 서버명과 도구명은 소문자 스네이크케이스
- 구분자는 더블 언더스코어(__)
- 하이픈(-)은 언더스코어(_)로 변환됨
```

---

### assembleToolPool 병합 로직

소스코드에서 확인된 도구 풀 구성 방식:

```
최종 도구 풀 = 내장 도구 + MCP 서버 도구들

우선순위 및 중복 처리:
1. 내장 도구 (Read, Write, Edit, Bash 등) 항상 포함
2. MCP 서버별 도구 추가
3. 동일 이름 충돌 시: 내장 도구 > MCP 도구
4. MCP 서버간 충돌 시: 나중에 로드된 서버 > 먼저 로드된 서버
```

---

### MCP 서버 설정 실전 예시

`.claude/settings.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/user/projects"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "database": {
      "command": "python",
      "args": ["-m", "mcp_server_sqlite", "--db-path", "./dev.db"],
      "env": {}
    },
    "custom-api": {
      "command": "node",
      "args": ["./mcp-servers/custom-api-server.js"],
      "env": {
        "API_BASE_URL": "https://api.example.com",
        "API_KEY": "${CUSTOM_API_KEY}"
      }
    }
  }
}
```

---

### deny rule로 MCP 서버 차단

특정 MCP 서버 전체 차단:

```json
{
  "permissions": {
    "deny": [
      "mcp__dangerous-server__*",
      "mcp__filesystem__delete_file",
      "mcp__filesystem__move_file"
    ],
    "allow": [
      "mcp__filesystem__read_file",
      "mcp__filesystem__list_directory",
      "mcp__filesystem__read_multiple_files"
    ]
  }
}
```

---

### 커스텀 MCP 서버 개발

간단한 커스텀 MCP 서버 예시 (`mcp-servers/project-info-server.js`):

```javascript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { execSync } from "child_process";
import { readFileSync } from "fs";

const server = new Server(
  { name: "project-info", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_git_log",
      description: "최근 Git 커밋 이력 조회",
      inputSchema: {
        type: "object",
        properties: {
          count: { type: "number", description: "조회할 커밋 수", default: 10 }
        }
      }
    },
    {
      name: "get_package_info",
      description: "package.json 정보 조회",
      inputSchema: { type: "object", properties: {} }
    }
  ]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "get_git_log") {
    const count = args?.count ?? 10;
    const log = execSync(`git log --oneline -${count}`).toString();
    return { content: [{ type: "text", text: log }] };
  }

  if (name === "get_package_info") {
    const pkg = JSON.parse(readFileSync("package.json", "utf-8"));
    return {
      content: [{
        type: "text",
        text: JSON.stringify({ name: pkg.name, version: pkg.version, dependencies: Object.keys(pkg.dependencies ?? {}) }, null, 2)
      }]
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

등록:

```json
{
  "mcpServers": {
    "project-info": {
      "command": "node",
      "args": ["./mcp-servers/project-info-server.js"]
    }
  }
}
```

사용:

```
# Claude에서 자동으로 다음 도구가 사용 가능
mcp__project-info__get_git_log
mcp__project-info__get_package_info
```

---

### MCP 서버 디버깅

```bash
# MCP 서버 직접 실행 테스트
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | \
  node ./mcp-servers/custom-api-server.js

# 연결 오류 진단
CLAUDE_CODE_MCP_DEBUG=1 claude

# 특정 서버 로그 확인
tail -f ~/.claude/logs/mcp-*.log
```

---

## 10. 커스텀 스킬 고급 작성법

### 스킬 로딩 경로

소스코드 분석에서 확인된 `loadSkillsDir` 로딩 순서:

```
1. ~/.claude/skills/          ← 전역 개인 스킬
2. .claude/skills/            ← 프로젝트 스킬
3. plugin 경로 (npm 패키지)   ← 설치된 스킬 플러그인
```

동일한 이름의 스킬이 여러 경로에 존재하면 우선순위가 높은 경로의 스킬이 사용된다 (낮은 번호 = 높은 우선순위). 프로젝트 스킬이 전역 스킬을 오버라이드할 수 있다.

---

### 기본 스킬 구조

`.claude/skills/my-skill.md`:

```markdown
---
name: my-skill
description: 스킬 설명
tools: ["Read", "Edit", "Bash"]
model: sonnet
---

# 스킬 지시사항

여기에 스킬 실행 지침을 작성합니다.

## 실행 단계

1. 첫 번째 단계
2. 두 번째 단계
3. 세 번째 단계
```

---

### frontmatter 옵션 전체 레퍼런스

```yaml
---
# 스킬 기본 정보
name: skill-name           # 슬래시 명령어 이름 (/skill-name)
description: "설명"         # 도움말에 표시될 설명
aliases: ["alt-name"]      # 대체 이름

# 모델 설정
model: sonnet              # haiku | sonnet | opus
                           # 기본값: sonnet

# 도구 권한
tools:                     # 허용할 도구 목록 (생략 시 모든 도구)
  - Read
  - Grep
  - Glob
  - Edit
  - Write
  - Bash
  - WebFetch
  - mcp__github__*         # MCP 도구도 지정 가능

# 실행 설정
shell: bash                # 내장 bash 명령어 실행 환경
effort: high               # low | medium | high (thinking budget)

# 훅 설정 (스킬 내 훅)
hooks:
  before: "echo '스킬 시작'"    # 스킬 실행 전 쉘 명령어
  after: "echo '스킬 완료'"     # 스킬 실행 후 쉘 명령어
  onError: "echo '오류 발생'"   # 오류 시 쉘 명령어

# 입력 인수
arguments:
  - name: target
    description: "대상 파일 또는 디렉토리"
    required: true
  - name: dry-run
    description: "실제 변경 없이 미리보기"
    required: false
    default: "false"
---
```

---

### 실전 스킬 예시 1: 코드 리뷰 자동화

`.claude/skills/code-review.md`:

```markdown
---
name: code-review
description: 변경된 파일의 코드 리뷰 수행
tools: ["Read", "Bash", "Grep"]
model: opus
effort: high
---

# 코드 리뷰 스킬

## 목적
변경된 코드를 분석하고 구체적인 개선사항을 제안합니다.

## 실행 단계

### 1. 변경사항 파악
```bash
git diff --name-only HEAD~1 HEAD
git diff HEAD~1 HEAD
```

### 2. 각 변경 파일 분석
변경된 각 파일에 대해 다음을 검토합니다:
- 버그 가능성 (null 체크, 경계값, 타입 불일치)
- 보안 취약점 (SQL 인젝션, XSS, 인증 누락)
- 성능 문제 (N+1 쿼리, 불필요한 재렌더링)
- 코드 품질 (가독성, 단일 책임 원칙, DRY)

### 3. 리뷰 보고서 작성
다음 형식으로 보고서를 작성합니다:

**[심각도: 높음/중간/낮음]**
- 위치: `파일명:줄번호`
- 문제: 문제 설명
- 제안: 개선 방법
- 예시 코드: (해당하는 경우)

### 4. 우선순위 요약
발견된 모든 이슈를 심각도 순으로 정렬하여 요약합니다.
```

---

### 실전 스킬 예시 2: 데이터베이스 마이그레이션 도우미

`.claude/skills/db-migrate.md`:

```markdown
---
name: db-migrate
description: Prisma 마이그레이션 안전하게 생성 및 검증
tools: ["Read", "Bash", "Edit"]
model: sonnet
hooks:
  before: "git stash list | head -5"
  after: "prisma migrate status 2>/dev/null || echo 'Prisma 미설치'"
---

# 데이터베이스 마이그레이션 스킬

## 실행 전 체크리스트
1. `prisma/schema.prisma` 현재 상태 확인
2. 보류 중인 마이그레이션 확인: `npx prisma migrate status`
3. 현재 브랜치 확인: `git branch --show-current`

## 마이그레이션 생성 단계

### 스키마 변경사항 분석
schema.prisma의 변경사항을 읽고:
- 추가된 모델/필드 파악
- 삭제된 모델/필드 파악 (데이터 손실 위험 경고)
- 변경된 제약조건 파악

### 마이그레이션 생성
```bash
npx prisma migrate dev --name [적절한_이름] --create-only
```

### 생성된 마이그레이션 SQL 검토
생성된 마이그레이션 파일을 읽고 다음을 확인:
- DROP TABLE / DROP COLUMN 문 (데이터 손실 위험)
- 인덱스 생성으로 인한 잠금 가능성
- 기본값 없는 NOT NULL 열 추가 (기존 행 오류)

### 안전 확인 후 적용
위험한 SQL이 없으면: `npx prisma migrate deploy`
위험한 SQL 발견 시: 사용자에게 경고하고 수동 검토 요청

## 주의사항
- 프로덕션 DB에는 절대 `prisma migrate dev` 사용 금지
- 데이터 손실 가능성 있는 마이그레이션은 반드시 백업 후 진행
```

---

### 실전 스킬 예시 3: 릴리즈 자동화

`.claude/skills/release.md`:

```markdown
---
name: release
description: 시맨틱 버저닝 기반 릴리즈 준비
tools: ["Read", "Edit", "Bash"]
model: sonnet
arguments:
  - name: bump
    description: "버전 증가 타입 (major|minor|patch)"
    required: true
hooks:
  before: "git status --short"
---

# 릴리즈 스킬

## 입력값
- bump: $ARGUMENTS (major | minor | patch)

## 실행 단계

### 1. 현재 상태 확인
```bash
git status
git log --oneline $(git describe --tags --abbrev=0)..HEAD
```

### 2. 버전 번호 계산
package.json의 현재 버전을 읽고 $ARGUMENTS에 따라 증가:
- major: 1.2.3 → 2.0.0
- minor: 1.2.3 → 1.3.0
- patch: 1.2.3 → 1.2.4

### 3. CHANGELOG 업데이트
`git log --oneline`으로 커밋 이력을 읽고 CHANGELOG.md에 추가:

```
## [새버전] - YYYY-MM-DD

### 추가됨
- 새 기능 목록

### 변경됨
- 변경 사항 목록

### 수정됨
- 버그 수정 목록
```

### 4. package.json 버전 업데이트
```bash
npm version $ARGUMENTS --no-git-tag-version
```

### 5. 최종 확인 요청
모든 변경사항을 사용자에게 보여주고 확인 후 커밋 진행:
```bash
git add package.json CHANGELOG.md
git commit -m "chore: release v[새버전]"
git tag -a v[새버전] -m "Release v[새버전]"
```
```

---

### 스킬 인수 처리 고급 패턴

```markdown
---
name: analyze-perf
description: 성능 분석 스킬
arguments:
  - name: target
    required: true
  - name: depth
    required: false
    default: "shallow"
---

# 성능 분석

대상: $ARG_TARGET
분석 깊이: $ARG_DEPTH

$ARG_DEPTH 값이 "deep"이면 전체 프로파일링을 수행하고,
"shallow"이면 표면적인 성능 지표만 확인합니다.
```

스킬 호출:

```
/analyze-perf src/api/heavy-endpoint.ts deep
```

---

### 스킬 간 체이닝

여러 스킬을 순차적으로 실행하는 마스터 스킬:

`.claude/skills/full-release-cycle.md`:

```markdown
---
name: full-release-cycle
description: 테스트 → 코드 리뷰 → 릴리즈 전체 사이클
tools: ["Bash", "Read"]
model: sonnet
---

# 전체 릴리즈 사이클

다음 단계를 순서대로 실행합니다:

## 1단계: 테스트 실행
```bash
pnpm run test:ci
```
테스트 실패 시 중단하고 실패 원인 보고.

## 2단계: 타입 검사
```bash
pnpm run typecheck
```
타입 오류 발견 시 중단하고 오류 목록 제공.

## 3단계: 코드 품질 확인
```bash
pnpm run lint
```

## 4단계: 빌드 검증
```bash
pnpm run build
```

## 5단계: 모든 단계 통과 시
사용자에게 patch/minor/major 중 어떤 버전으로 릴리즈할지 질문한 후
/release 스킬을 해당 인수로 실행합니다.
```

---

### 스킬 디버깅 및 테스트

```bash
# 스킬 파일 유효성 확인
claude /my-skill --help

# 스킬 dry-run (실제 실행 없이 계획만)
claude /my-skill --plan

# 스킬 로딩 확인
ls -la ~/.claude/skills/
ls -la .claude/skills/

# 스킬 목록 확인
claude /help
```

---

## 부록: 빠른 참조 카드

### 필수 환경변수

```bash
# 컨텍스트 관리
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=20000
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=8192

# 성능/디버깅
export CLAUDE_CODE_LOG_USAGE=true
export CLAUDE_CODE_LOG_DIR=~/.claude/logs
export CLAUDE_CODE_MCP_DEBUG=1
export CLAUDE_CODE_HOOK_DEBUG=1

# 모델 설정
export ANTHROPIC_MODEL=claude-sonnet-4-5  # 기본 모델 오버라이드
```

### 필수 단축키

| 단축키 | 동작 |
|--------|------|
| `Ctrl+C` | 현재 실행 중단 |
| `Ctrl+R` | 이전 명령어 검색 |
| `/compact` | 수동 컨텍스트 압축 |
| `/clear` | 대화 이력 초기화 |
| `/help` | 도움말 표시 |
| `/plan` | 계획 모드 전환 |

### 디렉토리 구조 베스트 프랙티스

```
프로젝트/
├── .claude/
│   ├── CLAUDE.md           # 프로젝트 지시사항
│   ├── settings.json       # 팀 공유 설정
│   ├── settings.local.json # 개인 설정 (gitignore)
│   ├── skills/             # 프로젝트 스킬
│   │   ├── deploy.md
│   │   └── db-migrate.md
│   ├── hooks/              # 프로젝트 훅
│   │   ├── pre-bash-check.sh
│   │   └── post-edit-format.sh
│   └── notepads/           # OMC 노트패드
│       └── main/
│           ├── learnings.md
│           └── decisions.md
├── docs/
│   ├── api-spec.md         # CLAUDE.md에서 @참조
│   └── coding-rules.md     # CLAUDE.md에서 @참조
└── ...
```

---

*이 가이드는 Claude Code 소스코드 분석을 기반으로 작성되었습니다. 버전 업데이트에 따라 일부 내용이 변경될 수 있습니다.*
