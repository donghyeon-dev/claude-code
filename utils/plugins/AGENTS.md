# utils/plugins 지식 베이스

## 개요
`utils/plugins/`는 플러그인 discovery, loading, validation, marketplace sync, startup reconciliation을 책임집니다.

## 어디를 먼저 볼지
| 작업 | 위치 | 메모 |
|------|------|------|
| 메인 로더 | `pluginLoader.ts` | 가장 큰 플러그인 오케스트레이션 핫스팟 |
| 마켓플레이스 상태 | `marketplaceManager.ts`, `officialMarketplace*.ts` | 원격 카탈로그/캐시 처리 |
| 검증/스키마 | `validatePlugin.ts`, `schemas.ts` | manifest correctness 경계 |
| 시작 시점 정합화 | `performStartupChecks.tsx`, `pluginStartupCheck.ts`, `reconciler.ts` | startup enforcement |
| 자산 타입별 로더 | `loadPluginCommands.ts`, `loadPluginAgents.ts`, `loadPluginHooks.ts`, `loadPluginOutputStyles.ts` | capability별 ingestion 분리 |

## 규약
- 플러그인 capability ingestion은 타입별로 나뉘어 있습니다. 이유가 있는 분리입니다.
- marketplace/install 상태와 validation은 별도 책임입니다.
- `utils/plugins/`는 discovery/load/validation/marketplace/startup을 맡고, `../../services/plugins/`는 서비스 계층 operation 쪽을 맡습니다.
- `refresh.ts`, `zipCache.ts` 같은 일부 파일은 로더 중심이 아니라 운영성 파일입니다.

## 안티패턴
- capability별 로더가 이미 있는데 모든 새 로직을 `pluginLoader.ts`에 몰아넣지 마세요.
- 편의상 validation/schema 경로를 우회하지 마세요.
- startup policy와 marketplace fetch/cache 로직을 섞지 마세요.

## 메모
- `utils/` 아래에 있지만 별도 규칙이 필요한 크기와 밀도를 가집니다.
- 플러그인 동작이 이상할 때는 ingestion → validation → startup checks를 분리해서 봐야 합니다.
