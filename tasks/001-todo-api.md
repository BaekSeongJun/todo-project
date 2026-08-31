# 001 · Todo API

## 고수준 명세

Todo에 대한 CRUD API 6종을 완성해 백엔드 핵심 기능을 마무리한다. 목록 조회는 서버 페이지네이션·상태 필터·정렬을 지원하고, 저장되는 HTML 콘텐츠는 jsoup으로 정제해 XSS를 차단하며, 타인 소유 리소스 접근은 존재 여부를 노출하지 않도록 404로 통일한다.

- 관련 FR ID: `FR-T02` ~ `FR-T07`, `FR-L01` ~ `FR-L04`, `FR-U08`
- 담당 Phase: `docs/ROADMAP.md` Phase 4 · Todo API (M1 — 핵심 MVP)
- 선행 Phase: Phase 2(인증) — JWT 필터·`SecurityConfig`·`ApiResponse<T>`를 그대로 재사용

## 관련 파일

- `todo-backend/src/main/java/com/example/todo/controller/TodoController.java` — API 6종 엔드포인트(`GET`/`POST /api/todos`, `GET`/`PUT`/`DELETE /api/todos/{id}`, `PATCH /api/todos/{id}/toggle`)
- `todo-backend/src/main/java/com/example/todo/service/TodoService.java` — CRUD 비즈니스 로직, `private getOwnedTodo` 소유권 검증 헬퍼
- `todo-backend/src/main/java/com/example/todo/repository/TodoRepository.java` — `findByIdAndUserId`, `findAllByUserIdAndCompletedOptional`
- `todo-backend/src/main/java/com/example/todo/dto/` — `TodoCreateRequest`·`TodoUpdateRequest`·`TodoResponse`·`TodoSortBy`
- `todo-backend/src/main/java/com/example/todo/util/HtmlSanitizer.java` — jsoup `Safelist` 기반 HTML 정제기
- `todo-backend/src/main/java/com/example/todo/entity/Todo.java` — `toggle()`·`update()` 도메인 메서드(이번 Phase에서 추가)
- `todo-backend/src/main/java/com/example/common/response/PageResponse.java` — 목록 응답 계약(PRD 8.1)
- `todo-backend/src/main/java/com/example/common/exception/TodoNotFoundException.java` — 소유권 검증 실패 예외(404)
- `todo-backend/src/main/java/com/example/common/entity/BaseEntity.java` — 참고용, `markDeleted()`/`isDeleted()` 재사용
- `todo-backend/pom.xml` — jsoup 1.18.3 의존성

## 수락 기준

- [x] 목록이 기본 size 10 · `page` 0-base · 기본 정렬 `createdAt desc`로 동작하고 `PageResponse<T>` 형식으로 응답한다 (FR-L01, FR-L02)
- [x] 상태 필터(`all`/`pending`/`completed`)와 정렬(`createdAt`/`dueDate`) 쿼리 파라미터가 화이트리스트로 동작한다 (FR-L03, FR-L04)
- [x] Soft Delete 후 목록·단건 조회 모두에서 제외된다 (FR-T06)
- [x] 타인 리소스 접근 시 404를 반환한다 (FR-T07)
- [x] `<script>`·`on*` 속성·`javascript:` 링크·`<img>` 태그가 제거되고, 허용 서식 태그는 보존된다 (FR-T03)
- [x] `./mvnw test`가 전체 통과한다(44건, Failures 0 · Errors 0 · Skipped 0)
- [x] 로컬 서버 실기동으로 Swagger 6종 노출, 토큰 없이 401, 실제 토큰으로 CRUD 전 과정이 `ApiResponse<T>` 형식으로 동작함을 확인한다
- [x] `docs/PRD.md` 1.3 표의 jsoup 설치 상태를 갱신한다

## 구현 단계

1. `pom.xml`에 jsoup 1.18.3 의존성 추가 및 `TodoNotFoundException`·`PageResponse<T>` 공통 컴포넌트 작성
2. `Todo` 엔티티에 `toggle()`·`update()` 도메인 메서드 추가, `TodoRepository`에 소유권 조회·필터 메서드 추가(각각 단위/슬라이스 테스트로 검증)
3. `HtmlSanitizer`(jsoup `Safelist` 화이트리스트) 작성 및 XSS 벡터별 단위 테스트 작성
4. `todo.dto` 4종(`TodoCreateRequest`/`TodoUpdateRequest`/`TodoResponse`/`TodoSortBy`) 작성
5. `TodoService` 작성 — 생성 시 정제, 단건 접근 시 소유권 검증, 목록 조회 시 필터·페이지네이션 조합
6. `TodoController` 작성 — API 6종, `Authentication`에서 userId 추출, `ApiResponse<T>` 직접 반환
7. `TodoServiceTest`·`HtmlSanitizerTest`·`TodoRepositoryTest`로 소유권 404·Soft Delete 제외·필터·정렬·정제 4개 시나리오를 회귀 테스트로 고정
8. 로컬 서버(`spring-boot:run`)를 실제로 기동해 Swagger 노출·401·전체 CRUD·타인 소유 404를 curl로 통합 검증
9. `docs/PRD.md`·`docs/ROADMAP.md` 동기화, `todo-backend` 저장소에 Phase 단위로 커밋(`[Phase 4] Todo CRUD API 구현` 형태), 루트 저장소에 문서 변경 커밋
