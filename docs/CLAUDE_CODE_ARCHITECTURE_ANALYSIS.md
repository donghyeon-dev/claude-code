# Claude Code 소스코드 아키텍처 분석

> 분석일: 2026-03-31 | 소스 위치: `/Users/parkdonghyeon/Dev/src/`

---

## 목차
1. [전체 아키텍처 개요](#1-전체-아키텍처-개요)
2. [핵심 모듈 구조](#2-핵심-모듈-구조)
3. [데이터 흐름](#3-데이터-흐름)
4. [도구(Tools) 시스템](#4-도구tools-시스템)
5. [명령어(Commands) 시스템](#5-명령어commands-시스템)
6. [시스템 프롬프트 아키텍처](#6-시스템-프롬프트-아키텍처)
7. [Permission 시스템](#7-permission-시스템)
8. [MCP (Model Context Protocol) 통합](#8-mcp-통합)
9. [서비스 레이어](#9-서비스-레이어)
10. [풀스택 개발자를 위한 인사이트](#10-풀스택-개발자를-위한-인사이트)

---

## 1. 전체 아키텍처 개요

### 기술 스택
| 구성 요소 | 기술 |
|-----------|------|
| **런타임** | Bun (빌드 시 `bun:bundle` feature flag 사용) |
| **언어** | TypeScript (strict) |
| **UI 프레임워크** | React + Ink (터미널 렌더링) |
| **상태관리** | 커스텀 Store (React Context + External Store 패턴) |
| **CLI 파싱** | Commander.js (`@commander-js/extra-typings`) |
| **스키마 검증** | Zod v4 |
| **API 통신** | Anthropic SDK (`@anthropic-ai/sdk`) |
| **MCP 프로토콜** | `@modelcontextprotocol/sdk` |
| **스타일링** | Chalk (터미널 컬러) |

### 설계 원칙
- **Feature Flag 기반 Dead Code Elimination**: `bun:bundle`의 `feature()` 함수로 빌드 타임에 불필요 코드 제거
- **Lazy Loading**: 순환 의존성 방지 및 시작 성능을 위해 `require()` 지연 로딩 광범위 사용
- **Memoization**: `lodash-es/memoize`로 비용이 큰 연산(설정 로딩, 명령어 조립) 캐싱
- **Fail-Closed Security**: Permission 시스템이 기본적으로 거부하고, 명시적 허용만 통과
- **Internal(ant) vs External 빌드 분리**: `process.env.USER_TYPE === 'ant'`로 내부 전용 기능 분리

### 프로젝트 규모
- **main.tsx**: 4,683줄 (CLI 진입점 + Commander 설정 + 모든 모드 분기)
- **Tool.ts**: 793줄 (도구 타입 시스템 정의)
- **query.ts**: ~68KB (LLM 쿼리 루프)
- **QueryEngine.ts**: ~46KB (쿼리 엔진 + 세션 관리)
- **utils/**: 329개 파일 (유틸리티 모듈)
- **components/**: 146개 항목 (Ink UI 컴포넌트)
- **tools/**: 43개 도구 디렉토리
- **commands/**: 103개 명령어 디렉토리
- **hooks/**: 87개 React 훅

---

## 2. 핵심 모듈 구조

```
src/
├── entrypoints/          # 진입점
│   ├── cli.tsx           # CLI 부트스트랩 (--version 패스트패스, 모드 분기)
│   ├── init.ts           # 초기화 (설정, 텔레메트리, OAuth, CA인증서)
│   ├── mcp.ts            # MCP 서버 모드 진입점
│   └── sdk/              # Agent SDK 진입점
│
├── main.tsx              # 메인 CLI 로직 (Commander 설정, 모든 모드 통합)
├── setup.ts              # 세션 설정 (Git, 워크트리, 권한, 훅 스냅샷)
├── context.ts            # 시스템/사용자 컨텍스트 (Git상태, CLAUDE.md)
│
├── Tool.ts               # 도구 타입 시스템 (Tool<Input,Output,Progress>)
├── tools.ts              # 도구 레지스트리 (getAllBaseTools, getTools, assembleToolPool)
├── tools/                # 43개 개별 도구 구현
│
├── commands.ts           # 명령어 레지스트리 (getCommands, 스킬 통합)
├── commands/             # 103개 슬래시 명령어 구현
│
├── query.ts              # LLM 쿼리 루프 (메시지 정규화, 도구 실행, 컴팩트)
├── QueryEngine.ts        # 쿼리 엔진 (세션 관리, 이력, 비용 추적)
│
├── state/                # 앱 상태 관리
│   ├── AppState.tsx      # React Context 기반 상태 프로바이더
│   ├── AppStateStore.ts  # 상태 타입 정의 + 기본값
│   └── store.ts          # External Store 구현
│
├── components/           # Ink UI 컴포넌트
│   ├── App.tsx           # 최상위 앱 컴포넌트
│   ├── Messages.tsx      # 메시지 목록 렌더링
│   ├── PromptInput/      # 사용자 입력 UI
│   ├── diff/             # 파일 변경 표시
│   ├── design-system/    # 디자인 시스템 (ThemeProvider, Box, Text)
│   ├── permissions/      # 권한 승인 UI
│   └── mcp/              # MCP 관련 UI
│
├── hooks/                # React 훅 (87개)
│   ├── useCanUseTool.tsx # 도구 사용 권한 확인
│   ├── useMergedTools.ts # 빌트인 + MCP 도구 병합
│   ├── useDynamicConfig.ts
│   └── ...
│
├── services/             # 서비스 레이어
│   ├── api/              # Claude API 통신
│   ├── mcp/              # MCP 클라이언트/서버
│   ├── lsp/              # Language Server Protocol
│   ├── analytics/        # 텔레메트리/분석
│   ├── compact/          # 컨텍스트 압축
│   ├── oauth/            # OAuth 인증
│   ├── plugins/          # 플러그인 관리
│   └── tools/            # 도구 실행 오케스트레이션
│
├── constants/            # 상수 정의
│   ├── prompts.ts        # 시스템 프롬프트 생성 (getSystemPrompt)
│   ├── systemPromptSections.ts # 프롬프트 섹션 캐싱 시스템
│   └── tools.ts          # 도구 관련 상수
│
├── skills/               # 스킬 시스템
│   ├── loadSkillsDir.ts  # 마크다운 스킬 로더
│   ├── bundledSkills.ts  # 번들 스킬
│   └── bundled/          # 번들 스킬 구현
│
├── types/                # 타입 정의
│   ├── message.ts        # 메시지 타입
│   ├── command.ts        # 명령어 타입
│   ├── permissions.ts    # 권한 타입
│   ├── hooks.ts          # 훅 타입
│   └── ids.ts            # ID 타입 (AgentId, SessionId)
│
├── utils/                # 유틸리티 (329개 파일)
│   ├── permissions/      # 권한 시스템
│   ├── model/            # 모델 관리
│   ├── hooks/            # 훅 유틸리티
│   ├── plugins/          # 플러그인 유틸리티
│   ├── settings/         # 설정 관리
│   ├── swarm/            # 에이전트 스웜
│   ├── git.ts            # Git 작업
│   ├── claudemd.ts       # CLAUDE.md 파싱
│   ├── systemPrompt.ts   # 시스템 프롬프트 빌드
│   └── ...
│
├── bridge/               # IDE 브리지 통신
├── ink/                  # Ink 커스텀 레이어 (렌더러, 이벤트)
├── memdir/               # 메모리 디렉토리 (자동 메모리)
├── coordinator/          # 코디네이터 모드 (에이전트 오케스트레이션)
├── vim/                  # Vim 모드 지원
├── voice/                # 음성 모드 지원
└── remote/               # 원격 세션 지원
```

---

## 3. 데이터 흐름

### 부팅 시퀀스
```
entrypoints/cli.tsx
  ├─ 패스트패스: --version → 즉시 출력 후 종료
  ├─ entrypoints/init.ts → 설정 로딩, 텔레메트리, OAuth
  ├─ main.tsx → Commander 파싱, 모드 결정
  │   ├─ setup.ts → CWD 설정, Git 상태, 워크트리, 훅 스냅샷
  │   ├─ context.ts → 시스템/사용자 컨텍스트 수집
  │   │   ├─ getSystemContext() → Git 상태 스냅샷
  │   │   └─ getUserContext() → CLAUDE.md, 현재 날짜
  │   └─ replLauncher.tsx → Ink REPL 시작
  └─ (비대화형) QueryEngine → query() 직접 호출
```

### 쿼리 루프 (사용자 메시지 → 응답)
```
사용자 입력
  │
  ├─ processUserInput() → 슬래시 명령어 파싱, 첨부파일 처리
  │
  ├─ QueryEngine.runMainLoop()
  │   ├─ fetchSystemPromptParts() → 시스템 프롬프트 조립
  │   ├─ normalizeMessagesForAPI() → 메시지 정규화
  │   ├─ query() → Claude API 호출
  │   │   ├─ API 스트리밍 응답 수신
  │   │   ├─ StreamingToolExecutor → 병렬 도구 실행
  │   │   │   ├─ validateInput() → 입력 검증
  │   │   │   ├─ checkPermissions() → 권한 확인
  │   │   │   ├─ canUseTool() → 훅/분류기 검사
  │   │   │   └─ tool.call() → 도구 실행
  │   │   └─ runTools() → 도구 결과 수집
  │   ├─ autoCompact → 토큰 임계치 초과 시 자동 압축
  │   └─ 루프 반복 (도구 호출이 있으면 계속)
  │
  └─ UI 업데이트 (Ink 렌더링)
```

### Task (서브에이전트) 시스템
```
AgentTool.call()
  ├─ generateTaskId('local_agent') → 고유 ID 생성
  ├─ createTaskStateBase() → 태스크 상태 초기화
  ├─ AppState.tasks에 등록
  ├─ 별도 프로세스/컨텍스트에서 쿼리 실행
  │   ├─ 독립적 AbortController
  │   ├─ 독립적 메시지 이력
  │   └─ 결과를 outputFile에 기록
  └─ TaskOutputTool/SendMessageTool로 결과 전달
```

---

## 4. 도구(Tools) 시스템

### 도구 타입 정의 (`Tool.ts`)

모든 도구는 `Tool<Input, Output, Progress>` 제네릭 인터페이스를 구현합니다:

```typescript
type Tool<Input, Output, Progress> = {
  name: string
  inputSchema: Input              // Zod 스키마
  call(): Promise<ToolResult>     // 실행 로직
  checkPermissions(): Promise<PermissionResult>  // 권한 검사
  prompt(): Promise<string>       // LLM에 보낼 도구 설명
  description(): Promise<string>  // 동적 설명
  isEnabled(): boolean
  isConcurrencySafe(): boolean    // 병렬 실행 가능 여부
  isReadOnly(): boolean           // 읽기 전용 여부
  isDestructive?(): boolean       // 파괴적 작업 여부
  maxResultSizeChars: number      // 결과 크기 제한
  // + 다양한 렌더링 메서드
}
```

`buildTool()` 헬퍼로 기본값 자동 주입 (fail-closed):
- `isConcurrencySafe` → `false` (안전하지 않다고 가정)
- `isReadOnly` → `false` (쓰기 작업이라고 가정)
- `checkPermissions` → `allow` (일반 권한 시스템에 위임)

### 전체 도구 카탈로그 (43개)

#### 핵심 도구 (항상 활성)
| 도구 | 설명 |
|------|------|
| **AgentTool** | 서브에이전트 생성 및 관리 |
| **BashTool** | 셸 명령어 실행 |
| **FileReadTool** | 파일 읽기 |
| **FileEditTool** | 파일 편집 (문자열 교체) |
| **FileWriteTool** | 파일 생성/덮어쓰기 |
| **GlobTool** | 파일 패턴 검색 |
| **GrepTool** | 파일 내용 검색 (ripgrep) |
| **WebFetchTool** | URL 가져오기 |
| **WebSearchTool** | 웹 검색 |
| **TodoWriteTool** | TODO 리스트 관리 |
| **SkillTool** | 스킬 실행 |
| **SendMessageTool** | 에이전트간 메시지 전달 |
| **AskUserQuestionTool** | 사용자에게 질문 |
| **NotebookEditTool** | Jupyter 노트북 편집 |
| **BriefTool** | 간결한 응답 생성 |

#### 계획/모드 도구
| 도구 | 설명 |
|------|------|
| **EnterPlanModeTool** | 계획 모드 진입 |
| **ExitPlanModeV2Tool** | 계획 모드 종료 |
| **EnterWorktreeTool** | Git 워크트리 진입 |
| **ExitWorktreeTool** | Git 워크트리 종료 |

#### 태스크 관리 도구 (TodoV2 활성 시)
| 도구 | 설명 |
|------|------|
| **TaskCreateTool** | 태스크 생성 |
| **TaskGetTool** | 태스크 조회 |
| **TaskUpdateTool** | 태스크 상태 업데이트 |
| **TaskListTool** | 태스크 목록 조회 |
| **TaskOutputTool** | 태스크 출력 읽기 |
| **TaskStopTool** | 태스크 중지 |

#### MCP 도구
| 도구 | 설명 |
|------|------|
| **ListMcpResourcesTool** | MCP 리소스 목록 |
| **ReadMcpResourceTool** | MCP 리소스 읽기 |
| **MCPTool** | MCP 서버 도구 실행 |
| **McpAuthTool** | MCP 인증 |

#### Feature Flag 기반 조건부 도구
| 도구 | 조건 |
|------|------|
| **REPLTool** | `USER_TYPE === 'ant'` |
| **SleepTool** | `PROACTIVE` or `KAIROS` |
| **CronCreateTool/DeleteTool/ListTool** | `AGENT_TRIGGERS` |
| **RemoteTriggerTool** | `AGENT_TRIGGERS_REMOTE` |
| **MonitorTool** | `MONITOR_TOOL` |
| **PowerShellTool** | Windows 환경 |
| **ToolSearchTool** | 도구 수 임계치 초과 시 |
| **TeamCreateTool/TeamDeleteTool** | 에이전트 스웜 활성 시 |
| **WorkflowTool** | `WORKFLOW_SCRIPTS` |
| **WebBrowserTool** | `WEB_BROWSER_TOOL` |
| **LSPTool** | `ENABLE_LSP_TOOL` 환경변수 |

### 도구 풀 조립 흐름
```
getAllBaseTools()          → 모든 빌트인 도구 (feature flag 기반 필터링)
  ↓
getTools(permCtx)         → deny 규칙 적용 + isEnabled() 필터
  ↓
assembleToolPool(permCtx, mcpTools) → 빌트인 + MCP 도구 병합 (이름 중복 시 빌트인 우선)
```

---

## 5. 명령어(Commands) 시스템

### 명령어 타입
```typescript
type Command = {
  type: 'local' | 'local-jsx' | 'prompt'
  name: string
  aliases?: string[]
  description: string
  source: 'builtin' | 'plugin' | 'bundled' | 'mcp' | SettingSource
  availability?: ('claude-ai' | 'console')[]
  loadedFrom?: 'skills' | 'plugin' | 'bundled' | 'commands_DEPRECATED' | 'mcp'
}
```

- **`local`**: 즉시 실행, 텍스트 출력 (예: `/cost`, `/clear`)
- **`local-jsx`**: JSX UI 렌더링 (예: `/mcp`, `/config`)
- **`prompt`**: LLM에 프롬프트 주입 (예: `/commit`, `/review`, 스킬)

### 주요 슬래시 명령어 (사용자 대면)

#### 세션 관리
| 명령어 | 설명 |
|--------|------|
| `/clear` | 대화 초기화 |
| `/compact` | 컨텍스트 압축 |
| `/cost` | 세션 비용 표시 |
| `/exit` | 종료 |
| `/resume` | 이전 세션 재개 |
| `/session` | 원격 세션 관리 |

#### 개발 도구
| 명령어 | 설명 |
|--------|------|
| `/commit` | Git 커밋 생성 (내부용) |
| `/diff` | 변경사항 표시 |
| `/review` | 코드 리뷰 |
| `/plan` | 계획 모드 전환 |
| `/files` | 추적 파일 목록 |
| `/branch` | 브랜치 관리 |

#### 설정/도구
| 명령어 | 설명 |
|--------|------|
| `/config` | 설정 관리 |
| `/mcp` | MCP 서버 관리 |
| `/permissions` | 권한 설정 |
| `/hooks` | 훅 설정 |
| `/model` | 모델 변경 |
| `/vim` | Vim 모드 전환 |
| `/theme` | 테마 변경 |

#### 고급 기능
| 명령어 | 설명 |
|--------|------|
| `/doctor` | 설치 진단 |
| `/memory` | 메모리 관리 |
| `/skills` | 스킬 관리 |
| `/plugin` | 플러그인 관리 |
| `/agents` | 에이전트 정의 관리 |
| `/tasks` | 백그라운드 태스크 관리 |
| `/thinkback` | 사고 과정 되돌아보기 |
| `/rewind` | 대화 되감기 |
| `/insights` | 세션 분석 리포트 |

### 명령어 소스 우선순위
```
loadAllCommands(cwd):
  1. bundledSkills      (번들 스킬)
  2. builtinPluginSkills (빌트인 플러그인 스킬)
  3. skillDirCommands   (.claude/skills/ 사용자 스킬)
  4. workflowCommands   (워크플로우 스크립트)
  5. pluginCommands     (외부 플러그인 명령어)
  6. pluginSkills       (외부 플러그인 스킬)
  7. COMMANDS()         (빌트인 명령어)
```

---

## 6. 시스템 프롬프트 아키텍처

### 프롬프트 구성 계층
```
constants/prompts.ts::getSystemPrompt()
  ├─ 기본 시스템 프롬프트 섹션들
  │   ├─ systemPromptSection('identity')     → AI 정체성
  │   ├─ systemPromptSection('tools')        → 도구 사용 지침
  │   ├─ systemPromptSection('environment')  → 환경 정보 (OS, 셸, 플랫폼)
  │   ├─ systemPromptSection('git')          → Git 커밋 지침
  │   ├─ systemPromptSection('tasks')        → 태스크 관리
  │   ├─ systemPromptSection('tone')         → 톤과 스타일
  │   ├─ systemPromptSection('security')     → 사이버 보안 지침
  │   └─ systemPromptSection('output')       → 출력 효율성
  │
  └─ resolveSystemPromptSections()
      → 캐싱: 같은 이름의 섹션은 한 번만 계산
      → cacheBreak: true인 섹션만 매 턴 재계산
```

### 시스템 프롬프트 우선순위 (`utils/systemPrompt.ts`)
```
buildEffectiveSystemPrompt():
  1. overrideSystemPrompt     → 존재 시 다른 모든 것 대체
  2. Coordinator 모드 프롬프트  → COORDINATOR_MODE 활성 시
  3. Agent 시스템 프롬프트      → --agent 지정 시
     └─ Proactive 모드: 기본 프롬프트에 추가
     └─ 일반 모드: 기본 프롬프트 대체
  4. Custom 시스템 프롬프트     → --system-prompt 지정 시
  5. 기본 시스템 프롬프트       → 표준 Claude Code 프롬프트
  + appendSystemPrompt        → 항상 끝에 추가
```

### 컨텍스트 주입 (`context.ts`)
```
getSystemContext() → 캐싱됨, 대화 시작 시 1회
  ├─ Git 상태 (브랜치, 최근 커밋, 변경사항)
  └─ 캐시 브레이커 (내부용)

getUserContext() → 캐싱됨, 대화 시작 시 1회
  ├─ CLAUDE.md 내용 (프로젝트~홈 디렉토리 탐색)
  └─ 현재 날짜
```

---

## 7. Permission 시스템

### 권한 모드
```typescript
type PermissionMode =
  | 'default'            // 매번 물어봄
  | 'plan'               // 계획 모드 (읽기만 허용)
  | 'autoEdit'           // 편집 자동 승인
  | 'fullAuto'           // 모두 자동 승인
  | 'bypassPermissions'  // 권한 우회 (샌드박스 전용)
```

### 권한 확인 흐름
```
도구 실행 요청
  ↓
1. Deny Rules 확인 (settings.json의 alwaysDenyRules)
   → 매칭 시 즉시 거부
  ↓
2. tool.validateInput() → 도구별 입력 검증
  ↓
3. tool.checkPermissions() → 도구별 권한 로직
  ↓
4. 일반 Permission 시스템
   ├─ alwaysAllowRules 확인
   ├─ alwaysAskRules 확인
   ├─ Permission Mode 기반 자동 승인/거부
   └─ 훅(PreToolUse) 실행
  ↓
5. canUseTool() → 최종 결정 (사용자 프롬프트 또는 자동)
```

### 도구 권한 규칙 (settings.json)
```json
{
  "permissions": {
    "allow": ["Read", "Glob", "Grep"],
    "deny": ["Bash(rm *)"],
    "ask": ["FileWrite"]
  }
}
```

패턴 매칭: `ToolName(패턴)` 형식으로 특정 입력에 대한 규칙 설정 가능

---

## 8. MCP 통합

### MCP 아키텍처
```
services/mcp/
  ├─ client.ts              # MCP 클라이언트 (도구/리소스 가져오기)
  ├─ MCPConnectionManager.tsx # 연결 관리 (React 컴포넌트)
  ├─ InProcessTransport.ts  # 인프로세스 전송
  ├─ SdkControlTransport.ts # SDK 제어 전송
  ├─ config.ts              # MCP 서버 설정
  ├─ types.ts               # MCP 타입 정의
  ├─ normalization.ts       # 도구 이름 정규화 (mcp__server__tool)
  ├─ auth.ts                # OAuth 인증
  ├─ officialRegistry.ts    # 공식 레지스트리
  └─ elicitationHandler.ts  # URL 인증 흐름
```

### MCP 도구 통합
- MCP 도구는 빌트인 도구와 같은 `Tool` 인터페이스로 정규화
- 이름 형식: `mcp__서버명__도구명`
- 빌트인 도구와 이름 충돌 시 빌트인 우선
- Deny Rules가 MCP 서버 단위 (`mcp__server`)로 전체 차단 가능

---

## 9. 서비스 레이어

### 주요 서비스

| 서비스 | 위치 | 역할 |
|--------|------|------|
| **API** | `services/api/` | Claude API 통신, 재시도, 에러 처리 |
| **Analytics** | `services/analytics/` | GrowthBook, 텔레메트리, 이벤트 로깅 |
| **Compact** | `services/compact/` | 컨텍스트 압축 (자동/수동) |
| **MCP** | `services/mcp/` | MCP 프로토콜 클라이언트 |
| **LSP** | `services/lsp/` | Language Server Protocol 통합 |
| **OAuth** | `services/oauth/` | OAuth 인증 흐름 |
| **Plugins** | `services/plugins/` | 플러그인 로딩/관리 |
| **PolicyLimits** | `services/policyLimits/` | 정책 기반 사용 제한 |
| **RemoteManagedSettings** | `services/remoteManagedSettings/` | 원격 관리 설정 |
| **SessionMemory** | `services/SessionMemory/` | 세션 메모리 관리 |
| **Tools** | `services/tools/` | 도구 실행 오케스트레이션 |
| **TokenEstimation** | `services/tokenEstimation.ts` | 토큰 수 추정 |

### 도구 실행 오케스트레이션 (`services/tools/`)
```
StreamingToolExecutor
  ├─ API 응답 스트리밍 중 도구 호출 감지
  ├─ isConcurrencySafe() → true인 도구는 병렬 실행
  └─ 결과를 메시지에 첨부

runTools()
  ├─ 도구 결과 수집
  ├─ applyToolResultBudget() → 결과 크기 제한
  └─ contextModifier 적용
```

---

## 10. 풀스택 개발자를 위한 인사이트

### A. CLAUDE.md를 최대한 활용하라

**소스코드에서 확인된 사실:**
- `context.ts`의 `getUserContext()`가 세션 시작 시 CLAUDE.md를 로딩
- 프로젝트 디렉토리부터 홈 디렉토리까지 계층적으로 탐색
- 내용이 시스템 프롬프트에 직접 주입됨

**실전 활용:**
```
~/.claude/CLAUDE.md          → 모든 프로젝트 공통 지침
프로젝트/.claude/CLAUDE.md   → 프로젝트별 지침
하위폴더/.claude/CLAUDE.md   → 모듈별 세부 지침
```

- 코딩 컨벤션, 아키텍처 결정, 금지 패턴을 명시하면 모든 대화에 반영
- `CLAUDE.md`에 "이 프로젝트는 Next.js 14 App Router를 사용하고, 서버 컴포넌트를 기본으로 한다"처럼 명시하면 매번 설명할 필요 없음

### B. 커스텀 스킬로 반복 작업 자동화

**소스코드에서 확인된 사실:**
- `skills/loadSkillsDir.ts`가 `.claude/skills/` 디렉토리에서 마크다운 파일 로딩
- Frontmatter로 도구 허용/금지, 모델 지정, 인자 정의 가능
- 스킬은 `/스킬이름` 또는 `SkillTool`로 호출 가능

**실전 활용:**
```markdown
<!-- .claude/skills/deploy.md -->
---
description: "프로덕션 배포 스크립트 실행"
tools: [Bash, FileRead]
---
1. 테스트 실행: npm test
2. 빌드: npm run build
3. 배포: npm run deploy
4. 헬스체크 확인
```

### C. Permission 규칙으로 안전하게 자동화

**소스코드에서 확인된 사실:**
- `filterToolsByDenyRules()`가 정규식 패턴 매칭 지원
- `preparePermissionMatcher()`로 도구별 커스텀 매칭 가능
- 패턴: `Bash(git *)` → git 명령만 허용

**실전 팁:**
```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Bash(npm test *)", "Bash(npm run build)",
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)"
    ]
  }
}
```

### D. 도구 실행의 병렬성 이해

**소스코드에서 확인된 사실:**
- `isConcurrencySafe()` → `true`인 도구만 병렬 실행
- Read, Glob, Grep 등 읽기 도구는 병렬 안전
- FileEdit, FileWrite 등 쓰기 도구는 순차 실행

**실전 팁:**
- 여러 파일을 동시에 읽어야 할 때 Claude가 자동으로 병렬화함
- 파일 수정은 순차적이므로 큰 리팩토링은 Agent Tool로 분리하면 더 빠름

### E. Agent Tool (서브에이전트) 전략적 활용

**소스코드에서 확인된 사실:**
- `Task.ts`의 TaskType: `local_bash`, `local_agent`, `remote_agent`, `in_process_teammate`
- 서브에이전트는 독립적 컨텍스트/AbortController/메시지 이력을 가짐
- `isolation: "worktree"` 옵션으로 Git 워크트리 격리 가능

**실전 팁:**
- 복잡한 작업을 독립적 서브태스크로 분할하면 메인 컨텍스트 오염 방지
- 워크트리 격리로 실험적 변경을 안전하게 테스트
- `run_in_background: true`로 병렬 작업 실행

### F. 컨텍스트 관리 전략

**소스코드에서 확인된 사실:**
- `services/compact/autoCompact.ts`가 토큰 임계치 초과 시 자동 압축
- `applyToolResultBudget()`가 도구 결과 크기 제한
- `maxResultSizeChars`로 도구별 결과 최대 크기 설정

**실전 팁:**
- `/compact` 명령으로 수동 압축 가능 (긴 세션에서 유용)
- 큰 파일 읽기 시 `offset`과 `limit` 파라미터 활용
- Grep 결과가 너무 많으면 `head_limit`으로 제한

### G. Hooks 시스템으로 워크플로우 커스터마이즈

**소스코드에서 확인된 사실:**
- `utils/hooks/` 디렉토리에 훅 인프라
- `PreToolUse`, `PostToolUse`, `SessionStart`, `FileChanged` 등 이벤트
- 훅은 settings.json에서 설정

**실전 팁:**
```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "echo 'Bash 실행 감지: $TOOL_INPUT'"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "FileEdit",
        "command": "npx prettier --write $FILE_PATH"
      }
    ]
  }
}
```

### H. MCP 서버로 도구 확장

**소스코드에서 확인된 사실:**
- MCP 도구는 빌트인 도구와 완전히 동등한 인터페이스로 통합
- `assembleToolPool()`에서 빌트인 + MCP 도구 병합
- MCP 리소스도 접근 가능 (ListMcpResourcesTool, ReadMcpResourceTool)

**실전 팁:**
- 프로젝트별 MCP 서버로 데이터베이스, 내부 API 등에 접근 가능
- `.claude/settings.json`의 `mcpServers`에서 설정
- 공식 레지스트리에서 검증된 MCP 서버 사용 권장

### I. 시스템 프롬프트 캐싱 이해

**소스코드에서 확인된 사실:**
- `systemPromptSection()`으로 정의된 섹션은 세션 동안 캐싱
- `DANGEROUS_uncachedSystemPromptSection()`만 매 턴 재계산 (캐시 브레이크)
- `/clear`와 `/compact`가 캐시 초기화

**실전 팁:**
- 시스템 프롬프트가 변경된 느낌이 들면 `/clear` 실행
- 캐시가 API 비용을 절약하므로 불필요한 캐시 브레이크 피하기

### J. Feature Flag 기반 숨겨진 기능들

**소스코드에서 확인된 feature flag들:**
| Flag | 기능 |
|------|------|
| `PROACTIVE` / `KAIROS` | 자율 에이전트 모드 |
| `COORDINATOR_MODE` | 다중 에이전트 코디네이터 |
| `AGENT_TRIGGERS` | 크론 기반 에이전트 트리거 |
| `VOICE_MODE` | 음성 인터페이스 |
| `BRIDGE_MODE` | IDE 브리지 모드 |
| `WORKFLOW_SCRIPTS` | 워크플로우 스크립트 실행 |
| `WEB_BROWSER_TOOL` | 웹 브라우저 도구 |
| `UDS_INBOX` | 유닉스 도메인 소켓 메시징 |
| `CONTEXT_COLLAPSE` | 컨텍스트 축소 |
| `HISTORY_SNIP` | 히스토리 스닙 |
| `TERMINAL_PANEL` | 터미널 패널 캡처 |
| `FORK_SUBAGENT` | 포크 서브에이전트 |
| `BUDDY` | 버디 모드 |

---

## 부록: 파일별 크기 순위 (상위)

| 파일 | 크기 | 역할 |
|------|------|------|
| `main.tsx` | ~804KB / 4,683줄 | CLI 메인 로직 |
| `query.ts` | ~69KB | LLM 쿼리 루프 |
| `interactiveHelpers.tsx` | ~57KB | 대화형 UI 헬퍼 |
| `QueryEngine.ts` | ~47KB | 쿼리 엔진 |
| `Tool.ts` | ~30KB | 도구 타입 시스템 |
| `commands.ts` | ~25KB | 명령어 레지스트리 |
| `dialogLaunchers.tsx` | ~23KB | 다이얼로그 런처 |
| `setup.ts` | ~21KB | 세션 설정 |
| `tools.ts` | ~17KB | 도구 레지스트리 |
| `history.ts` | ~14KB | 이력 관리 |
| `cost-tracker.ts` | ~11KB | 비용 추적 |

---

*이 문서는 Claude Code 소스코드를 직접 분석하여 작성되었습니다.*
