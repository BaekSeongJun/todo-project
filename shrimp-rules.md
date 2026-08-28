# AI Agent 운영 규칙 (shrimp-rules.md)

> 이 문서는 AI Agent(코딩 에이전트) 전용 운영 규칙이다. 일반 개발 지식이나 프로젝트 기능 설명은 포함하지 않는다.
> **최상위 출처는 항상 `CLAUDE.md`(루트) → `docs/PRD.md` → `docs/ROADMAP.md` → `docs/guides/`다.** 이 문서는 그 규칙을 재기술하지 않고, 작업 착수 전 확인 순서와 결정 기준만 압축한다. 상세 규칙 문구는 반드시 원본 문서에서 직접 확인한다.

## 1. 작업 착수 전 필수 확인 순서

- 어떤 작업이든 시작 전 **`CLAUDE.md`(루트)를 먼저 읽는다.** 이미 읽은 세션 안에서도 규칙이 갱신됐을 수 있으므로 장시간 세션에서는 재확인한다.
- 요구사항/스펙 관련 작업은 `docs/PRD.md`를 먼저 확인한다. `docs/ROADMAP.md`·`docs/guides/*`와 내용이 충돌하면 **PRD.md가 항상 이긴다.**
- FR ID(`FR-A04`, `FR-T06`, `FR-M07` 등)가 언급된 작업은 `docs/ROADMAP.md` 4장 추적 매트릭스에서 해당 ID의 Phase·완료조건을 확인한 뒤 착수한다.
- **현재 상태는 Phase 0(스캐폴딩)이다.** 다음 Phase 작업을 먼저 요청받아도, 앞 Phase의 `docs/ROADMAP.md` 3장 완료 조건이 아직 통과하지 않았다면 사용자에게 알리고 순서를 확인한다 — 임의로 건너뛰지 않는다.
- 프론트엔드 구현 패턴(컴포넌트 구조, 폼, 스타일링)은 `docs/guides/` 중 해당 가이드 파일을 먼저 연다: `project-structure.md`, `nextjs-16.md`, `component-patterns.md`, `styling-guide.md`, `forms-react-hook-form.md`.
- Spring Boot 4 / Spring Security 7 / Next.js 16의 API·문법이 확신되지 않으면 **추측하지 말고 `context7` MCP로 최신 문서를 조회하거나 사용자에게 질문한다.** 학습 데이터가 이 메이저 버전들과 어긋날 수 있다.

## 2. 저장소 구조와 커밋 범위

- 이 프로젝트는 **3개의 독립 git 저장소**로 구성된다: 루트(`D:\claude\todo-project`), `todo-backend/`, `todo-frontend/`. 통합 저장소가 아니다 — 절대 하나로 합치지 않는다.
- 백엔드 파일을 고쳤으면 `todo-backend/` 안에서 커밋하고, 프론트엔드 파일을 고쳤으면 `todo-frontend/` 안에서 커밋한다. 문서(`docs/`, 루트 `CLAUDE.md`)를 고쳤으면 루트 저장소에서 커밋한다. **한 커밋에 서로 다른 저장소의 변경을 섞지 않는다.**
- 커밋 메시지는 `<이모지> <타입>: <한글 설명>` 형식(commitlint 강제, 첫 줄 72자 이내, 마침표 금지). Phase 단위 커밋은 `[Phase N] 설명` 형태를 함께 쓴다.

## 3. 코드 작성 규칙 (프로젝트 특정)

### 백엔드 (`todo-backend/`)
- 베이스 패키지는 `com.example`만 쓴다. 새 클래스를 다른 루트 패키지에 만들지 않는다.
- Spring Boot 4 / Spring Security 7 기준 최신 API만 쓴다. `antMatchers` 등 구버전 Security API는 컴파일되지 않으므로 절대 쓰지 않는다.
- 웹 스타터 의존성을 추가할 때 아티팩트명은 `spring-boot-starter-webmvc`다 (`spring-boot-starter-web` 아님 — `todo-backend/pom.xml` 확인 결과 이미 `-webmvc`로 존재).
- `springdoc-openapi`를 추가할 경우 **3.1.0 이상**만 쓴다. 2.x는 Boot 4 미지원.

### 프론트엔드 (`todo-frontend/`)
- `src/` 디렉터리를 만들지 않는다. `app/`, `components/`, `lib/`, `hooks/`는 `todo-frontend/` 루트에 직접 둔다 (현재 구조가 이미 이렇게 되어 있음 — 유지한다).
- import 경로는 `@/*` alias(→ `./*`)를 쓴다.
- App Router 기준 Server Component를 기본으로 쓰고, `'use client'`는 상태·이벤트가 실제로 필요한 최소 범위에만 붙인다.
- Todo 생성·수정 UI는 별도 페이지가 아니라 `/todos` 내부 **다이얼로그**로 구현한다.
- OAuth 콜백 라우트 경로는 반드시 `/oauth/callback`이다 (`/oauth2/callback` 아님 — 오타 주의).
- shadcn/ui 설정은 `components.json`에서 style `radix-nova`, baseColor `neutral`을 유지한다 (`new-york` 등으로 바꾸지 않는다).

## 4. 기능 구현 표준

- **Soft Delete**: 물리 삭제(`DELETE FROM` / `repository.delete()`)를 쓰지 않는다. `deleted_at` 컬럼에 타임스탬프를 기록하는 방식만 쓴다. `Todo`·`Attachment` 엔티티에는 `@SQLRestriction("deleted_at IS NULL")`을 쓰고, `User` 엔티티에는 **절대 `@SQLRestriction`을 걸지 않는다** — ToOne 연관에서 `EntityNotFoundException`이 발생하기 때문. `User` 조회 시 삭제 여부 필터링은 Repository 메서드 조건으로 명시한다. `@NotFound` 애노테이션도 강제 eager 로딩을 유발하므로 쓰지 않는다.
- 관리자 전체 Todo 조회(`FR-M07`)는 JPA 연관 매핑 탐색이 아니라 **명시적 JOIN + DTO 프로젝션**으로 구현한다.
- **API 응답 계약**: 성공 응답은 `ApiResponse<T> = { success, data, message }`. 목록 응답의 `data`는 `PageResponse<T> = { content, page, size, totalElements, totalPages, first, last }`이며 `page`는 0-base로 반환한다(1-base 변환은 프론트 표시 단계에서만). 에러 응답은 래퍼 없는 평면 구조 `{ timestamp, status, code, message, errors[] }`이며 필드명은 반드시 `code`다(`errorCode` 아님). **프론트는 `code` 값을 하드코딩하지 않는다** — 값 체계가 Phase 2에서 확정되기 전까지 상수화하지 않는다.
- 컨트롤러는 JPA 엔티티를 직접 반환하지 않는다. 항상 응답 DTO로 변환해서 반환한다.
- 소유권 검증 실패(다른 사용자의 리소스 접근)는 **404**를 반환한다(403 아님 — 리소스 존재 여부를 숨기기 위함). `/api/admin/**` 경로의 역할 기반 차단만 **403**이다.
- 인증은 JWT Access Token 단독(HS256, 24시간 만료)만 쓴다. **Refresh Token을 도입하지 않는다.** JWT 필터는 매 요청마다 `userId`로 사용자를 조회해 `enabled = true AND deleted_at IS NULL`을 검사한다 — 이 검사를 생략하거나 캐싱으로 우회하지 않는다.
- 구글 OAuth2 로그인은 JWT를 URL 쿼리 파라미터로 넘기지 않는다. 1~2분 TTL의 일회용 코드를 발급해 프론트에 전달하고, 프론트는 `POST /api/auth/oauth/exchange`로 코드를 JWT와 교환한다. 코드 저장소는 **서버 인메모리**만 쓴다(DB 테이블, Redis 등 영속 저장소 금지).
- 소셜 로그인 사용자 조회 키는 `email`이다. `provider_id`는 가입 경로 기록용으로만 저장하고, 사용자 조회 조건으로 쓰지 않는다.
- 완료 토글·삭제 등 즉시 반응이 필요한 프론트 액션은 **낙관적 업데이트 + 실패 시 롤백 + 에러 토스트** 패턴으로 구현한다. React Query 쿼리 키에는 필터·정렬·페이지 값을 모두 포함시킨다(누락 시 페이지네이션 캐시가 어긋남).
- 다크 모드가 기본 테마다. 라이트 모드는 토글로만 제공한다. 그라데이션·글로우 효과·과한 그림자를 쓰지 않는다(뉴트럴 미니멀 톤 유지).

## 5. 프레임워크/라이브러리 사용 표준

- **설치되지 않은 라이브러리를 import하지 않는다.** 아래는 담당 Phase 이전에 쓰면 안 되는 대표 라이브러리다 (현재 Phase 0이므로 전부 아직 금지):
  - TanStack Query, next-themes → Phase 5
  - React Hook Form, Zod → Phase 6
  - Tiptap, Framer Motion → Phase 7
  - jsoup(백엔드) → Phase 4
- 라이브러리를 새로 설치했으면(백엔드 `pom.xml` 또는 프론트 `package.json` 변경) **같은 작업 안에서 `docs/PRD.md` 1.3 표를 ✅로 갱신한다.**
- 반대로 `pom.xml`/`package.json`을 살펴보다 `docs/PRD.md` 1.3 표에 없는 의존성을 발견하면, 표에 행을 추가해 동기화한다.
- Docker, Testcontainers를 어떤 형태로도 도입하지 않는다. 통합 테스트는 로컬 PostgreSQL의 `todolist_db_test` 스키마를 직접 사용한다.
- E2E 테스트 도구(Playwright, Cypress 등)를 신규로 프로젝트에 도입하지 않는다. (MCP로 연결된 Playwright 도구는 수동 확인 용도로만 쓰고, 프로젝트 코드에 E2E 테스트 스위트를 추가하지 않는다.)

## 6. 핵심 파일 상호작용 규칙

| 이 파일을 수정하면 | 반드시 함께 확인/수정할 것 |
|---|---|
| `todo-backend/pom.xml` 또는 `todo-frontend/package.json`에 의존성 추가 | `docs/PRD.md` 1.3 라이브러리 표를 ✅로 갱신 |
| FR ID가 걸린 기능 구현 | `docs/ROADMAP.md` 4장 추적 매트릭스 상태 확인/갱신 |
| API 응답 DTO, 엔티티 필드 변경 | `docs/PRD.md` 8장(API 계약) 및 데이터 모델 절과 어긋나지 않는지 대조 |
| `todo-frontend/AGENTS.md` | **직접 수정하지 않는다.** `next dev`가 자동 생성·재작성하는 파일이다. 내용이 바뀌어 있으면 지우지 말고 다른 변경분과 함께 그대로 커밋한다 |
| `todo-frontend/CLAUDE.md` | 이 파일은 `@AGENTS.md` 위임 한 줄만 유지한다. 규칙을 여기 직접 쓰지 않는다 |
| 루트 `CLAUDE.md` | PRD/ROADMAP과 표현이 어긋나지 않는지 재확인. 이 파일이 다른 모든 문서보다 우선하는 최종 규칙이다 |
| `.env*`, `application-local.yml` | 절대 커밋하지 않는다(`.gitignore`로 이미 제외됨을 재확인). 시크릿 12종은 PRD 9.1 표 기준 전부 외부 주입 |

## 7. AI 의사결정 기준 (모호한 상황 처리)

- **버전/문법이 불확실할 때**: 학습 데이터 추측 금지 → `context7` MCP로 해당 라이브러리 최신 문서 조회 → 그래도 불명확하면 사용자에게 질문. (Spring Boot 4 / Security 7 / Next.js 16은 모두 최신 메이저라 특히 이 규칙이 자주 적용된다.)
- **Phase 순서를 벗어난 요청을 받았을 때**: 즉시 구현하지 말고, 현재 Phase(0)와 앞 Phase 완료 조건 미충족 여부를 사용자에게 알리고 진행 여부를 확인한다.
- **PRD와 ROADMAP/guides가 충돌할 때**: PRD.md를 따른다. 충돌을 발견하면 사용자에게 알린다(임의로 한쪽을 무시하고 조용히 진행하지 않는다).
- **테이블에 없는 새 의존성을 설치해야 할 때**: 설치 자체는 담당 Phase 범위 안이면 진행하되, PRD 1.3 표 갱신을 같은 작업 단위로 묶는다(별도 후속 작업으로 미루지 않는다).
- **에러 처리·로딩 상태·테스트를 나중으로 미루고 싶은 유혹이 들 때**: 미루지 않는다. 해당 Phase 안에서 끝낸다(`CLAUDE.md` "작업 진행 방식" 규칙).

## 8. 금지 행위 목록

- ❌ Docker 또는 Testcontainers를 테스트/실행 환경에 도입
- ❌ `todo-frontend/`에 `src/` 디렉터리 생성
- ❌ JWT Refresh Token 메커니즘 추가
- ❌ 소셜 로그인 사용자 조회 시 `provider_id`를 조회 키로 사용
- ❌ `Todo`/`Attachment`/사용자 데이터의 물리 삭제(하드 delete)
- ❌ `User` 엔티티에 `@SQLRestriction` 또는 `@NotFound` 적용
- ❌ OAuth 인증 결과 JWT를 URL 파라미터로 프론트에 전달
- ❌ 소유권 검증 실패 시 403 반환(404여야 함) — 단 `/api/admin/**`은 예외적으로 403
- ❌ 담당 Phase 이전에 미설치 라이브러리 import (섹션 5 표 참조)
- ❌ 새 E2E 테스트 도구 도입
- ❌ 루트/`todo-backend`/`todo-frontend` 3개 저장소를 하나로 통합하거나, 한 커밋에 여러 저장소 변경 혼합
- ❌ `.env*`, `application-local.yml`, 시크릿 값을 소스에 하드코딩하거나 커밋
- ❌ `todo-frontend/AGENTS.md`의 `nextjs-agent-rules` 블록을 삭제
- ❌ API 에러 응답 필드명을 `errorCode`로 사용(반드시 `code`)
- ❌ 앞 Phase 완료 조건 미통과 상태에서 다음 Phase 작업에 임의로 착수
