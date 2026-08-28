# Todo List 서비스

개인 사용자가 자신의 할 일을 생성·정리·완료 처리할 수 있는 웹 기반 Todo 관리 서비스입니다. 이메일 또는 구글 계정으로 로그인해 본인의 할 일만 안전하게 관리하는 것을 목표로 하며, 팀 협업·공유·알림 같은 기능은 이번 범위에 포함하지 않습니다.

**기술 스택:** Spring Boot 4.1.1 백엔드 + Next.js 16.3.3 프론트엔드 모노레포. 로컬 환경에서 전 기능을 완성한 뒤 AWS로 이전하는 방식으로 개발합니다.

**개발 방식:** Claude Code를 활용한 바이브 코딩.

## 현재 진행 상태

**Phase 0(프로젝트 스캐폴딩) 진행 중.** 전체 진행 순서와 각 Phase의 완료 조건은 [`docs/ROADMAP.md`](docs/ROADMAP.md)를 참고하세요.

## 저장소 구조

이 프로젝트는 **3개의 독립된 git 저장소**로 구성됩니다. 각 저장소가 자체 `.git`을 가지며, 의도적으로 통합하지 않는 구조입니다.

```
todo-project/              # 루트 저장소 — 문서(docs/)와 프로젝트 전체 설정
├── docs/
│   ├── PRD.md             # 무엇을 만들 것인가 (기술 스택·데이터 모델·API 계약의 SSOT)
│   ├── ROADMAP.md          # 어떤 순서로 만들 것인가 (Phase 0~12)
│   └── guides/             # 프론트엔드 구현 패턴 가이드
├── todo-backend/           # 독립 저장소 — Spring Boot 백엔드 (com.example)
└── todo-frontend/          # 독립 저장소 — Next.js 16 프론트엔드
```

- 백엔드 파일을 고쳤다면 `todo-backend/` 안에서 커밋합니다.
- 프론트엔드 파일을 고쳤다면 `todo-frontend/` 안에서 커밋합니다.
- 문서(`docs/`, 루트 `CLAUDE.md`)를 고쳤다면 루트 저장소에서 커밋합니다.
- 한 커밋에 서로 다른 저장소의 변경을 섞지 않습니다.

## 문서 계층

문서가 사양의 출처입니다. 역할이 분리되어 있고 충돌 시 우선순위가 명시되어 있습니다.

| 문서 | 역할 | 우선순위 |
|---|---|---|
| `docs/PRD.md` | **무엇을** 만드는가 — 기술 스택·데이터 모델·API 계약의 SSOT | **최상위.** 다른 문서와 어긋나면 PRD를 따릅니다 |
| `docs/ROADMAP.md` | **어떤 순서로** 만드는가 — Phase 0~12, 각 Phase의 완료 조건 | PRD 하위 |
| `docs/guides/` | **어떻게** 만드는가 — 프론트엔드 구현 패턴 | PRD 하위 |

## 사전 요구사항

- JDK 21
- Node.js 20+
- PostgreSQL (로컬 설치, 스키마 `todolist_db` · 테스트 `todolist_db_test`)

## 로컬 개발 환경 실행

### 백엔드 (`todo-backend/`, Maven Wrapper)

```bash
./mvnw compile                              # 컴파일
./mvnw spring-boot:run                      # 실행 (DB_PASSWORD 환경변수 필수)
./mvnw test                                 # 전체 테스트
```

Windows PowerShell에서는 `.\mvnw.cmd`를 사용합니다.

설정은 `application.yml`(공통) + `application-local.yml`(로컬 전용, 커밋 금지) + `application-prod.yml`(운영) 프로파일로 분리되어 있으며, 로컬 실행 시 `application-local.yml`이 요구하는 `DB_PASSWORD` 등 환경변수를 직접 주입해야 합니다.

### 프론트엔드 (`todo-frontend/`)

```bash
npm run dev          # 개발 서버
npm run build         # 프로덕션 빌드
npm run typecheck     # tsc --noEmit
npm run lint          # eslint --max-warnings=0
npm run validate      # typecheck + lint + format:check (커밋 전 전체 검증)
```

## 작업 파일 규약

Phase 내부의 세부 작업은 [`/tasks/`](tasks/) 디렉터리에 `XXX-description.md` 형식으로 관리합니다. 형식 예시는 [`tasks/000-sample.md`](tasks/000-sample.md)를 참고하세요.
