# 사내 코딩 컨벤션 + Claude Code 실전 하네싱 가이드

> 기존 코드 스타일을 유지하면서 새 기능 추가, 리팩토링, 로직 수정을 하는 개발자를 위한 가이드

---

## 목차
1. [컨벤션 관리 전략: CLAUDE.md vs 스킬 vs 하이브리드](#1-컨벤션-관리-전략)
2. [CLAUDE.md 핵심 원칙 템플릿](#2-claudemd-핵심-원칙-템플릿)
3. [컨벤션 스킬 작성법](#3-컨벤션-스킬-작성법)
4. ["기존 코드처럼" 만들게 하는 프롬프트 전략](#4-기존-코드처럼-만들게-하는-프롬프트-전략)
5. [실전 시나리오별 워크플로우](#5-실전-시나리오별-워크플로우)
6. [Confluence → Claude Code 변환 가이드](#6-confluence--claude-code-변환-가이드)
7. [settings.json 권한 설정](#7-settingsjson-권한-설정)
8. [Hooks로 컨벤션 자동 검증](#8-hooks로-컨벤션-자동-검증)

---

## 1. 컨벤션 관리 전략

### 왜 하이브리드인가? (소스코드 근거)

Claude Code의 `context.ts`에서 CLAUDE.md는 세션 시작 시 1회 로딩되어 시스템 프롬프트에 주입됩니다:

```
getUserContext() → getClaudeMds() → getMemoryFiles()
  → 결과가 시스템 프롬프트의 "claudeMd" 섹션에 포함
  → 매 API 호출마다 이 내용이 전송됨 (토큰 소비)
```

반면 스킬(`skills/loadSkillsDir.ts`)은:
```
SkillTool.call() 시에만 마크다운 파일 전체 내용을 로딩
  → 호출 안 하면 이름+설명(frontmatter)만 존재
  → 토큰 절약
```

### 의사결정 매트릭스

| 컨벤션 길이 | 전략 | 이유 |
|------------|------|------|
| **~30줄 이하** | CLAUDE.md만 | 토큰 부담 적음, 항상 인지 |
| **30~100줄** | CLAUDE.md 요약 + 스킬 상세 | 핵심만 항상 인지, 상세는 필요시 |
| **100줄 이상** | CLAUDE.md 최소 원칙 + 스킬 분리 | 토큰 절약 필수 |
| **500줄 이상 (Confluence 전체)** | CLAUDE.md 원칙 + 스킬 여러 개 분리 | 도메인별 스킬 분리 |

### 세 가지 계층 구조

```
프로젝트/
├── .claude/
│   ├── CLAUDE.md              ← 핵심 원칙 (항상 로딩, ~30줄)
│   ├── skills/
│   │   ├── coding-convention.md    ← 전체 코딩 컨벤션 (필요 시 로딩)
│   │   ├── api-convention.md       ← API 설계 컨벤션
│   │   ├── test-convention.md      ← 테스트 컨벤션
│   │   └── review-checklist.md     ← 코드 리뷰 체크리스트
│   └── settings.json          ← 권한/훅 설정
```

---

## 2. CLAUDE.md 핵심 원칙 템플릿

**핵심: 짧고, 구체적이고, "기존 코드를 따르라"는 지시를 포함**

### 템플릿 A: 프론트엔드 프로젝트

```markdown
# 프로젝트 컨벤션

## 필수 원칙
- 새 코드는 반드시 같은 디렉토리의 기존 파일 패턴을 따른다
- 기존 코드를 먼저 읽고, 동일한 구조/네이밍/패턴으로 작성한다
- 컨벤션이 불확실하면 가장 최근에 수정된 유사 파일을 참조한다

## 기술 스택
- Next.js 14 (App Router), TypeScript strict
- Tailwind CSS + shadcn/ui
- Zustand (상태관리), React Query (서버 상태)
- Vitest + Testing Library (테스트)

## 네이밍 규칙
- 컴포넌트: PascalCase (UserProfile.tsx)
- 훅: camelCase, use 접두사 (useUserProfile.ts)
- 유틸: camelCase (formatDate.ts)
- API: camelCase, 동사 시작 (fetchUsers.ts)
- 타입: PascalCase, I/T 접두사 없음 (UserProfile, not IUserProfile)

## 금지 사항
- any 타입 사용 금지
- console.log 커밋 금지 (logger 사용)
- 인라인 스타일 금지 (Tailwind 사용)
- default export 금지 (named export만)

## 상세 컨벤션
- 전체 코딩 컨벤션: `/coding-convention` 스킬 참조
- API 컨벤션: `/api-convention` 스킬 참조
- 테스트 컨벤션: `/test-convention` 스킬 참조

## 작업 전 체크
- 관련 파일을 먼저 읽고 패턴을 파악한 후 작업 시작
- 비슷한 기능이 이미 구현되어 있는지 Grep으로 검색
```

### 템플릿 B: 백엔드 프로젝트

```markdown
# 프로젝트 컨벤션

## 필수 원칙
- 기존 코드와 동일한 패턴/구조/네이밍을 따른다
- 새 기능 추가 시 가장 유사한 기존 구현을 먼저 찾아 참조한다
- 리팩토링 시 기존 테스트가 깨지지 않도록 한다

## 기술 스택
- NestJS + TypeScript strict
- TypeORM + PostgreSQL
- Jest (테스트)
- Swagger (API 문서)

## 아키텍처 패턴
- Controller → Service → Repository 3계층
- DTO로 입출력 정의 (class-validator)
- Entity와 DTO를 분리 (직접 Entity 반환 금지)
- 에러는 커스텀 Exception으로 처리

## 네이밍
- 파일: kebab-case (user-profile.service.ts)
- 클래스: PascalCase (UserProfileService)
- 메서드: camelCase, 동사 시작 (findById, createUser)
- 상수: UPPER_SNAKE_CASE

## 상세 컨벤션: `/coding-convention` 스킬 참조
```

### 템플릿 C: 풀스택 모노레포

```markdown
# 프로젝트 컨벤션

## 필수 원칙
- 새 코드는 동일 모듈의 기존 패턴을 반드시 따른다
- 수정 전 기존 코드를 먼저 읽고 패턴을 파악한다
- 프론트/백 공통 타입은 packages/shared/에 정의

## 구조
- apps/web/ → Next.js 프론트엔드
- apps/api/ → NestJS 백엔드
- packages/shared/ → 공통 타입/유틸

## 핵심 규칙
- apps/web/ 작업 시 → apps/web/ 내의 기존 패턴 따름
- apps/api/ 작업 시 → apps/api/ 내의 기존 패턴 따름
- 타입 추가 시 → packages/shared/types/ 확인 후 추가
- API 변경 시 → 프론트/백 둘 다 수정

## 상세: `/coding-convention` 참조
```

---

## 3. 컨벤션 스킬 작성법

### 기본 구조

```markdown
<!-- .claude/skills/coding-convention.md -->
---
description: "사내 코딩 컨벤션 전체 가이드. 새 코드 작성이나 리팩토링 시 참조"
whenToUse: "코드 작성 시 컨벤션이 불확실할 때, 새 파일을 생성할 때, 리팩토링 할 때"
---

# 코딩 컨벤션 가이드

(여기에 Confluence의 컨벤션 내용을 마크다운으로 변환하여 작성)

## 1. 네이밍 규칙
...

## 2. 파일 구조
...

## 3. 에러 처리
...

## 4. 로깅
...

## 5. 테스트
...
```

### 도메인별 스킬 분리 (긴 컨벤션용)

**API 컨벤션 스킬:**
```markdown
<!-- .claude/skills/api-convention.md -->
---
description: "REST API 설계 및 구현 컨벤션. API 엔드포인트 생성, 수정 시 참조"
whenToUse: "API 엔드포인트를 새로 만들거나 수정할 때"
---

# API 설계 컨벤션

## URL 패턴
- 리소스: 복수형 (GET /users, POST /users)
- 단일 리소스: /users/:id
- 하위 리소스: /users/:id/posts
- 액션: /users/:id/activate (동사는 예외적으로 허용)

## 요청/응답 포맷
- 요청: camelCase JSON
- 응답: 표준 래퍼 사용
  ```json
  {
    "success": true,
    "data": { ... },
    "meta": { "page": 1, "total": 100 }
  }
  ```
- 에러: 표준 에러 포맷
  ```json
  {
    "success": false,
    "error": {
      "code": "USER_NOT_FOUND",
      "message": "사용자를 찾을 수 없습니다"
    }
  }
  ```

## 상태 코드
- 200: 조회 성공
- 201: 생성 성공
- 204: 삭제 성공 (본문 없음)
- 400: 잘못된 요청 (유효성 검증 실패)
- 401: 인증 필요
- 403: 권한 없음
- 404: 리소스 없음
- 409: 충돌 (중복 등)
- 500: 서버 에러

## DTO 패턴
- 요청 DTO: Create{Resource}Dto, Update{Resource}Dto
- 응답 DTO: {Resource}ResponseDto
- 목록 응답: {Resource}ListResponseDto
...
```

**테스트 컨벤션 스킬:**
```markdown
<!-- .claude/skills/test-convention.md -->
---
description: "테스트 작성 컨벤션. 단위/통합 테스트 작성 시 참조"
whenToUse: "테스트를 새로 작성하거나 기존 테스트를 수정할 때"
---

# 테스트 컨벤션

## 파일 위치
- 단위 테스트: 소스 파일 옆 (__tests__/ 또는 .test.ts)
- 통합 테스트: test/integration/
- E2E 테스트: test/e2e/

## 네이밍
- describe: 클래스/함수명
- it/test: "should + 행위 + 조건"

## 패턴
```typescript
describe('UserService', () => {
  describe('findById', () => {
    it('should return user when valid id given', async () => {
      // Given (준비)
      const userId = 'test-id';
      mockRepository.findOne.mockResolvedValue(mockUser);

      // When (실행)
      const result = await service.findById(userId);

      // Then (검증)
      expect(result).toEqual(mockUser);
      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: userId } });
    });

    it('should throw NotFoundException when user not found', async () => {
      // Given
      mockRepository.findOne.mockResolvedValue(null);

      // When & Then
      await expect(service.findById('invalid')).rejects.toThrow(NotFoundException);
    });
  });
});
```
...
```

### 스킬에서 자동 참조 트리거

`whenToUse` 필드를 잘 작성하면 Claude가 자동으로 관련 스킬을 찾아 호출합니다:

```markdown
---
whenToUse: "새 API 엔드포인트를 만들 때, REST API를 수정할 때, DTO를 작성할 때"
---
```

이렇게 하면 "새 유저 API를 만들어줘"라고 말했을 때 Claude가 자동으로 `/api-convention` 스킬을 호출하여 컨벤션을 참조합니다.

---

## 4. "기존 코드처럼" 만들게 하는 프롬프트 전략

### Claude Code 시스템 프롬프트에 이미 내장된 규칙

소스코드 `constants/prompts.ts`에서 확인된 Claude의 기본 행동 규칙:

```
"In general, do not propose changes to code you haven't read."
→ Claude는 코드를 먼저 읽어야 한다는 규칙이 있음

"Generally prefer editing an existing file to creating a new one"
→ 기존 파일 수정을 선호함
```

이 규칙을 **강화**하는 것이 핵심입니다.

### 전략 1: CLAUDE.md에 "참조 우선" 지시

```markdown
## 필수 워크플로우
1. 새 코드 작성 전에 같은 디렉토리의 기존 파일을 반드시 2개 이상 읽는다
2. 비슷한 기능이 이미 구현되어 있는지 Grep으로 검색한다
3. 기존 패턴과 동일한 구조로 작성한다
4. 네이밍, import 순서, 코드 구조가 기존 파일과 일치하는지 확인한다
```

### 전략 2: 구체적 프롬프트 패턴

**나쁜 프롬프트:**
```
유저 프로필 API를 만들어줘
```

**좋은 프롬프트:**
```
유저 프로필 조회 API를 만들어줘.
기존 src/modules/auth/ 에 있는 AuthController, AuthService 패턴을 따라서
src/modules/user/ 에 UserController, UserService를 만들어줘.
기존 코드의 에러 처리, DTO 패턴, 로깅 방식을 그대로 적용해.
```

**더 좋은 프롬프트:**
```
src/modules/auth/auth.controller.ts와 auth.service.ts를 먼저 읽고,
동일한 패턴으로 src/modules/user/ 에 유저 프로필 CRUD API를 만들어줘.
```

### 전략 3: 리팩토링 시

```
src/services/payment.service.ts의 processPayment 함수를 리팩토링해줘.
같은 파일의 다른 함수들(refundPayment, cancelPayment) 스타일을 따라서,
에러 처리와 로깅 패턴을 일관되게 만들어줘.
변경 전에 기존 테스트가 모두 통과하는지 확인해.
```

### 전략 4: 기능 추가 시

```
src/modules/order/order.service.ts에 주문 취소 기능을 추가해줘.
1. 먼저 이 파일과 order.controller.ts를 읽어줘
2. 기존 createOrder, updateOrder 패턴을 따라서 cancelOrder를 만들어줘
3. 기존 에러 처리 패턴(OrderNotFoundException 등)을 그대로 사용해
4. 테스트도 기존 order.service.spec.ts 패턴을 따라서 추가해
```

---

## 5. 실전 시나리오별 워크플로우

### 시나리오 A: 새 기능 추가

```
1단계: 컨텍스트 설정
  "src/modules/ 아래 기존 모듈 구조를 확인해줘" (Glob으로 탐색)

2단계: 유사 구현 참조
  "src/modules/product/ 를 읽어줘. 이 패턴을 따라서 review 모듈을 만들 거야"

3단계: 구현 요청
  "product 모듈과 동일한 패턴으로 review 모듈을 만들어줘.
   Controller, Service, Repository, DTO, Entity, Module 파일 모두 필요해.
   /coding-convention 스킬의 컨벤션을 따라줘"

4단계: 검증
  "테스트 실행해줘" → "린트 확인해줘" → "빌드 확인해줘"
```

### 시나리오 B: 기존 로직 수정

```
1단계: 현재 상태 파악
  "src/services/notification.service.ts를 읽고 sendEmail 함수의 동작을 설명해줘"

2단계: 수정 요청
  "sendEmail 함수에 재시도 로직을 추가해줘.
   같은 파일의 sendSMS 함수에 이미 구현된 재시도 패턴을 따라서 구현해"

3단계: 검증
  "기존 테스트가 통과하는지 확인하고, 새 테스트도 추가해줘"
```

### 시나리오 C: 리팩토링

```
1단계: 범위 확인
  "src/utils/helpers.ts에서 1000줄이 넘는 함수들을 찾아줘"

2단계: 참조 패턴 확인
  "src/utils/ 아래 다른 파일들은 어떤 구조로 분리되어 있어?
   가장 잘 정리된 파일을 참조 모델로 보여줘"

3단계: 리팩토링 요청
  "helpers.ts를 참조 모델과 동일한 패턴으로 분리해줘.
   기존 import를 쓰는 모든 파일도 함께 업데이트해줘.
   리팩토링 후 모든 테스트가 통과해야 해"
```

---

## 6. Confluence → Claude Code 변환 가이드

### 단계 1: Confluence 문서 구조 파악

Confluence 컨벤션 문서를 다음 카테고리로 분류:

| 카테고리 | CLAUDE.md | 스킬 |
|----------|-----------|------|
| 핵심 원칙 (5-10개) | O | - |
| 네이밍 규칙 | 요약만 | 상세 |
| 파일 구조 | 요약만 | 상세 |
| 코드 패턴 (예시 포함) | - | O |
| API 컨벤션 | - | O (별도 스킬) |
| 테스트 컨벤션 | - | O (별도 스킬) |
| Git/PR 규칙 | 요약만 | 상세 |
| 코드 리뷰 체크리스트 | - | O (별도 스킬) |

### 단계 2: CLAUDE.md에 핵심만 추출

Confluence에서 **"MUST", "NEVER", "ALWAYS"** 키워드가 있는 규칙만 추출:

```markdown
## 핵심 컨벤션 (위반 시 PR 리젝)
- MUST: 함수는 50줄 이하로 유지
- MUST: 모든 public API에 JSDoc 작성
- NEVER: any 타입 사용 금지
- NEVER: console.log 커밋 금지
- ALWAYS: 에러는 커스텀 Exception으로 처리
- ALWAYS: 새 기능에 단위 테스트 필수
```

### 단계 3: 스킬로 상세 변환

Confluence의 마크다운 export → 스킬 파일로 변환:

```bash
# Confluence에서 마크다운 export (페이지 → ... → Export → Markdown)
# export된 파일을 .claude/skills/ 로 복사
cp ~/Downloads/coding-convention.md .claude/skills/coding-convention.md
```

그 후 스킬 frontmatter 추가:
```markdown
---
description: "사내 코딩 컨벤션 전체 (Confluence 기반). 코드 작성 시 참조"
whenToUse: "새 코드를 작성하거나 리팩토링할 때 컨벤션이 불확실한 경우"
---

(Confluence 내용)
```

### 단계 4: 컨벤션이 업데이트될 때

Confluence 컨벤션이 변경되면:
1. 변경된 부분만 스킬 파일 업데이트
2. CLAUDE.md의 핵심 원칙이 바뀌었으면 CLAUDE.md도 업데이트
3. Claude Code에서 `/reload-plugins`로 스킬 리로드 (또는 새 세션 시작)

---

## 7. settings.json 권한 설정

사내 프로젝트에서 안전하게 사용하기 위한 권한 설정:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm test *)",
      "Bash(npm run lint *)",
      "Bash(npm run build)",
      "Bash(npm run typecheck)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git branch *)",
      "Bash(npx prisma generate)",
      "Bash(npx prisma migrate dev *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Bash(git reset --hard *)",
      "Bash(npm publish *)",
      "Bash(docker push *)",
      "Bash(curl * | sh)",
      "Bash(wget * | sh)"
    ]
  }
}
```

---

## 8. Hooks로 컨벤션 자동 검증

### PostToolUse: 파일 수정 후 자동 린트

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "if echo \"$TOOL_INPUT\" | grep -qE '\\.(ts|tsx)$'; then npx eslint --fix \"$(echo $TOOL_INPUT | jq -r '.file_path // .path // empty')\" 2>/dev/null; fi"
      }
    ]
  }
}
```

### PreToolUse: 위험한 파일 수정 방지

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "FILE=$(echo $TOOL_INPUT | jq -r '.file_path // .path // empty'); if echo \"$FILE\" | grep -qE '(migration|seed|\.env)'; then echo '{\"decision\": \"ask\", \"reason\": \"마이그레이션/시드/환경 파일 수정은 확인이 필요합니다\"}'; fi"
      }
    ]
  }
}
```

### SessionStart: 환경 검증

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "node -v && npm -v && echo 'Node/npm OK' || echo 'WARNING: Node.js 환경을 확인해주세요'"
      }
    ]
  }
}
```

---

## 요약: 최적 설정 체크리스트

- [ ] CLAUDE.md에 핵심 원칙 20-30줄 작성
- [ ] CLAUDE.md에 "기존 코드 패턴을 따르라" 지시 포함
- [ ] CLAUDE.md에 스킬 참조 안내 포함 (`/coding-convention` 참조)
- [ ] 전체 컨벤션을 `.claude/skills/coding-convention.md`로 분리
- [ ] 긴 컨벤션은 도메인별 스킬로 추가 분리 (api, test, review)
- [ ] 스킬 frontmatter에 `whenToUse` 작성 (자동 호출 트리거)
- [ ] settings.json에 안전 권한 설정
- [ ] Hooks로 린트/포맷 자동 실행 설정
- [ ] 프롬프트 시 "기존 파일 먼저 읽고 같은 패턴으로" 습관화

---

## 부록: 컨벤션 길이별 토큰 비용 추정

| 컨벤션 길이 | 대략 토큰 수 | CLAUDE.md에 넣을 때 세션 비용 영향 |
|------------|-------------|----------------------------------|
| 30줄 | ~200 토큰 | 미미 (권장) |
| 100줄 | ~700 토큰 | 약간 (허용 가능) |
| 300줄 | ~2,000 토큰 | 체감됨 (스킬 분리 권장) |
| 500줄 | ~3,500 토큰 | 상당함 (스킬 분리 필수) |
| 1000줄+ | ~7,000+ 토큰 | 과다 (반드시 스킬로 분리) |

> 참고: Claude Code의 auto compact 임계치는 `contextWindow - 33,000 토큰`입니다.
> CLAUDE.md가 너무 길면 실제 작업 공간이 줄어들어 더 자주 compact가 발생합니다.

---

*이 문서는 Claude Code 소스코드(context.ts, prompts.ts, skills/loadSkillsDir.ts) 분석에 기반합니다.*
