# ToPeak API 명세서

## 1. 문서 개요

학술제 1차 제출과 이후 개발 관리를 위한 최신 코드 기준 명세서다. 최신 FE/BE 분석 결과에 서비스 방향 검토사항을 반영했다. 분석 및 검토 기준일은 2026-10-03이며, 다음 소스 상태를 기준으로 Controller, Security, DTO, Entity, Service, Repository, Flyway, 테스트 및 FE 호출부를 조사했다.

- BE: `e7fb484b53ed71d02589ca9a49afeb0b6149df19`
- FE: `7b5659b32f08ebf4843df3c1691809cfabed86da`
- BE 패키지: `com.ggmount`. 서비스명 ToPeak와 Java 패키지명은 다르다.
- 코드 정적 분석 결과다. 운영 DB의 실제 데이터, migration 적용 여부, 외부 OAuth/날씨 서비스의 실응답을 확인한 결과는 아니다.

### 개발 주소와 인증

BE는 별도 port 설정이 없는 Spring Boot 기본 환경에서 `http://localhost:8080`, FE 기본 개발 주소는 `http://localhost:5173`이다. FE Vite proxy는 `/api`, `/oauth2`, `/login`을 BE에 전달한다. 배포 주소는 코드만으로 확정할 수 없다.

인증은 **Spring Security OAuth2 + 서버 세션**이다. JWT 발급/갱신 API는 없다. FE는 요청에 `credentials: 'include'`를 사용한다. 인증 후 세션 쿠키를 전달하며, 변경 요청은 `GET /api/csrf`에서 받은 `headerName`과 `token`을 헤더에 넣는다. 로그아웃에도 CSRF가 필요하다.

Security의 공개 허용 경로는 `/oauth2/authorization/**`, `/login/**`, `/error`, `/api/csrf`다. 그 외는 인증이 필요하다. 따라서 이 문서에서 **공개 프로필/공개 일지**란 다른 로그인 사용자에게 공개된다는 의미다. 비로그인 공개 API라는 의미가 아니다. 인증되지 않은 변경 요청은 CSRF 검사 순서에 따라 401보다 먼저 403을 받을 수 있다.

### 상태와 집계 기준

- **구현 완료**: 실제 Controller Mapping 또는 설정된 Security 처리 경로가 있다. FE 연결, 점수 계산, 서버 GPS 검증 완료까지 뜻하지 않는다.
- **구현 예정**: 별도 endpoint의 추가가 확정된 경우다. 현재 검토 결과 해당 API는 **0개**다.
- **FE 미연결**: BE API는 구현 완료지만 FE 호출이 연결되지 않은 상태다.
- **향후 기능 개선**: 기존 기능의 UX 또는 데이터 제공 방식 개선이다. 확정된 신규 API와 구분한다.
- **내부 비즈니스 로직 미구현**: 기존 API 흐름에 필요한 서버 내부 처리의 미구현이다. 신규 API를 의미하지 않는다.
- **부분 구현**: 이번 조사에서 분류한 API에는 없다. API의 제공 여부와 내부 로직의 미구현을 분리한다.
- Controller API 22개와 Security 경로 5개를 구분한다. Google/Kakao 시작·callback은 provider별 실제 경로로 각각 집계한다. 이미지 `kind`, 목록 항목 수, framework 기본 로그인/오류 페이지는 별도 API로 늘려 세지 않는다.

JSON 예시의 ID, 이름, URL, 수치는 형식 설명용이며 실제 DB 조회 결과를 의미하지 않는다. Java 정수/실수는 JSON number, 날짜는 `YYYY-MM-DD`, `Instant`는 UTC 등의 offset이 포함된 ISO-8601 문자열이다. 날씨의 `LocalDateTime`은 서울 현지 시각이며 offset이 없다. nullable 필드는 예시에 생략하지 않고 `null`로 표현할 수 있다.

## 2. API 전체 목록

| ID | 분류 | API | Method | Endpoint | 인증 | 구현 상태 |
| --- | --- | --- | --- | --- | --- | --- |
| A01 | 인증/Controller | CSRF 조회 | GET | `/api/csrf` | 불필요 | 구현 완료 |
| A02 | 인증/Security | Google 로그인 시작 | GET | `/oauth2/authorization/google` | 불필요 | 구현 완료 |
| A03 | 인증/Security | Kakao 로그인 시작 | GET | `/oauth2/authorization/kakao` | 불필요 | 구현 완료 |
| A04 | 인증/Security | Google callback | GET | `/login/oauth2/code/google` | 기존 로그인 불필요, OAuth state 검증 | 구현 완료 |
| A05 | 인증/Security | Kakao callback | GET | `/login/oauth2/code/kakao` | 기존 로그인 불필요, OAuth state 검증 | 구현 완료 |
| A06 | 인증/Security | 로그아웃 | POST | `/api/logout` | 세션 종료, CSRF 필요 | 구현 완료 |
| U01 | 회원 | 내 정보 조회 | GET | `/api/users/me` | 필요 | 구현 완료 |
| U02 | 회원 | 최초 설정/프로필 수정 | PATCH | `/api/users/me/profile` | 필요 | 구현 완료 |
| U03 | 회원 | 공개 프로필 조회 | GET | `/api/users/{id}/profile` | 필요 | 구현 완료 |
| U04 | 이미지 | 내 이미지 조회 | GET | `/api/users/me/images/{kind}` | 필요 | 구현 완료 |
| U05 | 이미지 | 내 이미지 저장 | PUT | `/api/users/me/images/{kind}` | 필요 | 구현 완료 |
| U06 | 이미지 | 내 이미지 삭제 | DELETE | `/api/users/me/images/{kind}` | 필요 | 구현 완료 |
| M01 | 산 | 산 목록 | GET | `/api/mountains` | 필요 | 구현 완료 |
| M02 | 코스 | 산별 코스 목록 | GET | `/api/mountains/{id}/courses` | 필요 | 구현 완료 |
| M03 | 코스 | 코스 상세/경로 | GET | `/api/courses/{id}` | 필요 | 구현 완료 |
| W01 | 날씨 | 산별 날씨 | GET | `/api/mountains/{id}/weather` | 필요 | 구현 완료 |
| H01 | 등산기록 | 산행 결과 저장 | POST | `/api/users/me/hiking-records` | 필요, 프로필 완료 | 구현 완료 |
| H02 | 등산기록 | 내 기록 목록 | GET | `/api/users/me/hiking-records` | 필요 | 구현 완료 |
| H03 | 등산기록 | 내 기록 상세 | GET | `/api/users/me/hiking-records/{id}` | 필요 | 구현 완료 |
| J01 | 등산일지 | 기록 기반 일지 생성 | POST | `/api/journals` | 필요, 프로필 완료 | 구현 완료 |
| J02 | 등산일지 | 내 일지 목록 | GET | `/api/users/me/journals` | 필요 | 구현 완료 |
| J03 | 등산일지 | 공개 일지 목록 | GET | `/api/journals` | 필요 | 구현 완료 |
| J04 | 등산일지 | 사용자 공개 일지 목록 | GET | `/api/users/{id}/journals` | 필요 | 구현 완료 |
| J05 | 등산일지 | 일지 상세 | GET | `/api/journals/{id}` | 필요 | 구현 완료 |
| J06 | 등산일지 | 일지 수정 | PATCH | `/api/journals/{id}` | 필요, 프로필 완료 | 구현 완료 |
| J07 | 등산일지 | 일지 삭제 | DELETE | `/api/journals/{id}` | 필요, 프로필 완료 | 구현 완료 |
| R01 | 랭킹 | 랭킹 조회 | GET | `/api/rankings` | 필요 | 구현 완료 |

아래 상세에서 별도 명시하지 않은 Path/Query/Request는 **없음**이다. JSON body를 전송하는 변경 요청은 `application/json`을 사용한다. 이미지 업로드는 `multipart/form-data`를 사용하며, body가 없는 DELETE 등의 요청에는 Content-Type이 반드시 필요하지 않다. Controller 변경 요청에는 CSRF가 필요하다. 일반적인 Security 오류(401/403)는 각 API의 주요 업무 상태 코드와 함께 적용한다. 204와 redirect는 JSON 응답이 아니다.

## 3. 인증 API

### A01. CSRF 조회

- **GET `/api/csrf` / 인증 불필요 / 구현 완료**
- 설명: Spring Security `CsrfToken`의 실제 헤더명과 토큰을 전달한다.
- Path/Query/Request Body: 없음.
- Response: `Map<String,String>`, 두 필드 모두 문자열. 주요 상태: 200.

```json
{
  "headerName": "X-CSRF-TOKEN",
  "token": "예시-CSRF-토큰"
}
```

변경 요청 헤더에는 반환된 `headerName`을 그대로 사용한다. 세션 변경/로그아웃 후 기존 토큰을 영구 재사용하는 규약은 없다.

### A02. Google 로그인 시작

- **GET `/oauth2/authorization/google` / 인증 불필요 / 구현 완료(Security)**
- 설명: 브라우저 이동으로 Google OAuth2 인증을 시작한다. Google 등록 scope는 `profile`, `email`이다.
- Path/Query/Request Body: 앱이 직접 지정하는 파라미터 없음. provider용 인가는 Security가 구성한다.
- Response Body: 애플리케이션 JSON 없음. 주요 상태: 302, provider 인증 화면으로 redirect.

### A03. Kakao 로그인 시작

- **GET `/oauth2/authorization/kakao` / 인증 불필요 / 구현 완료(Security)**
- 설명: 브라우저 이동으로 Kakao OAuth2 인증을 시작한다. 코드에서 명시한 Kakao scope는 없으며 동의 범위는 provider 앱 설정의 영향을 받는다.
- Path/Query/Request Body: 앱이 직접 지정하는 파라미터 없음.
- Response Body: 애플리케이션 JSON 없음. 주요 상태: 302, provider 인증 화면으로 redirect.

### A04. Google callback

- **GET `/login/oauth2/code/google` / 사전 로그인 불필요 / 구현 완료(Security)**
- 설명: provider 응답을 받아 저장된 OAuth 요청/state를 검증하고 세션 인증을 생성한다. 일반 FE 저장 API가 아니다.
- Path: 없음. Query: 성공 시 provider가 전달하는 `code`, `state` 문자열; 실패 시 `error` 등 OAuth 오류 값. Request Body: 없음.
- Response: 성공 302, `${FRONTEND_URL}/?oauth=success`로 redirect(기본 `http://localhost:5173`). 실패 401, 아래 JSON. JWT나 access token을 FE URL로 반환하지 않는다.

```json
{
  "error": "oauth_login_failed"
}
```

### A05. Kakao callback

- **GET `/login/oauth2/code/kakao` / 사전 로그인 불필요 / 구현 완료(Security)**
- Path: 없음. Query: provider의 `code`, `state` 또는 OAuth 오류 값. Request Body: 없음.
- 설명/Response: A04와 같은 Handler를 사용하며 성공 302, 실패 401이다. 실패 JSON은 A04 예시와 동일하다.

회원 식별은 `(provider, providerId)` 조합이다. Google은 `sub`, Kakao는 `id`를 사용하며 식별자가 없으면 인증 처리가 실패한다. 최초 로그인은 회원을 생성하고 최초 프로필 설정은 U02로 한다. 기존 회원 재로그인은 provider 이메일/사진을 갱신하지만 사용자가 설정한 닉네임·출생연도·프로필 완료 상태를 초기화하지 않는다. 동일 이메일만으로 서로 다른 provider 계정을 합치는 로직은 없다.

### A06. 로그아웃

- **POST `/api/logout` / 구현 완료(Security)**
- Path/Query/Request Body: 없음. CSRF 헤더 필요.
- 설명: 세션 무효화, 인증 해제, `JSESSIONID` 쿠키 삭제.
- Response Body: 없음. 주요 상태: 204, CSRF 오류 403.
- 로그아웃 필터는 일반 endpoint 인가보다 먼저 처리된다. 유효 CSRF를 가진 익명 요청에도 204가 가능하므로 반드시 401이 반환된다고 규정하지 않는다.

## 4. 회원 / 프로필 / 이미지 API

### 응답 모델

`MemberResponse`(U01/U02/U05):

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| userId | Long | 아니오 | 회원 ID |
| nickname | String | 가능 | 프로필 설정 전에는 미설정 가능 |
| birthYear | Integer | 가능 | 출생연도, 본인 응답에만 포함 |
| age | Integer | 가능 | 현재 연도 - 출생연도 + 1 |
| profileImageUrl | String | 가능 | 본인 업로드 이미지 우선, 없으면 OAuth 사진 |
| profileCompleted | boolean | 아니오 | 최초 설정 완료 여부 |
| score | Long | 가능 | 저장된 점수, 계산되지 않은 경우 null |
| backgroundImageUrl | String | 가능 | 본인 배경 이미지 URL |

아래는 U01/U02/U05에서 사용하는 동일 응답 예시다. `age`는 2026년 코드 계산 기준이다.

```json
{
  "userId": 7,
  "nickname": "산사람",
  "birthYear": 2003,
  "age": 24,
  "profileImageUrl": "/api/users/me/images/profile?v=123e4567-e89b-12d3-a456-426614174000",
  "profileCompleted": true,
  "score": null,
  "backgroundImageUrl": null
}
```

`PublicMemberResponse`(U03/R01):

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| userId | Long | 아니오 | 공개 프로필 이동용 ID |
| nickname | String | DB nullable | 공개 조회 대상은 프로필 완료 회원 |
| profileImageUrl | String | 가능 | OAuth provider 사진, 본인 업로드 이미지와 다름 |
| score | Long | 가능 | 저장된 점수 |

```json
{
  "userId": 7,
  "nickname": "산사람",
  "profileImageUrl": "https://example.com/provider-photo.jpg",
  "score": null
}
```

공개 DTO에는 출생연도, 나이, 이메일, provider명, providerId, 배경 이미지, 세션/OAuth 내부 정보가 없다. 업로드한 개인 프로필 이미지를 공개하는 API는 현재 없다. provider 사진 URL은 공개 응답에 들어갈 수 있다.

### U01. 내 정보 조회

- **GET `/api/users/me` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음.
- 설명: 현재 세션의 회원을 조회한다. 별도 `userId`를 받지 않는다. 프로필 미완료 회원도 조회 가능하다.
- Response: 위 `MemberResponse` JSON. `Cache-Control: no-store`. 주요 상태: 200, 인증 오류 401.

### U02. 최초 프로필 설정 / 프로필 수정

- **PATCH `/api/users/me/profile` / 인증 필요 / 구현 완료**
- Path/Query: 없음. Request: `MemberUpdateRequest`, 아래 두 필드 모두 필수다. PATCH이지만 한 필드만 보내는 부분 수정은 아니다.

| 필드 | 자료형 | 검증 |
| --- | --- | --- |
| nickname | String | null/공백 불가, 최대 30자; 저장 시 trim |
| birthYear | Integer | 필수, 1900 이상 현재 연도 이하 |

```json
{
  "nickname": "산사람",
  "birthYear": 2003
}
```

- 설명: 본인 이름·출생연도를 설정하고 프로필을 완료 처리한다. 닉네임 중복 금지 규칙은 없다. 이미지 수정은 U05/U06으로 한다.
- Response: 위 `MemberResponse` JSON. 주요 상태: 200, 입력 검증 400, 인증 401, CSRF 403.

### U03. 다른 사용자 공개 프로필

- **GET `/api/users/{id}/profile` / 인증 필요 / 구현 완료**
- Path: `id` Long, 대상 회원 ID. Query/Request Body: 없음.
- 설명: 대상 회원이 존재하고 프로필이 완료되어야 한다.
- Response: 위 `PublicMemberResponse` JSON. 주요 상태: 200, 대상 없음 404, 대상 프로필 미완료 403, 인증 401.

### U04. 내 이미지 조회

- **GET `/api/users/me/images/{kind}` / 인증 필요 / 구현 완료**
- Path: `kind` String, `profile` 또는 `background`. Query: `v` String, 선택적 revision. Request Body: 없음.
- 설명: 현재 회원의 이미지 bytes를 조회한다. `v` 생략 시 현재 이미지, 지정 시 저장된 revision과 같아야 한다. 프로필 완료는 필요하지 않다.
- Response: JSON 없음; `image/png` 또는 `image/jpeg` binary, `Cache-Control: no-store`, `X-Content-Type-Options: nosniff`.
- 주요 상태: 200, 이미지 없음/잘못된 kind/revision 불일치 404, 인증 401. 타 계정의 URL revision을 가져와도 그 계정의 bytes가 반환되지 않는다.

### U05. 내 이미지 저장

- **PUT `/api/users/me/images/{kind}` / 인증 필요 / 구현 완료**
- Path: `kind`는 U04와 같음. Query: 없음.
- Request Body: `multipart/form-data`, 필수 part `file`(`MultipartFile`). JSON 요청이 아니다. CSRF 필요.
- 설명: PNG/JPEG를 실제 디코딩·검증 후 재인코딩한다. 파일 5MiB 이하, 한 변 4096 이하, 총 16,000,000픽셀 이하. 요청 전체 제한은 설정상 6MB다. 프로필 완료는 필요하지 않다.
- Response: 위 `MemberResponse` JSON, `Cache-Control: no-store`.
- 주요 상태: 200, 비어 있거나 유효하지 않은 이미지/해상도 검증 400, 업로드 용량 초과 413, kind 오류 404, 인증 401, CSRF 403.

### U06. 내 이미지 삭제

- **DELETE `/api/users/me/images/{kind}` / 인증 필요 / 구현 완료**
- Path: `kind`는 U04와 같음. Query/Request Body: 없음.
- 설명: 본인 이미지 삭제. 이미지가 없어도 204이며 프로필 완료는 필요하지 않다. 프로필 업로드 삭제 후 본인 조회는 provider 사진으로 돌아갈 수 있다.
- Response Body: 없음. 주요 상태: 204, kind 오류 404, 인증 401, CSRF 403.

## 5. 산 / 코스 API

산·코스 Controller는 `JdbcTemplate`으로 Flyway 데이터에 접근한다. 이 영역에 별도 Entity/Service가 있다는 전제는 없다.

M01은 산 목록·선택용 기본 정보, M02는 선택한 산의 코스 목록, M03은 특정 코스의 상세·지도 경로를 제공한다. 코스 목록과 상세는 역할이 다르다. 별도의 산 상세 API는 추가하지 않는다. `region`, `address`, `description`을 위한 응답 설계도 포함하지 않는다. 특히 산 설명은 전체 산에 대한 일관되고 신뢰할 수 있는 수집 여부가 확정되지 않았다.

### M01. 산 목록

- **GET `/api/mountains` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음. 서버 검색·지역 필터 파라미터는 없다.
- 설명: 전체 산을 `sigungu_name`, `name` 순으로 반환한다. `city`는 DB `sigungu_name`, `courseCount`는 실제 연결된 코스 수다. V2 seed는 140개지만 응답 수는 현재 DB에 따른다.
- Response: `List<Mountain>`, 주요 상태 200, 인증 401.

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| id | long | 아니오 | 산 DB ID |
| name | String | 아니오 | 산 이름 |
| city | String | 아니오 | 시·군 이름 |
| height | double | 아니오 | 높이 m |
| lat | double | 아니오 | 정상 위도 |
| lng | double | 아니오 | 정상 경도 |
| courseCount | int | 아니오 | 코스 수 |

```json
[
  {
    "id": 1,
    "name": "예시산",
    "city": "예시시",
    "height": 800.0,
    "lat": 37.7,
    "lng": 127.3,
    "courseCount": 3
  }
]
```

FE 실등산 선택 화면은 이 목록을 받은 뒤 이름/시·군을 필터링한다. 기존 탐색 지도/추천·상세 화면에는 정적 데이터도 남아 있다. 현재 규모의 단순 필터를 위해 별도 검색/지역 API를 추가 설계하지 않는다.

### M02. 산별 코스 목록

- **GET `/api/mountains/{id}/courses` / 인증 필요 / 구현 완료**
- Path: `id` long, 산 ID. Query/Request Body: 없음.
- 설명: `length_km`, `id` 순. 산이 없으면 404, 산은 있지만 코스가 없으면 `[]`.
- Response: `List<Course>`; 주요 상태 200/404, 인증 401.

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| id | long | 아니오 | 코스 ID |
| mountainId | long | 아니오 | 연결된 산 ID |
| name | String | 아니오 | 코스 이름 |
| startName | String | 아니오 | 출발점 이름 |
| startLat | double | 아니오 | 출발점 위도 |
| startLng | double | 아니오 | 출발점 경도 |
| lengthKm | double | 아니오 | 출발점→정상 코스 거리 km |
| upMin | int | 아니오 | 예상 오름 시간 분 |
| downMin | int | 아니오 | 예상 하산 시간 분 |
| difficulty | String | 아니오 | DB 값 EASY/NORMAL/HARD |
| risk | String | 가능 | 위험 안내 |
| source | String | 아니오 | DB 값 FOREST/OSM |

```json
[
  {
    "id": 1,
    "mountainId": 1,
    "name": "예시 코스",
    "startName": "예시 입구",
    "startLat": 37.68,
    "startLng": 127.28,
    "lengthKm": 2.9,
    "upMin": 90,
    "downMin": 60,
    "difficulty": "NORMAL",
    "risk": null,
    "source": "OSM"
  }
]
```

### M03. 코스 상세 / 경로

- **GET `/api/courses/{id}` / 인증 필요 / 구현 완료**
- Path: `id` long, 코스 ID. Query/Request Body: 없음.
- 설명: `CourseDetail(Course course, List<Object> path)`. DB JSON text를 배열로 파싱한다. 현재 seed의 경로는 `[경도, 위도]` 좌표 배열이다. Controller가 모든 좌표의 형상/범위를 재검증하는 것은 아니다.
- Response: 아래 JSON. `course` 필드는 M02와 동일. 주요 상태 200/404, 인증 401. DB path JSON 손상 시 공통 400을 보장하지 않는다.

```json
{
  "course": {
    "id": 1,
    "mountainId": 1,
    "name": "예시 코스",
    "startName": "예시 입구",
    "startLat": 37.68,
    "startLng": 127.28,
    "lengthKm": 2.9,
    "upMin": 90,
    "downMin": 60,
    "difficulty": "NORMAL",
    "risk": null,
    "source": "OSM"
  },
  "path": [
    [
      127.28,
      37.68
    ],
    [
      127.3,
      37.7
    ]
  ]
}
```

## 6. 날씨 API

### W01. 산별 날씨

- **GET `/api/mountains/{id}/weather` / 인증 필요 / 구현 완료**
- Path: `id` long, 산 ID. Query/Request Body: 없음.
- 설명: 산의 DB 기상 격자와 기상청 단기예보를 사용한다. 최신 발표 기준은 현재 시각에서 10분을 뺀 뒤 02/05/08/11/14/17/20/23시 중 선택한다. 격자·발표 시각 기반 메모리 cache를 사용한다.
- Response: `WeatherResponse`, 아래 모델. 주요 상태 200, 산 없음 404, 기상청 키 미설정 503, 외부 호출/응답 오류 중 코드에서 처리하는 경우 502, 인증 401. 모든 비정상 외부 payload 예외가 동일 형식으로 처리되는 것은 아니다.
- FE 연결: 현재 W01 호출 없음. API 구현과 화면 연결을 구분한다.

| 모델 | 필드 | Java 자료형 | null |
| --- | --- | --- | --- |
| WeatherResponse | mountainId / mountainName | long / String | 아니오 |
| WeatherResponse | baseTime / days / source | LocalDateTime / List<Day> / String | 아니오 |
| Day | date / level / message | LocalDate / Level / String | 아니오 |
| Day | notes / hours | List<String> / List<HourForecast> | 아니오 |
| HourForecast | time | LocalDateTime | 아니오 |
| HourForecast | temperature | Double | 가능 |
| HourForecast | sky / pty / pop | Integer / Integer / Integer | 가능 |
| HourForecast | precipitationMm / snowCm | double / double | 아니오 |
| HourForecast | windSpeed / humidity | Double / Integer | 가능 |

```json
{
  "mountainId": 1,
  "mountainName": "예시산",
  "baseTime": "2026-10-03T08:00:00",
  "days": [
    {
      "date": "2026-10-03",
      "level": "GOOD",
      "message": "등산하기 좋은 날씨예요. 즐거운 산행 되세요!",
      "notes": [],
      "hours": [
        {
          "time": "2026-10-03T09:00:00",
          "temperature": 18.0,
          "sky": 1,
          "pty": 0,
          "pop": 10,
          "precipitationMm": 0.0,
          "snowCm": 0.0,
          "windSpeed": 2.0,
          "humidity": 60
        }
      ]
    }
  ],
  "source": "기상청 단기예보"
}
```

`message`는 `HikingWeatherRules`가 결정하며 위 예시는 실제 GOOD 문구다. `Level`은 GOOD/CAUTION/BAD다. 기온 °C, 강수확률 %, 강수량 mm, 적설 cm, 풍속 m/s, 습도 %다. SKY 1/3/4는 맑음/구름 많음/흐림, PTY 0/1/2/3/4는 없음/비/비·눈/눈/소나기다.

응답은 현재 시간대 이후 06~18시 예보를 대상으로 하며 날짜·시간 순으로 정렬한다. 해당 시간대가 없으면 날짜 자체를 생략할 수 있어 `days: []`도 가능하다. 위험도는 다음 조건 중 더 위험한 판정을 적용한다.

| 조건 | CAUTION | BAD |
| --- | --- | --- |
| 비/눈 PTY 시간 수 | 1시간 | 2시간 이상 |
| 강수확률 | 30% 이상 | 60% 이상 |
| 강수량 | 0 초과 | 3mm 이상 |
| 적설 | 0 초과 | 1cm 이상 |
| 풍속 | 7m/s 이상 | 10m/s 이상 |
| 더위 | 30°C 이상 | 33°C 이상 |
| 추위 | 0°C 이하 | -10°C 이하 |

누락된 예보 값은 위험 조건을 만들지 않을 수 있다. GOOD이 모든 관측값이 충분하다는 보장은 아니므로 결측 데이터 표시 정책은 검토 대상이다.

## 7. 실제 등산기록 API

### 실제 모델과 명칭

**실제 산행 활동은 `HikingActivity` / `hiking_activities`**다. 기존 이름 `HikingRecord` / `hiking_records`는 후기 게시글인 **등산일지**를 의미한다. HTTP 경로 `hiking-records`는 실제 활동을 제공하지만, Java의 `HikingRecordResponse`는 일지 응답이다. 이름만으로 서로 대응시키면 안 된다.

서비스 개념상 등산기록은 실제 산행에서 생성되는 활동 데이터이고, 등산일지는 저장된 등산기록을 선택하여 작성하는 선택적 후기다. 활동 데이터도 현재는 FE 결과를 저장하며 서버가 독립적으로 산행을 인증한 데이터는 아니다.

`HikingActivity`는 회원 ManyToOne, 산 ID 필수, 코스 ID 선택 관계를 갖는다. 산/코스는 Java에서는 scalar ID이며 DB FK로 연결된다. 회원은 세션에서 결정한다. 산 이름은 저장 시 DB에서 가져온 snapshot이다. `hikingDate`는 `startedAt`을 **Asia/Seoul**로 변환한 시작 날짜다. `completed`는 FE의 정상 도달 여부이며 종료 버튼을 눌렀다는 사실과 동일하지 않다. 클라이언트가 전달한 산행 완료 결과를 저장하는 값으로, 서버가 GPS 원본 경로를 기반으로 실제 정상 도달·등정을 독립 검증한 결과가 아니다.

### Request: HikingActivityRequest

| 필드 | Java 자료형 | 필수/검증 |
| --- | --- | --- |
| mountainId | Long | 필수, 양수 |
| courseId | Long | 선택/null 가능, 지정하면 양수 및 산과 일치 |
| startedAt | Instant | 필수, 현재 또는 과거 |
| endedAt | Instant | 필수, 현재 또는 과거, 시작 이상 |
| distanceMeters | Double | 필수, 유한한 0 이상 값, m |
| elapsedMs | Long | 필수, 0 이상, 종료-시작 간격 이하, ms |
| completed | Boolean | 필수, false 허용; FE의 정상 도달 결과, 서버 인증 아님 |
| clientRequestId | String | 필수, UUID 형식 문자열 |

```json
{
  "mountainId": 1,
  "courseId": null,
  "startedAt": "2026-10-01T00:00:00Z",
  "endedAt": "2026-10-01T03:00:00Z",
  "distanceMeters": 5800.0,
  "elapsedMs": 8040000,
  "completed": true,
  "clientRequestId": "123e4567-e89b-12d3-a456-426614174000"
}
```

사용자 ID, 산 이름, 산행 날짜, 원시 GPS 경로는 Request 필드가 아니다.

### Response: HikingActivityResponse

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| id | Long | 아니오 | 실제 활동 ID |
| mountainId | Long | 아니오 | 산 ID |
| mountainName | String | 아니오 | 저장 시 산 이름 |
| courseId | Long | 가능 | 코스 ID |
| startedAt / endedAt | Instant | 아니오 | 시작/종료 시각 |
| hikingDate | LocalDate | 아니오 | 서울 기준 시작 날짜 |
| distanceMeters | double | 아니오 | 이동 거리 m |
| elapsedMs | long | 아니오 | FE가 계산한 일시정지 제외 활동 시간(ms). 정지 상태의 시간도 포함될 수 있으며 서버는 전달값을 저장한다. |
| completed | boolean | 아니오 | 클라이언트가 보낸 정상 도달 여부; 서버 인증 아님 |
| journalId | Long | 가능 | 연결된 일지 ID; 없으면 null |

```json
{
  "id": 42,
  "mountainId": 1,
  "mountainName": "예시산",
  "courseId": null,
  "startedAt": "2026-10-01T00:00:00Z",
  "endedAt": "2026-10-01T03:00:00Z",
  "hikingDate": "2026-10-01",
  "distanceMeters": 5800.0,
  "elapsedMs": 8040000,
  "completed": true,
  "journalId": null
}
```

`elapsedMs`는 FE가 TRACKING 상태의 활동 시간을 누적한 값이다. 명시적·자동 일시정지 시간은 제외하지만 TRACKING 상태에서 움직이지 않은 시간도 포함될 수 있다. GPS 위치 표본이 없어도 일시정지되지 않은 TRACKING 시간은 증가할 수 있으며, BE는 전달값을 재계산하지 않고 저장한다.

### H01. 실제 산행 결과 저장

- **POST `/api/users/me/hiking-records` / 인증 및 본인 프로필 완료 필요 / 구현 완료**
- Path/Query: 없음. Request Body: 위 `HikingActivityRequest` JSON.
- 설명: 완료한 GPS 세션의 요약 결과를 저장한다. 산 존재, 코스의 산 일치, 시간·거리의 형식/범위를 검사한다.
- Response: 위 `HikingActivityResponse` JSON. 신규 저장도 **200**이며 201이 아니다.
- 주요 상태: 200, 입력/산·코스 불일치 400, 산 없음 404, 동일 요청 ID를 다른 내용으로 재사용 409, 프로필 미완료/CSRF 403, 인증 401.

멱등성: 회원별 `(member_id, client_request_id)` UNIQUE와 회원 행 `FOR UPDATE`로 저장을 직렬화한다. 같은 UUID와 동일한 산/코스/시간/거리/완등 값이면 기존 기록을 200으로 반환한다. 같은 UUID에 다른 값이면 409다. 다른 UUID로 같은 산행을 보내는 것을 막는 산행 진위 검증은 아니다.

서버는 GPS 경로·속도·고도·위치 정확도를 받지 않으며 거리를 재계산하거나 정상 도달을 검증하지 않는다. 거리/시간/완료 결과는 FE 계산값이다. `completed=true`는 서버가 인증한 실제 완등을 의미하지 않는다. 점수 갱신도 하지 않는다. 저장 API는 구현 완료지만 이러한 내부 로직은 별도 미구현이다.

### H02. 내 등산기록 목록

- **GET `/api/users/me/hiking-records` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음. 프로필 완료 여부는 목록 읽기를 막지 않는다.
- 설명: 현재 회원의 모든 활동을 `startedAt DESC, id DESC` 순으로 조회한다. pagination 없음. 연결 일지를 확인하여 `journalId`를 포함한다.
- Response: `List<HikingActivityResponse>`. 예시는 H01 객체를 배열에 담은 형태이며 빈 경우 `[]`다.

```json
[
  {
    "id": 42,
    "mountainId": 1,
    "mountainName": "예시산",
    "courseId": null,
    "startedAt": "2026-10-01T00:00:00Z",
    "endedAt": "2026-10-01T03:00:00Z",
    "hikingDate": "2026-10-01",
    "distanceMeters": 5800.0,
    "elapsedMs": 8040000,
    "completed": true,
    "journalId": 10
  }
]
```

- 주요 상태: 200, 인증 401. FE 기록 탭·일지 후보 목록에 연결되어 있다.

### H03. 내 등산기록 상세

- **GET `/api/users/me/hiking-records/{id}` / 인증 필요 / 구현 완료**
- Path: `id` Long, 실제 활동 ID. Query/Request Body: 없음.
- 설명: ID와 현재 회원을 함께 조회한다. 다른 사용자의 기록도 없는 기록과 동일하게 404다.
- Response: H01의 `HikingActivityResponse` JSON; 일지 작성 후 `journalId`가 해당 ID로 바뀐다.
- 주요 상태: 200/404, 인증 401. 현재 FE에는 이 endpoint 호출이 없다.

활동 수정/삭제/타인 활동 공개/원시 GPS 조회 API는 현재 없다. 이번 문서는 필요가 확정되지 않은 해당 API를 예정 목록에 추가하지 않는다.

## 8. 등산일지 API

### 관계 / legacy 호환

`HikingRecord`는 회원과 연결된 일지이며 `HikingActivity`를 optional OneToOne로 참조한다. 원칙은 **활동 1 : 일지 0..1**이다. V4에서 `hiking_records.hiking_activity_id` nullable FK와 UNIQUE를 추가했다. 기존 일지는 값을 null로 남겨 보존하며 활동을 임의 생성하거나 억지로 연결하지 않는다.

새 일지 생성 시 본인 활동을 비관적 쓰기 lock으로 조회하고 이미 일지가 있는지 검사한다. 서비스 검사와 DB UNIQUE 모두 중복을 막는다. 새 일지의 산 이름과 날짜는 활동에서 복사하며 최초 생성 이후 수정하지 않는다. 정상 도달 `completed=true`만 후보로 제한하는 조건은 없다.

### J01 Request: HikingRecordRequest

| 필드 | Java 자료형 | 필수/검증 |
| --- | --- | --- |
| hikingRecordId | Long | 필수 양수, **HikingActivity.id**를 가리킴 |
| title | String | 필수, 공백 불가, 최대 100자 |
| content | String | 필수, 최대 10,000자, 빈 문자열 허용 |
| isPublic | Boolean | 필수, false 가능 |

```json
{
  "hikingRecordId": 42,
  "title": "가을 산행",
  "content": "천천히 걸으며 풍경을 즐겼다.",
  "isPublic": false
}
```

이 필드 이름은 HTTP 계약 그대로이며 Java 일지 Entity의 ID라는 뜻이 아니다. `mountainName`, `hikingDate`는 생성 DTO 필드가 아니다. 테스트에서 위조한 산/날짜 추가값은 사용되지 않고 활동 값으로 저장된다.

### J06 Request: JournalUpdateRequest

| 필드 | Java 자료형 | 필수/검증 |
| --- | --- | --- |
| title | String | 필수, 공백 불가, 최대 100자 |
| content | String | 필수, 최대 10,000자, 빈 문자열 가능 |
| isPublic | Boolean | 필수 |
| hikingRecordId | Long | 생략/null만 허용(@Null) |
| mountainName | String | 생략/null만 허용(@Null) |
| hikingDate | LocalDate | 생략/null만 허용(@Null) |

```json
{
  "title": "수정한 산행 후기",
  "content": "내용을 보완했다.",
  "isPublic": true
}
```

산/날짜/연결 ID를 non-null로 보내면 같은 값이라도 400이다. 공개 여부만 보내는 PATCH도 필수 제목/내용 검증으로 실패한다. FE는 수정 가능한 세 필드만 보낸다.

### Response: HikingRecordResponse

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| id | Long | 아니오 | 일지 ID |
| userId | Long | 아니오 | 작성자 ID |
| nickname | String | DB nullable | 작성자 닉네임 |
| mountainName | String | 아니오 | 생성 시 활동의 산 이름, legacy는 기존 값 |
| title | String | 아니오 | 제목 |
| content | String | 아니오 | 내용 |
| hikingDate | LocalDate | 아니오 | 활동의 서울 기준 시작 날짜, legacy는 기존 값 |
| isPublic | boolean | 아니오 | 공개 여부 |
| hikingRecordId | Long | 가능 | 연결된 실제 활동 ID, legacy null |

```json
{
  "id": 10,
  "userId": 7,
  "nickname": "산사람",
  "mountainName": "예시산",
  "title": "가을 산행",
  "content": "천천히 걸으며 풍경을 즐겼다.",
  "hikingDate": "2026-10-01",
  "isPublic": false,
  "hikingRecordId": 42
}
```

일지 목록도 같은 DTO로 전체 content를 포함한다. 회원 출생연도/이메일/OAuth 정보, 활동의 원시 경로·거리·시각은 이 DTO에 없다. 공개 일지에는 연결 활동 숫자 ID는 포함되지만 H03의 본인 제한으로 타인의 활동 상세를 읽을 수 없다.

### J01. 등산기록 기반 일지 생성

- **POST `/api/journals` / 인증 및 프로필 완료 필요 / 구현 완료**
- Path/Query: 없음. Request Body: 위 `HikingRecordRequest` JSON.
- 설명: 로그인 회원이 소유한 실제 활동으로만 생성한다. 산·날짜는 서버가 활동에서 결정한다.
- Response: 위 `HikingRecordResponse` JSON. 주요 상태: **200**, 입력 400, 활동 없음/타인 활동 404, 중복 일지 409, 프로필 미완료/CSRF 403, 인증 401.

### J02. 내 등산일지 목록

- **GET `/api/users/me/journals` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음.
- 설명: 본인의 공개·비공개·legacy 일지 모두, `hikingDate DESC, id DESC`. pagination/개수 제한 없음. 프로필 완료를 추가로 요구하지 않는다.
- Response: `List<HikingRecordResponse>`, 주요 상태 200, 인증 401.

```json
[
  {
    "id": 10,
    "userId": 7,
    "nickname": "산사람",
    "mountainName": "예시산",
    "title": "가을 산행",
    "content": "천천히 걸으며 풍경을 즐겼다.",
    "hikingDate": "2026-10-01",
    "isPublic": false,
    "hikingRecordId": 42
  }
]
```

### J03. 공개 등산일지 목록

- **GET `/api/journals` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음.
- 설명: `isPublic=true`만, `hikingDate DESC, id DESC` 최신 최대 100개. pagination 없음. 이 feed 조회는 작성자의 프로필 완료 여부를 추가 조건으로 검사하지 않는다.
- Response: `List<HikingRecordResponse>`, 예시 아래. 주요 상태 200, 인증 401.

```json
[
  {
    "id": 10,
    "userId": 7,
    "nickname": "산사람",
    "mountainName": "예시산",
    "title": "가을 산행",
    "content": "천천히 걸으며 풍경을 즐겼다.",
    "hikingDate": "2026-10-01",
    "isPublic": true,
    "hikingRecordId": 42
  }
]
```

### J04. 다른 사용자 공개 일지 목록

- **GET `/api/users/{id}/journals` / 인증 필요 / 구현 완료**
- Path: `id` Long, 작성자 ID. Query/Request Body: 없음.
- 설명: 대상 회원 존재 및 프로필 완료를 검사하고 공개 일지만 조회한다. 자신의 ID를 넣어도 비공개 일지는 포함되지 않는다. 순서는 J02와 같고 pagination/개수 제한은 없다.
- Response: J03 예시와 같은 `List<HikingRecordResponse>`; 빈 경우 `[]`.
- 주요 상태: 200, 대상 없음 404, 대상 프로필 미완료 403, 인증 401.

### J05. 일지 상세

- **GET `/api/journals/{id}` / 인증 필요 / 구현 완료**
- Path: `id` Long, 일지 ID. Query/Request Body: 없음.
- 설명: 공개 일지는 로그인 사용자가 읽을 수 있고 비공개 일지는 작성자만 읽는다. 읽기에 별도 프로필 완료 조건은 없다.
- Response: 위 `HikingRecordResponse` JSON. legacy 예시는 아래와 같다.

```json
{
  "id": 11,
  "userId": 7,
  "nickname": "산사람",
  "mountainName": "기존 산 이름",
  "title": "기존 일지",
  "content": "기존 내용",
  "hikingDate": "2026-09-01",
  "isPublic": false,
  "hikingRecordId": null
}
```

- 주요 상태: 200, 없는 일지/타인 비공개 일지 404, 인증 401.

### J06. 일지 수정

- **PATCH `/api/journals/{id}` / 인증·프로필 완료·작성자 권한 필요 / 구현 완료**
- Path: `id` Long, 일지 ID. Query: 없음. Request Body: 위 `JournalUpdateRequest` JSON.
- 설명: 제목·내용·공개 여부만 수정한다. 연결 활동·산·날짜는 변경 불가. legacy 일지도 기존 산/날짜와 null 연결을 유지하며 수정 가능하다.
- Response: 같은 `HikingRecordResponse` 구조, 예시 아래.

```json
{
  "id": 10,
  "userId": 7,
  "nickname": "산사람",
  "mountainName": "예시산",
  "title": "수정한 산행 후기",
  "content": "내용을 보완했다.",
  "hikingDate": "2026-10-01",
  "isPublic": true,
  "hikingRecordId": 42
}
```

- 주요 상태: 200, 입력/불변 필드 지정 400, 없음/타인 일지 404, 프로필 미완료/CSRF 403, 인증 401.

### J07. 일지 삭제

- **DELETE `/api/journals/{id}` / 인증·프로필 완료·작성자 권한 필요 / 구현 완료**
- Path: `id` Long, 일지 ID. Query/Request Body: 없음.
- 설명: 일지를 물리 삭제한다. 실제 활동은 삭제하지 않으므로 같은 활동으로 새 일지를 다시 작성할 수 있다. H02/H03의 `journalId`는 다시 null이 된다.
- Response Body: 없음. 주요 상태 204, 없음/타인 일지 404, 프로필 미완료/CSRF 403, 인증 401.

## 9. 랭킹 / 점수 API

### R01. 전체 랭킹

- **GET `/api/rankings` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음. 기간/지역 필터, pagination 없음.
- 설명: 프로필 완료 회원 최대 100명. 점수 non-null 우선, `score DESC, id ASC`. 점수가 null인 회원도 결과에 포함될 수 있다.
- Response: `List<PublicMemberResponse>`; 서버가 순위 번호 필드를 반환하지 않는다. 주요 상태 200, 인증 401.

```json
[
  {
    "userId": 7,
    "nickname": "산사람",
    "profileImageUrl": "https://example.com/provider-photo.jpg",
    "score": 120
  },
  {
    "userId": 8,
    "nickname": "걷는사람",
    "profileImageUrl": null,
    "score": null
  }
]
```

`members.score`와 점수 조회는 구현되어 있다. 본인 점수는 U01, 타인 점수는 U03, 랭킹 점수는 R01에서 제공하므로 별도의 중복 score 조회 API는 필요하지 않다.

점수 정책의 우선순위는 **산행 다양성 > 산행 빈도 > 산의 높이**다. 그러나 **점수 자동 계산 및 `members.score` 갱신 로직은 미구현**이다. H01 저장과 회원 score 갱신은 연결되지 않는다. DB의 저장값을 읽는 랭킹 API가 있다는 이유로 점수 시스템 전체를 완료로 표현하지 않는다. FE 랭킹은 null/비유한 점수를 0으로 표시하는 처리도 있으므로 미계산과 실제 0점을 구분하는 UX 검토가 필요하다.

## 10. 구현 예정 API 및 향후 개선

### 구현 예정 API

현재 검토 결과 **0개**다. 산 정보는 M01/M02/M03을 활용하며 별도 산 상세·검색·지역 필터 API를 추가하지 않는다. 산 이름 및 시·군 검색/필터링은 M01 결과를 FE에서 처리한다. 점수 조회도 U01/U03/R01을 활용하므로 별도 endpoint를 추가하지 않는다.

### 향후 기능 개선: 타 사용자 프로필 이미지

현재 공개 프로필 U03과 랭킹 R01은 OAuth 공급자의 `profileImageUrl`을 사용한다. ToPeak에 직접 업로드한 사진은 본인 이미지 API에서 관리되며 타 사용자에게 제공하는 구조는 없다. 향후 직접 등록한 사진도 다른 사용자의 프로필 및 랭킹 화면에 표시하도록 이미지 제공 방식을 개선할 예정이다. 구체적인 endpoint와 구현 방식은 실제 개발 시 현재 구조를 기준으로 결정한다. 이는 **향후 기능 개선**이며 확정된 구현 예정 API가 아니다.

### 내부 비즈니스 로직: 점수 자동 계산

실제 등산기록을 기반으로 점수를 자동 산정·갱신하는 로직은 현재 미구현이며 향후 서버 내부에 추가할 예정이다. **산행 다양성 > 산행 빈도 > 산의 높이** 순으로 반영한다. 서로 다른 산의 경험을 가장 중요하게, 꾸준한 산행 빈도를 그다음으로, 산의 높이를 보조 요소로 평가한다. 동일 산 재방문은 인정하되 새로운 산 방문보다 낮은 가치를 부여한다. 속도·평균 속도는 경쟁 점수에서 제외하고 개인 산행기록 정보로만 사용한다. 구체적인 공식·가중치는 미확정이며 실제 운영 데이터와 서비스 운영 과정에서 조정할 수 있도록 한다. 별도 점수 계산·조회 API는 추가하지 않는다.

## 11. 실제 오류 응답

`GlobalExceptionHandler`는 특정 예외를 Map으로 반환한다. 별도 공통 `ErrorResponse` DTO는 없고 `ErrorCode`는 비어 있는 enum이다. `BusinessException` 선언만으로 공통 business 오류 처리 체계가 있는 것으로 간주하지 않는다.

| 발생 위치/조건 | 상태 | error | message |
| --- | --- | --- | --- |
| Security 인증 진입점 | 401 | unauthorized | 로그인이 필요합니다. 다시 로그인해주세요. |
| Security CSRF 실패 | 403 | invalid_csrf | 인증 토큰이 없거나 만료되었습니다. /api/csrf 조회 후 반환된 헤더와 토큰으로 다시 요청해주세요. |
| Security 기타 접근 거부 | 403 | forbidden | 이 요청에 대한 권한이 없습니다. |
| OAuth 실패 Handler | 401 | oauth_login_failed | **필드 없음** |
| MethodArgumentNotValidException | 400 | validation_failed | 첫 번째 검증 오류의 defaultMessage |
| HttpMessageNotReadableException | 400 | invalid_request | 입력 형식을 확인해주세요. |
| MaxUploadSizeExceededException | 413 | image_too_large | 이미지는 5MiB 이하로 올려주세요. |
| ResponseStatusException | 예외가 지정한 상태 | request_failed | reason, 없으면 요청을 처리할 수 없습니다. |

```json
{
  "error": "request_failed",
  "message": "등산기록을 찾을 수 없습니다."
}
```

```json
{
  "error": "request_failed",
  "message": "이미 등산일지를 작성한 기록입니다."
}
```

위 예시는 각각 J01의 404, J01의 409에서 사용하는 reason이다. H03의 404는 reason 없이 발생하므로 message는 `요청을 처리할 수 없습니다.`다. 모든 예외를 포괄하는 Handler는 아니다. DB 무결성 예외, path 타입 변환, multipart 필수 part 누락 등 명시적으로 처리하지 않은 오류에 위 두 필드 형식을 일괄 보장하지 않는다. 프레임워크 기본 오류 응답은 별도 실행 환경에 따라 확인해야 한다.

## 12. FE-BE 연결과 현재 사용자 흐름

FE `src/api.ts`에 함수가 있다는 사실과 화면에서 호출한다는 사실을 구분하여 `App.tsx`, profile/ranking/hiking component, hooks 및 테스트를 함께 확인했다.

| 기능 | 연결 상태 | 코드에서 확인한 동작/제한 |
| --- | --- | --- |
| Google/Kakao·현재 회원·로그아웃·CSRF | 구현 완료 + FE 연결 | 브라우저 OAuth 이동, 세션 cookie, 변경 요청 CSRF 조회 |
| 최초 프로필/수정 | 구현 완료 + FE 연결 | nickname/birthYear를 U02로 저장 |
| 개인 프로필/배경 이미지 | 구현 완료 + FE 연결 | 업로드/삭제 hooks, 본인 URL binary 조회 |
| 공개 프로필·사용자 공개 일지 | 구현 완료 + FE 연결 | U03/J04, 점수와 provider 사진 포함 |
| 실등산용 산·코스·경로 | 구현 완료 + FE 연결 | DB ID 선택, M01/M02/M03, 경로 좌표 검증 |
| 기존 탐색 지도·추천·산 상세 | FE 정적 자료 사용 | 실등산 데이터 화면과 완전히 통일되어 있지 않음; 산 기본 정보·코스는 M01/M02/M03 활용, 별도 API 추가 없음 |
| 날씨 | BE 구현 완료 + FE 미연결 | W01 호출 없음 |
| GPS 진행 | FE 메모리 | 위치 수집/필터/구간/일시정지; 원시 경로 서버 저장 없음 |
| 종료 결과 저장 | 구현 완료 + FE 연결 | H01, 동일 clientRequestId로 재시도 |
| 내 등산기록 탭·일지 후보 | 구현 완료 + FE 연결 | H02, journalId가 null인 활동만 작성 후보 |
| 등산기록 상세 | BE 구현 완료 + FE 미연결 | H03 호출 함수/화면 연결 없음 |
| 일지 생성·조회·수정·삭제·공개 | 구현 완료 + FE 연결 | J01~J07, 새 글과 수정 body 구분 |
| 랭킹 | 구현 완료 + FE 연결 | R01, 공개 프로필로 userId 전달 |
| 마이페이지 산행 통계 | 기존 API 연결, 활동 집계 미연결 | 횟수 일부는 일지 기준; 거리 값은 0, 실제 활동 기준 집계 아님 |
| 월 목표 | FE localStorage | DB member_monthly_goals 테이블은 있으나 API/활성 저장 서비스 없음 |
| 즐겨찾기·수동 완등 표시 | FE localStorage | 실제 산행 completed/서버 검증과 별도; 회원별 분리도 검토 대상 |

### 등산 종료부터 일지 저장까지

1. `HikingPage`에서 서버 산·코스를 선택하고 GPS 세션을 시작한다. 시작 시각과 `crypto.randomUUID()` 저장 요청 ID를 만든다.
2. `useHikingSession`이 위치를 필터링하여 메모리에 구간·거리·활동 시간을 관리한다. 일시정지와 가시성 변경을 처리한다.
3. 종료 시 `ActiveHike`가 산/코스 ID, 시작/종료 시각, 거리, elapsedMs, 정상 도달 여부, 요청 UUID만 H01로 보낸다. GPS 점 배열은 보내지 않는다.
4. 저장 실패 재시도는 동일 UUID를 사용한다. 저장 성공 후 일지 작성 이동이 가능하다. 저장되지 않은 세션은 페이지 이탈/새로고침 시 메모리 유실 가능성이 있다.
5. 일지 작성 화면이 H02를 조회하고 `journalId === null`인 자신의 활동을 선택한다. 저장 완료 화면에서 이동한 경우 해당 활동 ID를 선택한다.
6. 선택 전 산/날짜 영역은 숨기고 저장 버튼은 비활성화한다. 후보가 없을 때 안내는 유지하며 `등산 시작하기` 버튼은 없다. 선택 후 산/날짜는 읽기 전용 정보로 표시한다.
7. FE는 활동 ID와 제목/내용/공개 여부만 J01에 보낸다. 서버가 소유권·중복을 검증하고 산/날짜를 활동에서 결정한다. 중복 409 시 후보 목록을 다시 조회한다.
8. 수정은 J06으로 제목/내용/공개 여부만 전송한다. legacy 일지는 활동 선택 없이 기존 산/날짜를 유지한다.

현재 App에는 기존 모의 등산 화면 코드도 남아 있으나 실제 GPS 산행 component와 동일한 저장 기능으로 표현해서는 안 된다. 마이페이지 통계는 기존 H02 데이터의 FE 집계로 개선 가능한 영역이며 새로운 API가 반드시 필요한 것은 아니다.

## 13. 개인정보 / 권한 확인

- 본인 전용 프로필·이미지·활동·일지 API는 인증 principal에서 회원을 결정한다. 클라이언트가 userId를 지정하여 소유자를 바꾸는 요청 필드는 없다.
- H03은 타인 활동 ID를 404로 숨긴다. H02는 본인만 반환한다. 원시 GPS 데이터는 서버 저장/조회 API 자체가 없다.
- J01은 활동 ID와 현재 회원을 함께 lock 조회하므로 타인 활동으로 일지를 만들 수 없다. 존재하지 않는 ID도 404다.
- 비공개 일지는 J03/J04에 포함되지 않고 J05에서는 작성자 외 404다. J06/J07은 작성자만 가능하며 타인 ID는 404다.
- 공개 DTO에는 birthYear/age/email/provider/providerId/인증정보가 없다. 본인 U01 응답의 birthYear/age가 공개 응답에 그대로 재사용되지 않는다.
- 개인 업로드 이미지 endpoint는 본인만 읽는다. 공개 프로필 사진은 provider URL이다. FE에서 본인 업로드와 다른 사용자가 보는 사진이 다를 수 있다. 직접 업로드 사진을 공개 프로필·랭킹에도 표시하는 개선 방향은 정해졌으며 구체적 제공 방식은 개발 시 결정한다.
- 공개 일지의 활동 ID 노출은 활동 상세 접근 권한을 부여하지 않는다. 연결 ID까지 숨길 필요가 있는지는 별도 제품 정책이다.
- 프로필이 미완료여도 기존 활동/본인 일지/접근 가능한 상세 읽기는 가능하지만 활동·일지 쓰기는 완료가 필요하다. 공개 사용자 프로필과 사용자별 공개 일지 목록은 대상의 완료 상태도 검사한다.

## 14. 코드 대조 근거와 검토 필요 사항

### 주요 근거 파일

BE 경로는 `BE/src/main/java/com/ggmount/` 아래다.

| 조사 항목 | 근거 |
| --- | --- |
| 모든 Controller Mapping | global/auth/oauth/SessionController.java, member/controller/MemberController.java, PersonalDataController.java, mountain/controller/MountainController.java, weather/WeatherController.java, hiking/controller/HikingActivityController.java, HikingRecordController.java, ranking/controller/RankingController.java |
| 인증/인가 | global/config/SecurityConfig.java, global/auth/oauth/*, member/service/OAuthMemberService.java, BE/src/main/resources/application.yml |
| 회원/사진 계약 | member/dto/*, domain/Member.java, service/MemberService.java, PersonalDataService.java, ImageUploadValidator.java, repository/* |
| 활동/일지 계약과 소유권 | hiking/dto/*, domain/*, service/*, repository/* |
| 산/코스/날씨 | MountainController.java, weather/* |
| 점수 읽기/랭킹 | member/domain/Member.java, member/dto/*, ranking/service/RankingService.java, member/repository/MemberRepository.java |
| 오류 | global/exception/GlobalExceptionHandler.java, ErrorCode.java, BusinessException.java, OAuth Handler와 SecurityConfig |
| FE 실제 호출 | FE/src/api.ts, App.tsx, components/hiking/*, components/profile/*, components/ranking/*, hooks/useMemberImage.ts, useMonthlyGoals.ts, usePersistentIds.ts, FE/vite.config.ts |
| schema/호환 | BE/src/main/resources/db/migration/V1__member_and_hiking_records.sql, V2__mountains.sql, V3__courses.sql, V3_1__member_personal_data.sql, V4__hiking_activities_and_journal_link.sql |

### Migration 및 기존 데이터

V1 회원/일지 → V2 산 → V3 코스 → V3.1 개인 이미지·월 목표 → V4 실제 활동과 일지 연결 순서다. V4는 기존 일지 삭제·잘못된 연결을 하지 않는다. nullable UNIQUE는 여러 legacy null 연결을 허용하며 새 일지는 서비스에서 필수 활동 ID를 요구한다. legacy를 신규 활동으로 전환하는 보정 API/자동 backfill은 없다.

### 테스트 코드가 검증하는 범위

기존 테스트 소스를 읽어 명세와 대조했다. 이번 문서 작업에서 빌드/테스트를 새로 실행한 것은 아니며 아래 항목은 **작성된 테스트의 검증 내용**이다.

| 테스트 영역 | 명세 대조 내용 |
| --- | --- |
| ProfileJournalIntegrationTests | 본인 활동 기반 생성, 위조 산/날짜 무시, 타인/없는 활동 차단, 중복 409, 불변 필드 400, 삭제 후 재작성, legacy 조회·수정·권한, 멱등 저장·시간·코스 검증·CSRF |
| JournalMigrationTests | V3.1 legacy 데이터가 V4 이후 유지되고 연결 null, 연결 UNIQUE |
| 이미지 통합 테스트 | 업로드 검증, revision 및 계정 접근 제한 |
| OAuth mapper/Handler 테스트 | provider 식별 및 redirect/실패 응답 |
| SeedData/날씨 규칙 테스트 | seed와 코스 경로, 날씨 판정; 외부 live 테스트는 조건부 |
| FE/tests | 세션/CSRF 요청, 활동 저장 body·재시도 ID, 일지 생성/수정 계약, GPS 계산·필터·지도/코스 처리 |

실제 Controller Method/URI 22개, Security 처리 경로 5개, DTO 필드, 소유권 필터, V4 관계, FE 호출을 대조했다. 구현 완료 API는 모두 유지했고 구현 예정 API는 0개다. 문서 작업으로 source/test/migration을 수정하지 않는다.

### 향후 결정이 필요한 사항

1. **이미지 제공 방식 개선**: 직접 업로드한 프로필 사진을 타 사용자 프로필/랭킹에 표시할 구체적 제공 방식과 권한·응답 계약. 신규 endpoint는 확정하지 않았다.
2. **점수 내부 로직 구현**: 산행 다양성 > 산행 빈도 > 산의 높이 및 재방문 활동의 차등 반영에 따른 공식·가중치·갱신 시점, 중복 산행 처리. 재방문은 인정하되 새로운 산 방문보다 낮은 가치를 부여하며 속도·평균 속도는 경쟁 점수에서 제외한다. 공식·가중치는 미확정이며 운영 과정에서 조정한다. 자동 계산·갱신은 미구현이며 legacy 일지에 활동을 임의 연결하여 점수를 부여하지 않는다.
3. **GPS 저장/검증 정책**: 현재 FE가 GPS를 측정·필터링하고 산행 결과를 계산하며 서버는 요약 결과만 저장한다. 독립적인 GPS 진위·정상 도달 검증과 원시 경로 저장은 미구현으로, 결과 저장만으로 실제 등정을 완전히 검증한다고 보지 않는다. 향후 지역 기반 챌린지를 운영한다면 챌린지별 참여·완료 인정 조건과 필요한 검증 수준을 별도로 정의한다. 필요한 경우 서버 검증 수준, 원시 GPS 저장 여부, 보관 기간·위치정보 정책을 결정한다. 챌린지는 확장 계획이며 신규 API는 확정하지 않는다.
4. **등산기록 삭제 정책**: 삭제 기능이 필요한지, 필요한 경우 연결 일지와 점수에 미치는 영향. 현재 활동 삭제 API는 없다.
5. **기타 데이터·UX**: 활동/일지 목록 pagination, null score 표시, 마이페이지 활동 기반 집계, 기존 정적 산 화면과 M01/M02/M03 데이터 연결. 별도 산 상세 API나 설명 데이터 계약을 추가하지 않는다.
6. **운영/오류 처리**: 날씨 결측·빈 예보 표시, 외부 payload 처리, cache와 키 설정, 세션 cookie/HTTPS/proxy/CORS 설정. `WebConfig` 선언만으로 완성된 CORS 정책이 있다고 표현할 수 없다.
7. **기존 데이터·명명 관리**: legacy null 연결 보존과 향후 연결 필요성, 클래스명과 HTTP 용어 차이(`HikingRecord`=일지, `hikingRecordId`=활동 ID). 자동 추정 연결이나 rename을 확정하지 않았다.

## 15. 최종 요약

| 구분 | 개수 |
| --- | --- |
| 구현 완료 | **27**: Controller 22 + Security 5 |
| 구현 예정 | **0** |
| 부분 구현 | **0** |

- **지원되는 주요 기능**: Google/Kakao 세션 로그인, CSRF/로그아웃, 최초 프로필 및 수정, 본인 이미지, 공개 프로필, 산·코스·경로, 날씨 조회, 실제 활동 저장·본인 조회, 기록 기반 일지 CRUD/공개, 랭킹·저장 점수 읽기.
- **별도 구현 예정 API**: 없음. 산 목록·코스 목록·코스 상세를 활용하며 산 상세·검색·지역 필터·점수 조회 endpoint를 추가하지 않는다.
- **향후 기능 개선**: 직접 업로드 프로필 사진을 타 사용자 프로필과 랭킹에도 표시하도록 제공 방식을 개선한다. 현재 구현되어 있지 않으며 구체적 endpoint는 개발 시 결정한다.
- **API는 있지만 FE 미연결**: 날씨 W01, 실제 활동 상세 H03. 마이페이지에는 기존 H02 활동 데이터를 통계에 활용하는 연결이 남아 있다.
- **FE는 있지만 BE가 없는/정적·로컬 기능**: 정적 산 상세, localStorage 월 목표·즐겨찾기·수동 완등 표시. 월 목표 DB 테이블만으로 API 구현이라고 보지 않는다.
- **API 외 내부 로직 미구현**: 산행 다양성 > 산행 빈도 > 산의 높이 정책에 따른 점수 자동 계산·갱신, H01 활동 저장과 members.score 연결, 서버 GPS 진위/거리/정상 도달 검증, 원시 GPS 저장. 조회/저장 API 완료 상태와 구분한다.
- **권한/호환**: 타인 활동·타인 비공개 일지·타인 수정/삭제 차단, 공개 DTO 개인정보 제한, 활동 1개당 최대 일지 1개, 삭제 후 재작성 가능, legacy null 연결 보존.
- **점수 정책**: 동일 산 재방문도 인정하되 새로운 산 방문보다 낮은 가치로 차등 반영한다. 속도·평균 속도는 개인 기록에만 사용하며 경쟁 점수에서 제외한다. 공식·가중치는 미확정이며 운영 데이터와 운영 과정에서 조정한다.
- **향후 챌린지 검증**: 현재 결과 저장은 실제 완등 인증이 아니다. 지역 챌린지별 참여·완료 인정 조건과 필요한 서버 검증 수준, GPS 저장·보관 정책은 별도로 정의한다. 신규 챌린지 API는 확정하지 않는다.
- **최종 결정 필요**: 업로드 사진 제공 방식, 점수 공식·갱신, GPS 저장·검증, 활동 삭제 필요성, legacy 처리, 목록/통계 UX 및 운영 인증·날씨 설정. 확정되지 않은 신규 API는 설계하지 않았다.
