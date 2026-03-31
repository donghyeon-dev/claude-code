# Claude Code 개발자 유형별 완전 가이드

> 이 문서는 다양한 개발자 페르소나별로 Claude Code를 효과적으로 활용하는 방법을 안내합니다.
> 각 섹션은 실무에서 바로 사용할 수 있는 워크플로우, 프롬프트 예시, 설정 템플릿을 포함합니다.

---

## 목차

1. [프론트엔드 개발자 (React/Next.js/Vue)](#1-프론트엔드-개발자-reactnextjsvue)
2. [백엔드 개발자 (Node.js/Python/Go)](#2-백엔드-개발자-nodejspythongo)
3. [DevOps/인프라 엔지니어](#3-devops인프라-엔지니어)
4. [주니어 개발자/학습자](#4-주니어-개발자학습자)
5. [풀스택 개발자](#5-풀스택-개발자)
6. [모바일 개발자 (React Native/Flutter)](#6-모바일-개발자-react-nativeflutter)

---

## 1. 프론트엔드 개발자 (React/Next.js/Vue)

프론트엔드 개발자는 UI 컴포넌트 생성, 스타일 디버깅, 반응형 디자인, 상태 관리 등 다양한 작업을 Claude Code와 함께 수행할 수 있습니다. 시각적 결과물을 빠르게 만들어내야 하는 프론트엔드 특성상, Claude Code의 파일 탐색 및 수정 기능을 적극 활용하는 것이 중요합니다.

### 핵심 워크플로우

#### 워크플로우 1: 컴포넌트 생성 및 구조화

React 또는 Vue 컴포넌트를 새로 만들 때 Claude Code는 프로젝트의 기존 패턴을 분석하여 일관성 있는 컴포넌트를 생성합니다. 먼저 기존 컴포넌트 구조를 파악한 뒤 새 컴포넌트를 작성하는 것이 좋습니다.

**단계:**
1. 프로젝트의 컴포넌트 구조 파악 (`src/components/` 탐색)
2. 기존 컴포넌트 패턴 분석 (네이밍, Props 타입, 스타일 방식)
3. 새 컴포넌트 생성 요청
4. 스토리북 또는 테스트 파일 자동 생성

#### 워크플로우 2: CSS/Tailwind 디버깅

스타일 문제가 발생했을 때 Claude Code의 Glob과 Grep 기능으로 관련 클래스와 스타일 정의를 빠르게 찾아낼 수 있습니다.

**단계:**
1. 문제가 되는 컴포넌트 파일 특정
2. Grep으로 동일한 클래스를 사용하는 파일 검색
3. Tailwind 설정 파일(`tailwind.config.js`) 확인
4. `globals.css` 또는 커스텀 유틸리티 파일 분석
5. 충돌 클래스 수정 또는 커스텀 유틸리티 추가

#### 워크플로우 3: 반응형 디자인 작업

모바일 퍼스트 또는 데스크탑 퍼스트 디자인을 구현할 때 미디어 쿼리와 Tailwind의 반응형 접두사를 효과적으로 관리합니다.

**단계:**
1. 현재 뷰포트별 스타일 상태 파악
2. 브레이크포인트 정의 확인 (`tailwind.config.js` 또는 CSS 변수)
3. 각 뷰포트 크기에 맞는 스타일 추가/수정
4. 접근성(ARIA) 속성 함께 점검

#### 워크플로우 4: 테스트 자동 생성

컴포넌트 로직과 사용자 인터랙션에 대한 테스트를 자동으로 생성하여 품질을 확보합니다.

**단계:**
1. 테스트할 컴포넌트의 Props와 이벤트 핸들러 파악
2. Jest/Vitest + Testing Library 테스트 생성
3. 엣지 케이스(빈 값, 긴 텍스트, 오류 상태) 테스트 추가
4. 스냅샷 테스트 또는 시각적 회귀 테스트 연결

#### 워크플로우 5: 성능 최적화

번들 크기, 렌더링 성능, 코드 스플리팅 등 프론트엔드 성능 이슈를 분석하고 개선합니다.

**단계:**
1. 번들 분석 결과 파일 공유 (webpack-bundle-analyzer 등)
2. 불필요한 재렌더링 원인 파악
3. `useMemo`, `useCallback`, `React.memo` 적용
4. 이미지 최적화 및 lazy loading 구현

### 실전 프롬프트 예시

```
# 컴포넌트 생성
"src/components/ui/ 디렉토리를 확인하고, 기존 컴포넌트 패턴에 맞게
UserProfileCard 컴포넌트를 만들어줘. Props는 name, avatar, role, email이고
Tailwind CSS로 스타일링해줘. TypeScript 타입도 포함해줘."

# 스타일 디버깅
"버튼 컴포넌트에서 hover 상태가 모바일에서 이상하게 보여.
src/components/Button.tsx와 관련 CSS 파일을 분석하고
문제 원인을 찾아서 수정해줘."

# 반응형 수정
"src/pages/dashboard/page.tsx의 사이드바가 태블릿(768px~1024px)에서
메인 컨텐츠와 겹쳐. Tailwind 반응형 클래스로 수정해줘."

# 테스트 생성
"src/components/LoginForm.tsx에 대한 Vitest + Testing Library 테스트를
작성해줘. 이메일 유효성 검사, 비밀번호 길이 체크, 제출 성공/실패 케이스를
모두 커버해야 해."

# 성능 분석
"아래 컴포넌트 코드를 분석해서 불필요한 재렌더링 원인을 찾아줘.
useEffect 의존성 배열과 콜백 함수 메모이제이션 문제가 있는지 확인해줘."

# API 연동
"src/hooks/ 디렉토리를 확인하고 기존 패턴에 맞게
useUserData 커스텀 훅을 만들어줘. React Query v5를 사용하고
로딩, 에러, 성공 상태를 모두 처리해야 해."

# 접근성 개선
"src/components/Modal.tsx를 WAI-ARIA 접근성 표준에 맞게 수정해줘.
포커스 트랩, ESC 키 닫기, role='dialog' 속성이 필요해."
```

### CLAUDE.md 템플릿 (React/Next.js 프로젝트용)

```markdown
# 프로젝트 설정

이 프로젝트는 Next.js 14 App Router를 사용합니다.

## 기술 스택
- 프레임워크: Next.js 14 (App Router)
- 언어: TypeScript 5.x
- 스타일링: Tailwind CSS v3
- UI 라이브러리: shadcn/ui
- 상태관리: Zustand
- 서버 상태: TanStack Query (React Query) v5
- 폼: React Hook Form + Zod
- 테스트: Vitest + Testing Library

## 컴포넌트 규칙
- 서버 컴포넌트를 기본으로 사용
- 클라이언트 컴포넌트는 파일 최상단에 'use client' 지시문 필요
- 컴포넌트 파일명: PascalCase (예: UserProfile.tsx)
- 훅 파일명: camelCase, use 접두사 (예: useUserData.ts)
- 컴포넌트는 src/components/{domain}/ 디렉토리에 배치

## 스타일링 규칙
- Tailwind CSS 클래스 사용 (인라인 CSS 사용 금지)
- 커스텀 유틸리티: src/app/globals.css에 정의
- 컴포넌트별 변형(variant)은 shadcn/ui의 cva() 패턴 사용
- 다크모드: dark: 접두사 사용 (class 기반 다크모드)

## 디렉토리 구조
- src/app/ - Next.js 라우트와 레이아웃
- src/components/ - 재사용 가능한 UI 컴포넌트
- src/components/ui/ - shadcn/ui 기반 기본 컴포넌트
- src/hooks/ - 커스텀 React 훅
- src/lib/ - 유틸리티 함수
- src/store/ - Zustand 스토어
- src/types/ - TypeScript 타입 정의

## 코드 스타일
- 함수 컴포넌트만 사용 (클래스 컴포넌트 금지)
- Props 타입은 interface로 정의 (type alias 대신)
- 기본 내보내기(default export) 사용
- 절대 경로 임포트: @/로 시작 (예: @/components/Button)

## 금지 사항
- any 타입 사용 금지
- console.log 코드에 남기지 말 것
- 직접 DOM 조작 금지 (ref 사용)
```

### settings.json 권한 설정 (프론트엔드)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run dev)",
      "Bash(npm run build)",
      "Bash(npm run test)",
      "Bash(npm run lint)",
      "Bash(npm install *)",
      "Bash(npx shadcn-ui@latest add *)",
      "Bash(git diff)",
      "Bash(git status)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/review` | 현재 변경사항 코드 리뷰 요청 |
| `/test` | 선택한 파일에 대한 테스트 생성 |
| `/explain` | 선택한 코드 블록 설명 요청 |
| `/fix` | 현재 린트 에러 또는 타입 에러 수정 |
| `/optimize` | 선택 코드 성능 최적화 |

---

## 2. 백엔드 개발자 (Node.js/Python/Go)

백엔드 개발자는 API 설계, 데이터베이스 관리, 비즈니스 로직 구현, 성능 최적화 등 서버 사이드 작업에서 Claude Code를 활용합니다. 코드의 안전성과 확장성이 중요한 백엔드 영역에서 Claude Code는 패턴 분석과 테스트 자동화에 특히 유용합니다.

### 핵심 워크플로우

#### 워크플로우 1: API 엔드포인트 설계 및 구현

RESTful API 또는 GraphQL 엔드포인트를 설계할 때 기존 라우팅 패턴을 분석하고 일관성 있는 구조로 새 엔드포인트를 추가합니다.

**단계:**
1. 기존 라우터/컨트롤러 파일 구조 파악
2. 미들웨어 체인 확인 (인증, 유효성 검사, 에러 처리)
3. 새 엔드포인트 요구사항 명세화
4. 컨트롤러 → 서비스 → 레포지토리 레이어 순서로 구현
5. OpenAPI/Swagger 문서 업데이트

#### 워크플로우 2: DB 마이그레이션 작성

스키마 변경이 필요할 때 안전한 마이그레이션 스크립트를 작성하고 롤백 계획을 수립합니다.

**단계:**
1. 현재 스키마 파일 분석
2. 변경 사항 영향도 검토 (외래 키, 인덱스, 기존 데이터)
3. 마이그레이션 스크립트 생성 (up/down 포함)
4. 시딩 데이터 업데이트
5. 마이그레이션 테스트 환경 검증

#### 워크플로우 3: 테스트 자동화

단위 테스트, 통합 테스트, 엔드포인트 테스트를 체계적으로 작성하여 회귀 방지를 강화합니다.

**단계:**
1. 테스트할 서비스/함수 로직 파악
2. Mock 및 Stub 전략 결정 (DB 모킹, 외부 API 모킹)
3. 해피 패스와 엣지 케이스 테스트 작성
4. 통합 테스트로 실제 DB 연동 검증
5. CI 파이프라인에 테스트 커버리지 임계값 설정

#### 워크플로우 4: 성능 프로파일링 및 최적화

느린 쿼리, 메모리 누수, CPU 병목을 분석하고 개선합니다.

**단계:**
1. 프로파일링 도구 실행 결과 공유 (py-spy, Go pprof, clinic.js 등)
2. N+1 쿼리 패턴 탐지
3. 캐싱 전략 검토 (Redis, 인메모리)
4. 데이터베이스 인덱스 최적화 제안
5. 비동기 처리 및 큐 시스템 도입 검토

#### 워크플로우 5: 보안 취약점 검토

인증, 입력 검증, SQL 인젝션, 민감 데이터 노출 등 보안 이슈를 자동으로 탐지합니다.

**단계:**
1. 인증/인가 미들웨어 검토
2. 입력 유효성 검사 누락 지점 탐색
3. 환경 변수 및 시크릿 관리 방식 확인
4. OWASP Top 10 기준 코드 스캔
5. 보안 헤더 설정 확인

### 실전 프롬프트 예시

```
# API 엔드포인트 생성
"src/routes/ 디렉토리를 분석하고 기존 라우터 패턴에 맞게
POST /api/v1/orders 엔드포인트를 구현해줘.
Zod로 요청 바디 유효성 검사, JWT 인증 미들웨어 적용,
트랜잭션으로 주문 생성 + 재고 차감을 원자적으로 처리해야 해."

# Prisma 마이그레이션
"User 모델에 refreshToken 필드(String, nullable)와
lastLoginAt 필드(DateTime, nullable)를 추가하는
Prisma 마이그레이션을 만들어줘. 기존 데이터 호환성 유지해야 해."

# 파이썬 단위 테스트
"src/services/payment_service.py를 분석하고
pytest로 단위 테스트를 작성해줘. Stripe API는 unittest.mock으로 모킹하고,
결제 성공, 카드 거절, 네트워크 오류 케이스를 모두 커버해야 해."

# Go API 개발
"internal/handler/ 패턴을 참고해서
GET /api/products/:id 핸들러를 구현해줘.
Redis 캐싱 (TTL 5분), PostgreSQL 폴백,
Not Found 시 404 응답을 포함해야 해."

# 성능 최적화
"아래 SQL 쿼리가 프로덕션에서 2초 이상 걸려.
실행 계획(EXPLAIN ANALYZE 결과)을 보고
인덱스 전략과 쿼리 최적화 방안을 제안해줘."

# 보안 검토
"src/middleware/auth.ts를 검토해서 JWT 검증 로직의
보안 취약점을 찾아줘. 토큰 만료 처리, 알고리즘 검증,
비밀 키 강도 등을 확인해줘."
```

### CLAUDE.md 템플릿 (Node.js 백엔드 프로젝트용)

```markdown
# 백엔드 프로젝트 설정

## 기술 스택
- 런타임: Node.js 20 LTS
- 프레임워크: Express.js 4.x / Fastify 4.x
- 언어: TypeScript 5.x
- ORM: Prisma 5.x
- 데이터베이스: PostgreSQL 15
- 캐시: Redis 7
- 인증: JWT (jose 라이브러리)
- 유효성 검사: Zod
- 테스트: Jest + Supertest
- 문서: Swagger/OpenAPI 3.0

## 아키텍처 패턴
- 레이어드 아키텍처: Router → Controller → Service → Repository
- 의존성 주입: tsyringe 사용
- 에러 처리: 커스텀 AppError 클래스, 글로벌 에러 핸들러
- 로깅: Pino (JSON 포맷, 레벨: info/warn/error)

## 디렉토리 구조
- src/routes/ - Express 라우터
- src/controllers/ - 요청/응답 처리
- src/services/ - 비즈니스 로직
- src/repositories/ - 데이터 접근 레이어
- src/middleware/ - 인증, 유효성 검사, 에러 처리
- src/types/ - TypeScript 타입
- src/utils/ - 유틸리티 함수
- prisma/ - 스키마 및 마이그레이션

## 코딩 규칙
- 모든 컨트롤러 메서드는 async/await 사용
- try/catch 대신 asyncHandler 래퍼 사용
- 환경 변수는 src/config/env.ts에서 Zod로 검증
- 데이터베이스 ID는 UUID (cuid2) 사용
- API 응답은 { success, data, error } 형태로 통일

## 금지 사항
- any 타입 사용 금지
- 직접 process.env 접근 금지 (config 모듈 사용)
- console.log 사용 금지 (logger 사용)
- Prisma 쿼리를 컨트롤러에 직접 작성 금지
```

### CLAUDE.md 템플릿 (Python/FastAPI 프로젝트용)

```markdown
# FastAPI 백엔드 프로젝트 설정

## 기술 스택
- 프레임워크: FastAPI 0.110+
- 언어: Python 3.11+
- ORM: SQLAlchemy 2.0 (async)
- 마이그레이션: Alembic
- 유효성 검사: Pydantic v2
- 비동기 DB 드라이버: asyncpg
- 캐시: Redis (aioredis)
- 인증: python-jose (JWT)
- 테스트: pytest + pytest-asyncio + httpx

## 코딩 규칙
- 타입 힌트 필수 (mypy strict 모드)
- 비동기 함수 우선 (async def)
- 의존성 주입: FastAPI Depends() 패턴
- 스키마 분리: RequestSchema, ResponseSchema 별도 정의
- 환경 변수: pydantic BaseSettings 사용

## 디렉토리 구조
- app/api/ - 라우터
- app/core/ - 설정, 보안
- app/models/ - SQLAlchemy 모델
- app/schemas/ - Pydantic 스키마
- app/services/ - 비즈니스 로직
- app/repositories/ - DB 쿼리
- tests/ - 테스트 파일
```

### settings.json 권한 설정 (백엔드)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run dev)",
      "Bash(npm run test)",
      "Bash(npm run test:coverage)",
      "Bash(npx prisma migrate dev)",
      "Bash(npx prisma migrate reset)",
      "Bash(npx prisma studio)",
      "Bash(npx prisma generate)",
      "Bash(python -m pytest *)",
      "Bash(alembic upgrade head)",
      "Bash(alembic downgrade *)",
      "Bash(go test ./...)",
      "Bash(go build ./...)",
      "Bash(docker-compose up *)",
      "Bash(redis-cli *)"
    ],
    "deny": [
      "Bash(DROP TABLE *)",
      "Bash(rm -rf /)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/review` | API 엔드포인트 설계 검토 |
| `/test` | 서비스 레이어 단위 테스트 생성 |
| `/fix` | TypeScript/Python 타입 에러 수정 |
| `/explain` | 복잡한 쿼리나 비즈니스 로직 설명 |
| `/optimize` | 쿼리 성능 또는 알고리즘 최적화 |

---

## 3. DevOps/인프라 엔지니어

DevOps 엔지니어는 자동화, 안정성, 반복 가능성을 추구합니다. Claude Code는 복잡한 YAML 파일 작성, Terraform 모듈 설계, 쉘 스크립트 자동화 등에서 실수를 줄이고 작업 속도를 크게 높여줍니다.

### 핵심 워크플로우

#### 워크플로우 1: CI/CD 파이프라인 구성

GitHub Actions 또는 GitLab CI 파이프라인을 프로젝트 요구사항에 맞게 설계하고 최적화합니다.

**단계:**
1. 현재 빌드/테스트/배포 프로세스 파악
2. 병렬 실행 가능한 작업 분리
3. 캐싱 전략 적용 (Docker 레이어, npm 캐시, Gradle 캐시)
4. 환경별 배포 전략 설정 (dev/staging/prod)
5. 알림 및 승인 게이트 설정

#### 워크플로우 2: Docker 이미지 최적화

Dockerfile을 멀티스테이지 빌드로 최적화하고 이미지 크기를 최소화합니다.

**단계:**
1. 현재 Dockerfile 분석
2. 멀티스테이지 빌드 적용
3. .dockerignore 파일 최적화
4. 보안 베스트 프랙티스 적용 (non-root 사용자, 최소 권한)
5. docker-compose 환경 구성

#### 워크플로우 3: IaC(Infrastructure as Code) 작성

Terraform 또는 Pulumi로 클라우드 인프라를 코드로 관리합니다.

**단계:**
1. 타겟 인프라 아키텍처 정의
2. 모듈 구조 설계 (네트워크, 컴퓨팅, 데이터베이스, 보안)
3. 상태 관리 백엔드 설정 (S3, GCS, Terraform Cloud)
4. 변수 및 출력값 정의
5. 플랜 검토 후 적용

#### 워크플로우 4: 모니터링 및 알림 설정

Prometheus, Grafana, PagerDuty 등을 활용한 모니터링 체계를 구축합니다.

**단계:**
1. 핵심 메트릭 정의 (SLI/SLO 기반)
2. Prometheus 수집 규칙 작성
3. Grafana 대시보드 JSON 생성
4. 알림 규칙 정의 (임계값, 경보 채널)
5. 인시던트 런북 작성

#### 워크플로우 5: 로그 분석 자동화

대용량 로그에서 패턴을 찾고 자동화된 분석 스크립트를 작성합니다.

**단계:**
1. 로그 형식 파악 (JSON, 텍스트, 구조화된 포맷)
2. 관심 패턴 정의 (에러율, 레이턴시 분포, 이상 패턴)
3. 분석 스크립트 또는 쿼리 작성 (jq, awk, Elasticsearch 쿼리)
4. 자동화 스케줄링 (cron, Airflow)
5. 리포트 생성 및 배포

### 실전 프롬프트 예시

```
# GitHub Actions 워크플로우
"Node.js 20 + pnpm 프로젝트용 GitHub Actions 워크플로우를 만들어줘.
PR에서는 lint, test, build를 병렬 실행하고,
main 브랜치 푸시 시 Docker 이미지 빌드 후 ECR에 푸시,
그다음 ECS 서비스를 자동 업데이트해야 해.
캐시 설정도 최적화해줘."

# Dockerfile 최적화
"현재 Dockerfile을 분석하고 멀티스테이지 빌드로 최적화해줘.
최종 이미지는 node:20-alpine 기반으로 non-root 사용자를 사용하고,
보안 취약점을 최소화해야 해. 이미지 크기 목표는 200MB 이하."

# Terraform 모듈
"AWS ECS Fargate 클러스터를 위한 Terraform 모듈을 만들어줘.
VPC, 서브넷, ALB, ECS 클러스터, 태스크 정의, 서비스를 포함하고
변수로 환경(dev/staging/prod)별 설정을 분리해야 해."

# Kubernetes 매니페스트
"이 Node.js 앱을 위한 Kubernetes Deployment, Service, Ingress,
HorizontalPodAutoscaler 매니페스트를 작성해줘.
리소스 제한, 리브니스/레디니스 프로브,
롤링 업데이트 전략도 포함해야 해."

# 쉘 스크립트 자동화
"매일 새벽 2시에 PostgreSQL 데이터베이스를 S3에 백업하는
bash 스크립트를 만들어줘. 30일 이상 된 백업 자동 삭제,
백업 성공/실패 시 Slack 알림, 로그 파일 저장을 포함해야 해."

# 로그 분석
"Nginx 액세스 로그에서 지난 1시간 동안 5xx 에러 비율,
평균 응답 시간, 상위 10개 느린 엔드포인트를 추출하는
jq + awk 스크립트를 작성해줘."
```

### CLAUDE.md 템플릿 (DevOps/인프라 프로젝트용)

```markdown
# 인프라 프로젝트 설정

## 환경 정보
- 클라우드: AWS (us-east-1 기본, ap-northeast-2 보조)
- 컨테이너 오케스트레이션: EKS 1.29
- IaC: Terraform 1.7+ (OpenTofu 호환)
- CI/CD: GitHub Actions
- 컨테이너 레지스트리: ECR
- 모니터링: Prometheus + Grafana + PagerDuty
- 로그: CloudWatch + Elasticsearch

## Terraform 규칙
- 모든 리소스에 tags 블록 필수 (Environment, Project, Owner, ManagedBy)
- 모듈은 modules/ 디렉토리에 도메인별로 분리
- 상태 파일: S3 + DynamoDB 잠금
- terraform.tfvars는 gitignore (환경별 변수는 CI에서 주입)
- plan 결과 반드시 리뷰 후 apply

## Docker 규칙
- 베이스 이미지: 특정 버전 태그 고정 (latest 금지)
- 멀티스테이지 빌드 필수
- non-root 사용자 실행 필수
- HEALTHCHECK 지시문 포함

## Kubernetes 규칙
- 네임스페이스로 환경 분리
- 리소스 requests/limits 항상 명시
- PodDisruptionBudget 설정
- NetworkPolicy로 트래픽 제한

## 보안 정책
- IAM 최소 권한 원칙
- Secrets는 AWS Secrets Manager 또는 Kubernetes Secrets (SOPS 암호화)
- 컨테이너 이미지 취약점 스캔 (Trivy) CI 필수
- 보안 그룹: 0.0.0.0/0 인바운드 금지 (특별 승인 시 제외)

## 배포 전략
- dev: 자동 배포 (main 브랜치)
- staging: 자동 배포 + 스모크 테스트
- prod: 수동 승인 + 카나리 배포 (10% → 50% → 100%)
```

### settings.json 권한 설정 (DevOps)

```json
{
  "permissions": {
    "allow": [
      "Bash(terraform init)",
      "Bash(terraform plan *)",
      "Bash(terraform validate)",
      "Bash(terraform fmt *)",
      "Bash(docker build *)",
      "Bash(docker-compose *)",
      "Bash(kubectl get *)",
      "Bash(kubectl describe *)",
      "Bash(kubectl logs *)",
      "Bash(helm lint *)",
      "Bash(helm template *)",
      "Bash(aws * --dry-run)",
      "Bash(ansible-playbook --check *)"
    ],
    "deny": [
      "Bash(terraform apply)",
      "Bash(terraform destroy)",
      "Bash(kubectl delete *)",
      "Bash(aws ec2 terminate-instances *)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/review` | Terraform 플랜 또는 YAML 설정 검토 |
| `/explain` | 복잡한 인프라 구성 설명 |
| `/fix` | YAML 구문 오류 또는 Terraform 에러 수정 |
| `/test` | Terraform 모듈 테스트(Terratest) 생성 |
| `/optimize` | 비용 또는 성능 최적화 제안 |

---

## 4. 주니어 개발자/학습자

Claude Code는 주니어 개발자에게 경험 많은 시니어가 항상 옆에 있는 것과 같은 환경을 제공합니다. 코드를 이해하고, 디버깅하고, 더 나은 방식을 배우는 과정에서 Claude Code를 적극적으로 활용하세요. 단순히 코드를 받아쓰는 것보다 "왜 이렇게 작성하나요?"를 함께 묻는 습관이 중요합니다.

### 핵심 워크플로우

#### 워크플로우 1: 코드 이해와 설명 요청

처음 보는 코드나 이해하기 어려운 개념을 만났을 때 Claude Code에게 단계별로 설명을 요청합니다.

**단계:**
1. 이해하지 못한 코드 블록 선택
2. 전체 맥락(파일 경로, 관련 함수)과 함께 설명 요청
3. 더 쉬운 비유나 예시 추가 요청
4. 관련 문서나 학습 자료 추천 요청

#### 워크플로우 2: 코드 리뷰 및 개선점 파악

작성한 코드에 대한 피드백을 받고 더 나은 방식을 배웁니다.

**단계:**
1. 작성한 코드를 공유하고 리뷰 요청
2. 개선점과 그 이유 파악
3. 개선된 버전과 원본 비교
4. 새로 배운 패턴을 직접 적용해보기

#### 워크플로우 3: 에러 메시지 디버깅

에러 메시지를 이해하고 스스로 해결하는 능력을 키웁니다.

**단계:**
1. 에러 메시지와 스택 트레이스 전체 복사
2. 발생 상황과 시도한 해결 방법 설명
3. 에러 원인 이해 후 수정
4. 비슷한 에러를 피하는 방법 학습

#### 워크플로우 4: 단계별 기능 구현

큰 기능을 작은 단계로 나누어 하나씩 구현하며 학습합니다.

**단계:**
1. 구현할 기능을 가능한 작게 분리
2. 각 단계별로 무엇을 구현할지 먼저 설명 요청
3. 직접 작성 시도 후 피드백 받기
4. 테스트 코드 함께 작성하여 동작 확인

#### 워크플로우 5: 알고리즘 및 자료구조 학습

이론과 실제 코드를 연결하여 컴퓨터 과학 개념을 깊이 이해합니다.

**단계:**
1. 개념 설명과 실제 사용 사례 요청
2. 단순한 예시 구현 코드 요청
3. 시간/공간 복잡도 분석 학습
4. 실제 프로젝트에 적용 연습

### 실전 프롬프트 예시

```
# 코드 설명 요청
"아래 코드에서 reduce() 함수가 어떻게 동작하는지
초등학생도 이해할 수 있게 단계별로 설명해줘.
어떤 값이 언제 변하는지 각 반복 단계를 보여줘.
[코드 붙여넣기]"

# 내 코드 리뷰
"로그인 기능을 구현했는데 코드 리뷰를 해줘.
초보 개발자로서 어떤 부분이 문제이고,
더 나은 방식은 무엇인지 친절하게 설명해줘.
왜 그렇게 해야 하는지 이유도 알려줘.
[코드 붙여넣기]"

# 에러 디버깅
"아래 에러가 발생해서 1시간째 해결 못하고 있어.
에러 메시지가 무슨 뜻인지, 왜 발생하는지,
어떻게 수정해야 하는지 설명해줘.
TypeError: Cannot read properties of undefined (reading 'map')
[스택 트레이스 붙여넣기]"

# 개념 학습
"JavaScript의 Promise와 async/await를
실제 예시 코드와 함께 설명해줘.
언제 어떤 것을 사용해야 하는지,
실수하기 쉬운 패턴도 알려줘."

# 단계별 구현 가이드
"To-do 앱의 '할 일 삭제' 기능을 구현하고 싶어.
React와 useState를 사용하는데,
어떤 순서로 구현하면 좋을지 단계별로 알려줘.
각 단계에서 내가 직접 해볼 수 있도록 힌트만 주고,
막히면 코드를 보여줘."

# 알고리즘 이해
"이진 탐색 알고리즘을 배우고 있어.
1. 개념 설명 (실생활 비유 포함)
2. JavaScript로 구현한 코드
3. 코드 한 줄씩 설명
4. 선형 탐색과 비교해서 언제 유리한지
순서로 알려줘."

# 베스트 프랙티스 학습
"내가 작성한 아래 함수가 동작하긴 하는데
시니어 개발자라면 어떻게 다르게 작성할지 궁금해.
차이점과 그 이유를 배우고 싶어.
[코드 붙여넣기]"
```

### 단계별 학습 전략

#### 읽기 단계 (코드 파악)
```
"이 파일이 전체 프로젝트에서 어떤 역할을 하는지 설명해줘.
어떤 함수들이 있고, 각각 무슨 일을 하는지 개요를 알려줘."
```

#### 이해 단계 (동작 원리 파악)
```
"이 함수가 실제로 어떻게 실행되는지
입력값 A를 넣으면 어떤 과정을 거쳐 출력값 B가 나오는지
순서대로 설명해줘."
```

#### 수정 단계 (기존 코드 변경)
```
"이 기능에 로딩 상태 표시를 추가하고 싶어.
어디를 어떻게 수정해야 하는지, 왜 그 위치를 선택했는지 알려줘."
```

#### 작성 단계 (새 기능 구현)
```
"설명을 들었으니 직접 비슷한 기능을 구현해볼게.
코드를 바로 보여주기 전에 어떤 접근 방식이 있는지 먼저 알려줘."
```

### CLAUDE.md 템플릿 (학습자용)

```markdown
# 학습 환경 설정

## 나의 수준
- 경력: 입문 (6개월 미만) / 주니어 (1-2년)
- 주력 언어: JavaScript, Python
- 학습 중인 기술: React, Node.js, SQL

## 선호하는 설명 방식
- 기술 용어를 사용할 때는 반드시 쉬운 설명 추가
- 코드 변경 시 왜 그렇게 했는지 이유 설명
- 실생활 비유를 통한 개념 설명 선호
- 한 번에 너무 많은 개념 제시 자제

## 학습 목표
- 이해하지 못한 채 코드를 복사하지 않기
- 각 기능 구현 후 반드시 테스트 작성
- 에러를 만나면 직접 해결 시도 후 도움 요청

## 프로젝트 정보
- 사용 언어: JavaScript (ES2022)
- 프레임워크: React 18
- 패키지 매니저: npm
- 테스트: Jest + Testing Library

## 요청 사항
- 코드를 바로 주기보다 단계별 힌트 제공 선호
- 완성된 코드에는 주석으로 설명 포함
- 더 배울 수 있는 관련 주제나 문서 링크 제안
```

### settings.json 권한 설정 (학습자)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm start)",
      "Bash(npm test)",
      "Bash(npm install *)",
      "Bash(node *)",
      "Bash(python *)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git reset --hard *)",
      "Bash(git push --force)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/explain` | 선택한 코드 상세 설명 |
| `/review` | 내 코드 피드백 요청 |
| `/fix` | 에러 수정 도움 요청 |
| `/test` | 내 코드 테스트 작성 |
| `/help` | Claude Code 사용법 안내 |

### 주니어 개발자를 위한 팁

1. **"왜?"를 항상 물어보세요**: 코드를 받았을 때 "왜 이렇게 작성했나요?"를 함께 물으면 더 많이 배울 수 있습니다.

2. **에러를 두려워하지 마세요**: 에러 메시지를 그대로 붙여넣고 "이게 무슨 뜻이에요?"라고 물어보는 것이 성장의 시작입니다.

3. **작게 시작하세요**: 큰 기능을 한 번에 구현하려 하지 말고, 가장 작은 동작 단위부터 시작하세요.

4. **직접 해보세요**: 힌트를 받았으면 직접 코드를 작성해보고, 막히면 다시 물어보는 방식이 효과적입니다.

5. **코드를 읽는 능력을 키우세요**: "이 파일에서 가장 중요한 로직을 찾아줘"처럼 탐색 요청을 자주 연습하세요.

---

## 5. 풀스택 개발자

풀스택 개발자는 프론트엔드와 백엔드를 모두 다루기 때문에 두 영역 사이의 계약(API 인터페이스, 타입 공유)을 일관되게 관리하는 것이 중요합니다. Claude Code는 양쪽 코드를 동시에 이해하고 수정할 수 있어 풀스택 작업에 최적화되어 있습니다.

### 핵심 워크플로우

#### 워크플로우 1: API 계약 기반 개발 (Contract-First)

타입을 공유 레이어에 먼저 정의하고, 프론트엔드와 백엔드가 이를 기반으로 구현합니다.

**단계:**
1. `packages/shared/types/` 또는 `shared/` 디렉토리에 공통 타입 정의
2. OpenAPI 스펙 또는 tRPC 라우터를 계약으로 사용
3. 백엔드 구현 및 타입 일치 확인
4. 프론트엔드 API 클라이언트 자동 생성 또는 타입 import
5. E2E 테스트로 계약 검증

#### 워크플로우 2: 프론트/백 병렬 개발

독립적인 기능을 동시에 개발하여 병목을 제거합니다.

**단계:**
1. 기능을 프론트엔드 독립 작업과 백엔드 독립 작업으로 분리
2. API 목(Mock)으로 프론트엔드 선행 개발
3. 백엔드 API 구현 완료 후 목 교체
4. 통합 테스트로 최종 검증

#### 워크플로우 3: 모노레포 구조 관리

Turborepo, Nx, pnpm workspace 기반의 모노레포에서 패키지 간 의존성을 관리합니다.

**단계:**
1. 공유 패키지 식별 (ui, utils, types, config)
2. 패키지 간 의존성 그래프 파악
3. 빌드 순서 및 캐시 전략 최적화
4. 공유 ESLint/TypeScript 설정 관리

#### 워크플로우 4: 풀스택 기능 엔드-투-엔드 구현

데이터베이스에서 UI까지 하나의 기능을 완전히 구현합니다.

**단계:**
1. DB 스키마 설계 및 마이그레이션
2. 백엔드 API 엔드포인트 구현
3. 프론트엔드 UI 컴포넌트 및 상태 관리
4. API 연동 및 에러 처리
5. E2E 테스트 작성

#### 워크플로우 5: 성능 모니터링 및 디버깅

클라이언트와 서버를 모두 포함한 전체 요청 흐름을 분석합니다.

**단계:**
1. 브라우저 Performance 탭과 서버 메트릭 동시 분석
2. 네트워크 워터폴 분석
3. 서버 응답 시간 및 클라이언트 렌더링 시간 분리
4. 병목 지점 특정 후 최적화

### 실전 프롬프트 예시

```
# 풀스택 기능 구현
"사용자가 게시글을 작성하면 실시간으로 다른 사용자에게 알림이 가는 기능을 구현해줘.
- 백엔드: packages/api/에 WebSocket 엔드포인트 (Socket.io)
- 프론트엔드: packages/web/에 알림 토스트 컴포넌트
- 공유 타입: packages/shared/에 NotificationEvent 타입 정의
먼저 공유 타입부터 설계하고 순서대로 구현해줘."

# API 타입 동기화
"packages/api/src/routes/users.ts와
packages/web/src/api/users.ts를 분석해서
타입 불일치 문제를 찾아줘. 공유 타입은
packages/shared/types/에 위치해야 해."

# 모노레포 설정
"Turborepo + pnpm workspace 기반 모노레포를 설정해줘.
패키지 구조:
- apps/web (Next.js)
- apps/api (Fastify)
- packages/ui (공유 컴포넌트)
- packages/shared (공유 타입)
빌드 파이프라인과 개발 환경 설정을 포함해줘."

# E2E 테스트
"Playwright로 회원가입 → 로그인 → 게시글 작성 → 로그아웃
플로우 전체를 테스트하는 E2E 테스트를 작성해줘.
테스트 환경에서 백엔드 API를 그대로 사용해야 해."

# tRPC 설정
"apps/api의 tRPC 라우터와 apps/web의 tRPC 클라이언트를 연동해줘.
기존 Express 미들웨어와 함께 동작하고,
React Query 통합도 포함해야 해."
```

### CLAUDE.md 템플릿 (풀스택 모노레포용)

```markdown
# 풀스택 모노레포 설정

## 모노레포 구조
- 패키지 매니저: pnpm 8.x
- 모노레포 도구: Turborepo 2.x
- 루트 설정: turbo.json, pnpm-workspace.yaml

## 패키지 구조
- apps/web/ - Next.js 14 프론트엔드
- apps/api/ - Fastify 4 백엔드
- packages/ui/ - 공유 React 컴포넌트 (shadcn/ui 기반)
- packages/shared/ - 공유 타입, 유틸리티, 상수
- packages/eslint-config/ - 공유 ESLint 설정
- packages/tsconfig/ - 공유 TypeScript 설정

## API 계약 관리
- 공유 타입은 packages/shared/types/에 위치
- API 응답 타입: ApiResponse<T> 제네릭 사용
- tRPC로 엔드-투-엔드 타입 안전성 보장
- OpenAPI 스펙은 apps/api/openapi.yaml

## 프론트엔드 (apps/web)
- Next.js 14 App Router
- Tailwind CSS + shadcn/ui
- TanStack Query v5 (서버 상태)
- Zustand (클라이언트 상태)
- Playwright (E2E 테스트)

## 백엔드 (apps/api)
- Fastify 4 + TypeScript
- Prisma ORM (PostgreSQL)
- Redis (캐시, 세션, 큐)
- tRPC v11
- Jest + Supertest (단위/통합 테스트)

## 개발 명령어
- 전체 개발 서버: pnpm dev (Turborepo 병렬 실행)
- 전체 빌드: pnpm build
- 전체 테스트: pnpm test
- 특정 앱만: pnpm --filter @myapp/web dev

## 코딩 규칙
- 패키지 간 import는 패키지명 사용 (@myapp/shared)
- 상대 경로 import는 같은 패키지 내에서만 허용
- 공유 로직은 반드시 packages/shared로 이동
- 각 앱의 business 로직은 해당 앱 패키지 내에 유지
```

### settings.json 권한 설정 (풀스택/모노레포)

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm *)",
      "Bash(npx turbo *)",
      "Bash(npx prisma *)",
      "Bash(npm run *)",
      "Bash(npx playwright *)",
      "Bash(docker-compose up *)",
      "Bash(docker-compose down)",
      "Bash(git *)"
    ],
    "deny": [
      "Bash(rm -rf node_modules)",
      "Bash(git push --force)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/review` | API 계약 및 타입 일관성 검토 |
| `/test` | E2E 또는 통합 테스트 생성 |
| `/fix` | 타입 불일치 또는 빌드 에러 수정 |
| `/explain` | 모노레포 의존성 관계 설명 |
| `/optimize` | 번들 크기 또는 빌드 시간 최적화 |

---

## 6. 모바일 개발자 (React Native/Flutter)

모바일 개발은 플랫폼별 차이, 네이티브 모듈, 빌드 환경 설정 등 복잡한 요소가 많습니다. Claude Code는 크로스 플랫폼 코드 관리, 플랫폼별 분기 처리, 빌드 에러 해결에서 특히 유용합니다.

### 핵심 워크플로우

#### 워크플로우 1: 크로스 플랫폼 컴포넌트 관리

iOS와 Android에서 공통으로 동작하는 컴포넌트를 설계하면서 플랫폼별 UI 차이를 처리합니다.

**단계:**
1. 공통 로직과 플랫폼별 UI 분리 설계
2. `Platform.OS` 또는 `.ios.tsx`/`.android.tsx` 파일 분기 전략 결정
3. 플랫폼별 스타일 차이 처리 (SafeAreaView, 상태바 등)
4. Expo/React Native CLI 기반 실행 및 테스트

#### 워크플로우 2: 네이티브 모듈 디버깅

카메라, 위치, 푸시 알림 등 네이티브 기능 관련 에러를 해결합니다.

**단계:**
1. 에러 발생 플랫폼 및 버전 파악 (iOS/Android, OS 버전)
2. 권한 설정 파일 확인 (Info.plist, AndroidManifest.xml)
3. 네이티브 모듈 링킹 상태 확인
4. Metro 번들러 캐시 초기화 후 재빌드

#### 워크플로우 3: 빌드 에러 해결

Xcode, Android Studio, Gradle 빌드 에러를 분석하고 해결합니다.

**단계:**
1. 에러 로그 전체 공유 (스크롤하지 말고 전체 로그)
2. 최근 변경 사항 확인 (라이브러리 업데이트, 네이티브 모듈 추가)
3. 의존성 충돌 분석
4. 플랫폼별 설정 파일 점검 (Podfile, build.gradle)
5. 클린 빌드 후 재시도

#### 워크플로우 4: 성능 최적화

리스트 렌더링, 이미지 로딩, 애니메이션 성능을 개선합니다.

**단계:**
1. 느린 화면 특정 (Flipper, React Native Debugger)
2. FlatList 최적화 (keyExtractor, getItemLayout, windowSize)
3. 이미지 캐싱 및 크기 최적화
4. Reanimated 2/3로 JS 스레드 부하 감소
5. Hermes 엔진 활성화 여부 확인

#### 워크플로우 5: 앱스토어 제출 준비

앱 서명, 메타데이터, 스크린샷, 릴리스 빌드를 자동화합니다.

**단계:**
1. 앱 버전 및 빌드 번호 업데이트 자동화
2. 릴리스 빌드 스크립트 작성
3. Fastlane 또는 EAS Build 설정
4. 환경별 설정 분리 (개발/스테이징/프로덕션)
5. 코드 서명 설정 검토

### 실전 프롬프트 예시

```
# 컴포넌트 생성 (React Native)
"React Native에서 iOS와 Android 모두에서 잘 동작하는
커스텀 날짜 피커 컴포넌트를 만들어줘.
iOS는 네이티브 DateTimePicker, Android는 커스텀 모달을 사용하고
TypeScript 타입과 사용 예시도 포함해줘."

# 네이티브 모듈 에러
"iOS 시뮬레이터에서 react-native-camera를 사용하면
아래 에러가 발생해. Info.plist 설정과 Pod 설정을 확인하고
해결 방법을 알려줘.
[에러 메시지 붙여넣기]"

# 빌드 에러 해결
"Android 빌드가 아래 Gradle 에러로 실패해.
android/build.gradle과 android/app/build.gradle을 확인하고
수정해줘.
[Gradle 에러 로그 붙여넣기]"

# FlatList 최적화
"현재 FlatList가 1000개 이상의 아이템에서 느려.
현재 구현 코드를 분석하고 성능 최적화 방법을 적용해줘.
getItemLayout, memo, keyExtractor 최적화를 포함해야 해."

# Fastlane 설정
"iOS App Store와 Google Play에 자동 배포하는
Fastlane 설정 파일을 만들어줘.
베타(TestFlight/Firebase App Distribution)와
프로덕션 배포 lane을 분리하고,
버전 번호 자동 증가도 포함해야 해."

# Flutter 위젯 생성
"Flutter로 iOS의 Cupertino 디자인 스타일에 맞는
커스텀 액션 시트 위젯을 만들어줘.
애니메이션, 다크 모드 지원, 접근성(Semantics) 포함해야 해."

# 상태 관리 (React Native)
"현재 Context API로 관리하는 인증 상태를 Zustand로 마이그레이션해줘.
AsyncStorage와 연동해서 앱 재시작 후에도 로그인 상태가 유지되어야 해."
```

### CLAUDE.md 템플릿 (React Native 프로젝트용)

```markdown
# React Native 프로젝트 설정

## 환경 정보
- React Native: 0.73.x (New Architecture 활성화)
- Expo: Managed Workflow / Bare Workflow (해당사항 선택)
- JavaScript 엔진: Hermes
- 언어: TypeScript 5.x
- 패키지 매니저: yarn

## 주요 라이브러리
- 네비게이션: React Navigation 6 (Native Stack)
- 상태관리: Zustand + React Query
- 애니메이션: React Native Reanimated 3
- 스타일링: StyleSheet API + Shopify Restyle
- 폼: React Hook Form
- 스토리지: MMKV (AsyncStorage 대체)
- 이미지: react-native-fast-image
- 아이콘: react-native-vector-icons (Ionicons)
- 테스트: Jest + React Native Testing Library

## 지원 플랫폼
- iOS: 15.0 이상
- Android: API 29 (Android 10) 이상

## 디렉토리 구조
- src/screens/ - 화면 컴포넌트
- src/components/ - 재사용 UI 컴포넌트
- src/navigation/ - 네비게이션 설정
- src/hooks/ - 커스텀 훅
- src/store/ - Zustand 스토어
- src/api/ - API 클라이언트
- src/utils/ - 유틸리티
- src/types/ - TypeScript 타입
- android/ - Android 네이티브 코드
- ios/ - iOS 네이티브 코드 (CocoaPods)

## 코딩 규칙
- Platform.OS 분기는 플랫폼별 파일(.ios.tsx, .android.tsx) 분리 선호
- 스타일은 StyleSheet.create() 또는 Restyle 테마 사용 (인라인 스타일 금지)
- 리스트는 항상 FlatList 또는 FlashList 사용 (ScrollView 내 map 금지)
- 이미지는 항상 크기 지정 + FastImage 사용

## 빌드 명령어
- iOS: yarn ios (시뮬레이터)
- Android: yarn android (에뮬레이터)
- 릴리스 빌드: yarn build:ios / yarn build:android
- 클린 빌드: yarn clean && pod install (iOS)
```

### CLAUDE.md 템플릿 (Flutter 프로젝트용)

```markdown
# Flutter 프로젝트 설정

## 환경 정보
- Flutter: 3.19.x
- Dart: 3.3.x
- 대상 플랫폼: iOS 15+, Android API 29+

## 아키텍처
- 상태관리: Riverpod 2.x (AsyncNotifier 패턴)
- 라우팅: go_router
- 의존성 주입: Riverpod Provider
- 네트워크: Dio + Retrofit
- 직렬화: json_serializable + Freezed

## 디렉토리 구조 (Feature-First)
- lib/features/{feature}/
  - data/ (레포지토리 구현, 데이터소스)
  - domain/ (엔티티, 유스케이스, 레포지토리 인터페이스)
  - presentation/ (위젯, 프로바이더, 상태)
- lib/core/ (공통 유틸, 테마, 라우터)
- lib/shared/ (공유 위젯, 상수)

## 코딩 규칙
- 모든 공개 API에 Dart Doc 주석
- const 생성자 가능하면 항상 사용
- Freezed로 불변 데이터 클래스 생성
- 위젯 분리: 200줄 초과 시 별도 파일로 분리
```

### settings.json 권한 설정 (모바일 개발)

```json
{
  "permissions": {
    "allow": [
      "Bash(yarn *)",
      "Bash(npm run *)",
      "Bash(npx react-native *)",
      "Bash(pod install)",
      "Bash(pod update *)",
      "Bash(flutter *)",
      "Bash(adb *)",
      "Bash(xcrun *)",
      "Bash(fastlane *)",
      "Bash(eas build *)",
      "Bash(eas submit *)"
    ],
    "deny": [
      "Bash(rm -rf ios/Pods)",
      "Bash(rm -rf android/.gradle)"
    ]
  }
}
```

### 자주 쓰는 슬래시 명령어

| 명령어 | 용도 |
|--------|------|
| `/review` | 컴포넌트 성능 및 접근성 검토 |
| `/fix` | 빌드 에러 또는 린트 에러 수정 |
| `/explain` | 네이티브 모듈 동작 원리 설명 |
| `/test` | UI 컴포넌트 테스트 생성 |
| `/optimize` | FlatList 또는 애니메이션 성능 최적화 |

---

## 공통 팁: Claude Code를 더 잘 활용하는 방법

### 컨텍스트를 충분히 제공하세요

Claude Code는 프로젝트 파일을 읽을 수 있지만, 처음에는 어떤 파일이 중요한지 알지 못합니다. 관련 파일 경로를 명시적으로 언급하고, 어떤 기술 스택을 사용하는지 알려주면 훨씬 정확한 답변을 받을 수 있습니다.

```
# 컨텍스트 부족한 요청 (나쁜 예)
"로그인 안 돼"

# 컨텍스트 충분한 요청 (좋은 예)
"src/app/api/auth/login/route.ts에서 로그인 API가 있는데,
Next.js 14 App Router + Prisma + JWT 환경에서
POST 요청 시 500 에러가 발생해. 에러 로그: [로그 붙여넣기]"
```

### CLAUDE.md를 항상 최신으로 유지하세요

프로젝트의 기술 스택, 코딩 규칙, 디렉토리 구조가 변경될 때마다 CLAUDE.md를 업데이트하세요. 이것이 Claude Code가 프로젝트를 이해하는 가장 중요한 문서입니다.

### 반복 작업은 명령어화하세요

자주 반복하는 요청이 있다면 슬래시 명령어로 만들거나, CLAUDE.md에 "이 유형의 요청에는 항상 이렇게 해줘"라고 명시하세요.

### 코드 변경 전 항상 확인하세요

Claude Code가 코드를 수정하기 전에 "무엇을 어떻게 변경할 계획인지 먼저 알려줘"라고 요청하는 습관을 들이면 의도치 않은 변경을 방지할 수 있습니다.

---

*이 가이드는 Claude Code의 기능과 개발자 경험을 바탕으로 작성되었습니다.*
*각 섹션의 CLAUDE.md 템플릿은 프로젝트 상황에 맞게 수정하여 사용하세요.*
