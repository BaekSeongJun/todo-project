# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

개인용 Todo 관리 서비스. **Spring Boot 4.1.1 백엔드 + Next.js 16.3.3 프론트엔드 모노레포**이며, 로컬에서 전 기능을 완성한 뒤 AWS로 이전하는 방식으로 진행한다.

**현재 상태: Phase 0(스캐폴딩) 진행 중.** 백엔드는 `TodoBackendApplication` 하나, 프론트는 `create-next-app` 초기 상태다. 문서(`docs/`)가 코드보다 크게 앞서 있으므로, **구현 전에 반드시 해당 Phase의 명세를 읽는다.**

## 문서 계층 (중요)

이 저장소는 문서가 사양의 출처다. 역할이 분리되어 있고 **충돌 시 우선순위가 명시**되어 있다.

| 문서 | 역할 | 우선순위 |
|---|---|---|
| `docs/PRD.md` | **무엇을** 만드는가 — 기술 스택·데이터 모델·API 계약의 SSOT | **최상위.** 다른 문서와 어긋나면 PRD를 따른다 |
| `docs/ROADMAP.md` | **어떤 순서로** 만드는가 — Phase 0~12, 각 Phase의 완료 조건 | PRD 하위 |
| `docs/guides/` | **어떻게** 만드는가 — 프론트엔드 구현 패턴 4종 | PRD 하위 |

- 요구사항은 **FR ID**(`FR-A04`, `FR-T06`, `FR-M07` …)로 참조된다. 작업 시 담당 FR ID를 ROADMAP 4장 추적 매트릭스에서 확인한다.
- `docs/guides/`: `project-structure.md`(폴더 구조·네이밍), `nextjs-16.md`(App Router 규칙), `component-patterns.md`, `styling-guide.md`(Tailwind v4 + shadcn), `forms-react-hook-form.md`
- PRD 1.3 표를 **양방향으로 동기화**한다: 라이브러리를 설치하면 즉시 ✅로 갱신하고, 표에 없는데 `pom.xml`/`package.json`에 있는 의존성을 발견하면 표에 추가한다.

## 개발 명령어

### 백엔드 (`todo-backend/`, Maven Wrapper)

```bash
./mvnw compile                              # 컴파일
./mvnw spring-boot:run                      # 실행 (DB_PASSWORD 환경변수 필수)
./mvnw test                                 # 전체 테스트
./mvnw test -Dtest=TodoServiceTest          # 단일 테스트 클래스
./mvnw test -Dtest=TodoServiceTest#소유권_검증 # 단일 테스트 메서드
```

Windows PowerShell에서는 `.\mvnw.cmd`를 사용한다.

### 프론트엔드 (`todo-frontend/`)

```bash
npm run dev          # 개발 서버
npm run build        # 프로덕션 빌드 (M1 게이트 조건)
npm run typecheck    # tsc --noEmit
npm run lint         # eslint --max-warnings=0
npm run format       # prettier --write .
npm run validate     # typecheck + lint + format:check (커밋 전 전체 검증)
```

**`next lint`는 Next.js 16에서 제거되었다.** `npm run lint`(ESLint 직접 호출)를 쓴다.

Git 훅(husky, `todo-frontend/`에 설치): pre-commit은 `lint-staged`, commit-msg는 `commitlint`, pre-push는 `npm run typecheck`.

### 커밋 메시지 형식

`<이모지> <타입>: <한글 설명>` (예: `✨ feat: Todo 목록 페이지네이션 구현`). commitlint가 강제하며, 첫 줄 72자 이내·마침표 금지. ROADMAP 6장은 Phase 단위 커밋에 `[Phase 4] Todo CRUD API 구현` 형태를 쓴다.

## 아키텍처의 핵심 결정

### 백엔드

- **베이스 패키지 `com.example`**, JDK 21, Maven. Lombok annotation processor 설정 완료.
- **Spring Boot 4 / Security 7**이다. `antMatchers` 등 구 API는 컴파일되지 않는다. 스타터명도 `spring-boot-starter-web`이 아니라 **`-webmvc`**다. 문법이 불확실하면 추측하지 말고 확인한다.
- **인증은 JWT Access Token 단독**(HS256, 24시간). **Refresh Token 도입 금지.** 다만 JWT 필터는 매 요청 `userId`로 사용자를 조회해 `enabled = true AND deleted_at IS NULL`을 확인한다(FR-A04-1) — 비활성 계정의 기존 토큰을 차단하기 위한 것으로, stateless 원칙의 예외가 아니다.
- **구글 OAuth2는 일회용 코드 교환 방식**이다. JWT를 URL에 노출하지 않고 1~2분 TTL의 일회용 코드를 프론트에 전달한 뒤 `POST /api/auth/oauth/exchange`로 교환한다. 코드는 **서버 인메모리 저장소**에 둔다(테이블·Redis 금지).
- 소셜 로그인 사용자 조회 키는 **`email`**이다. `provider_id`는 가입 경로 기록용이며 조회에 쓰지 않는다(FR-A09).

### Soft Delete 규약 (전 계층 관통)

- 물리 삭제를 하지 않는다. `deleted_at TIMESTAMP`를 기록하고 모든 조회에서 제외한다.
- `Todo`·`Attachment`는 **`@SQLRestriction("deleted_at IS NULL")`**.
- **`users`에는 `@SQLRestriction`을 걸지 않는다.** ToOne 연관 대상이 Soft Delete되면 `EntityNotFoundException`이 발생하기 때문이며, 대신 Repository 메서드에 조건을 명시한다. `@NotFound`는 강제 eager 로딩을 유발하므로 금지. Hibernate `@SoftDelete`도 boolean 기반이라 사용하지 않는다.
- 관리자의 전체 할 일 조회(FR-M07)는 연관 매핑 탐색 대신 **명시적 `JOIN` + DTO 프로젝션**으로 작성한다.

### API 응답 계약 (PRD 8.1)

- 성공: `ApiResponse<T>` = `{ success, data, message }`
- 목록: `data`가 `PageResponse<T>` = `{ content, page, size, totalElements, totalPages, first, last }`. **`page`는 0-base**이며 표시 단계에서만 1-base로 변환한다.
- 에러: 래퍼 없는 평면 구조 `{ timestamp, status, code, message, errors[] }`. 필드명은 `code`(`errorCode` 아님)이며, 값 체계는 Phase 2에서 확정한다. **프론트는 `code` 값을 하드코딩하지 않는다.**
- 엔티티를 직접 노출하지 않고 응답 DTO만 반환한다.
- **소유권 검증 실패는 404**(존재 여부를 숨기기 위해 403을 쓰지 않는다). 단 `/api/admin/**` 역할 차단은 **403**이다.

### 프론트엔드

- **`src/` 디렉터리를 사용하지 않는다.** `app/`, `components/`, `lib/`, `hooks/`가 `todo-frontend/` 루트에 직접 있다. 경로 alias는 `@/*` → `./*`.
- App Router. Server Components 우선, `'use client'`는 최소한으로.
- 생성·수정은 별도 페이지가 아니라 `/todos` 내 **다이얼로그**로 처리한다. OAuth 콜백 경로는 `/oauth/callback`(`/oauth2/callback` 아님).
- shadcn/ui style은 **`radix-nova`**, baseColor `neutral`(new-york 아님).
- 완료 토글·삭제는 **낙관적 업데이트 + 실패 시 롤백 + 에러 토스트**. React Query 쿼리 키에 필터·정렬·페이지를 모두 포함해 페이지네이션 캐시와 어긋나지 않게 한다.
- 다크 모드가 **기본**이고 라이트 토글을 제공한다. 그라데이션·글로우·과한 그림자는 쓰지 않는다(뉴트럴 미니멀).

## 금지 사항과 함정

- **Docker / Testcontainers 사용 금지.** 통합테스트는 로컬 PostgreSQL의 `todolist_db_test` 스키마를 직접 쓴다.
- **미설치 라이브러리를 `import` 하지 않는다.** PRD 1.3 표에서 설치 상태가 ❌인 것은 담당 Phase에서 설치한 뒤 사용한다. (예: TanStack Query·next-themes는 Phase 5, React Hook Form+Zod는 Phase 6, Tiptap·Framer Motion은 Phase 7, jsoup은 Phase 4)
- **springdoc-openapi는 3.1.0 이상.** 2.x는 Boot 4 미지원이고 3.0.x도 Jackson 마찰이 보고되어 있다.
- **시크릿을 소스에 하드코딩하지 않는다.** PRD 9.1 환경변수 표의 12종(`DB_*`, `JWT_SECRET`, `GOOGLE_*`, `APP_*`, `NEXT_PUBLIC_API_BASE_URL`)은 전부 외부 주입이다. `.env*`·`application-local.yml`은 커밋 금지.
- **⚠️ 중첩 git 저장소:** 현재 `todo-backend/.git`과 `todo-frontend/.git`이 독립 저장소로 존재하고 루트 저장소는 커밋이 0개다. ROADMAP 6장은 **루트 단일 저장소**를 전제하므로 Phase 0에서 통합해야 한다. 그 전까지는 루트에서 `git add`를 해도 하위 프로젝트 파일이 추적되지 않는다.
- 프론트엔드 자동화 테스트는 범위 밖이며, **E2E 도구를 신규 도입하지 않는다.**
- `todo-frontend/AGENTS.md`의 `nextjs-agent-rules` 블록은 `next dev`가 자동 생성·재작성한다. 지우지 말고 변경분과 함께 커밋한다.

## 작업 진행 방식

- **한 번에 한 Phase만.** 앞 Phase의 완료 조건(ROADMAP 3장)을 통과하기 전에 다음으로 넘어가지 않는다.
- Phase 내부 세부 작업은 `/tasks/XXX-description.md`(예: `001-scaffolding.md`)로 관리한다.
- 에러 처리·로딩 상태·테스트를 뒤로 미루지 않고 해당 Phase 안에서 끝낸다.
- **버전이나 API가 확신되지 않으면 추측하지 말고 질문한다.** Boot 4 / Security 7 / Next 16은 모두 최신 메이저라 학습 데이터와 어긋날 수 있다.
- 게이트: **M1 = Phase 8 종료 시 PRD 12.1 인수 기준 전부**, **M2 = Phase 11 종료 시 PRD 12.1 + 12.2 전부(24개)**.
