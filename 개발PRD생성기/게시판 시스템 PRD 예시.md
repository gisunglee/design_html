# 🚀 게시판 관리 시스템 - Cursor AI 최적화 PRD

## 📋 프로젝트 개요
**담당 개발자:** 김개발
**작성일:** 2024-12-23

회원제 게시판 시스템으로 사용자 인증, 게시글 CRUD, 댓글, 파일 첨부, 관리자 기능을 제공합니다. Spring Boot와 JSP를 활용하여 안정적이고 확장 가능한 웹 애플리케이션을 구현합니다.

## ⚙️ 기술 스택 및 개발 표준

### 🔧 기술 스택
- **Backend:** Spring Boot 2.7.x
- **Frontend:** JSP, Bootstrap 5, jQuery
- **Database:** MySQL 8.0
- **Server:** Apache Tomcat 9.0

### 📁 패키지 구조
```
com.company.board
├── controller (웹 컨트롤러)
│   ├── api (REST API 컨트롤러)
│   └── web (JSP 뷰 컨트롤러)
├── service (비즈니스 로직)
├── repository (데이터 접근)
├── dto (데이터 전송 객체)
│   ├── request (요청 DTO)
│   └── response (응답 DTO)
├── entity (JPA 엔티티)
├── config (설정 클래스)
├── exception (커스텀 예외)
└── util (유틸리티 클래스)
```

### 🛡️ Exception 처리 표준
- GlobalExceptionHandler를 사용한 전역 예외 처리
- 커스텀 Exception 클래스 생성 (BoardException, UserException)
- 에러 코드 체계: ERR001(인증 실패), ERR002(권한 없음), ERR003(데이터 없음)
- ResponseEntity로 일관된 에러 응답 반환
- 로그는 ERROR 레벨로 기록, 사용자에게는 친화적 메시지 제공

### 🎯 Controller 응답 표준
- REST API: ResponseEntity<ApiResponse<T>> 사용
- 성공: HttpStatus.OK, 데이터 포함
- 실패: 적절한 HTTP 상태코드, 에러메시지 포함
- JSP 페이지: ModelAndView 사용
- JSON 응답 형태: `{"success": true, "message": "", "data": {}}`

### 💾 쿼리 작성 표준
- JPA Query Methods 우선 사용 (findByTitleContaining 등)
- 복잡한 쿼리는 @Query 어노테이션 활용
- Native Query는 최소화, 필요시 성능상 이유 명시
- 모든 파라미터는 @Param 어노테이션 사용
- 페이징은 Pageable과 Page<T> 활용
- 삭제는 논리삭제(deleted_yn) 우선 적용

## 🗄️ 데이터베이스 설계

### 📊 tb_user (사용자 테이블)

| 컬럼명 | 데이터타입 | 제약조건 | 설명 |
|--------|------------|----------|------|
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 사용자 ID |
| username | VARCHAR(50) | UNIQUE, NOT NULL | 로그인 ID |
| password | VARCHAR(255) | NOT NULL | 암호화된 비밀번호 |
| email | VARCHAR(100) | UNIQUE, NOT NULL | 이메일 |
| nickname | VARCHAR(50) | NOT NULL | 닉네임 |
| role | VARCHAR(20) | DEFAULT 'USER' | 권한 (USER, ADMIN) |
| created_at | DATETIME | NOT NULL | 생성일시 |
| updated_at | DATETIME | | 수정일시 |
| deleted_yn | CHAR(1) | DEFAULT 'N' | 삭제여부 |

### 📊 tb_board (게시판 테이블)

| 컬럼명 | 데이터타입 | 제약조건 | 설명 |
|--------|------------|----------|------|
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 게시글 ID |
| title | VARCHAR(200) | NOT NULL | 제목 |
| content | TEXT | NOT NULL | 내용 |
| writer_id | BIGINT | NOT NULL, FK(tb_user.id) | 작성자 ID |
| view_count | INT | DEFAULT 0 | 조회수 |
| created_at | DATETIME | NOT NULL | 생성일시 |
| updated_at | DATETIME | | 수정일시 |
| deleted_yn | CHAR(1) | DEFAULT 'N' | 삭제여부 |

### 📊 tb_comment (댓글 테이블)

| 컬럼명 | 데이터타입 | 제약조건 | 설명 |
|--------|------------|----------|------|
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 댓글 ID |
| board_id | BIGINT | NOT NULL, FK(tb_board.id) | 게시글 ID |
| writer_id | BIGINT | NOT NULL, FK(tb_user.id) | 작성자 ID |
| content | TEXT | NOT NULL | 댓글 내용 |
| created_at | DATETIME | NOT NULL | 생성일시 |
| updated_at | DATETIME | | 수정일시 |
| deleted_yn | CHAR(1) | DEFAULT 'N' | 삭제여부 |

### 📊 tb_file (첨부파일 테이블)

| 컬럼명 | 데이터타입 | 제약조건 | 설명 |
|--------|------------|----------|------|
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 파일 ID |
| board_id | BIGINT | NOT NULL, FK(tb_board.id) | 게시글 ID |
| original_name | VARCHAR(255) | NOT NULL | 원본 파일명 |
| stored_name | VARCHAR(255) | NOT NULL | 저장된 파일명 |
| file_path | VARCHAR(500) | NOT NULL | 파일 경로 |
| file_size | BIGINT | NOT NULL | 파일 크기 |
| content_type | VARCHAR(100) | NOT NULL | MIME 타입 |
| created_at | DATETIME | NOT NULL | 업로드일시 |

## ⭐ 기능 명세서

### 🔥 사용자 인증 관리

**설명:** Spring Security를 활용한 회원가입, 로그인, 권한 관리 기능

**구현 단계:**
1. User Entity 및 UserDetails 구현
2. Spring Security 설정 (SecurityConfig)
3. 회원가입 API 및 JSP 페이지 구현
4. 로그인/로그아웃 처리
5. 권한별 접근 제어 설정

**API 명세:**
```
POST /api/auth/register
Request: { username, password, email, nickname }
Response: { success, message, userId }

POST /api/auth/login
Request: { username, password }
Response: { success, message, token }

POST /api/auth/logout
Response: { success, message }
```

---

### 🔥 게시글 목록 조회

**설명:** 페이징, 검색, 정렬 기능이 포함된 게시글 목록 조회

**구현 단계:**
1. 검색 조건 처리 (제목, 내용, 작성자)
2. 페이징 파라미터 설정 (기본 20개씩)
3. 정렬 옵션 처리 (최신순, 조회수순)
4. Repository 쿼리 메소드 구현
5. JSP에서 목록 출력 및 페이징 UI

**API 명세:**
```
GET /api/board
Params: page(0), size(20), search, searchType(title/content/writer), sort(created_at/view_count)
Response: { 
  content: [게시글 배열], 
  totalElements, 
  totalPages, 
  currentPage,
  hasNext,
  hasPrevious
}
```

---

### 🔥 게시글 작성

**설명:** 제목, 내용, 파일 첨부가 가능한 게시글 작성 기능

**구현 단계:**
1. 게시글 작성 폼 JSP 페이지 구현
2. 클라이언트 사이드 유효성 검증 (제목/내용 필수)
3. 파일 업로드 처리 (MultipartFile)
4. 게시글 데이터 저장 (트랜잭션 처리)
5. 파일 정보 저장 및 물리적 파일 저장
6. 성공시 목록 페이지로 리다이렉트

**API 명세:**
```
POST /api/board
Request: { title, content, files[] }
Content-Type: multipart/form-data
Response: { success, message, boardId }
```

---

### ⭐ 게시글 상세 조회

**설명:** 게시글 내용과 댓글, 첨부파일을 조회하고 조회수 증가

**구현 단계:**
1. 게시글 ID로 기본 정보 조회
2. 조회수 증가 처리 (동일 세션 중복 방지)
3. 댓글 목록 조회 (작성자 정보 포함)
4. 첨부파일 목록 조회
5. 상세 페이지 JSP 구현 (수정/삭제 권한 체크)

**API 명세:**
```
GET /api/board/{id}
Response: { 
  board: {게시글 정보},
  comments: [댓글 배열],
  attachments: [첨부파일 배열],
  canEdit: boolean,
  canDelete: boolean
}
```

---

### ⭐ 게시글 수정

**설명:** 작성자 또는 관리자만 게시글 수정 가능

**구현 단계:**
1. 권한 체크 (작성자 본인 또는 관리자)
2. 기존 데이터 조회 및 폼에 표시
3. 수정 데이터 유효성 검증
4. 첨부파일 추가/삭제 처리
5. 게시글 정보 업데이트
6. 수정 완료 후 상세 페이지로 이동

**API 명세:**
```
PUT /api/board/{id}
Request: { title, content, files[], deleteFileIds[] }
Response: { success, message }
```

---

### ⭐ 게시글 삭제

**설명:** 논리삭제를 통한 게시글 삭제 (관리자는 물리삭제 가능)

**구현 단계:**
1. 권한 체크 (작성자 본인 또는 관리자)
2. 삭제 확인 모달 표시
3. 논리삭제 처리 (deleted_yn = 'Y')
4. 관련 댓글도 함께 논리삭제
5. 첨부파일은 보관 (관리자 판단)
6. 목록 페이지로 리다이렉트

**API 명세:**
```
DELETE /api/board/{id}
Response: { success, message }
```

---

### 📝 댓글 관리

**설명:** 게시글에 대한 댓글 작성, 수정, 삭제 기능

**구현 단계:**
1. 댓글 작성 폼 구현 (AJAX)
2. 댓글 목록 실시간 업데이트
3. 댓글 수정/삭제 권한 체크
4. 댓글 페이징 처리 (10개씩)
5. 대댓글 기능 (선택사항)

**API 명세:**
```
POST /api/comment
Request: { boardId, content }
Response: { success, message, commentId }

PUT /api/comment/{id}
Request: { content }
Response: { success, message }

DELETE /api/comment/{id}
Response: { success, message }
```

---

### 📝 파일 다운로드

**설명:** 첨부파일 다운로드 및 접근 권한 제어

**구현 단계:**
1. 파일 ID로 파일 정보 조회
2. 파일 존재 여부 확인
3. 다운로드 권한 체크
4. 파일 스트림 응답
5. 다운로드 로그 기록

**API 명세:**
```
GET /api/file/download/{fileId}
Response: File Stream with proper headers
```

## 💼 비즈니스 로직 및 제약사항

### 🔐 인증/권한 요구사항
- 모든 게시글 작성/수정/삭제는 로그인 필수
- 게시글 조회는 비로그인 사용자도 가능
- 관리자는 모든 게시글/댓글 관리 권한
- 세션 타임아웃: 30분 (Remember-me 7일)
- 비밀번호는 BCrypt로 암호화 저장

### ✅ 데이터 검증 규칙
- 제목: 필수, 1-200자, HTML 태그 제거
- 내용: 필수, 최대 5000자, XSS 방지 처리
- 파일: 최대 10MB, jpg/png/pdf/doc/xlsx만 허용
- 이메일: 유효한 이메일 형식, 중복 불가
- 사용자명: 영문+숫자 조합, 4-20자, 중복 불가

### 📋 비즈니스 규칙
- 게시글 작성자만 수정/삭제 가능 (관리자 예외)
- 삭제된 게시글은 관리자만 확인 가능
- 댓글 작성시 알림 기능 (선택사항)
- 첨부파일은 게시글 삭제시에도 보관
- 같은 IP에서 동일 게시글 조회수는 24시간 1회만 증가

### 🚀 성능 요구사항
- 페이지 로딩: 3초 이내
- 동시 접속: 500명
- 게시글 목록: 페이지당 20개씩
- 파일 업로드: 진행률 표시, 최대 10MB
- 데이터베이스 인덱스: title, created_at, writer_id

### ⚠️ 예외 상황 처리
- 파일 업로드 실패시 사용자 알림 및 재시도 옵션
- 네트워크 오류시 자동 재시도 (최대 3회)
- 권한 없는 접근시 로그인 페이지 이동
- 데이터베이스 오류시 관리자 알림 및 사용자 안내
- 세션 만료시 현재 작성 내용 임시 저장

## 🧪 테스트 시나리오

### 🔬 단위 테스트
- UserService 메소드별 기능 테스트
- BoardService CRUD 로직 테스트
- FileService 업로드/다운로드 테스트
- Repository 쿼리 메소드 테스트
- Validation 어노테이션 검증 테스트
- Exception Handler 동작 테스트

### 🔗 통합 테스트
- Controller API 엔드포인트 전체 테스트
- Spring Security 인증/권한 테스트
- 데이터베이스 트랜잭션 롤백 테스트
- 파일 업로드/다운로드 전체 플로우 테스트
- JSP 페이지 렌더링 테스트

### 👤 사용자 시나리오 테스트
1. **회원가입 → 로그인 → 게시글 작성 → 수정 → 삭제**
2. **비로그인 사용자 게시글 조회 → 댓글 작성 시도 (실패)**
3. **파일 첨부 게시글 작성 → 다운로드 → 파일 삭제**
4. **관리자 로그인 → 모든 게시글 관리**
5. **검색 기능 → 페이징 → 정렬 변경**

### 📊 성능 테스트
- 대용량 데이터 조회 성능 (게시글 10만건)
- 동시 접속자 부하 테스트 (500명)
- 파일 업로드 성능 테스트 (10MB 파일)
- 메모리 사용량 모니터링 (힙 메모리 1GB 이하)
- 데이터베이스 커넥션 풀 최적화

## 🎯 Cursor AI 구현 가이드

### 📝 구현 순서
1. **Entity 및 Repository 생성** - 데이터베이스 스키마 기반
2. **Security 설정** - 인증/권한 처리
3. **Service 계층 구현** - 비즈니스 로직 포함
4. **Controller 구현** - API 명세 기준
5. **JSP 화면 개발** - Bootstrap 활용
6. **Exception Handler 구성** - 표준 에러 처리
7. **테스트 코드 작성** - 단위/통합 테스트
8. **성능 최적화** - 인덱스, 캐시 적용

### 💡 중요 고려사항
- 모든 코드는 정의된 표준을 준수해야 합니다
- Exception 처리는 GlobalExceptionHandler를 활용하세요
- Controller 응답은 ApiResponse 형태로 일관성 유지
- 데이터 검증은 Bean Validation (@Valid) 활용
- 페이징과 검색 기능을 모든 목록에 포함
- XSS, SQL Injection 방지 필수 적용
- 파일 업로드시 보안 검증 철저히 수행

### 🔍 완료 체크리스트
- [ ] User, Board, Comment, File Entity 생성 및 관계 설정
- [ ] Spring Security 설정 및 인증 구현
- [ ] Repository 인터페이스 및 쿼리 메소드 정의
- [ ] Service 비즈니스 로직 구현 (트랜잭션 포함)
- [ ] REST API Controller 구현 (에러 처리 포함)
- [ ] JSP 화면 구현 (Bootstrap, jQuery 활용)
- [ ] 파일 업로드/다운로드 기능 구현
- [ ] GlobalExceptionHandler 구현
- [ ] 단위 테스트 작성 (Service, Repository)
- [ ] 통합 테스트 작성 (Controller, Security)
- [ ] 성능 최적화 (인덱스, 쿼리 튜닝)
- [ ] 보안 검증 (XSS, CSRF, 파일 업로드)

---

**📌 이 PRD는 Cursor AI에 최적화되어 작성되었습니다. 각 단계별로 순차적으로 구현하시기 바랍니다.**

### 🚀 Phase 1 구현 예시
```
Cursor AI에게 다음과 같이 요청하세요:

"위 PRD를 기반으로 1단계 Entity 클래스들을 생성해주세요. 
User, Board, Comment, File Entity를 JPA 어노테이션과 함께 구현하고, 
Entity 간 관계도 설정해주세요. 
표준에 따라 BaseEntity도 함께 만들어주세요."
``` 