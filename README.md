# p002

[jhs512/p-14172-2](https://github.com/jhs512/p-14172-2)의 `068` 커밋에서 시작한 프로젝트입니다.

## 69강 : 문제 소개

### 전제

- AI를 활용해도 되지만, "문제를 풀어줘, 해줘" 식은 금지
- 모든 구현은 TDD를 따른다.
- 테스트 케이스를 나중에 추가해도 된다.
- 로그인/로그아웃 기능은 구현하지 않는다.
- 특정 요청을 1번 회원으로서 진행하고 싶다면 주소에 `?actorId=1`을 붙인다.
- actorId가 필요한 액션 : 내 정보, 글 작성, 글 수정, 글 삭제, 댓글 작성, 댓글 수정, 댓글 삭제

### 예외 클래스

| 예외 | 발생 조건 |
| --- | --- |
| `com.back.global.GlobalExceptionHandler.MemberDuplicateUsernameException` | 회원가입 시 중복된 username이 입력되었다면 |
| `com.back.global.GlobalExceptionHandler.UnauthenticatedException` | actorId가 필요한 액션에서 actorId 파라미터가 없거나, 존재하지 않는 actorId라면 |
| `com.back.global.GlobalExceptionHandler.AccessDeniedException` | 글/댓글의 수정/삭제에서 actorId가 작성자가 아니라면 |

인증 실패 (`throw new UnauthenticatedException();`)

- 응답코드 : 401
- 응답본문 : `{"resultCode": "401-1", "msg": "로그인 후 이용해주세요.", "data": null}`

권한이 없을 때

- 응답코드 : 403
- 응답본문 : `{"resultCode": "403-1", "msg": "권한이 없습니다.", "data": null}`

### 문제 1 : 회원가입 API

`POST /api/v1/members/join`

- 요청본문 : `{"username": "string", "password": "string", "name": "string"}`
- 요청본문 제약 : username(Size(4, 30), 유니크), password(Size(8, 30)), name(Size(2, 30))
- 응답상태 : 201
- 응답본문 : `{"resultCode": "201-1", "msg": "%s님 환영합니다. 회원가입이 완료되었습니다.", "data": null}`

힌트 1 : 중복된 username이 입력되었다면? `throw new MemberDuplicateUsernameException(reqBody.username);`

- 응답코드 : 409
- 응답본문 : `{"resultCode": "409-1", "msg": "%s(은)는 이미 사용중인 username 입니다.", "data": null}`

### 문제 2 : 내 정보 API

`GET /api/v1/members/me?actorId={작성자-번호}`

- 응답상태 : 200
- 응답본문 : `{"id": 번호, "createDate": "날짜", "updateDate": "날짜", "username": "string", "name": "string"}`
- MemberDto에는 비밀번호가 없어야 한다.

### 문제 3 : 샘플 데이터 생성할 때 회원 6명도 만든다.

- system(시스템), admin(관리자), user1(유저1), user2(유저2), user3(유저3), user4(유저4)
- 비밀번호는 `12345678`로 한다.

### 문제 4 : 모든 글과 댓글은 작성자(author 필드)를 가진다.

| 대상 | 작성자 |
| --- | --- |
| 1번 글 | user1 |
| 2번 글 | user2 |
| 3번 글 | user3 |
| 1번 댓글 | user1 |
| 2번 댓글 | user2 |
| 3번 댓글 | user3 |
| 4번 댓글 | user1 |
| 5번 댓글 | user2 |

### 문제 5 : 글 작성 시 작성자 정보가 필요하다.

`POST /api/v1/posts?actorId={작성자-번호}`

### 문제 6 : 댓글 작성 시 작성자 정보가 필요하다.

`POST /api/v1/posts/{글-번호}/comments?actorId={작성자-번호}`

### 문제 7 : 글 삭제 시 작성자 본인이어야 한다.

`DELETE /api/v1/posts/{글-번호}?actorId={작성자-번호}`

### 문제 8 : 글 수정 시 작성자 본인이어야 한다.

`PUT /api/v1/posts/{글-번호}?actorId={작성자-번호}`

### 문제 9 : 댓글 삭제 시 작성자 본인이어야 한다.

`DELETE /api/v1/posts/{글-번호}/comments/{댓글-번호}?actorId={작성자-번호}`

### 문제 10 : 댓글 수정 시 작성자 본인이어야 한다.

`PUT /api/v1/posts/{글-번호}/comments/{댓글-번호}?actorId={작성자-번호}`
