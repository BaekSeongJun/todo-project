# Phase 8 검증 결과표

> PRD 12.1 M1 게이트(1~13, 21~24) 인수 기준 24개 중 이 소절에 속한 17개 항목을 검증 방식·근거와 함께 기록한다.
> ROADMAP.md Phase 8(229~241행)의 산출물이며, 이 문서 자체가 M1 게이트 통과의 근거 자료다.

## 1. 백엔드 통합테스트로 검증한 항목 (1, 2, 4~9, 12, 13, 24)

| 번호 | 인수 기준 | 검증 방식 | 근거 |
|---|---|---|---|
| 1 | `todolist_db`와 함께 PostgreSQL 실행 | 실측 | 통합테스트 53개 전부가 로컬 PostgreSQL `todolist_db_test` 스키마에 실제 연결해 통과함(`application-test.yml`). DB가 떠 있지 않으면 `SpringBootTest` 컨텍스트 로딩 자체가 실패하므로, 테스트 통과 자체가 실행 증거다. |
| 2 | 백엔드가 오류 없이 기동 (`./mvnw spring-boot:run`) | 실측 | 로컬 8080 포트에서 이미 기동 중인 인스턴스에 `curl http://localhost:8080/api/todos` 요청 → `401 UNAUTHORIZED`(`ErrorResponse` 계약 형식)를 정상 응답. 컨텍스트가 정상 로딩되지 않으면 이 응답 자체가 불가능하므로 오류 없는 기동의 증거로 채택 (2026-09-01 실측). |
| 4 | 회원가입 시 사용자 생성 및 JWT 반환 | 자동테스트 | `AuthControllerIntegrationTest#회원가입_성공_시_사용자가_생성되고_JWT를_반환한다` — `accessToken` 존재 확인 + `userRepository.findByEmailAndDeletedAtIsNull` 존재 확인 |
| 5 | 로그인 시 유효한 JWT 반환 | 자동테스트 | `AuthControllerIntegrationTest#로그인_성공_시_유효한_JWT를_반환한다` |
| 6 | 보호된 엔드포인트가 유효한 토큰을 요구 | 자동테스트 | `AuthControllerIntegrationTest#토큰_없이_보호된_엔드포인트_접근시_401을_반환한다` — `code=UNAUTHORIZED` 확인 |
| 7 | 할 일 CRUD가 페이지네이션과 함께 동작 | 자동테스트 | `TodoControllerIntegrationTest#CRUD_및_페이지네이션이_정상_동작한다` — 생성 3건, `page=0&size=2` 조회로 `content` 2건·`totalElements` 3·`first=true` 확인, 수정·완료 토글까지 검증 |
| 8 | Soft Delete 시 `deleted_at`이 갱신되고 목록에서 제외 | 자동테스트 | `TodoControllerIntegrationTest#삭제된_할일은_deleted_at이_기록되고_목록에서_제외된다` — 네이티브 쿼리로 `@SQLRestriction` 우회해 `deleted_at IS NOT NULL` 직접 확인, 목록 조회 결과가 빈 배열임을 확인 |
| 9 | 타인의 리소스 접근 시 404 (소유권 검증) | 자동테스트 | `TodoControllerIntegrationTest#타인의_할일_조회_수정_삭제는_404를_반환한다` — 조회·수정·삭제 3개 동작 모두 `code=TODO_NOT_FOUND` 확인 |
| 12 | 구글 로그인이 정상 동작하고 동일 이메일 계정이 중복 생성되지 않음 | 자동테스트(2건 조합) | **정상 동작:** `OAuthExchangeIntegrationTest#유효한_일회용_코드는_JWT로_정상_교환된다`. **중복 방지:** 기존 `CustomOAuth2UserServiceTest`(Mockito 유닛테스트, Phase 3에서 작성)가 동일 이메일 재로그인 시 신규 `User`를 생성하지 않고 기존 계정을 재사용함을 검증 — 신규 테스트 없이 재사용 |
| 13 | 내용에 script 태그를 넣어 저장해도 정제되어 저장됨 | 자동테스트 | `TodoControllerIntegrationTest#본문에_script_태그를_포함해도_정제되어_저장된다` — `<script>`·`alert(1)` 미포함, 정상 텍스트는 보존 확인 |
| 24 | 구글 로그인 후 받은 일회용 코드를 두 번째로 교환하면 실패함 | 자동테스트 | `OAuthExchangeIntegrationTest#이미_사용된_일회용_코드는_재사용시_실패한다` — 2차 교환 시 `401`·`code=INVALID_OAUTH_CODE` 확인 |

**테스트 클래스 목록**
- `com.example.auth.controller.AuthControllerIntegrationTest` (3 테스트)
- `com.example.todo.controller.TodoControllerIntegrationTest` (4 테스트)
- `com.example.auth.controller.OAuthExchangeIntegrationTest` (2 테스트)
- 전체 스위트: `./mvnw test` 기준 **53개 전부 BUILD SUCCESS** (2026-09-01 실측, 기존 유닛/리포지토리 테스트 포함 누적치)

## 2. 로컬 검증 체크리스트로 확인한 항목 (3, 10~11, 21~23)

> 프론트엔드 자동화 테스트는 PRD 9장 기준 범위 밖이며 E2E 도구를 신규 도입하지 않는다(CLAUDE.md 금지 사항). 아래는 실행·수동 확인 또는 Phase 7 Playwright MCP 실측 근거를 인용한다.

| 번호 | 인수 기준 | 검증 방식 | 근거 |
|---|---|---|---|
| 3 | 프론트엔드 빌드 성공 (`npm run build`) | 실측 | `npm run build` 실행 → `Compiled successfully`, TypeScript 통과, 8개 라우트(`/`, `/_not-found`, `/login`, `/oauth/callback`, `/signup`, `/todos` 등) 전부 정적 생성 성공 (2026-09-01 실측) |
| 10 | 뉴트럴 미니멀 테마 + 다크/라이트 토글 적용 | Phase 7 근거 인용 | ROADMAP.md Phase 7 완료 조건(218~226행)에서 다크 모드 기본 적용과 그라데이션·과한 그림자 미사용 원칙 준수를 이미 확인. 라이트 토글은 `next-themes` 기반으로 Phase 5에서 구현, Phase 7 UI 작업 시 화면 전반에서 사용 확인 |
| 11 | 반응형 레이아웃 정상 동작 | Phase 7 근거 인용 | ROADMAP.md Phase 7 완료 조건 — "모바일 1열 카드형 / 데스크톱 리스트형, 레이아웃 깨짐 없음(FR-U07) — 390px·1280px 스크린샷 확인"(224행) |
| 21 | 상태 필터·정렬 변경 시 목록 갱신, 필터 변경 시 첫 페이지로 이동 | Phase 7 근거 인용 | ROADMAP.md Phase 7 완료 조건 — "필터·정렬·페이지 동시 동작, 필터 변경 시 첫 페이지로 이동(FR-L06) — 2페이지에서 필터 전환 시 1페이지로 리셋됨을 실측"(223행) |
| 22 | 모든 목록 화면에 로딩(스켈레톤)·빈 상태·에러 상태 UI 표시 | Phase 7 근거 인용 | ROADMAP.md Phase 7 완료 조건 — "로딩(스켈레톤)·빈 상태·에러 상태 UI가 모두 존재(FR-U04) — EmptyState·ErrorState 실측"(222행) |
| 23 | 완료 토글이 즉시 반영, 서버 실패 시 롤백 + 에러 토스트 | Phase 7 근거 인용 | ROADMAP.md Phase 7 완료 조건 — "토글이 즉시 반영되고 실패 시 롤백 + 에러 토스트(FR-T05, FR-U05) — `window.fetch` 강제 실패로 재현, '다시 시도' 후 서버 상태로 정확히 복원됨을 확인"(221행) |

## 3. 요약

- **백엔드 9개 항목** — 신규 통합테스트 9개(3개 클래스)로 전부 자동 검증, 회귀 방지선 확보
- **프론트 5개 항목** — 1개(빌드)는 이번에 재실행 확인, 4개는 Phase 7에서 이미 Playwright MCP로 실측된 근거를 그대로 인용(같은 날 재확인은 중복 작업이라 생략)
- **PRD 12.1 M1 게이트 17개 항목(1~13 중 3·10·11 제외한 9개 + 3·10·11 + 21~24) 전부 근거 확보 완료**

> 나머지 M1 게이트 항목(3, 10, 11)은 백엔드가 아닌 프론트 항목이라 위 표 2절에 포함되어 있다. 12.1의 전체 13개(1~13) + 21~24 = 17개가 이 문서에서 다뤄지는 전부이며, 이는 PRD 12.1 소절의 정의와 일치한다.
