# PRD — Todo List 서비스

| 항목 | 내용 |
|---|---|
| 문서 버전 | 1.5 |
| 작성일 | 2026-08-26 |
| 최종 개정일 | 2026-08-31 |
| 상태 | 개발 착수 전 확정본 |
| 관련 문서 | `ROADMAP.md`(실행 순서와 Phase 정의), `docs/guides/`(프론트 개발 가이드) |

> 이 문서는 **무엇을 만들 것인가**를 정의하며, 기술 스택·데이터 모델·API 계약의 **단일 출처(SSOT)**다.
> 어떤 문서(`ROADMAP.md`, `docs/guides/`)든 이 PRD와 어긋나면 **PRD를 따른다.**
> **어떤 순서로 만들지**는 `ROADMAP.md`가, **프론트 구현 패턴**은 `docs/guides/`가 담당한다.

---

## 1. 제품 개요

### 1.1 한 줄 정의

개인 사용자가 자신의 할 일을 생성·정리·완료 처리할 수 있는 웹 기반 Todo 관리 서비스.

### 1.2 배경과 개발 방식

**배경:** 이메일 또는 구글 계정으로 로그인해 자신의 할 일만 안전하게 관리하는 것을 목표로 한다. 팀 협업, 공유, 알림 같은 기능은 이번 범위에 넣지 않고 개인 사용자의 기본 흐름을 완결성 있게 구현하는 데 집중한다.

**개발 방식:** Claude Code를 활용한 바이브 코딩. 로컬 환경에서 프론트엔드와 백엔드를 모두 개발한 뒤, 완성 후 AWS로 이전한다.

### 1.3 기술 스택

> **이 절이 스택 사실의 단일 출처(SSOT)다.** 버전과 설치 상태가 다른 문서와 어긋나면 이 표를 따른다.
> **⚠️ "설치 상태"가 ❌인 라이브러리는 `import` 하지 않는다.** 해당 Phase에서 설치한 뒤 사용하며, 설치 시 이 표를 함께 갱신한다.
> **⚠️ 양방향 동기화 의무.** 이 표에 없으나 `pom.xml`·`package.json`에 존재하는 의존성을 발견하면 **이 표를 즉시 갱신한다.** 표와 실물이 어긋난 상태를 방치하지 않는다.
> **"설치 상태"는 의존성 선언 여부만을 뜻한다.** ✅는 `pom.xml`·`package.json`에 이미 있다는 의미이며, **코드에서 이미 쓰고 있다는 뜻이 아니다.** 따라서 ✅ 항목은 어느 Phase에서도 설치 단계가 필요 없고, ❌ 항목만 비고에 적힌 Phase에서 설치한 뒤 이 표를 ✅로 갱신한다.

#### 백엔드

| 항목 | 버전 | 설치 상태 | 비고 |
|---|---|---|---|
| Spring Boot | 4.1.1 | ✅ | 스타터명이 `-webmvc` (구 `-web` 아님) |
| JDK | 21 | ✅ | |
| Maven | Wrapper (`./mvnw`) | ✅ | 베이스 패키지 `com.example` |
| Spring Security | 7.x | ✅ | Boot 4 BOM 관리. 구 API(`antMatchers` 등) 금지 |
| OAuth2 Client | Boot 4 BOM | ✅ | `spring-boot-starter-security-oauth2-client`. 구글 로그인 (FR-A07) |
| Spring Data JPA / Hibernate | Boot 4 BOM | ✅ | |
| PostgreSQL Driver | Boot 4 BOM | ✅ | |
| jjwt (JWT) | 0.12.6 | ✅ | **설치 완료 — Phase 2에서 설치 단계 불필요.** 코드 사용은 Phase 2부터 시작한다 |
| Bean Validation | Boot 4 BOM | ✅ | |
| Lombok | Boot 4 BOM | ✅ | annotation processor 설정 완료. 엔티티·DTO 보일러플레이트 축소 |
| Spring Boot DevTools | Boot 4 BOM | ✅ | 로컬 전용(optional). 운영 빌드에 포함하지 않는다 |
| springdoc-openapi | 3.1.0 | ✅ | **설치 완료(Phase 2).** Maven Central에서 3.1.0 확인 후 설치. 로컬 기동 시 Swagger UI 정상 표시, prod 프로파일에서는 비활성화(404) 확인 완료 |
| jsoup | 1.18.3 | ✅ | **설치 완료(Phase 4).** Maven Central에서 1.18.3 확인 후 설치. HTML 정제 (FR-T03). `Safelist`는 FR-T02 툴바 8종으로 한정. **`preserveRelativeLinks` 활성화 금지**(`javascript:` 우회 취약점) 준수 확인 |
| spring-boot-starter-mail | Boot 4 BOM | ❌ Phase 9 | 메일 발송 (FR-R01). 로컬은 로그 출력 구현체 사용 |
| AWS SDK v2 (`s3`) | 2.x | ❌ Phase 12 | S3 저장소·presigned URL (FR-F03, FR-F05) |

#### 프론트엔드

| 항목 | 버전 | 설치 상태 | 비고 |
|---|---|---|---|
| Next.js | 16.3.3 | ✅ | App Router. `next lint` 제거됨 |
| React / React DOM | 19.2.8 | ✅ | |
| TypeScript | ^5 | ✅ | |
| Tailwind CSS | ^4 | ✅ | CSS-first (`@tailwindcss/postcss`) |
| shadcn/ui (CLI) | ^4.19.0 | ✅ | style `radix-nova`, baseColor `neutral` |
| radix-ui | ^1.6.7 | ✅ | 통합 패키지 |
| lucide-react | ^1.34.0 | ✅ | |
| shadcn 부속 유틸 | — | ✅ | `clsx`, `tailwind-merge`, `class-variance-authority`, `tw-animate-css`. shadcn/ui 설치 시 함께 들어옴 |
| ESLint / Prettier / husky / lint-staged / commitlint | — | ✅ | 코드 품질 도구 |
| TanStack Query | ^5.102.8 | ✅ | **설치 완료(Phase 5).** 서버 상태 관리 |
| next-themes | ^0.4.6 | ✅ | **설치 완료(Phase 5).** 다크/라이트 토글 (FR-U02) |
| React Hook Form + Zod | — | ❌ Phase 6 | 폼 검증 |
| Tiptap | — | ❌ Phase 7 | 리치 텍스트 에디터 (FR-T02) |
| Framer Motion | — | ❌ Phase 7 | 상태 변화 애니메이션 (FR-U06) |

#### 데이터베이스 · 인프라

| 항목 | 값 | 비고 |
|---|---|---|
| PostgreSQL | 로컬 설치 | 스키마 `todolist_db`, 테스트 `todolist_db_test` |
| Node.js | 20+ | 프론트 빌드 |
| 배포 (예정) | Amplify(FE) · EC2(BE) · RDS(DB) · S3(파일) · SES(메일) | Phase 12 |

#### 금지 사항

- **Docker 사용 금지.** 통합테스트는 로컬 PostgreSQL을 직접 사용한다 (Testcontainers 미사용).
- **Refresh Token 도입 금지.** Access Token 단독 운영 (FR-A04).
- **위 표에 없는 프레임워크·BaaS로 대체 금지.** 스택 변경이 필요하면 이 절을 먼저 개정한다.

---

## 2. 목표와 비목표

### 목표
- 이메일 회원가입/로그인과 구글 소셜 로그인을 지원하는 안전한 인증 체계
- 비밀번호를 잊은 사용자가 스스로 복구할 수 있는 재설정 흐름
- 할 일 생성·조회·수정·완료·삭제의 전 과정이 매끄럽게 동작
- 할 일에 참고 자료를 파일로 첨부
- 목록의 페이지네이션·필터·정렬
- 삭제 데이터의 복구 가능성 확보(Soft Delete)
- 관리자가 사용자와 전체 현황을 파악하고 문제 계정을 차단할 수 있는 관리 기능
- 로컬에서 완성 후 코드 수정 없이 AWS로 이전 가능한 구조

### 비목표 (이번 범위에서 제외)
- 팀/조직 기능, 할 일 공유 및 협업
- 이메일·푸시 알림, 리마인더 (비밀번호 재설정 메일은 예외)
- **반복 일정** (매일/매주 자동 생성)
- 본문(Tiptap) 내 이미지 삽입 — 첨부파일은 별도 영역으로만 지원한다. 에디터 툴바에 이미지 버튼을 두지 않으며, 본문 HTML 정제(FR-T03) 시 `<img>` 태그도 제거 대상이다
- 하위 할 일(체크리스트), 캘린더 뷰
- 회원가입 시 이메일 인증
- 통계 대시보드의 고도화(차트, 기간별 추이) — 요약 수치만 제공
- 다국어 지원, 모바일 앱

---

## 3. 타깃 사용자

| 구분 | 설명 |
|---|---|
| 주 사용자 | 개인 할 일을 웹에서 관리하려는 일반 사용자 (role: USER) |
| 관리자 | 서비스 운영자. 사용자 현황 파악과 문제 계정 차단을 담당 (role: ADMIN) |
| 사용 환경 | 데스크톱 브라우저 중심, 모바일 브라우저 지원 |
| 전제 | 각 사용자는 자신의 데이터만 접근한다. 사용자 간 데이터 공유는 없다 |

---

## 4. 사용자 스토리

| ID | 스토리 |
|---|---|
| US-01 | 사용자로서 이메일과 비밀번호로 가입해 내 계정을 만들고 싶다 |
| US-02 | 사용자로서 구글 계정으로 간편하게 로그인하고 싶다 |
| US-03 | 사용자로서 로그인해야만 내 할 일에 접근되기를 원한다 |
| US-04 | 사용자로서 제목과 상세 내용, 마감일, 우선순위를 지정해 할 일을 등록하고 싶다 |
| US-05 | 사용자로서 상세 내용을 굵게·목록 등으로 서식을 넣어 작성하고 싶다 |
| US-06 | 사용자로서 할 일 목록을 최신순 또는 마감일순으로 보고 싶다 |
| US-07 | 사용자로서 완료/미완료로 걸러서 보고 싶다 |
| US-08 | 사용자로서 목록이 길어져도 페이지 단위로 편하게 탐색하고 싶다 |
| US-09 | 사용자로서 체크박스를 누르면 기다림 없이 즉시 완료 표시되기를 원한다 |
| US-10 | 사용자로서 할 일을 삭제해도 데이터가 곧바로 완전히 사라지지는 않기를 원한다 |
| US-11 | 사용자로서 눈이 편한 다크 모드와 밝은 라이트 모드를 골라 쓰고 싶다 |
| US-12 | 사용자로서 휴대폰에서도 레이아웃이 깨지지 않고 사용하고 싶다 |
| US-13 | 사용자로서 비밀번호를 잊었을 때 이메일로 재설정 링크를 받아 스스로 복구하고 싶다 |
| US-14 | 사용자로서 할 일에 참고 파일을 첨부하고 나중에 내려받고 싶다 |
| US-15 | 관리자로서 가입한 사용자 목록과 전체 현황을 확인하고 싶다 |
| US-16 | 관리자로서 문제가 되는 계정을 비활성화해 접근을 막고 싶다 |
| US-17 | 서비스 운영자로서 일반 사용자가 관리 기능에 접근할 수 없다는 것을 보장받고 싶다 |

---

## 5. 기능 요구사항

### 5.1 인증 (AUTH)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-A01 | 회원가입 ID는 **이메일만** 사용한다(`username` 필드 없음). 이메일 형식을 검증한다 | 필수 |
| FR-A01-1 | 이름(`name`)은 **필수, 1~50자**다. 소셜 가입 시에는 제공자 프로필의 이름을 사용하며, 없으면 이메일의 로컬 파트를 사용한다 | 필수 |
| FR-A02 | 비밀번호는 **6자 이상**이며 BCrypt(strength 10)로 해싱해 저장한다 | 필수 |
| FR-A03 | 이미 존재하는 이메일로 가입 시 409와 안내 메시지를 반환한다 | 필수 |
| FR-A04 | 로그인 성공 시 **JWT Access Token(24시간 만료)** 을 발급한다. Refresh Token은 사용하지 않는다 | 필수 |
| FR-A04-1 | JWT 인증 필터는 토큰 서명 검증 후 `userId`로 사용자를 조회해 **`enabled = true`이고 `deleted_at IS NULL`** 임을 확인한다. 불충족 시 401을 반환한다. 이는 FR-M06(비활성 계정의 기존 토큰 차단)을 충족하기 위한 것이며, **Refresh Token 도입이 아니므로 FR-A04와 상충하지 않는다.** 조회는 PK 단건이며 `@PreAuthorize`용 사용자 정보 로딩과 공유한다 | 필수 |
| FR-A05 | 토큰은 클라이언트 **localStorage**에 저장하고 모든 요청에 `Authorization: Bearer` 헤더로 전송한다 | 필수 |
| FR-A06 | 토큰 만료·무효 시 401을 반환하고, 프론트는 토큰을 삭제한 뒤 로그인 화면으로 이동하며 만료 안내를 표시한다 | 필수 |
| FR-A07 | **구글 OAuth2 로그인**을 지원한다. 백엔드 리다이렉트 방식으로 처리한다 | 필수 |
| FR-A08 | OAuth 성공 시 JWT를 URL에 직접 노출하지 않고, 단기(1~2분) 일회용 코드를 프론트에 전달한 뒤 교환 API로 JWT를 발급한다. 코드는 1회만 사용 가능하다. **코드는 서버 인메모리 저장소에 보관**하며 TTL 경과 또는 1회 사용 시 즉시 폐기한다. 단일 인스턴스 배포를 전제하므로 **별도 테이블이나 Redis를 두지 않는다** | 필수 |
| FR-A09 | 구글 계정의 이메일이 기존 계정과 같으면 **기존 계정에 연결**한다. 새 계정을 만들지 않는다 | 필수 |
| FR-A10 | 인증되지 않은 사용자는 할 일 관련 모든 API와 화면에 접근할 수 없다 | 필수 |
| FR-A11 | 로그아웃은 **클라이언트 동작**이다. localStorage의 토큰을 삭제하고 `/login`으로 이동하며 React Query 캐시를 비운다. JWT는 stateless이므로 별도 서버 API를 두지 않는다 | 필수 |

### 5.2 할 일 관리 (TODO)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-T01 | 할 일은 제목(필수, 1~200자), 내용(선택, 서식 있는 텍스트), 마감일(선택), 우선순위(HIGH/MEDIUM/LOW, 기본 MEDIUM), 완료여부를 가진다 | 필수 |
| FR-T02 | 내용은 **Tiptap 에디터**로 작성한다. 툴바는 굵게·기울임·밑줄·취소선·목록·제목·인용·링크로 제한한다. 저장 형식은 **HTML 문자열**(`editor.getHTML()`)이며 JSON을 사용하지 않는다(7장 `content TEXT`와 일치) | 필수 |
| FR-T03 | 내용 HTML은 **서버에서 정제(sanitize)** 한 뒤 저장한다. script 태그, on* 이벤트 속성, javascript: 링크는 제거한다 | 필수 |
| FR-T04 | 할 일의 생성·조회·수정·삭제가 가능하다 | 필수 |
| FR-T05 | 완료 상태는 체크박스로 토글한다 | 필수 |
| FR-T06 | **삭제는 Soft Delete**로 처리한다. 물리 삭제하지 않고 `deleted_at`을 기록하며, 이후 모든 조회에서 제외한다 | 필수 |
| FR-T07 | 모든 단건 접근(조회/수정/토글/삭제)에서 **소유권을 검증**한다. 타인의 리소스 접근 시 404를 반환한다(존재 여부를 노출하지 않기 위해 403을 쓰지 않는다). 단 **역할 기반 경로 차단(`/api/admin/**`)은 403**을 반환한다 — 리소스 존재 여부와 무관한 권한 문제이며, 관리자 API의 존재 자체는 비밀이 아니기 때문이다 | 필수 |

### 5.3 목록·탐색 (LIST)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-L01 | 목록은 서버 페이지네이션으로 제공한다. 기본 페이지 크기 **10**, 기본 정렬 **최신순(createdAt desc)** | 필수 |
| FR-L02 | 응답에 현재 페이지, 페이지 크기, 전체 건수, 전체 페이지 수, 첫/마지막 페이지 여부를 포함한다 | 필수 |
| FR-L03 | 상태 필터를 제공한다: 전체(기본) / 미완료 / 완료 | 필수 |
| FR-L04 | 정렬은 최신순 / 마감일순을 제공한다 | 필수 |
| FR-L05 | **재사용 가능한 페이지네이션 컴포넌트**를 별도로 구현한다. 페이지 번호, 이전/다음, 생략(…) 처리, 현재 페이지 강조, 모바일 축약형을 지원한다 | 필수 |
| FR-L06 | 필터·정렬·페이지 상태가 서로 충돌 없이 동작하며, 필터 변경 시 첫 페이지로 이동한다 | 필수 |

### 5.4 UI/UX

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-U01 | 디자인 방향은 **뉴트럴 미니멀**이다. 무채색 스케일 + 액센트 컬러 1개, 헤어라인 보더, 넉넉한 여백, 타이포 중심 위계 | 필수 |
| FR-U02 | **다크 모드를 기본**으로 하고 라이트 모드 토글을 제공한다. 두 모드 모두 텍스트 대비가 충분해야 한다 | 필수 |
| FR-U03 | 그라데이션·글로우·과한 그림자는 사용하지 않는다 | 필수 |
| FR-U04 | 모든 목록 화면은 **로딩(스켈레톤) / 빈 상태 / 에러 상태** UI를 갖춘다 | 필수 |
| FR-U05 | 완료 토글과 삭제는 **낙관적 업데이트**로 즉시 반영하고, 실패 시 원래 상태로 롤백하며 에러 토스트를 표시한다 | 필수 |
| FR-U06 | 애니메이션은 목록 진입/삭제/완료 토글 등 상태 변화에만 절제해서 사용한다 | 권장 |
| FR-U07 | 반응형: 모바일 1열 카드형, 데스크톱 리스트형. 모바일에서 레이아웃이 깨지지 않는다 | 필수 |
| FR-U08 | 에러 메시지는 무슨 일이 있었는지와 다음에 무엇을 할지를 한 문장으로 안내한다. 서버 스택트레이스를 노출하지 않는다 | 필수 |

### 5.5 비밀번호 재설정 (RESET)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-R01 | 이메일을 입력하면 재설정 링크를 메일로 발송한다. 로컬 개발에서는 링크를 로그로 출력한다 | 필수 |
| FR-R02 | 토큰은 30분 만료, 1회용이며 DB에는 해시로 저장한다 | 필수 |
| FR-R03 | 가입되지 않은 이메일이나 소셜 전용 계정이어도 **동일한 성공 응답**을 반환한다(계정 존재 여부 노출 금지) | 필수 |
| FR-R04 | 재설정 시 새 비밀번호는 6자 이상이며 확인 입력과 일치해야 한다 | 필수 |
| FR-R05 | 재설정 성공 시 해당 사용자의 미사용 토큰을 모두 무효화한다 | 필수 |
| FR-R06 | 만료·재사용·위조 토큰은 "유효하지 않거나 만료된 링크입니다"로 동일하게 처리한다 | 필수 |
| FR-R07 | 동일 이메일의 연속 요청을 제한한다(1분 내 재요청 차단) | 권장 |

### 5.6 파일 첨부 (FILE)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-F01 | 할 일 하나에 최대 **5개**, 개당 최대 **10MB**까지 첨부할 수 있다 | 필수 |
| FR-F02 | 허용 형식: jpg, png, webp, gif, pdf, docx, xlsx, txt, zip. 확장자와 Content-Type을 모두 검증한다 | 필수 |
| FR-F03 | 저장소는 인터페이스로 추상화한다. 로컬 개발은 디스크, AWS는 S3를 사용하며 환경변수로 전환한다 | 필수 |
| FR-F04 | 저장 키는 UUID 기반으로 생성하고 원본 파일명은 DB에만 보관한다 | 필수 |
| FR-F05 | 다운로드는 소유권 검증 후 **단기 만료 URL**로 제공한다. `FileStorage` 인터페이스는 `generateDownloadUrl(storedKey, ttl)`을 정의하며, **S3 구현체는 presigned URL**을, **로컬 구현체는 서명된 만료 토큰이 포함된 자체 다운로드 엔드포인트 URL**을 반환한다. 어느 경우에도 저장소 원본(S3 버킷·파일시스템 경로)에 직접 접근할 수 없다 | 필수 |
| FR-F06 | 첨부 삭제는 Soft Delete로 처리하며 실제 파일은 즉시 지우지 않는다 | 필수 |
| FR-F07 | 업로드 UI는 드래그앤드롭과 파일 선택을 지원하고, 진행 상태·파일명·크기를 표시한다. 이미지에는 썸네일을 보여준다 | 필수 |
| FR-F08 | 목록에서 첨부가 있는 할 일에 클립 아이콘과 개수를 표시한다 | 권장 |

### 5.7 관리자 (ADMIN)

| ID | 요구사항 | 우선순위 |
|---|---|---|
| FR-M01 | 사용자는 `USER` 또는 `ADMIN` 권한을 가지며, JWT에 권한 정보를 포함한다 | 필수 |
| FR-M02 | **관리자 API는 서버에서 권한을 검사한다.** SecurityConfig와 `@PreAuthorize`를 이중으로 적용하며, 프론트 화면 가드는 보안 수단으로 인정하지 않는다 | 필수 |
| FR-M03 | 최초 관리자는 SQL로 직접 지정한다. 회원가입 등 일반 경로로 ADMIN이 될 수 없다 | 필수 |
| FR-M04 | 관리자는 사용자 목록을 페이지네이션으로 조회하고 이메일로 검색할 수 있다 | 필수 |
| FR-M05 | 관리자는 계정을 활성/비활성으로 전환할 수 있다. 자기 자신은 비활성화할 수 없다 | 필수 |
| FR-M06 | 비활성화된 계정은 로그인이 차단되며, 이미 발급된 토큰으로도 접근할 수 없다. 구현은 **FR-A04-1**(JWT 필터의 매 요청 사용자 상태 확인)에 따른다 | 필수 |
| FR-M07 | 관리자는 전체 할 일을 작성자와 함께 페이지네이션으로 조회할 수 있다. 작성자 정보는 **`JOIN` + DTO 프로젝션**으로 한 번에 조회해 N+1을 방지한다(7장 조회 규칙 참조) | 필수 |
| FR-M08 | 관리자는 요약 통계(전체 사용자 수, 활성 사용자 수, 전체 할 일 수, 완료 수)를 볼 수 있다 | 필수 |
| FR-M09 | 관리자 응답에 비밀번호 해시 등 민감정보를 포함하지 않는다 | 필수 |
| FR-M10 | 일반 사용자가 `/admin`에 직접 접근하면 접근 불가 안내 화면을 표시한다 | 필수 |

---

## 6. 화면 정의

| 라우트 | 화면 | 주요 요소 | 인증 |
|---|---|---|---|
| `/login` | 로그인 | 이메일·비밀번호 입력, 구글 로그인 버튼, 회원가입 링크, 에러/만료 안내 | X |
| `/signup` | 회원가입 | 이메일·비밀번호·이름 입력, 검증 메시지 | X |
| `/oauth/callback` | OAuth 콜백 | 일회용 코드 교환, 로딩 표시, 실패 시 로그인으로 이동 | X |
| `/todos` | 할 일 목록(메인) | 필터·정렬, 목록, 페이지네이션, 생성 버튼, 로딩/빈/에러 상태 | O |
| `/todos` 내 다이얼로그 | 생성·수정 | 제목, Tiptap 내용, 마감일, 우선순위, 첨부파일 영역 | O |
| `/forgot-password` | 재설정 요청 | 이메일 입력, 항상 동일한 안내 문구 | X |
| `/reset-password` | 비밀번호 변경 | 토큰 검증, 새 비밀번호·확인 입력 | X |
| `/admin` | 관리자 | 요약 통계 카드, 사용자 목록 탭, 전체 할 일 탭 | ADMIN |
| 공통 헤더 | — | 로고, 테마 토글, 사용자 메뉴/로그아웃, (ADMIN일 때만) 관리자 메뉴 | — |

---

## 7. 데이터 모델

DB 스키마 이름: **`todolist_db`** (테스트용: `todolist_db_test`)

### users
`id`, `email`(unique), `password`(nullable — 소셜 전용 계정), `name`, `provider`(LOCAL/GOOGLE), `provider_id`, `role`(USER/ADMIN), `enabled`, `created_at`, `updated_at`, `deleted_at`

> **소셜 로그인 조회 키는 `email`이다.** FR-A09가 "구글 계정의 이메일이 기존 계정과 같으면 기존 계정에 연결"로 규정하므로, OAuth2 성공 처리는 `provider_id`가 아니라 **`email`로 사용자를 조회**한다. `provider`·`provider_id`는 **가입 경로 기록용**이며 조회 키로 쓰지 않는다(따라서 별도 인덱스를 두지 않는다).

### todos
`id`, `user_id`(FK), `title`, `content`(TEXT, 정제된 HTML), `completed`, `due_date`, `priority`(HIGH/MEDIUM/LOW), `created_at`, `updated_at`, `deleted_at`

### password_reset_tokens
`id`, `user_id`(FK), `token`(해시 저장), `expires_at`, `used_at`, `created_at`

### attachments
`id`, `todo_id`(FK), `original_name`, `stored_key`, `content_type`, `size_bytes`, `created_at`, `deleted_at`

### 공통 컬럼 규약

모든 테이블이 아래 규약을 따른다.

**예외:**
- `password_reset_tokens`는 Soft Delete 대상이 아니므로 `deleted_at`·`updated_at`을 두지 않는다.
- `attachments`는 업로드 후 내용이 변경되지 않으므로 `updated_at`을 두지 않는다. (`deleted_at`은 FR-F06에 따라 유지)

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | BIGSERIAL | PK | 자동 증가. UUID를 사용하지 않는다 |
| `created_at` | TIMESTAMP | NOT NULL | 생성 시각 |
| `updated_at` | TIMESTAMP | NOT NULL | 수정 시각 |
| `deleted_at` | TIMESTAMP | NULL | Soft Delete 표식. NULL이면 유효한 행 |

- **명명 규칙:** 테이블·컬럼 모두 snake_case. 엔티티 필드는 camelCase로 두고 매핑에 맡긴다.
- **조회 규칙:** `deleted_at IS NULL` 필터를 모든 조회에 적용한다 (FR-T06). 구현은 `@SQLRestriction("deleted_at IS NULL")`을 사용한다.
  - ⚠️ **`@SQLRestriction`은 해당 엔티티의 모든 쿼리에 항상 적용되며 비활성화·파라미터화할 수 없다.** ToOne 연관(`Todo.user`, `Attachment.todo`)의 대상이 Soft Delete된 경우 Hibernate가 대상을 찾지 못해 `EntityNotFoundException`이 발생한다. `@NotFound`로 회피하면 해당 연관이 **강제 eager 로딩**되므로 사용하지 않는다.
  - 따라서 **`users`에는 `@SQLRestriction`을 걸지 않고** Repository 메서드에서 `deleted_at IS NULL` 조건을 명시한다. 관리자 조회(FR-M07)는 연관 매핑 탐색 대신 **명시적 `JOIN` + DTO 프로젝션**으로 작성해 N+1과 예외를 동시에 피한다.
  - Hibernate 6.4+의 `@SoftDelete`는 boolean 플래그 기반이라 `deleted_at TIMESTAMP` 규약과 맞지 않으므로 **사용하지 않는다.**
- **연관관계:** FK는 `{대상테이블단수}_id` 형식 (`user_id`, `todo_id`).
- **Enum:** DB에는 문자열(VARCHAR)로 저장한다. `@Enumerated(EnumType.STRING)`을 사용하며 ordinal 저장을 금지한다.

### 인덱스

| 테이블 | 인덱스 | 목적 |
|---|---|---|
| todos | `(user_id, deleted_at, created_at)` | 목록 조회 + 최신순 정렬 (FR-L01) |
| todos | `(user_id, deleted_at, due_date)` | 마감일순 정렬 (FR-L04). 필터와 정렬을 한 인덱스로 처리한다 |
| attachments | `(todo_id, deleted_at)` | 할 일별 첨부 조회 |
| users | `(email)` unique | 로그인·중복 가입 검사 (FR-A03) |
| password_reset_tokens | `(token)` | 재설정 토큰 조회 (FR-R02) |

---

## 8. API 요약

| 메서드 | 경로 | 설명 | 인증 |
|---|---|---|---|
| POST | `/api/auth/signup` | 회원가입 | X |
| POST | `/api/auth/login` | 로그인 (JWT 반환) | X |
| GET | `/api/auth/me` | 내 정보 | O |
| GET | `/oauth2/authorization/google` | 구글 로그인 시작 | X |
| POST | `/api/auth/oauth/exchange` | 일회용 코드 → JWT 교환 | X |
| GET | `/api/todos` | 목록 (page, size, status, sort) | O |
| POST | `/api/todos` | 생성 | O |
| GET | `/api/todos/{id}` | 단건 조회 | O |
| PUT | `/api/todos/{id}` | 수정 | O |
| PATCH | `/api/todos/{id}/toggle` | 완료 토글 | O |
| DELETE | `/api/todos/{id}` | Soft Delete | O |
| POST | `/api/auth/password/forgot` | 재설정 메일 요청 | X |
| POST | `/api/auth/password/reset` | 토큰으로 비밀번호 변경 | X |
| POST | `/api/todos/{id}/attachments` | 첨부 업로드 | O |
| GET | `/api/todos/{id}/attachments` | 첨부 목록 | O |
| GET | `/api/attachments/{id}/download-url` | 단기 만료 다운로드 URL 발급 | O |
| GET | `/api/attachments/{id}/download` | 서명 토큰(`token`) 검증 후 파일 스트리밍 — **로컬 저장소 전용** | X |
| DELETE | `/api/attachments/{id}` | 첨부 Soft Delete | O |
| GET | `/api/admin/users` | 사용자 목록 (page, size, email) | ADMIN |
| PATCH | `/api/admin/users/{id}/status` | 계정 활성/비활성 | ADMIN |
| GET | `/api/admin/todos` | 전체 할 일 조회 | ADMIN |
| GET | `/api/admin/stats` | 요약 통계 | ADMIN |
| GET | `/actuator/health` | 헬스체크 | X |

- 응답 DTO만 반환하며 엔티티를 직접 노출하지 않는다.
- API 문서는 SpringDoc OpenAPI(Swagger UI)로 제공하고, 운영 환경에서는 비활성화한다.

**첨부 다운로드 두 엔드포인트의 관계 (FR-F05)**

`download-url`은 **소유권을 검증한 뒤** 저장소 구현체가 만든 단기 만료 URL을 반환한다. 프론트는 반환된 URL을 그대로 열기만 하며, 저장소 종류를 알 필요가 없다.

| `APP_STORAGE_TYPE` | `download-url`이 반환하는 값 | `/download` 사용 여부 |
|---|---|---|
| `local` | `{API_BASE}/api/attachments/{id}/download?token={서명토큰}` | 사용함 |
| `s3` | S3 presigned URL | 사용 안 함 (호출되지 않음) |

- `/download`가 인증 헤더를 요구하지 않는 이유는 **서명 토큰 자체가 대상 첨부 ID·만료 시각을 담고 서버 키로 서명**되어 있기 때문이다. 토큰 검증은 `download-url` 단계에서 이미 끝난 소유권 검증을 대체한다.
- 토큰이 없거나 위조·만료된 경우 404를 반환한다(존재 여부를 노출하지 않는다, FR-T07과 동일한 원칙).
- 이 경로는 로컬 저장소 전용이므로, S3 전환 시 **프론트 코드는 변경되지 않는다**(9장 이식성).

### 8.1 응답 형식

**모든 성공 응답은 `ApiResponse<T>`로 감싼다.**

```json
{ "success": true, "data": { }, "message": null }
```

| 필드 | 타입 | 설명 |
|---|---|---|
| `success` | boolean | 성공 시 항상 `true` |
| `data` | T | 실제 페이로드. 본문이 없으면 `null` |
| `message` | string \| null | 사용자에게 보일 안내가 있을 때만 채운다 |

**목록 응답의 `data`는 `PageResponse<T>`다.** (FR-L02)

```json
{
  "success": true,
  "data": {
    "content": [],
    "page": 0, "size": 10,
    "totalElements": 0, "totalPages": 0,
    "first": true, "last": true
  },
  "message": null
}
```

`page`는 **0부터 시작**한다. 프론트에서 1-base로 보여줄 경우 표시 단계에서만 변환한다.

**에러 응답**은 래퍼를 쓰지 않고 아래 평면 구조로 통일한다.

```json
{
  "timestamp": "2026-08-27T10:00:00Z",
  "status": 400,
  "code": "VALIDATION_FAILED",
  "message": "입력값을 확인해 주세요.",
  "errors": [{ "field": "email", "message": "이메일 형식이 아닙니다." }]
}
```

| 필드 | 설명 |
|---|---|
| `code` | 문자열 에러 코드. 프론트 분기용 |
| `message` | 사용자에게 그대로 보여줄 한 문장 (FR-U08) |
| `errors[]` | 필드 검증 실패 시에만 존재. `field`/`message` 쌍 |

> **에러 코드 값 체계는 Phase 2에서 확정한다.** 그 전까지 프론트는 `code`를 문자열로만 다루고 값을 하드코딩하지 않는다.
> 필드명은 `code`이며 `errorCode`가 아니다. 프론트는 `code` 값을 하드코딩하지 않는다.

---

## 9. 비기능 요구사항

| 구분 | 요구사항 |
|---|---|
| 보안 | 비밀번호 BCrypt 해싱, JWT HS256, 시크릿은 환경변수로만 주입, 모든 입력값 서버 검증, HTML 정제, 업로드 파일 형식·용량 검증, 관리자 API 서버 권한 검사, CORS는 지정된 출처만 허용(와일드카드 금지), 운영에서 Swagger 차단 |
| 데이터 | 물리 삭제 금지, 소유권 검증 필수, 운영 DB는 자동 백업 활성화, S3 버킷 퍼블릭 액세스 차단 |
| 성능 | 목록 조회는 페이지 단위(10건)로만 조회하며 전체 로딩하지 않는다. 정렬 기준에 따라 `(user_id, deleted_at, created_at)`(최신순) 또는 `(user_id, deleted_at, due_date)`(마감일순) 복합 인덱스를 사용해 필터와 정렬을 한 인덱스로 처리한다(7장 인덱스). 인증된 요청마다 JWT 검증 시 **사용자 PK 조회 1회**가 발생한다(FR-A04-1) |
| 이식성 | URL·DB 접속정보·CORS 출처를 소스에 하드코딩하지 않고 환경변수로 분리하며, 파일 저장과 메일 발송은 인터페이스로 추상화해 값 교체만으로 AWS 이전이 가능해야 한다 |
| 운영 | 로그는 표준출력, `/actuator/health` 제공, 서버 재시작 시 자동 기동 |
| 접근성 | 다크·라이트 모두 충분한 텍스트 대비, 키보드로 주요 흐름(입력·제출·토글) 조작 가능 |
| 테스트 | 백엔드 통합테스트 작성(프론트 자동화 테스트는 범위 밖). Docker/Testcontainers 미사용 |

### 9.1 환경변수 분리 원칙

아래 값은 **소스에 하드코딩하지 않는다.** 로컬은 `application-local.yml`(커밋 금지)과 `.env.local`, 운영은 배포 환경의 환경변수로 주입한다.

| 환경변수 | 용도 | 로컬 → AWS 전환 |
|---|---|---|
| `DB_URL` / `DB_USERNAME` / `DB_PASSWORD` | DB 접속 | 로컬 PostgreSQL → RDS |
| `JWT_SECRET` | JWT HS256 서명 키 | 운영은 별도 값 사용 |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | OAuth2 클라이언트 | 리다이렉트 URI 재등록 필요 |
| `APP_CORS_ALLOWED_ORIGINS` | 허용 출처 | 로컬 주소 → 운영 도메인 |
| `APP_PASSWORD_RESET_URL` | 재설정 링크 베이스 URL | 로컬 주소 → 운영 도메인 |
| `APP_OAUTH_REDIRECT_URL` | 구글 로그인 성공 후 프론트 콜백 베이스 URL | 로컬 http://localhost:3000/oauth/callback → 운영 도메인 (FR-A08) |
| `APP_STORAGE_TYPE` | `local` \| `s3` | 구현체 전환 스위치 (FR-F03) |
| `APP_UPLOAD_DIR` | 로컬 저장소의 파일 저장 루트 경로 | `local`일 때만 사용. S3 전환 후 미사용 |
| `APP_S3_BUCKET` / `APP_S3_REGION` | S3 버킷명·리전 | `s3`일 때만 사용. 로컬에서는 미설정 |
| `APP_DOWNLOAD_URL_TTL_SECONDS` | 다운로드 URL 만료 시간(초) | 로컬·S3 공통 (FR-F05) |
| `APP_MAIL_TYPE` | `log` \| `ses` | 구현체 전환 스위치 (FR-R01) |
| `NEXT_PUBLIC_API_BASE_URL` | 프론트가 호출할 API 주소 | 로컬 주소 → 운영 도메인 |

- 프로파일을 `local` / `prod`로 분리하고, 공통 설정만 `application.yml`에 둔다.
- **시크릿이 담긴 파일(`.env*`, `application-local.yml`)은 커밋하지 않는다.**
- 코드는 값이 무엇인지 알 필요가 없어야 한다. 파일 저장·메일 발송은 인터페이스로 추상화해 **환경변수 교체만으로** AWS 이전이 끝나야 한다(FR-F03, FR-R01).

---

## 10. 제약 조건

- 기술 스택과 버전은 **1.3 기술 스택** 절에 고정되어 있으며 임의 변경할 수 없다. 변경이 필요하면 1.3 절을 먼저 개정한다.
- **Docker를 사용하지 않는다.** 통합테스트는 로컬 PostgreSQL의 `todolist_db_test`를 사용한다.
- 백엔드 베이스 패키지는 `com.example`, 빌드는 Maven.
- 개발은 전부 로컬에서 진행하고, AWS 작업은 모든 기능·테스트 완료 후에 착수한다.
- 파일 저장은 로컬 개발 시 디스크, AWS 이전 후 S3를 사용하며 코드 변경 없이 전환된다.
- 메일 발송은 로컬에서 로그 출력으로 대체하고, AWS 이전 시 SES로 교체한다.
- 모든 주석과 커밋 메시지는 한글로 작성한다.

---

## 11. 개발 마일스톤

| 단계 | 내용 | 산출물 |
|---|---|---|
| Phase 0 | 모노레포 스캐폴딩, 로컬 환경 구성 | 빌드되는 빈 프로젝트 |
| Phase 1 | DB 스키마, 엔티티, Repository, Soft Delete | 테이블 생성 확인 |
| Phase 2 | Spring Security 7 + JWT 인증 | 가입/로그인/보호 API |
| Phase 3 | 구글 OAuth2 + 일회용 코드 교환 | 소셜 로그인 동작 |
| Phase 4 | Todo CRUD API, 페이지네이션, 소유권 검증, 예외 처리, HTML 정제 | API 완성 |
| Phase 5 | 프론트 디자인 토큰, API 클라이언트, **React Query·next-themes 설치 및 설정**, 공통 컴포넌트(페이지네이션 포함) | 공통 토대 |
| Phase 6 | 인증 화면, OAuth 콜백, 라우트 보호 | 로그인 흐름 완성 |
| Phase 7 | 할 일 화면, 낙관적 업데이트, Tiptap | 메인 기능 완성 |
| Phase 8 | 백엔드 통합테스트 + 로컬 검증 체크리스트 | 핵심 기능 완성본 |
| Phase 9 | 비밀번호 재설정 (토큰 발급, 메일 추상화, 재설정 화면) | 계정 복구 흐름 |
| Phase 10 | 파일 첨부 (저장소 추상화, 업로드/다운로드/삭제) | 첨부 기능 |
| Phase 11 | 관리자 페이지 (role, 권한 검사, 사용자·할일·통계) | 관리 기능 |
| Phase 12 | AWS 이전 (RDS → S3·SES 전환 → EC2 → Amplify → 운영 점검) | 운영 배포 |

---

## 12. 인수 기준

인수 기준은 검증 시점에 따라 두 묶음으로 나눈다. **ROADMAP의 게이트는 번호 범위가 아니라 아래 소절 이름으로 참조한다** — 항목이 추가돼도 게이트 정의가 어긋나지 않게 하기 위함이다.

### 12.1 M1 게이트 (Phase 8 종료 시점) — 핵심 MVP

1. `todolist_db`와 함께 PostgreSQL 실행
2. 백엔드가 오류 없이 기동 (`./mvnw spring-boot:run`)
3. 프론트엔드 빌드 성공 (`npm run build`)
4. 회원가입 시 사용자 생성 및 JWT 반환
5. 로그인 시 유효한 JWT 반환
6. 보호된 엔드포인트가 유효한 토큰을 요구
7. 할 일 CRUD가 페이지네이션과 함께 동작
8. Soft Delete 시 `deleted_at`이 갱신되고 목록에서 제외
9. 타인의 리소스 접근 시 404 (소유권 검증)
10. 뉴트럴 미니멀 테마 + 다크/라이트 토글 적용
11. 반응형 레이아웃 정상 동작
12. 구글 로그인이 정상 동작하고 동일 이메일 계정이 중복 생성되지 않음
13. 내용에 script 태그를 넣어 저장해도 정제되어 저장됨
21. 상태 필터(전체/미완료/완료)와 정렬(최신순/마감일순) 변경 시 목록이 갱신되고, 필터 변경 시 첫 페이지로 이동함 (FR-L03·L04·L06)
22. 모든 목록 화면에 로딩(스켈레톤)·빈 상태·에러 상태 UI가 표시됨 (FR-U04)
23. 완료 토글이 즉시 반영되고, 서버 실패 시 원래 상태로 롤백되며 에러 토스트가 표시됨 (FR-U05)
24. 구글 로그인 후 받은 일회용 코드를 두 번째로 교환하면 실패함 (FR-A08)

> 21~24는 나중에 추가돼 번호가 뒤에 있으나 **Phase 7까지 구현이 끝나므로 M1에서 검증한다.** 번호는 기존 참조를 깨지 않기 위해 유지한다.

### 12.2 M2 게이트 (Phase 11 종료 시점) — 확장 기능

> **번호가 14부터 시작하고 21~24가 12.1에 있는 것은 의도된 배치다.** 인수 기준은 작성 순서대로 번호를 매겼고, 이후 12.1/12.2로 **검증 시점에 따라 재분류**했다. 기존 참조를 깨지 않기 위해 번호는 그대로 두었으므로, 번호의 연속성이 아니라 **소속 소절**을 기준으로 읽는다. 두 소절을 합치면 1~24가 빠짐없이 채워진다.

14. 재설정 링크로 비밀번호 변경 후 새 비밀번호로 로그인 가능하며, 같은 토큰 재사용은 실패
15. 미가입 이메일로 재설정 요청 시에도 동일한 성공 응답
16. 파일 업로드·다운로드·삭제가 동작하고, 용량·형식·개수 제한이 서버에서 적용됨
17. 타인의 할 일에 첨부 업로드 시 404
18. USER 권한으로 `/api/admin/**` 접근 시 403, 토큰 없이 접근 시 401
19. 관리자 화면에서 계정 비활성화 후 해당 계정의 로그인과 기존 토큰 사용이 모두 차단됨 (FR-A04-1)
20. 관리자가 자기 자신을 비활성화할 수 없음

> **M2 게이트는 12.1 + 12.2 전부(24개)를 통과해야 한다.** M1에서 통과한 항목도 확장 기능 추가로 깨지지 않았는지 재확인한다.

---

## 13. 리스크와 대응

| 리스크 | 영향 | 대응 |
|---|---|---|
| Spring Boot 4 / Security 7의 신규 문법을 도구가 구버전으로 작성 | 컴파일 실패, 잘못된 보안 설정 | **1.3 절**에 버전·금지사항 고정(`antMatchers` 등 구 API 금지). 불확실하면 추측하지 말고 질문하도록 규칙화 |
| springdoc-openapi 버전 문제 | 기동 실패 | 2.x는 Boot 4 미지원이라 금지. **3.1.0 이상**으로 고정하고 Phase 2에서 기동을 확인한다. 3.0.x에도 Jackson 2/3 관련 이슈가 보고되어 있으므로 실패 시 이슈 트래커를 확인한다 |
| `@SQLRestriction`이 ToOne 연관에서 `EntityNotFoundException` 유발 | 관리자 조회 실패 | `users`에는 제한을 걸지 않고 Repository 조건으로 처리, FR-M07은 JOIN + DTO 프로젝션 (**7장 조회 규칙**) |
| 로컬 저장소에 presigned URL 개념이 없어 추상화 실패 | AWS 이전 시 코드 수정 발생 | `FileStorage.generateDownloadUrl(key, ttl)` 계약으로 통일. 로컬은 서명 토큰 URL 반환 (FR-F05) |
| Amplify의 Next.js 16 지원 여부 불확실 | 배포 방식 변경 필요 | **Phase 11 종료 시점**에 공식 문서를 확인해 OPEN-01을 결정하고, 미지원 시 정적 export 또는 EC2 `next start`로 전환 |
| localStorage 토큰의 XSS 노출 | 계정 탈취 | 사용자 입력 HTML 서버 정제 필수, 외부 스크립트 최소화 |
| 낙관적 업데이트와 페이지네이션 캐시 키 불일치 | 목록이 어긋나 보임 | 필터·정렬·페이지를 포함한 쿼리 키 설계, 실패 시 롤백 + invalidate |
| 로컬 전용 코드가 AWS에서 깨짐 | 배포 지연 | 환경변수 분리 원칙(**9장 이식성**), Phase 12-1 사전 점검 |
| 관리자 권한 검사를 프론트 가드에만 의존 | 권한 우회 | SecurityConfig + `@PreAuthorize` 이중 검사, 통합테스트로 403/401 검증 |
| 업로드 파일을 통한 악성 파일 유입 | 서버·사용자 피해 | 확장자·Content-Type 화이트리스트, 용량·개수 제한, UUID 키 저장, 버킷 퍼블릭 차단 |
| 재설정 응답으로 가입 여부가 드러남 | 계정 열거 공격 | 모든 경우 동일 응답, 요청 빈도 제한 |
| 로컬 디스크 저장이 EC2 재배포 시 소실 | 첨부 유실 | `FileStorage` 추상화로 AWS에서는 S3 사용 |

---

## 14. 미결정 사항

**Phase 11 종료 시점(M2 게이트)에 네 건을 모두 결정한다. Phase 12 착수의 선행조건이며, 미결정 상태로는 AWS 이전을 시작하지 않는다.**

| ID | 항목 | 선택지 |
|---|---|---|
| OPEN-01 | 프론트 배포 방식 | Amplify SSR / 정적 export / EC2에서 `next start` — Amplify의 Next.js 16 지원 확인 후 결정 |
| OPEN-02 | HTTPS 적용 방식 | Nginx + Let's Encrypt / ALB + ACM |
| OPEN-03 | 도메인 | 자체 도메인 사용 / Amplify 기본 도메인 + EC2 퍼블릭 주소 |
| OPEN-04 | 운영 메일 발송 | SES 샌드박스 해제 여부와 발신 도메인 검증 방식 |

> OPEN-03의 결정에 따라 구글 클라우드 콘솔의 승인된 리다이렉트 URI, `APP_CORS_ALLOWED_ORIGINS`, `APP_PASSWORD_RESET_URL` 값이 함께 달라진다(9.1 참조).

---

## 15. 변경 이력

| 버전 | 일자 | 내용 |
|---|---|---|
| 1.0 | 2026-08-26 | 최초 작성 |
| 1.1 | 2026-08-27 | **PRD 자립화 개정.** ① 존재하지 않는 `CLAUDE.md`·`claude-code-프롬프트.md` 위임 6곳 제거 ② **1.3 기술 스택** 절 신설(버전·설치 상태, `docs/guides/`가 SSOT로 참조하던 죽은 앵커 해소) ③ **8.1 응답 형식** 신설(`ApiResponse<T>`/`PageResponse<T>` 확정) ④ 7장 공통 컬럼 규약·인덱스 명문화 ⑤ 9.1 환경변수 분리 원칙 신설 ⑥ FR-A01-1(이름 제약)·FR-A11(로그아웃) 추가 ⑦ FR-F09 삭제(2장 비목표와 중복) ⑧ 관리자 사용자 목록 검색 파라미터 명시 ⑨ 13장 Amplify 확인 시점 오기(Phase 9 → Phase 11 종료) 정정 |
| 1.2 | 2026-08-27 | **기술 검증(prd-validator) 반영.** ① **1.3 표를 실물과 동기화** — 설치됨 4종(Lombok, OAuth2 Client, DevTools, shadcn 유틸) 추가, 미설치 3종(jsoup, starter-mail, AWS SDK s3) 추가, 양방향 동기화 의무 명시 ② **FR-A04-1 신설** — FR-M06(비활성 계정 토큰 차단)과 stateless JWT의 조정 방식 확정 ③ **FR-F05 재작성** — presigned URL을 `generateDownloadUrl(key, ttl)` 계약으로 추상화해 로컬/S3 양립 ④ **7장 조회 규칙에 `@SQLRestriction` 정책 명시** — ToOne 연관 `EntityNotFoundException` 회피, `users` 제외, FR-M07 DTO 프로젝션 ⑤ springdoc 3.1.0 이상으로 정정(근거 보강) ⑥ 11장 Phase 5에 React Query·next-themes 설치 명시 ⑦ 인덱스를 `(user_id, deleted_at, due_date)`로 교체 ⑧ FR-A08 일회용 코드 저장소(인메모리) 확정 ⑨ FR-T02 저장 형식(HTML) 명시 ⑩ FR-T07에 403/404 층위 구분 추가 ⑪ `attachments` `updated_at` 예외 명시 ⑫ 인수 기준 21~24 추가 (FR-L03/L04/L06, FR-U04, FR-U05, FR-A08 커버) |
| 1.3 | 2026-08-27 | **재검증 회귀 수정.** ① **12장을 12.1(M1 게이트) / 12.2(M2 게이트) 소절로 분리** — ROADMAP이 번호 범위 대신 소절 이름으로 참조하게 해 항목 추가 시 게이트 정의가 어긋나는 문제를 구조적으로 차단. 신규 21~24는 Phase 7까지 구현이 끝나므로 12.1에 배치 ② 9장 성능의 `(user_id, deleted_at, *)` 축약을 7장과 동일하게 2개 인덱스로 전개 |
| 1.4 | 2026-08-28 | **ROADMAP 검토에서 역으로 발견된 명세 공백 보정.** ① **8장에 첨부 다운로드 엔드포인트 `GET /api/attachments/{id}/download` 신설** — FR-F05가 "로컬 구현체는 자체 다운로드 엔드포인트 URL을 반환한다"고 규정했으나 8장에 그 경로가 없어, 구현자가 임의로 정해야 했고 저장소별로 프론트 코드가 갈릴 위험이 있었다. 저장소 무관 공통 경로로 확정하고, `download-url`과의 관계·인증 없이 동작하는 이유·실패 시 404를 표로 명시(9장 이식성 보강) ② **9.1에 파일 저장소 환경변수 4종 추가** — `APP_UPLOAD_DIR`(로컬 저장 루트), `APP_S3_BUCKET`/`APP_S3_REGION`, `APP_DOWNLOAD_URL_TTL_SECONDS`. FR-F03·FR-F05가 요구하는 값들의 주입 경로가 표에 없어 9장 이식성("소스에 하드코딩하지 않는다")을 만족할 수 없던 공백 보정 ③ **7장 `users`에 소셜 로그인 조회 키를 `email`로 명시** — FR-A09가 이메일 기준 계정 연결을 규정하므로 `provider_id`는 가입 경로 기록용이며 조회 키가 아님을 못박고, 별도 인덱스를 두지 않는 근거를 남김 ④ **12.2에 번호 배치 각주 추가** — 12.1에만 있던 설명을 12.2에도 붙여 14~20/21~24의 번호 구멍을 누락으로 오인하지 않게 함 ⑤ **1.3 "설치 상태" 열의 의미 명문화** — ✅는 의존성 선언 여부일 뿐 코드 사용 여부가 아님을 표 안내에 추가하고, jjwt 비고의 "의존성만 추가, 아직 미사용"을 설치 단계 불필요로 정정(✅와 "미사용"이 한 칸에 섞여 설치 단계 판단에 혼선을 주던 문제) |
| 1.5 | 2026-08-28 | **APP_OAUTH_REDIRECT_URL 환경변수 신설** — Phase 3 구현 중 프론트 콜백 URL 관리 공백 발견 |
