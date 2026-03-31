# Claude Code 프롬프트 엔지니어링 & 하네스 기법 가이드

> 소스 코드 분석을 기반으로 한 Claude Code 활용 심화 가이드

---

## 목차

1. [시스템 프롬프트의 실제 구조](#1-시스템-프롬프트의-실제-구조)
2. [CLAUDE.md 작성 모범 사례](#2-claudemd-작성-모범-사례-dodont)
3. [커스텀 스킬 작성법](#3-커스텀-스킬-작성법)
4. [에이전트 정의 작성 가이드](#4-에이전트-정의-claudeagents-작성-가이드)
5. [OMC 플러그인 하네스 강화 설정](#5-omc-플러그인-하네스-강화-설정)
6. [Claude가 더 잘 이해하는 프롬프트 패턴](#6-claude가-더-잘-이해하는-프롬프트-패턴)
7. [환경변수 기반 고급 설정](#7-환경변수-기반-고급-설정)

---

## 1. 시스템 프롬프트의 실제 구조

### getSystemPrompt()의 섹션 순서

Claude Code의 `prompts.ts`에서 `getSystemPrompt()`는 다음 순서로 시스템 프롬프트를 조립한다.

```
identity → tools → environment → git → tasks → tone → security → output
```

각 섹션이 의미하는 바:

| 섹션 | 역할 |
|------|------|
| **identity** | Claude의 역할 정의, 자기 소개 방식 |
| **tools** | 사용 가능한 도구 목록과 사용 정책 |
| **environment** | 운영체제, 셸, 작업 디렉토리 등 실행 환경 |
| **git** | git 저장소 여부, 브랜치, 커밋 스타일 |
| **tasks** | 태스크 관리(TodoWrite/TodoRead) 활용 방침 |
| **tone** | 응답 톤, 간결성, 언어 스타일 |
| **security** | 보안 제약 및 허가되지 않은 행동 금지 |
| **output** | 출력 형식, 마크다운 사용 여부 |

### 정적 섹션 vs 동적 섹션

```typescript
// 정적 섹션: 글로벌로 캐싱됨 (매번 재계산 안 함)
function systemPromptSection(content: string): CachedSection {
  return { type: 'cached', content }
}

// 동적 섹션: 매 턴마다 재계산 (사용자별 or 컨텍스트별)
function DANGEROUS_uncachedSystemPromptSection(content: string): UncachedSection {
  return { type: 'uncached', content }
}
```

**캐싱 경계 (`SYSTEM_PROMPT_DYNAMIC_BOUNDARY`):**

```
[글로벌 캐시 구간]
  identity
  tools
  environment (일부)
  ...
──────── SYSTEM_PROMPT_DYNAMIC_BOUNDARY ────────
[사용자별 동적 구간]
  CLAUDE.md 내용
  현재 작업 디렉토리
  git 상태
  커스텀 지시사항
```

실용적 의미: 글로벌 캐시 구간은 API 비용이 절감되므로, **자주 변하지 않는 지시사항은 CLAUDE.md 상단에** 두는 것이 유리하다.

### buildEffectiveSystemPrompt() 우선순위

시스템 프롬프트는 여러 소스를 병합하며, 우선순위는 다음과 같다.

```
override > coordinator > agent > custom > default + append
```

```typescript
// 예시: 우선순위 적용 흐름
const effectivePrompt = buildEffectiveSystemPrompt({
  override: process.env.SYSTEM_PROMPT_OVERRIDE,  // 최우선
  coordinator: coordinatorPrompt,                  // 멀티에이전트 조율자
  agent: agentDefinition.systemPrompt,             // 에이전트 정의
  custom: userCustomPrompt,                        // 사용자 커스텀
  default: defaultSystemPrompt,                    // 기본값
  append: appendSection,                           // 기본값에 추가
})
```

**활용 팁:** 특정 에이전트에게 강한 제약을 부여하려면 `override` 레벨에 지시사항을 주입하면 된다. 반면 기본 동작에 추가적인 규칙만 더하고 싶다면 `append`를 활용한다.

---

## 2. CLAUDE.md 작성 모범 사례 (Do/Don't)

### CLAUDE.md 계층 구조

```
~/.claude/CLAUDE.md          ← 전역: 모든 프로젝트에 적용
  ↓ 병합
프로젝트/.claude/CLAUDE.md   ← 프로젝트: 해당 저장소에 적용
  ↓ 병합
하위폴더/.claude/CLAUDE.md   ← 로컬: 특정 서브디렉토리에만 적용
```

이 계층은 `getUserContext()` 함수에서 다음 경로로 로딩된다.

```typescript
getUserContext()
  → filterInjectedMemoryFiles()   // 중복 제거, 주입 우선순위 처리
  → getClaudeMds()                // .claude/CLAUDE.md 파일 탐색
  → getMemoryFiles()              // 추가 메모리 파일 로딩
```

### Do: 이렇게 작성하라

```markdown
# 프로젝트 컨벤션

## 코드 스타일
- TypeScript 사용, `any` 타입 금지
- 함수형 컴포넌트만 사용 (클래스 컴포넌트 금지)
- 파일명: kebab-case (예: user-profile.tsx)

## 아키텍처 결정
- 상태 관리: Zustand 사용 (Redux 금지)
- API 호출: React Query + axios 조합
- 라우팅: React Router v6 (useNavigate 사용)

## 금지 패턴
- console.log 프로덕션 코드에 삽입 금지
- any 타입 캐스팅 금지
- 직접 DOM 조작 금지 (ref 활용)

## 선호 라이브러리
- 날짜: date-fns (moment.js 금지)
- 스타일: Tailwind CSS (styled-components 금지)
- 폼: React Hook Form
```

### Don't: 이렇게 하지 마라

```markdown
# 나쁜 예시

## 절대 하지 마세요
- 코드를 수정하지 마세요              ← 너무 광범위한 금지
- 항상 완벽하게 작성해야 합니다       ← 추상적, 측정 불가
- 다음 코드를 참고하세요:             ← 코드 직접 삽입 (토큰 낭비)
  [200줄의 코드 예시]

## 동시에 적용되는 모순 규칙
- 주석을 항상 영어로 작성하세요
- 모든 주석은 한국어로 작성하세요    ← 충돌!
```

**핵심 원칙:**

| 항목 | 권장 |
|------|------|
| 길이 | 500 토큰 이내 (핵심만) |
| 형식 | 간결한 목록형, 계층 최소화 |
| 내용 | 측정 가능한 구체적 지시 |
| 갱신 | 프로젝트 변경 시 즉시 업데이트 |

### 환경변수와의 관계

```bash
# CLAUDE.md 로딩 비활성화 (디버깅 시 유용)
export CLAUDE_CODE_DISABLE_CLAUDE_MDS=true

# --bare 모드: 최소 설정으로 실행 (CLAUDE.md 무시)
claude --bare
```

`--bare` 모드는 CLAUDE.md를 포함한 모든 커스텀 컨텍스트를 비활성화하며, 순수 기본 동작만 실행된다. 성능 테스트나 격리 환경 구축 시 활용한다.

---

## 3. 커스텀 스킬 작성법

### 스킬 파일 위치

```
.claude/
└── skills/
    ├── my-skill.md          ← 커스텀 스킬
    ├── code-review.md
    └── deploy-check.md
```

### Frontmatter 옵션 전체

```yaml
---
description: "이 스킬이 하는 일을 한 줄로 설명"
tools:
  - Bash
  - Read
  - Edit
  - Glob
  - Grep
model: sonnet          # haiku | sonnet | opus
shell: bash            # 스킬 내 셸 명령 실행 환경
effort: low            # low | medium | high (thinking budget)
hooks:
  pre: "./scripts/pre-skill.sh"
  post: "./scripts/post-skill.sh"
paths:
  - "src/**"           # 이 스킬이 접근할 수 있는 경로
  - "tests/**"
whenToUse: |
  사용자가 코드 리뷰를 요청할 때,
  또는 PR 작성 전 검토가 필요할 때 자동 호출
disableModelInvocation: false   # true면 모델 자동 호출 안 함
---
```

### 인자 치환: {{ARG_NAME}} 패턴

```markdown
---
description: "특정 파일의 타입 오류를 수정"
tools: [Read, Edit, Bash]
---

# 타입 오류 수정 스킬

대상 파일: {{FILE_PATH}}
수정 범위: {{SCOPE}}

## 지시사항

1. `{{FILE_PATH}}` 파일을 읽어라
2. TypeScript 타입 오류를 모두 찾아라
3. 각 오류를 {{SCOPE}} 범위 내에서 수정하라
4. 수정 후 `tsc --noEmit`로 검증하라
```

호출 시:

```bash
# 슬래시 명령으로 인자 전달
/fix-types FILE_PATH=src/utils/auth.ts SCOPE=function
```

또는 Claude에게 자연어로:

```
auth.ts 파일의 타입 오류를 함수 범위 내에서만 수정해줘
```

### 스킬 로딩 순서

```
bundled 스킬 (내장)
  ↓
builtinPlugin 스킬 (플러그인 기본 제공)
  ↓
skillDir 스킬 (.claude/skills/)
  ↓
workflow 스킬 (워크플로 정의)
  ↓
pluginSkills (외부 플러그인)
  ↓
COMMANDS (최종 명령 목록)
```

**우선순위:** 나중에 로딩된 스킬이 동명의 이전 스킬을 덮어쓴다. 따라서 `.claude/skills/`의 커스텀 스킬로 내장 스킬을 오버라이드할 수 있다.

### 모델 자동 호출 조건

```typescript
// 스킬이 자동으로 모델에 의해 호출되려면:
const willAutoInvoke =
  skill.type === 'prompt' &&
  !skill.disableModelInvocation &&
  (skill.description || skill.whenToUse)
```

`whenToUse` 필드가 있으면 Claude가 상황에 맞게 스킬을 자동 선택한다. 명시적 슬래시 명령 없이도 트리거된다.

### 실전 예시: 코드 리뷰 스킬

```markdown
---
description: "PR 전 코드 품질 검토"
tools: [Read, Grep, Bash]
model: sonnet
effort: medium
whenToUse: |
  사용자가 "코드 리뷰", "PR 전 검토", "품질 확인"을 요청할 때
---

# 코드 리뷰 체크리스트

## 검토 대상
{{TARGET_PATH}} 경로의 변경사항을 검토한다.

## 단계

1. **변경 파일 파악**
   ```bash
   git diff --name-only HEAD~1
   ```

2. **각 파일 읽기 및 분석**
   - 타입 안전성 확인
   - 중복 코드 탐지
   - 네이밍 컨벤션 준수 여부

3. **보안 취약점 스캔**
   ```bash
   grep -r "eval\|innerHTML\|dangerouslySetInnerHTML" {{TARGET_PATH}}
   ```

4. **테스트 커버리지 확인**
   변경된 로직에 대응하는 테스트가 있는지 확인

## 출력 형식
- 심각도: 높음/중간/낮음
- 위치: 파일명:라인
- 설명: 문제점과 개선 방안
```

---

## 4. 에이전트 정의 (.claude/agents/) 작성 가이드

### AgentDefinition 타입 구조

```typescript
interface AgentDefinition {
  agentType: string          // 에이전트 식별자 (예: "code-reviewer")
  systemPrompt: string       // 에이전트 전용 시스템 프롬프트
  tools?: string[]           // 허용 도구 목록 (미지정 시 전체)
  model?: ModelTier          // haiku | sonnet | opus
  memory?: MemoryConfig      // 메모리/컨텍스트 설정
  description?: string       // 에이전트 설명 (오케스트레이터가 참조)
}
```

### 파일 구조

```
.claude/
└── agents/
    ├── code-reviewer.md     ← 커스텀 에이전트 정의
    ├── db-expert.md
    └── security-auditor.md
```

### 에이전트 정의 파일 예시

```markdown
---
agentType: db-expert
model: opus
tools:
  - Read
  - Bash
  - Grep
description: "데이터베이스 쿼리 최적화 및 스키마 설계 전문가"
memory:
  enabled: true
  scope: session
---

당신은 PostgreSQL과 MySQL 데이터베이스 전문가입니다.

## 역할
- 쿼리 성능 분석 및 최적화
- 인덱스 설계 및 추천
- N+1 쿼리 문제 탐지
- 정규화/비정규화 트레이드오프 분석

## 행동 원칙
- EXPLAIN ANALYZE로 실행 계획을 항상 확인하라
- 인덱스 추천 전 카디널리티를 고려하라
- 마이그레이션 스크립트는 항상 롤백 가능하게 작성하라
- 프로덕션 데이터에 직접 쿼리 실행을 권장하지 마라

## 출력 형식
최적화 제안 시 반드시 다음을 포함하라:
1. 현재 문제점
2. 개선 방안
3. 예상 성능 향상 수치
4. 적용 리스크
```

### 빌트인 에이전트 vs 커스텀 에이전트

```typescript
// 빌트인 에이전트 판별
function isBuiltInAgent(agentType: string): boolean {
  return BUILT_IN_AGENT_TYPES.includes(agentType)
}

// 커스텀 에이전트 판별
function isCustomAgent(agentType: string): boolean {
  return !isBuiltInAgent(agentType) && customAgentExists(agentType)
}
```

**빌트인 에이전트** (Claude Code 기본 제공):
- 코드 검색, 파일 탐색 등 범용 작업
- 업데이트 시 자동으로 개선됨

**커스텀 에이전트** (`.claude/agents/`에 정의):
- 도메인 특화 전문성 주입 가능
- 프로젝트별 맞춤 설정 가능
- `systemPrompt`로 완전한 행동 제어

### Task 도구에서 subagent_type 지정

```javascript
// Task 도구 호출 예시 (오케스트레이터에서 사용)
Task({
  subagent_type: "db-expert",          // 커스텀 에이전트 이름
  prompt: `
    users 테이블의 쿼리 성능을 분석하고 인덱스를 최적화하라.
    현재 평균 응답 시간이 500ms인데 100ms 이하로 줄여야 한다.
  `
})
```

**oh-my-claudecode 플러그인의 에이전트 지정 패턴:**

```javascript
// 플러그인 에이전트는 "oh-my-claudecode:" 접두사 사용
Task({
  subagent_type: "oh-my-claudecode:executor",
  model: "sonnet",
  prompt: "src/api/auth.ts에 JWT 검증 미들웨어를 추가하라"
})

Task({
  subagent_type: "oh-my-claudecode:architect",
  model: "opus",
  prompt: "현재 인증 시스템의 보안 취약점을 분석하라"
})
```

---

## 5. OMC 플러그인 하네스 강화 설정

### oh-my-claudecode 에이전트 티어링

oh-my-claudecode는 복잡도에 따라 세 개의 모델 티어로 에이전트를 분류한다.

```
LOW tier   → Haiku   (빠르고 저렴, 단순 작업)
MEDIUM tier → Sonnet  (균형, 대부분의 작업)
HIGH tier  → Opus    (강력하고 느림, 복잡한 추론)
```

| 도메인 | LOW | MEDIUM | HIGH |
|--------|-----|--------|------|
| 분석 | architect-low | architect-medium | architect |
| 실행 | executor-low | executor | executor-high |
| 탐색 | explore | explore-medium | explore-high |
| 프론트엔드 | designer-low | designer | designer-high |
| 보안 | security-reviewer-low | - | security-reviewer |
| 테스트 | - | qa-tester | qa-tester-high |

**작업별 에이전트 선택 가이드:**

```
빠른 코드 조회          → explore (haiku)
파일/패턴 탐색          → explore-medium (sonnet)
복잡한 아키텍처 분석    → explore-high (opus)
단순 한 줄 수정         → executor-low (haiku)
기능 구현               → executor (sonnet)
복잡한 리팩토링         → executor-high (opus)
간단한 디버깅           → architect-low (haiku)
복잡한 버그 추적        → architect (opus)
```

### 병렬 실행 전략

**Ultrawork: 최대 병렬 실행**

```bash
# 명시적 활성화
/oh-my-claudecode:ultrawork

# 키워드로 자동 활성화
ulw: 모든 TypeScript 오류를 수정하라
ultrawork: 전체 테스트 스위트를 통과시켜라
```

ultrawork는 독립적인 작업들을 동시에 여러 에이전트에 분산하여 실행한다. 파일 충돌 감지 및 소유권 조율 기능이 내장되어 있다.

**Autopilot: 완전 자율 실행**

```bash
# 아이디어에서 동작하는 코드까지 자율 실행
autopilot: 사용자 인증 REST API를 빌드해줘

# 단계:
# 1. 자동 계획 수립
# 2. 요구사항 분석
# 3. 병렬 구현
# 4. 지속적 검증
# 5. 자기 수정 반복
```

**Swarm: N개 에이전트 협업**

```bash
# 5개 에이전트가 공유 태스크 풀에서 작업 분담
/swarm 5:executor "모든 린트 오류를 수정하라"

# 내부 동작:
# - 태스크 풀: pending → claimed → done
# - 5분 타임아웃 후 자동 해제
# - 모든 태스크 완료 시 종료
```

**Pipeline: 순차 에이전트 체인**

```bash
# 내장 파이프라인 프리셋 사용
/pipeline review       # explore → architect → critic → executor
/pipeline implement    # planner → executor → tdd-guide
/pipeline debug        # explore → architect → build-fixer

# 커스텀 파이프라인
/pipeline explore:haiku -> architect:opus -> executor:sonnet
```

### .omc-config.json 설정

```json
{
  "defaultExecutionMode": "ultrawork",
  "modelPreferences": {
    "exploration": "haiku",
    "implementation": "sonnet",
    "review": "opus"
  },
  "parallelWorkers": {
    "max": 5,
    "timeout": 300000
  },
  "verification": {
    "requireArchitectApproval": true,
    "runTestsBeforeComplete": true
  }
}
```

**defaultExecutionMode 동작:**

```
"fast" 또는 "parallel" 키워드 사용 시
  ↓
명시적 키워드 확인 (ulw, eco 등)
  ↓ 없으면
.omc-config.json의 defaultExecutionMode 읽기
  ↓ 없으면
ultrawork가 기본값
```

### Notepad Wisdom System

에이전트 작업 중 발생하는 학습/결정/이슈를 추적하는 지식 관리 시스템.

```
.omc/notepads/{plan-name}/
├── learnings.md     ← 기술적 발견, 코드 패턴
├── decisions.md     ← 아키텍처/설계 결정
├── issues.md        ← 알려진 이슈 및 우회법
└── problems.md      ← 블로커 및 도전과제
```

**API 활용 예시:**

```typescript
// 학습 내용 기록
addLearning({
  plan: "auth-system",
  content: "JWT 토큰 갱신 시 race condition 발생 가능. Refresh Token을 DB에 저장하여 검증해야 함."
})

// 결정 사항 기록
addDecision({
  plan: "auth-system",
  content: "Redis 대신 PostgreSQL로 세션 저장. 이유: 기존 인프라 활용, 추가 서비스 의존성 최소화"
})

// 이전 세션 지식 참조
const wisdom = getWisdomSummary("auth-system")
// → 이전 세션의 학습/결정/이슈 요약 반환
```

---

## 6. Claude가 더 잘 이해하는 프롬프트 패턴

### 시스템 프롬프트에서 확인된 Claude의 행동 규칙

Claude Code의 시스템 프롬프트에는 다음과 같은 행동 규칙이 내장되어 있다. 이를 활용하면 프롬프트 효율을 높일 수 있다.

**규칙 1: "do not propose changes to code you haven't read"**

Claude는 파일을 읽지 않고 수정을 제안하지 않는다. 이를 활용한 프롬프트 패턴:

```
# 좋은 방식
auth.ts 파일을 읽고 JWT 검증 로직을 개선해줘

# 더 구체적인 방식 (Claude가 자동으로 읽게 유도)
src/middleware/auth.ts에서 토큰 만료 처리 부분을 찾아서 에러 메시지를 개선해줘
```

**규칙 2: "If an approach fails, diagnose why before switching tactics"**

실패 시 원인 파악 후 전략을 변경하도록 내장되어 있다. 디버깅 프롬프트 패턴:

```
# 효과적인 디버깅 요청
테스트가 실패하고 있어. 무작정 수정하지 말고:
1. 실패 원인을 먼저 진단해줘
2. 최소한 2가지 해결 접근법을 제시해줘
3. 각 접근법의 트레이드오프를 설명해줘
4. 가장 안전한 방법을 선택해서 수정해줘
```

**규칙 3: "Break down and manage your work with TaskCreate"**

복잡한 작업은 태스크로 분할하게 되어 있다. 활용 방법:

```
# 복잡한 작업은 명시적으로 분할 요청
다음 기능을 구현해줘. 먼저 태스크 목록을 만들고, 각 태스크를 순서대로 완료해:
- 사용자 프로필 페이지
- 프로필 수정 API
- 이미지 업로드 기능
- 변경 이력 로그
```

**규칙 4: "prefer editing existing file to creating new one"**

기존 파일 수정을 선호한다. 새 파일 생성이 필요한 경우:

```
# 새 파일이 필요한 이유를 명시하면 더 잘 따름
utils/validation.ts 파일을 새로 만들어줘.
기존 파일들에서 중복 사용되는 검증 로직을 이곳으로 추출하는 것이 목적이야.
```

### 효과적인 프롬프트 구조

```
[목표] → [제약조건] → [구체적 지시] → [검증 기준]
```

**실전 예시:**

```
목표: 사용자 인증 미들웨어 추가
제약조건:
  - 기존 Express 라우터 구조 유지
  - JWT 라이브러리는 jsonwebtoken 사용
  - 에러 응답은 RFC 7807 Problem Details 형식

구체적 지시:
  1. src/middleware/ 디렉토리에 auth.middleware.ts 생성
  2. 토큰 검증 실패 시 401 반환
  3. 만료된 토큰은 refreshToken 엔드포인트로 리다이렉트
  4. 미들웨어를 src/routes/protected.ts에 적용

검증 기준:
  - tsc --noEmit 통과
  - 기존 테스트 모두 통과
  - 새 미들웨어 단위 테스트 작성 및 통과
```

### 슬래시 명령어 활용 전략

```bash
# 계획 먼저 수립 (구현 전 검토 가능)
/plan 사용자 권한 시스템을 리팩토링하고 싶어

# 컨텍스트 관리 (긴 세션에서 토큰 절약)
/compact

# 특정 도구만 사용하도록 제한
/oh-my-claudecode:ecomode fix TypeScript errors in src/

# 완료까지 중단 없이 실행
/oh-my-claudecode:ralph 전체 마이그레이션 완료해줘
```

### 멀티에이전트 프롬프트 패턴

```
# 병렬 탐색 요청
3개 에이전트를 병렬로 사용해서:
- 에이전트 1: 프론트엔드 컴포넌트 파악
- 에이전트 2: 백엔드 API 엔드포인트 파악
- 에이전트 3: 데이터베이스 스키마 파악
결과를 종합해서 인증 흐름을 설명해줘

# 검증 포함 구현 요청
기능을 구현한 후 반드시:
1. lsp_diagnostics로 타입 오류 0건 확인
2. 테스트 실행 후 결과 첨부
3. architect 에이전트의 코드 리뷰 통과
위 세 단계를 완료한 후에만 완료 선언해줘
```

### Context Window 관리 패턴

```bash
# 긴 세션에서 컨텍스트 관리
/compact        # 대화 요약으로 토큰 절약

# 중요 결정사항은 메모리에 저장
중요: 우리가 선택한 인증 방식 (JWT + Redis)을 기억해줘

# 세션 간 컨텍스트 유지
.omc/notepads에 현재까지의 결정사항을 저장해줘
```

---

## 7. 환경변수 기반 고급 설정

### 핵심 환경변수 목록

| 환경변수 | 기본값 | 설명 |
|----------|--------|------|
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 모델 기본값 | 컨텍스트 윈도우 크기 오버라이드 |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | false | CLAUDE.md 로딩 비활성화 |
| `CLAUDE_CODE_SIMPLE` | false | 최소 도구 모드 |
| `ENABLE_LSP_TOOL` | false | LSP 도구 활성화 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | false | 자동 메모리 비활성화 |

### CLAUDE_CODE_AUTO_COMPACT_WINDOW

컨텍스트 윈도우 크기를 조정하여 자동 압축(auto-compact) 임계값을 제어한다.

```bash
# 더 큰 윈도우 (압축 빈도 감소, 토큰 소비 증가)
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=150000

# 더 작은 윈도우 (압축 빈도 증가, 응답 속도 향상)
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=50000
```

**사용 시나리오:**
- 큰 코드베이스 탐색 → 큰 윈도우 설정
- 반복적인 단순 작업 → 작은 윈도우로 속도 향상

### CLAUDE_CODE_DISABLE_CLAUDE_MDS

```bash
# CLAUDE.md 완전 비활성화
export CLAUDE_CODE_DISABLE_CLAUDE_MDS=true

# 특정 세션에서만 비활성화
CLAUDE_CODE_DISABLE_CLAUDE_MDS=true claude

# 활용 시나리오:
# - A/B 테스트: CLAUDE.md 영향 전후 비교
# - 디버깅: CLAUDE.md가 원인인지 격리 확인
# - 공유 환경: 개인 설정 없이 기본 동작 실행
```

### CLAUDE_CODE_SIMPLE

최소 도구 모드로, Bash, Read, Edit 세 가지 도구만 활성화된다.

```bash
export CLAUDE_CODE_SIMPLE=true

# 활성화된 도구:
# - Bash: 쉘 명령 실행
# - Read: 파일 읽기
# - Edit: 파일 편집

# 비활성화된 도구:
# - Glob, Grep, WebSearch, Task 등
```

**활용 시나리오:**
- 보안 제약이 강한 환경
- 특정 도구만 허용된 샌드박스
- 단순한 자동화 스크립트에서 Claude 활용

### ENABLE_LSP_TOOL

Language Server Protocol 도구를 활성화하여 IDE 수준의 코드 분석을 제공한다.

```bash
export ENABLE_LSP_TOOL=true

# 활성화되는 기능:
# - lsp_diagnostics: 파일별 타입/린트 오류
# - lsp_diagnostics_directory: 프로젝트 전체 오류
# - lsp_goto_definition: 정의로 이동
# - lsp_find_references: 참조 찾기
# - lsp_rename: 심볼 이름 변경
# - lsp_hover: 심볼 정보 조회
# - lsp_workspace_symbols: 심볼 검색
```

**LSP 도구 활용 워크플로:**

```bash
# 1. LSP 서버 확인
lsp_servers()

# 2. 파일 진단
lsp_diagnostics({ file: "src/auth.ts" })

# 3. 프로젝트 전체 진단
lsp_diagnostics_directory({ path: "src/", strategy: "tsc" })

# 4. 오류 수정 후 재확인
lsp_diagnostics({ file: "src/auth.ts" })
```

### CLAUDE_CODE_DISABLE_AUTO_MEMORY

Claude가 자동으로 대화 내용을 메모리에 저장하는 기능을 비활성화한다.

```bash
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true

# 비활성화되는 동작:
# - 중요 결정사항 자동 메모리 저장
# - 세션 간 컨텍스트 자동 유지

# 활용 시나리오:
# - 민감한 정보를 다루는 작업
# - 각 세션을 독립적으로 실행
# - 메모리 파일 오염 방지
```

### 환경변수 조합 예시

```bash
# 프로덕션 자동화 환경 (최소 도구, CLAUDE.md 없음)
export CLAUDE_CODE_SIMPLE=true
export CLAUDE_CODE_DISABLE_CLAUDE_MDS=true
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
claude --bare

# 개발 환경 (LSP 활성화, 큰 컨텍스트 윈도우)
export ENABLE_LSP_TOOL=true
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=120000
claude

# CI/CD 파이프라인 환경
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=80000
claude -p "테스트를 실행하고 실패한 테스트를 수정해줘"
```

---

## 부록: 빠른 참조표

### 자주 쓰는 CLAUDE.md 템플릿

```markdown
# [프로젝트명] 개발 규칙

## 기술 스택
- 언어: TypeScript 5.x
- 런타임: Node.js 20 LTS
- 프레임워크: Express 4.x
- DB: PostgreSQL 15 + Prisma ORM
- 테스트: Jest + Supertest

## 코드 규칙
- `any` 타입 금지
- 함수당 최대 30줄
- 순환 참조 금지
- 파일당 최대 200줄

## 커밋 규칙
- Conventional Commits 형식
- 예: feat(auth): add JWT refresh token support

## 금지 사항
- console.log 프로덕션 코드 삽입
- TODO 주석 남기기
- 타입 어설션 남용

## 테스트 정책
- 모든 API 엔드포인트는 통합 테스트 필수
- 비즈니스 로직은 단위 테스트 필수
- 커버리지 80% 이상 유지
```

### 에이전트 선택 빠른 가이드

```
질문이 있어요 (단순)      → Claude에게 직접
파일 찾아줘               → explore (haiku)
코드 탐색해줘             → explore-medium (sonnet)
아키텍처 분석해줘         → architect (opus)
한 줄 수정해줘            → executor-low (haiku)
기능 구현해줘             → executor (sonnet)
복잡한 리팩토링해줘       → executor-high (opus)
UI 컴포넌트 만들어줘      → designer (sonnet)
보안 취약점 찾아줘        → security-reviewer (opus)
테스트 작성해줘           → qa-tester (sonnet)
```

### 검증 체크리스트

```
구현 완료 전 반드시 확인:
[ ] lsp_diagnostics 오류 0건
[ ] 빌드 통과 (tsc --noEmit or npm run build)
[ ] 관련 테스트 모두 통과
[ ] CLAUDE.md 규칙 위반 없음
[ ] console.log, TODO, HACK 코드 없음
[ ] 새 파일은 기존 패턴과 일치
```

