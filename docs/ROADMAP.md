# ROADMAP — Todo List 서비스

| 항목 | 내용 |
|---|---|
| 문서 버전 | 1.4 |
| 기준 문서 | `PRD.md` v1.4 (SSOT) |
| 관련 문서 | `docs/guides/` (프론트 구현 패턴) |
| 개발 방식 | Claude Code 바이브 코딩, 로컬 개발 후 AWS 이전 |

> PRD가 **무엇을 만들지**, `docs/guides/`가 **어떻게 만들지**를 정한다면, 이 문서는 **어떤 순서로 언제까지 만들지**를 정한다.
> 이 문서와 PRD가 어긋나면 **PRD를 따른다.**

---

## 1. 진행 원칙

1. **한 번에 한 Phase만.** 앞 Phase의 완료 조건을 통과하기 전에 다음으로 넘어가지 않는다.
2. **Phase 종료 = 커밋.** 완료 조건을 통과하면 즉시 커밋하고, 실패하면 그 자리에서 고친다.
   - ⚠️ 이 원칙은 **단일 git 저장소**를 전제한다. Phase 0에서 중첩 `.git` 정리(6장 참조)를 끝내기 전까지는 성립하지 않는다.
3. **로컬 우선.** Phase 0~11은 전부 로컬에서 개발한다. AWS는 Phase 12에서만 다룬다.
4. **뒤로 미루지 않는다.** 각 Phase에 필요한 에러 처리·로딩 상태·테스트를 그 Phase 안에서 끝낸다.
5. **막히면 멈춘다.** Claude Code가 버전이나 API를 확신하지 못하면 추측하게 두지 말고 질문하게 한다.
6. **라이브러리는 담당 Phase에서 설치한다.** PRD 1.3 표에서 설치 상태가 ❌인 라이브러리는 그 Phase 전에 `import`하지 않는다. **설치 후에는 PRD 1.3 표의 설치 상태를 ✅로 갱신**하는 것까지가 그 Phase의 완료 조건이다(PRD 1.3 양방향 동기화 의무).
7. **작업 파일 규약.** Phase 내부의 세부 작업은 `/tasks/XXX-description.md`(예: `001-scaffolding.md`)로 관리한다. 각 파일은 고수준 명세·관련 파일·수락 기준·구현 단계를 포함한다. Phase 상세의 산출물이 곧 작업 파일의 상위 단위다.

---

## 2. 마일스톤 개요

| 마일스톤 | 범위 | 목표 상태 |
|---|---|---|
| **M1 — 핵심 MVP** | Phase 0~8 | 로그인하고 할 일을 관리할 수 있는 완성된 서비스가 로컬에서 동작 |
| **M2 — 확장 기능** | Phase 9~11 | 계정 복구, 파일 첨부, 관리 기능까지 갖춘 완전한 기능 세트 |
| **M3 — 운영 배포** | Phase 12 | AWS에서 실제 도메인으로 서비스 구동 |

```
M1 ─────────────────────────────► M2 ──────────────► M3
0  1  2  3  4  5  6  7  8        9  10  11          12
└─ 백엔드 ─┘  └─ 프론트 ─┘ 테스트  └─ 확장 ─┘        배포
```

---

## 3. Phase 상세

### M1 — 핵심 MVP

#### Phase 0 · 프로젝트 스캐폴딩
- **목표:** 빌드되는 빈 모노레포와 로컬 개발 환경, 그리고 **단일 git 저장소**
- **선행:** 없음 (JDK 21, Node 20+, PostgreSQL 로컬 설치 필요)
- **커버:** 비기능 — 이식성 (PRD 9.1 환경변수 분리 원칙)
- **산출물:**
  - `todo-backend`, `todo-frontend` 모노레포 배치 (PRD 1.3 · `docs/guides/project-structure.md`)
  - **루트 단일 git 저장소** — `todo-backend/.git`, `todo-frontend/.git` 제거 후 루트 저장소로 통합
  - 루트 `.gitignore` (백엔드 `target/`, 프론트 `node_modules/`·`.next/`, `.env*`, `application-local.yml`, 업로드 디렉터리 포함)
  - 루트 `README.md` (한글)
  - **`application.yml`(공통) + `application-local.yml`(로컬, 커밋 금지) + `application-prod.yml`(운영)** 프로파일 분리
  - `/tasks/` 디렉터리와 `000-sample.md` (진행 원칙 7)
- **완료 조건:**
  - `./mvnw compile` 성공 · `npm run build` 성공
  - 폴더 구조가 `docs/guides/project-structure.md`와 일치
  - **루트에서 `git status`가 `todo-backend`·`todo-frontend`의 파일들을 정상 추적** (중첩 저장소 없음, `git log`에 최초 커밋 존재)
  - `.env*`·`application-local.yml`이 `git check-ignore`로 무시됨을 확인
  - PRD 9.1 표의 환경변수가 모두 `application-local.yml`(또는 `.env.local`)에서 주입되고 소스에 하드코딩되지 않음
- **⚠️ 현재 상태(2026-08-28 기준, 미완료):**
  - 루트 `.gitignore`·`README.md` 없음, 루트 저장소 커밋 0개
  - `todo-backend/.git`·`todo-frontend/.git`이 **독립 저장소로 존재** → 진행 원칙 2와 6장 형상 관리 전략과 충돌
  - 백엔드 설정이 `application.properties` **단일 파일**이라 PRD 9.1의 `application-local.yml` 규약과 형식·분리 모두 불일치 → **`.yml` 전환 + 프로파일 분리**가 이 Phase의 작업
- **주의:** 여기서 환경변수 분리를 제대로 해두지 않으면 Phase 12에서 전부 되돌아와야 한다

#### Phase 1 · DB 스키마와 엔티티
- **목표:** 데이터 계층 완성
- **선행:** Phase 0
- **커버:** FR-T01, FR-T06, PRD 7장(데이터 모델·공통 컬럼 규약·인덱스)
- **DDL 생성 전략 (이 Phase에서 확정):**
  - **로컬 개발 구간(Phase 1~11)은 JPA `ddl-auto: update`로 테이블을 생성**한다. 별도 `db/init.sql`을 DDL의 출처로 두지 않는다.
  - PRD 7장 인덱스 4종과 `users.email` unique는 `ddl-auto: update`만으로는 생성되지 않으므로 **엔티티의 `@Table(indexes = ..., uniqueConstraints = ...)`에 선언**해 Hibernate가 만들게 한다.
    - `todos`: `(user_id, deleted_at, created_at)`, `(user_id, deleted_at, due_date)`
    - `attachments`: `(todo_id, deleted_at)` — 실제 생성은 Phase 10
    - `users`: `(email)` unique
    - `password_reset_tokens`: `(token)` — 실제 생성은 Phase 9
  - 스키마 생성(`CREATE SCHEMA todolist_db`)과 최초 관리자 지정(FR-M03) 같은 **JPA가 만들 수 없는 것만** `db/init.sql`에 둔다. 이 파일은 애플리케이션이 자동 실행하지 않으며 수동 적용 대상이다.
  - **`ddl-auto: validate` 전환은 Phase 12-2(prod 프로파일)에서 수행**한다. 로컬 프로파일은 끝까지 `update`를 유지한다.
- **산출물:** `db/init.sql`(스키마 생성 + 관리자 지정 SQL 전용), BaseEntity, `User`·`Todo` 엔티티, Repository
- **완료 조건:**
  - 테이블·컬럼이 snake_case로 정상 생성 (`id` BIGSERIAL, `created_at`/`updated_at`/`deleted_at` 규약 준수)
  - Enum이 `@Enumerated(EnumType.STRING)`으로 VARCHAR 저장됨 (ordinal 금지)
  - Soft Delete 동작 — `Todo`는 `@SQLRestriction("deleted_at IS NULL")`, **`users`에는 걸지 않고 Repository 조건으로 처리**(PRD 7장 조회 규칙)
  - `todolist_db` 스키마에 PRD 7장 인덱스가 실제로 생성됐음을 `\di`로 확인
- **주의:** Hibernate 6.4+의 `@SoftDelete`는 boolean 기반이라 `deleted_at TIMESTAMP` 규약과 맞지 않으므로 사용하지 않는다. `@NotFound`도 강제 eager 로딩을 유발하므로 금지 (PRD 7장)

#### Phase 2 · 인증 (Security + JWT)
- **목표:** 이메일 가입/로그인과 토큰 인증
- **선행:** Phase 1
- **커버:** FR-A01, **FR-A01-1**, FR-A02 ~ FR-A06, **FR-A04-1**, FR-A10
- **설치:** **springdoc-openapi 3.1.0 이상** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** JwtTokenProvider, JWT 인증 필터, SecurityConfig, AuthController(`/api/auth/signup`·`/login`·`/me`), `ApiResponse<T>` 래퍼, Swagger 설정
- **완료 조건:**
  - 가입·로그인 시 **24시간 만료 JWT(HS256)** 발급, `JWT_SECRET`은 환경변수 주입 (FR-A04)
  - 비밀번호 **6자 이상** 검증 + **BCrypt strength 10** 해싱 (FR-A02)
  - 이름 **1~50자 필수** 서버 검증 (FR-A01-1)
  - 중복 이메일 가입 시 **409** (FR-A03)
  - 토큰 없이 보호 API 호출 시 401 (FR-A10)
  - **JWT 필터가 매 요청 `enabled = true`·`deleted_at IS NULL`을 확인**(FR-A04-1, Phase 11의 FR-M06 전제)
  - 모든 성공 응답이 `ApiResponse<T>` 형식 (PRD 8.1)
  - Swagger UI 정상 기동, **운영 프로파일에서는 비활성화** 설정
  - **PRD 1.3 표의 springdoc-openapi 설치 상태 갱신**
- **확정 과제:** **에러 코드(`code`) 값 체계를 이 Phase에서 확정**한다 (PRD 8.1). 확정 전까지 프론트는 `code`를 하드코딩하지 않는다
- **리스크:** Spring Security 7 문법. 구 API(`antMatchers` 등)가 나오면 즉시 중단하고 교정
- **주의:** springdoc-openapi는 **3.1.0 이상**을 사용한다. 2.x는 Boot 4 미지원이며, 3.0.x도 Jackson 2/3 관련 이슈가 보고되어 있다. 기동 실패 시 버전을 먼저 의심할 것 (PRD 13장)

#### Phase 3 · 구글 OAuth2
- **목표:** 소셜 로그인
- **선행:** Phase 2, 구글 클라우드 콘솔 클라이언트 발급(`GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET`)
- **커버:** FR-A07 ~ FR-A09
- **산출물:** CustomOAuth2UserService, OAuth2 SuccessHandler, **인메모리 일회용 코드 저장소**, `POST /api/auth/oauth/exchange`
- **완료 조건:**
  - 구글 로그인 후 JWT 획득 (`/oauth2/authorization/google` 진입)
  - **JWT를 URL에 노출하지 않고** 단기(1~2분) 일회용 코드만 프론트에 전달 (FR-A08)
  - 코드 재사용 시 실패, TTL 경과 시 실패 — **별도 테이블·Redis 없이 인메모리 저장소**로 구현 (FR-A08)
  - 동일 이메일 계정 중복 생성 없이 **기존 계정에 연결** — 조회 키는 **`email`이며 `provider_id`가 아니다**(PRD 7장 `users`). `provider`·`provider_id`는 가입 경로 기록용 (FR-A09)
  - 소셜 가입 시 이름은 제공자 프로필 이름, 없으면 이메일 로컬 파트 (FR-A01-1)
- **주의:** 소셜 전용 계정은 `password`가 NULL이다 (PRD 7장 `users`). Phase 9의 FR-R03이 이 계정을 다룬다

#### Phase 4 · Todo API
- **목표:** 백엔드 기능 완성
- **선행:** Phase 2
- **커버:** FR-T02 ~ FR-T07, FR-L01 ~ FR-L04, FR-U08
- **설치:** **jsoup 1.18.x** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** Todo CRUD API 6종(PRD 8장), `PageResponse<T>`, HTML 정제기(jsoup), GlobalExceptionHandler
- **완료 조건:**
  - 목록이 **기본 size 10 · `page` 0-base · 기본 정렬 createdAt desc**로 동작하고 응답이 `PageResponse<T>` 형식(`content`/`page`/`size`/`totalElements`/`totalPages`/`first`/`last`) (FR-L01, FR-L02, PRD 8.1)
  - 상태 필터(전체/미완료/완료)와 정렬(최신순/마감일순) 쿼리 파라미터 동작 (FR-L03, FR-L04)
  - Soft Delete 후 목록·단건 조회 모두에서 제외 (FR-T06)
  - **타인 리소스 접근 시 404** (FR-T07)
  - HTML 정제 — `<script>`, `on*` 속성, `javascript:` 링크, **`<img>` 태그 제거**. `Safelist`는 FR-T02 툴바 8종(굵게·기울임·밑줄·취소선·목록·제목·인용·링크)으로 한정하며 **`preserveRelativeLinks` 활성화 금지** (FR-T03, PRD 1.3·2장 비목표)
  - 에러 응답이 **평면 구조**(`timestamp`/`status`/`code`/`message`/`errors[]`)로 통일되고 스택트레이스를 노출하지 않음 (FR-U08, PRD 8.1)
  - **PRD 1.3 표의 jsoup 설치 상태 갱신**
- **게이트:** 여기까지 통과하면 백엔드 핵심이 끝난다. Swagger로 전 API를 한 번 훑고 넘어갈 것

#### Phase 5 · 프론트 토대
- **목표:** 화면을 만들기 전의 공통 기반
- **선행:** Phase 4 (API 스펙·에러 코드 체계 확정)
- **커버:** FR-U01 ~ FR-U03, FR-L05
- **설치:** **TanStack Query, next-themes** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** 디자인 토큰(`app/globals.css`), API 클라이언트(`lib/api/client.ts` — JWT 자동 첨부·401 처리), `providers/QueryProvider.tsx`·`providers/ThemeProvider.tsx`, Pagination·EmptyState·ErrorState·Skeleton, 공통 헤더·ThemeToggle
- **완료 조건:**
  - `npm run build` 성공
  - **뉴트럴 미니멀** 토큰 적용 — 무채색 스케일 + 액센트 1개, 그라데이션·글로우·과한 그림자 없음 (FR-U01, FR-U03)
  - **다크 모드 기본** + 라이트 토글, 두 모드 모두 텍스트 대비 충분 (FR-U02)
  - Pagination이 첫/중간/마지막 페이지에서 모두 정상 — 페이지 번호·이전/다음·생략(…)·현재 페이지 강조·모바일 축약형. **0-base 응답을 표시 단계에서만 1-base로 변환** (FR-L05, PRD 8.1)
  - **`QueryProvider`가 루트 레이아웃에 마운트되고 `queryClient` 인스턴스에 접근 가능**(Phase 6의 FR-A11 캐시 비움, Phase 7의 FR-U05 낙관적 업데이트 전제)
  - API 클라이언트가 `NEXT_PUBLIC_API_BASE_URL` 환경변수를 사용하고 주소를 하드코딩하지 않음 (PRD 9.1)
  - **PRD 1.3 표의 TanStack Query·next-themes 설치 상태 갱신**

#### Phase 6 · 인증 화면
- **목표:** 로그인 흐름 완성
- **선행:** Phase 3(OAuth 교환 API), Phase 5(API 클라이언트·QueryProvider·공통 컴포넌트)
- **커버:** FR-A05, FR-A06, FR-A10, **FR-A11**(로그아웃), **FR-A01-1**(이름 1~50자 폼 검증)
- **설치:** **React Hook Form + Zod** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** `/login`, `/signup`, `/oauth/callback`, `lib/auth/token.ts`, `lib/schemas/`(Zod), useAuth, 라우트 가드
- **완료 조건:**
  - 가입 → 자동 로그인 → `/todos` 진입
  - 토큰을 **localStorage**에 저장하고 모든 요청에 `Authorization: Bearer` 첨부 (FR-A05)
  - 토큰 없이 보호 라우트 접근 시 `/login`으로 리다이렉트 (FR-A10)
  - 401 응답 시 토큰 삭제 + 로그인 화면 이동 + **만료 안내 표시** (FR-A06)
  - **로그아웃 시 토큰 삭제 + `/login` 이동 + React Query 캐시 비움**. 서버 API를 호출하지 않는다 (FR-A11)
  - Zod 스키마의 **비밀번호 6자 이상**, **이름 1~50자** 규칙이 서버 검증과 일치 (FR-A02, FR-A01-1)
  - `/oauth/callback`이 **JWT가 아닌 일회용 코드**를 받아 교환 API를 호출하고, 실패 시 로그인으로 이동 (FR-A08)
  - **PRD 1.3 표의 React Hook Form·Zod 설치 상태 갱신**

#### Phase 7 · 할 일 화면
- **목표:** 메인 기능 완성
- **선행:** Phase 6
- **커버:** FR-T01 ~ FR-T05, FR-L03 ~ FR-L06, FR-U04 ~ FR-U07
- **설치:** **Tiptap, Framer Motion** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** `/todos`, 생성·수정 다이얼로그, `TiptapEditor`, React Query 훅(`hooks/useTodos.ts`), 낙관적 업데이트
- **완료 조건:**
  - 제목(1~200자)·내용·마감일·우선순위(기본 MEDIUM) 입력과 CRUD 동작 (FR-T01, FR-T04)
  - Tiptap 툴바가 **8종(굵게·기울임·밑줄·취소선·목록·제목·인용·링크)으로 제한**되고 **이미지 버튼 없음**, 저장은 `editor.getHTML()` **HTML 문자열** (FR-T02, PRD 2장 비목표)
  - 토글이 **즉시 반영되고 실패 시 롤백 + 에러 토스트** (FR-T05, FR-U05)
  - 로딩(스켈레톤)·빈 상태·에러 상태 UI가 모두 존재 (FR-U04)
  - 필터·정렬·페이지 동시 동작, **필터 변경 시 첫 페이지로 이동** (FR-L06)
  - 모바일 1열 카드형 / 데스크톱 리스트형, 레이아웃 깨짐 없음 (FR-U07)
  - 애니메이션은 목록 진입·삭제·완료 토글 등 상태 변화에만 절제 사용 (FR-U06)
  - **PRD 1.3 표의 Tiptap·Framer Motion 설치 상태 갱신**
- **리스크:** 낙관적 업데이트의 쿼리 키. 필터·정렬·페이지를 키에 포함하지 않으면 목록이 어긋난다 (PRD 13장)

#### Phase 8 · 통합테스트와 로컬 검증
- **목표:** 핵심 기능의 회귀 방지선 확보
- **선행:** Phase 7
- **커버:** PRD **12.1 M1 게이트 인수 기준** 전부 / 비기능 — 테스트
- **산출물:**
  - `todolist_db_test` 스키마 (**로컬 PostgreSQL 직접 사용, Docker/Testcontainers 금지** — PRD 1.3 금지 사항)
  - 백엔드 통합테스트 — 가입/로그인·보호 API 401·Todo CRUD+페이지네이션·Soft Delete 제외·타인 리소스 404·HTML 정제·OAuth 코드 재사용 실패
  - 백엔드 검증 결과표
  - **프론트 로컬 검증 체크리스트** — 필터·정렬 갱신, 로딩/빈/에러 상태 UI, 낙관적 롤백, 다크/라이트 토글, 반응형
- **완료 조건:** 전체 테스트 통과 · PRD **12.1** 전부 통과
  - 12.1의 1~9·12~13·24는 **백엔드 통합테스트**로 검증
  - 12.1의 3·10~11·21~23은 **로컬 검증 체크리스트**로 확인 (**프론트 자동화 테스트는 PRD 9장 기준 범위 밖 — E2E 도구를 새로 도입하지 않는다**)
- **🚩 M1 게이트:** 여기서 한 번 멈추고 실제로 써본다. UX 문제는 확장 기능 전에 고치는 편이 싸다

---

### M2 — 확장 기능

> 세 Phase는 서로 의존하지 않으므로 **순서를 바꿔도 된다.** 아래는 난이도가 낮은 순서다.

#### Phase 9 · 비밀번호 재설정
- **선행:** Phase 8
- **커버:** FR-R01 ~ FR-R07
- **설치:** **spring-boot-starter-mail** (PRD 1.3 ❌ → 설치 후 표 갱신)
- **산출물:** `password_reset_tokens` 엔티티·테이블(`(token)` 인덱스 포함), `MailSender` 인터페이스 + **로그 출력 구현체**(`APP_MAIL_TYPE=log`), `POST /api/auth/password/forgot`·`/reset`, `/forgot-password`·`/reset-password` 화면
- **완료 조건:**
  - 로그에 출력된 링크로 비밀번호 변경 후 로그인 성공 (FR-R01)
  - 토큰이 **30분 만료·1회용·DB에는 해시로 저장** (FR-R02)
  - 미가입 이메일·**소셜 전용 계정**도 동일한 성공 응답 (FR-R03)
  - 새 비밀번호 6자 이상 + 확인 입력 일치 (FR-R04)
  - 재설정 성공 시 **해당 사용자의 미사용 토큰 전부 무효화** (FR-R05)
  - 만료·재사용·위조 토큰이 **동일 메시지**로 처리 (FR-R06)
  - 1분 내 재요청 차단 (FR-R07, 권장)
  - `password_reset_tokens`에 `deleted_at`·`updated_at`을 두지 않음 (PRD 7장 예외)
  - `APP_PASSWORD_RESET_URL`이 환경변수로 주입됨 (PRD 9.1)
  - 통합테스트: 토큰 재사용 실패, 미가입 이메일 동일 응답 (PRD 12.2 항목 14·15)
  - **PRD 1.3 표의 spring-boot-starter-mail 설치 상태 갱신**

#### Phase 10 · 파일 첨부
- **선행:** Phase 8
- **커버:** FR-F01 ~ FR-F08
- **산출물:** `attachments` 엔티티·테이블(`(todo_id, deleted_at)` 인덱스 포함), `FileStorage` 인터페이스 + `LocalFileStorage`, 첨부 API **5종**(업로드/목록/다운로드URL 발급/**`GET /api/attachments/{id}/download`**/삭제 — PRD 8장), 첨부 UI
- **완료 조건:**
  - 할 일당 **최대 5개, 개당 최대 10MB** 서버 검증 (FR-F01)
  - 허용 형식 9종(jpg/png/webp/gif/pdf/docx/xlsx/txt/zip)을 **확장자와 Content-Type 모두** 검증 (FR-F02)
  - 저장 키는 UUID 기반, 원본 파일명은 DB에만 보관 (FR-F04)
  - **`FileStorage.generateDownloadUrl(storedKey, ttl)` 계약**으로 통일 — 로컬 구현체는 **`GET /api/attachments/{id}/download?token=...` URL**을 반환하고, 저장소 원본 경로에 직접 접근할 수 없음 (FR-F05, PRD 8장)
  - `/download`가 **서명 토큰만으로 인증**되고(헤더 불필요), 토큰 없음·위조·만료 시 **404** (PRD 8장)
  - **프론트는 `download-url`이 준 URL을 그대로 열기만 한다** — 저장소 종류를 분기하지 않으며, Phase 12-3 S3 전환 시 프론트 코드 무변경 (PRD 9장 이식성)
  - 첨부 삭제는 **Soft Delete**, 실제 파일은 지우지 않음 (FR-F06)
  - `APP_STORAGE_TYPE=local|s3`로 구현체 전환, **`APP_UPLOAD_DIR`·`APP_DOWNLOAD_URL_TTL_SECONDS`를 환경변수로 주입**(저장 경로·TTL 하드코딩 금지) (FR-F03, PRD 9.1)
  - 업로드 UI가 드래그앤드롭·파일 선택·진행 상태·파일명·크기 표시, 이미지 썸네일 (FR-F07)
  - 목록에 클립 아이콘 + 개수 표시 (FR-F08, 권장)
  - **타인 할 일에 첨부 업로드 시 404** (FR-T07, PRD 12.2 항목 17)
  - 통합테스트: 용량·형식·개수 제한, 타인 할 일 404 (PRD 12.2 항목 16·17)
  - `attachments`에 `updated_at`을 두지 않음 (PRD 7장 예외)
- **주의:** `S3FileStorage`는 여기서 만들지 않는다. Phase 12-3에서 인터페이스에 끼운다

#### Phase 11 · 관리자 페이지
- **선행:** Phase 8
- **커버:** FR-M01 ~ FR-M10
- **산출물:** `role`·`enabled` 컬럼, 권한 검사(SecurityConfig + `@PreAuthorize` **이중**), 관리자 API 4종(PRD 8장), `/admin` 화면(통계 카드·사용자 탭·전체 할 일 탭), 최초 관리자 지정 SQL(`db/init.sql`)
- **완료 조건:**
  - JWT에 권한 정보 포함 (FR-M01)
  - **USER 토큰으로 `/api/admin/**` 접근 시 403, 토큰 없이 접근 시 401** (FR-M02, FR-T07 예외 규칙, PRD 12.2 항목 18)
  - 최초 관리자는 **SQL로만** 지정 — 회원가입 등 일반 경로로 ADMIN 승격 불가 (FR-M03)
  - 사용자 목록 페이지네이션 + 이메일 검색(`email` 파라미터) (FR-M04)
  - 계정 활성/비활성 전환, **자기 자신 비활성화 불가** (FR-M05, PRD 12.2 항목 20)
  - 비활성화 계정의 로그인과 **기존 토큰 사용이 모두 차단** — 구현은 Phase 2의 FR-A04-1에 의존 (FR-M06, PRD 12.2 항목 19)
  - 전체 할 일 조회를 **`JOIN` + DTO 프로젝션**으로 처리해 N+1 방지, 연관 매핑 탐색 금지 (FR-M07, PRD 7장 조회 규칙·13장 리스크)
  - 요약 통계 4종(전체 사용자·활성 사용자·전체 할 일·완료 수) (FR-M08)
  - 응답에 비밀번호 해시 등 민감정보 없음 (FR-M09)
  - USER가 `/admin` 직접 접근 시 접근 불가 안내 화면 (FR-M10)
- **리스크:** 권한 검사가 프론트 가드에만 걸리는 실수. **프론트 가드는 보안 수단으로 인정하지 않으며** 통합테스트로 403/401을 반드시 확인 (FR-M02, PRD 13장)
- **🚩 M2 게이트:** PRD **12.1 + 12.2 인수 기준 전부(24개)** 통과. M1에서 통과한 항목도 확장 기능 추가로 깨지지 않았는지 재확인한다
- **결정 과제:** 여기서 **OPEN-01~04를 모두 결정**한다 (PRD 14장). **Amplify의 Next.js 16 지원 여부를 공식 문서로 확인**해 OPEN-01을 정하며, 미결정 상태로는 Phase 12에 착수하지 않는다

---

### M3 — 운영 배포

#### Phase 12 · AWS 이전
| 단계 | 내용 | 완료 조건 |
|---|---|---|
| 12-1 | 배포 준비 점검 (하드코딩·시크릿·빌드·권한) | **PRD 9.1 환경변수 전 항목**이 외부 주입이며 소스 하드코딩 0건(파일 저장소 4종 `APP_UPLOAD_DIR`·`APP_S3_BUCKET`·`APP_S3_REGION`·`APP_DOWNLOAD_URL_TTL_SECONDS` 포함) · 시크릿 파일이 커밋 이력에 없음 |
| 12-2 | RDS PostgreSQL 연결 | prod 프로파일로 기동. **`ddl-auto: validate` 통과** — 로컬 `update`로 만들어진 스키마를 RDS에 반영한 뒤 검증한다(Phase 1 DDL 전략) |
| 12-3 | S3·SES 구현체 전환 | `APP_STORAGE_TYPE=s3`·`APP_MAIL_TYPE=ses`로 **환경변수만 바꿔 동작 전환**, 기존 서비스 코드·**프론트 코드 무변경**. `download-url`이 presigned URL을 반환하고 **`/api/attachments/{id}/download`는 더 이상 호출되지 않음**(PRD 8장), `APP_S3_BUCKET`·`APP_S3_REGION` 주입, **버킷 퍼블릭 액세스 차단** |
| 12-4 | EC2 백엔드 배포 (systemd + Nginx) | `/actuator/health` 200, 재부팅 후 자동 기동, HTTPS 적용(OPEN-02 결정안) |
| 12-5 | Amplify 프론트 배포 | 운영 도메인 접속, `APP_CORS_ALLOWED_ORIGINS`·`NEXT_PUBLIC_API_BASE_URL`·`APP_PASSWORD_RESET_URL`·구글 리다이렉트 URI 갱신 (OPEN-03 연동) |
| 12-6 | 최종 운영 점검 | **운영에서 Swagger 차단** · CORS 와일드카드 없음 · RDS 자동 백업 활성화 · S3 퍼블릭 차단 · 로그 표준출력 |

- **선행:** Phase 11 완료 + **OPEN-01~04 결정 완료**
- **설치:** **AWS SDK v2 (`s3`)** — 12-3에서 설치 (PRD 1.3 ❌ → 설치 후 표 갱신)
- **선결 과제:** Amplify의 Next.js 16 지원 여부 확인 결과(OPEN-01)에 따라 12-5를 정적 export 또는 EC2 `next start`로 대체
- **커버:** FR-F03·FR-F05(S3 구현체), FR-R01(SES 구현체), 비기능 — 이식성·보안·운영

---

## 4. 요구사항 추적 매트릭스

> **표기 규칙:** `~`는 연속 범위이며, `A01-1`처럼 하이픈이 붙은 **서브 ID는 범위에 포함되지 않으므로 괄호 안에 개별 명시**한다.

| 요구사항 | 담당 Phase |
|---|---|
| FR-A01 ~ A06 (**A01-1, A04-1 포함**), A10, A11 (인증·토큰) | 2, 6 |
| FR-A07 ~ A09 (구글 OAuth2) | 3, 6 |
| FR-T01 ~ T07 (할 일) | 1, 4, 7 |
| FR-L01 ~ L06 (목록·페이지네이션) | 4, 5, 7 |
| FR-U01 ~ U08 (UI/UX) | 4(U08), 5, 7 |
| FR-R01 ~ R07 (비밀번호 재설정) | 9, 12-3(SES 전환) |
| FR-F01 ~ F08 (파일 첨부) | 10, 12-3(S3 전환) |
| FR-M01 ~ M10 (관리자) | 2(A04-1 전제), 11 |
| PRD 7장 — 데이터 모델·인덱스·Soft Delete 규약 | 1, 9, 10, 11 |
| PRD 8.1 — `ApiResponse`/`PageResponse`/평면 에러 | 2, 4 |
| 비기능 — 이식성 (9.1 환경변수) | 0, 5, 12-1 |
| 비기능 — 보안 | 2, 3, 4, 10, 11, 12-6 |
| 비기능 — 테스트 (백엔드 통합테스트, 프론트 자동화 제외) | 8, 9, 10, 11 |
| PRD 12.1 인수 기준(1~13, 21~24) | 8 (M1 게이트) |
| PRD 12.2 인수 기준(14~20) | 9(14·15), 10(16·17), 11(18·19·20) — M2 게이트에서 24개 전부 재확인 |
| PRD 14장 OPEN-01 ~ 04 | 11(결정) → 12(적용) |

---

## 5. 진행 체크리스트

**M1 — 핵심 MVP**
- [ ] Phase 0 · 스캐폴딩 (**중첩 `.git` 정리 · 프로파일 분리 포함**)
- [ ] Phase 1 · 엔티티와 Repository (**DDL 전략 확정**)
- [ ] Phase 2 · 인증 (**springdoc 설치 · 에러 코드 체계 확정**)
- [ ] Phase 3 · 구글 OAuth2
- [ ] Phase 4 · Todo API (**jsoup 설치**)
- [ ] Phase 5 · 프론트 토대 (**TanStack Query · next-themes 설치**)
- [ ] Phase 6 · 인증 화면 (**React Hook Form · Zod 설치**)
- [ ] Phase 7 · 할 일 화면 (**Tiptap · Framer Motion 설치**)
- [ ] Phase 8 · 통합테스트 · **M1 게이트 (PRD 12.1)**

**M2 — 확장 기능**
- [ ] Phase 9 · 비밀번호 재설정 (**spring-boot-starter-mail 설치**)
- [ ] Phase 10 · 파일 첨부
- [ ] Phase 11 · 관리자 페이지 · **M2 게이트 (PRD 12.1 + 12.2, 24개)**
- [ ] OPEN-01~04 결정 (Phase 11 종료 시점)

**M3 — 운영 배포**
- [ ] 12-1 배포 준비 점검
- [ ] 12-2 RDS (**`ddl-auto: validate` 전환**)
- [ ] 12-3 S3 · SES (**AWS SDK v2 설치**)
- [ ] 12-4 EC2
- [ ] 12-5 Amplify
- [ ] 12-6 운영 점검

**문서 동기화 (상시)**
- [ ] 라이브러리 설치 시마다 **PRD 1.3 표의 설치 상태 갱신** (진행 원칙 6)

---

## 6. 형상 관리

- **저장소는 루트 `todo-project` 단일 git 저장소 하나다.** `todo-backend`·`todo-frontend`는 하위 디렉터리일 뿐이며 **각자의 `.git`을 두지 않는다**(서브모듈·중첩 저장소 금지). PRD 1.3의 모노레포 전제와 일치시키기 위함이다.
  - ⚠️ **현재 `todo-backend/.git`·`todo-frontend/.git`이 독립 저장소로 존재한다.** Phase 0에서 이를 제거하고 루트 저장소로 통합해야 아래 브랜치·태그 전략과 진행 원칙 2가 성립한다. (`todo-frontend`는 커밋 1개, `todo-backend`는 커밋 0개이므로 이력 손실은 사실상 없다)
- 브랜치: `main`(항상 동작하는 상태) + Phase별 작업 브랜치 `feat/phase-N-요약`
- Phase 완료 조건을 통과하면 `main`에 병합하고 태그를 남긴다 (`phase-4-todo-api` 형식)
- 커밋 메시지는 한글, `[Phase 4] Todo CRUD API 구현` 형태
- 시크릿이 담긴 파일(`.env*`, `application-local.yml`, 업로드 디렉터리)은 절대 커밋하지 않는다 (PRD 9.1)

---

## 7. 일정

각 Phase의 예상·실제 소요는 진행하면서 채운다. 처음 몇 Phase를 진행한 뒤 실제 속도를 보고 이후 일정을 잡는 편이 정확하다.

| Phase | 예상 | 실제 | 비고 |
|---|---|---|---|
| 0 | | | 중첩 `.git` 정리 포함 |
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
| 6 | | | |
| 7 | | | |
| 8 | | | M1 게이트 |
| 9 | | | |
| 10 | | | |
| 11 | | | M2 게이트 · OPEN 결정 |
| 12 | | | |

---

## 8. 변경 이력

| 버전 | 일자 | 내용 |
|---|---|---|
| 1.0 | 2026-08-26 | 최초 작성. PRD v1.0 기준, Phase 0~12 정의 |
| 1.1 | 2026-08-27 | **PRD v1.2 검증 반영.** ① 존재하지 않는 `CLAUDE.md`·`claude-code-프롬프트.md` 참조 3곳 제거(헤더 2곳, Phase 0 완료 조건) ② Phase 0 완료 조건을 `docs/guides/project-structure.md` 기준으로 교체 ③ Phase 2에 FR-A04-1 커버·완료 조건 추가, springdoc 3.1.0 주의사항 명시 ④ Phase 5에 TanStack Query·next-themes 설치와 `QueryProvider` 마운트 완료 조건 추가 ⑤ 기준 문서를 PRD v1.2로 갱신 |
| 1.2 | 2026-08-27 | **재검증 회귀 수정.** ① 게이트 참조를 **번호 범위에서 PRD 소절 이름으로 교체** — Phase 8은 "PRD 12.1 전부", M2 게이트는 "12.1 + 12.2 전부(24개)". 인수 기준이 늘어도 게이트 정의가 어긋나지 않는다 ② Phase 8 산출물에 프론트 로컬 검증 체크리스트 추가(인수 기준 21~23 검증 경로 확보) ③ 추적 매트릭스에 **서브 ID 표기 규칙** 신설, FR-A01-1·A04-1·A11 누락 보정 ④ Phase 6 커버에 FR-A11·FR-A01-1 추가 및 로그아웃 완료 조건 명시 ⑤ 기준 문서를 PRD v1.3으로 갱신 |
| 1.3 | 2026-08-28 | **코드베이스 실사 기반 개정.** ① **중첩 `.git` 문제 명시** — `todo-backend/.git`·`todo-frontend/.git`이 독립 저장소로 존재해 6장 형상 관리(단일 `main` + Phase 브랜치·태그)와 진행 원칙 2가 성립하지 않는 문제를 6장과 Phase 0에 기록하고, **저장소 통합을 Phase 0 산출물·완료 조건으로 추가** ② **Phase 0 실사 반영** — 루트 `.gitignore`·`README.md` 부재, 커밋 0개, `application.properties` 단일 파일 상태를 "현재 상태(미완료)"로 명기하고, PRD 9.1에 맞춰 **`application.yml`/`-local.yml`/`-prod.yml` 프로파일 분리**를 산출물로 추가 ③ **Phase 1에 DDL 생성 전략 신설** — `db/init.sql`이 실재하지 않고 코드는 `ddl-auto=update`인 불일치를 해소. 로컬은 `update`, PRD 7장 인덱스 4종·unique는 `@Table(indexes=...)`로 선언, `db/init.sql`은 스키마 생성·관리자 지정 SQL 전용, **`validate` 전환은 Phase 12-2**로 못박음 ④ **미설치 라이브러리 설치 단계를 전 Phase에 명시** — Phase 2 springdoc-openapi, Phase 4 jsoup, Phase 6 React Hook Form+Zod, Phase 7 Tiptap·Framer Motion, Phase 9 spring-boot-starter-mail, Phase 12-3 AWS SDK v2(s3) 누락 보정 ⑤ **진행 원칙 6 신설** — 설치 후 **PRD 1.3 표 갱신**을 완료 조건화(PRD 1.3 양방향 동기화 의무) ⑥ **진행 원칙 7 신설** — `/tasks/XXX-description.md` 작업 파일 규약 명문화, Phase 0에 `/tasks/` 디렉터리 생성 추가 ⑦ **완료 조건을 측정 가능한 형태로 재작성** — 각 Phase에 FR ID를 붙이고 수치·응답 형식(6자·24h·10건·0-base·404/403/409·`ApiResponse`/`PageResponse`/평면 에러)을 명시 ⑧ **Phase 2에 에러 코드 체계 확정 과제 추가**(PRD 8.1 위임 사항이 어느 Phase에도 없던 누락 보정) ⑨ **Phase 4에 FR-T03 정제 범위 구체화** — `<img>` 제거·`Safelist` 8종 한정·`preserveRelativeLinks` 금지(PRD 1.3·2장 비목표) ⑩ **Phase 5 커버에 FR-U02 다크 기본, Phase 7에 Tiptap 툴바 8종·이미지 버튼 금지** 명시 ⑪ **Phase 8 검증 경로 이원화 명문화** — 12.1 항목별로 통합테스트 담당분과 로컬 체크리스트 담당분을 분리하고 **E2E 도구 신규 도입 금지**(PRD 9장 테스트) 명시 ⑫ **Phase 12 각 단계 완료 조건을 PRD 9장·9.1 기준 검증 가능 항목으로 교체**(모호한 "점검 10항목/13항목" 제거) ⑬ **추적 매트릭스 보강** — FR-U08(Phase 4), FR-R01/F03·F05의 12-3 전환, FR-M06의 Phase 2 전제, PRD 7장·8.1·12.1·12.2·14장 행 추가 ⑭ **진행 체크리스트에 설치·확정 과제 표시**와 문서 동기화 상시 항목 추가 ⑮ Phase 11에 OPEN-01~04 결정 과제를 별도 항목으로 분리(PRD 13·14장) |
| 1.4 | 2026-08-28 | **PRD v1.4 반영 동기화.** ① 기준 문서를 PRD v1.4로 갱신 ② **Phase 3** — FR-A09 완료 조건에 소셜 로그인 조회 키가 `email`이며 `provider_id`가 아님을 명시(PRD 7장 `users` 신설 규정) ③ **Phase 10** — 첨부 API를 4종에서 **5종**으로 정정(PRD 8장에 신설된 `GET /api/attachments/{id}/download` 반영), `/download`가 서명 토큰만으로 인증되고 실패 시 404라는 조건 추가, **프론트가 저장소 종류를 분기하지 않는다**는 이식성 조건 추가, `APP_UPLOAD_DIR`·`APP_DOWNLOAD_URL_TTL_SECONDS` 주입 조건 추가(PRD 9.1 신설분) ④ **Phase 12-1** — 점검 대상을 "환경변수 8종"에서 **"9.1 전 항목"**으로 교체하고 파일 저장소 4종을 예시로 명시(PRD 9.1이 12종으로 늘어난 것 반영) ⑤ **Phase 12-3** — S3 전환 시 `/download`가 호출되지 않는다는 동작 변화와 프론트 코드 무변경, `APP_S3_BUCKET`·`APP_S3_REGION` 주입을 완료 조건에 추가 |
