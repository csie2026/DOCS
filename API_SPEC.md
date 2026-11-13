# ToPeak API 명세서

## 1. 문서 개요

학술제 최종 제출과 개발 관리를 위한 최신 코드 기준 명세서다. 기존 1~15장 분류·API ID·표·예시 형식을 유지하고, 2026-11-13의 다음 소스를 Controller·DTO·Security·서비스·예외 처리·테스트·FE 실제 호출과 교차 대조했다.

- BE: `bd102653758f104cb9369d288ce1f1ebda35c337` (`dev`, 공개 보너스 보존 PR #47 포함).
- FE: `66ceb1fe288d9047cd821b986b4c33e4f030e75b` (`dev` 기준 코드, 프로필 저장 결과 확인 포함).
- BE 패키지: `com.ggmount`. 서비스명 ToPeak와 Java 패키지명은 다르다.
- 배포 DB·OAuth 실제 왕복·기상청 live 응답을 확인한 문서가 아니다. 같은 BE HEAD의 별도 신규 PostgreSQL 14.24 검증 결과는 14장에 구분한다. 문서 작업에서 제품 코드/테스트/migration은 수정하지 않았다.
- 현재 정책 원본은 [점수·통계 정책](SCORE_POLICY.md), GPS의 상세 호환 이력은 [산행 무결성](hiking-activity-integrity.md)에서 관리한다.

### 목차

- [1. 문서 개요](#1-문서-개요)
- [2. API 전체 목록](#2-api-전체-목록)
- [3. 인증 API](#3-인증-api)
- [4. 회원 / 프로필 / 이미지 API](#4-회원--프로필--이미지-api)
- [5. 산 / 코스 API](#5-산--코스-api)
- [6. 날씨 API](#6-날씨-api)
- [7. 실제 등산기록 API](#7-실제-등산기록-api)
- [8. 등산일지 API](#8-등산일지-api)
- [9. 랭킹 / 점수 API](#9-랭킹--점수-api)
- [10. 구현 예정 API 및 향후 개선](#10-구현-예정-api-및-향후-개선)
- [11. 실제 오류 응답](#11-실제-오류-응답)
- [12. FE-BE 연결과 현재 사용자 흐름](#12-fe-be-연결과-현재-사용자-흐름)
- [13. 개인정보 / 권한 확인](#13-개인정보--권한-확인)
- [14. 코드 대조 근거와 검토 필요 사항](#14-코드-대조-근거와-검토-필요-사항)
- [15. 최종 요약](#15-최종-요약)

### 개발 주소와 인증

BE는 별도 port 설정이 없는 Spring Boot 기본 환경에서 `http://localhost:8080`, FE 기본 개발 주소는 `http://localhost:5173`이다. FE Vite proxy는 `/api`, `/oauth2`, `/login`을 BE에 전달한다. 배포 주소는 코드만으로 확정할 수 없다.

인증은 **Spring Security OAuth2 + 서버 세션**이다. JWT 발급/갱신 API는 없다. FE는 요청에 `credentials: 'include'`를 사용한다. 인증 후 세션 쿠키를 전달하며, 변경 요청은 `GET /api/csrf`에서 받은 `headerName`과 `token`을 헤더에 넣는다. 로그아웃에도 CSRF가 필요하다.

Security의 공개 허용 경로는 `/oauth2/authorization/**`, `/login/**`, `/error`, `/api/csrf`다. 그 외는 인증이 필요하다. 따라서 이 문서에서 **공개 프로필/공개 일지**란 다른 로그인 사용자에게 공개된다는 의미다. 비로그인 공개 API라는 의미가 아니다. 인증되지 않은 변경 요청은 CSRF 검사 순서에 따라 401보다 먼저 403을 받을 수 있다.

### 상태와 집계 기준

- **구현 완료**: Controller Mapping 또는 Security 처리 경로를 확인한 상태다. FE 사용·테스트 범위·실제 현장 이동 검증까지 자동으로 뜻하지 않는다.
- **FE 미연결**: BE 구현은 있지만 현재 FE 화면 호출이 없는 경로다. 사용하지 않는 API helper와 실제 화면 연결을 구분한다.
- **향후 계획/미구현**: 활동 수정·삭제, GPS 원본 조회, 산 상세 전용·검색·필터·즐겨찾기·GPS 시작/일시정지 API는 현재 없다. 신규 API 추가가 확정됐다고 적지 않는다.
- Controller **28개**와 Security **5개**, 총 **33개 Method/URI**다. provider별 로그인/callback은 각각 세고 이미지 kind·월 값·목록 항목 수·framework 기본 오류 페이지는 별도 API로 세지 않는다.
- JSON body의 정수/실수는 number다. 날짜는 `YYYY-MM-DD`, Instant는 offset을 포함한 ISO-8601 문자열, 날씨 LocalDateTime은 서울 현지 시각(offset 없음), 목표 month는 `YYYY-MM`이다. 예시 ID·좌표·수치는 형식 설명용이며 현재 DB 실데이터를 의미하지 않는다.
- nullable 필드는 null을 허용한다. FE가 이전 BE 호환을 위해 일부 필드를 optional로 선언해도 현행 BE record의 응답 필드가 생략된다는 뜻은 아니다. 알려지지 않은 JSON 필드 전체를 금지하는 strict schema는 구현하지 않았다.
- Java Long 범위와 JavaScript 안전 정수 범위는 다르다. FE는 회원/활동 ID 등에 safe integer 검증을 사용한다. 대규모 ID/총점 범위 확대 시 별도 호환 검토가 필요하다.

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
| U07 | 이미지 | 공개 업로드 프로필 이미지 | GET | `/api/users/{userId}/images/profile?v={revision}` | 필요 | 구현 완료 |
| U08 | 회원 | 회원탈퇴 | DELETE | `/api/users/me` | 필요, CSRF | 구현 완료 |
| H04 | 점수 | 본인 점수 재산정 | POST | `/api/users/me/hiking-records/recalculate-score` | 필요, 프로필 완료 | 구현 완료 |
| R02 | 랭킹 | 상위 목록·본인 전체 순위 | GET | `/api/rankings/summary` | 필요 | 구현 완료 |
| G01 | 목표 | 월 목표 조회 | GET | `/api/users/me/monthly-goals/{month}` | 필요, 프로필 완료 | 구현 완료 |
| G02 | 목표 | 월 목표 교체/해제 | PUT | `/api/users/me/monthly-goals/{month}` | 필요, 프로필 완료 | 구현 완료 |

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
- Response: 성공 302, `${FRONTEND_URL}/?oauth=success`로 redirect(기본 `http://localhost:5173`). 실패도 302, `${FRONTEND_URL}/?oauth=failed`로 redirect하며 `Cache-Control: no-store`를 설정한다. 실패 원문·JWT·access token을 FE URL로 반환하지 않는다. 현재 Handler는 실패 401 JSON을 반환하지 않는다.

### A05. Kakao callback

- **GET `/login/oauth2/code/kakao` / 사전 로그인 불필요 / 구현 완료(Security)**
- Path: 없음. Query: provider의 `code`, `state` 또는 OAuth 오류 값. Request Body: 없음.
- 설명/Response: A04와 같은 Handler를 사용하며 성공·실패 모두 302다. FE는 `oauth=failed`를 읽어 실패를 표시하고 해당 query를 정리한다.

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
| provider | String | 아니오(현행 회원 DB) | google/kakao, 본인에게만 제공 |
| email | String | 가능 | 제공자 동의/등록 상태에 따라 없을 수 있음 |
| joinedAt | Instant | 가능 | 신규 회원 가입 시각; V8 이전 회원은 추정하지 않고 null |

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
  "backgroundImageUrl": null,
  "provider": "google",
  "email": null,
  "joinedAt": "2026-11-01T00:00:00Z"
}
```

`PublicMemberResponse`(R01, R02 항목의 공통 공개 필드):

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| userId | Long | 아니오 | 공개 프로필 이동용 ID |
| nickname | String | DB nullable | 공개 조회 대상은 프로필 완료 회원 |
| profileImageUrl | String | 가능 | 업로드 프로필의 U07 revision URL 우선, 없으면 OAuth 사진 |
| score | Long | 가능 | 저장된 점수 |

```json
{
  "userId": 7,
  "nickname": "산사람",
  "profileImageUrl": "https://example.com/provider-photo.jpg",
  "score": null
}
```

공개 DTO에는 출생연도·나이·이메일·provider·providerId·joinedAt·배경 이미지·세션/OAuth 내부 정보가 없다. 업로드 프로필 이미지의 공개 조회는 U07이며 **인증이 필요**하다.

U03은 위 네 필드에 아래 두 필드를 추가한 `PublicProfileResponse`다. R01/R02와 U03의 객체를 동일한 필드 집합이라고 가정하지 않는다.

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| totalHikeCount | long | 아니오 | 저장 거리≥500m AND elapsedMs≥1,200,000 또는 서버 검증 완등인 활동 수 |
| completedHikeCount | long | 아니오 | completed=true AND gpsValidationVersion=1인 활동 수 |

```json
{
  "userId": 7,
  "nickname": "산사람",
  "profileImageUrl": null,
  "score": 270,
  "totalHikeCount": 3,
  "completedHikeCount": 1
}
```

같은 날 같은 산의 별도 활동도 각각 세며 점수 상태/원장/일지 유무를 조건으로 추가하지 않는다. 상세 통계 기준과 FE 계산은 9장의 통계 절을 참조한다.

### U01. 내 정보 조회

- **GET `/api/users/me` / 인증 필요 / 구현 완료**
- Path/Query/Request Body: 없음.
- 설명: 현재 세션의 회원을 조회한다. 별도 `userId`를 받지 않는다. 프로필 미완료 회원도 조회 가능하다.
- Response: 위 `MemberResponse` JSON. `Cache-Control: no-store`. 주요 상태: 200, 인증 오류 401.
- 현재 응답은 기존 필드에 provider/email/joinedAt도 포함한다. 이 세 필드는 본인 전용이며 미상 가입일은 null이다.

### U02. 최초 프로필 설정 / 프로필 수정

- **PATCH `/api/users/me/profile` / 인증 필요 / 구현 완료**
- Path/Query: 없음. Request: `MemberUpdateRequest`, 아래 세 필드 모두 필수다. PATCH이지만 한 필드만 보내는 부분 수정은 아니다.

| 필드 | 자료형 | 검증 |
| --- | --- | --- |
| expectedUserId | Long | 필수 양수; 인증 Principal의 내부 회원 ID와 일치해야 함 |
| nickname | String | null/공백 불가, 최대 30자; 저장 시 trim |
| birthYear | Integer | 필수, 1900 이상 현재 연도 이하 |

```json
{
  "expectedUserId": 7,
  "nickname": "산사람",
  "birthYear": 2003
}
```

- 설명: 본인 이름·출생연도를 설정하고 프로필을 완료 처리한다. 닉네임 중복 금지 규칙은 없다. 이미지 수정은 U05/U06으로 한다.
- Response: 위 `MemberResponse` JSON. 성공 PATCH의 Controller는 별도 Cache-Control을 지정하지 않으며 Security 기본 헤더와 구분한다. 주요 상태: 200, 누락·null·0·음수·파싱 불가 ID/입력 검증 400, 인증 401, CSRF 403, 계정 불일치 409.
- 수정 대상을 expectedUserId로 선택하지 않는다. Principal의 회원을 찾고 **변경 전에** ID를 비교한다. 불일치 응답:

```json
{
  "error": "profile_account_mismatch",
  "message": "로그인 계정이 변경되었습니다. 현재 계정을 다시 확인해주세요."
}
```

message의 정확한 문구는 `ProfileAccountMismatch`를 따른다. 클라이언트는 문구 대신 status/error로 분기한다.

**FE 내부 저장 확인(추가 BE endpoint가 아님):** 입력/회원/화면 범위를 고정하고 CSRF 준비 후 GET U01로 계정을 재확인한 뒤 PATCH한다. PATCH 응답 뒤 GET U01이 실패하면 저장 완료·확인 실패로, 쓰기 전송 후 네트워크/timeout/5xx/잘못된 성공 응답은 저장 결과 불명확으로 구분한다. '저장 결과 확인'은 GET U01만 수행하며 PATCH를 자동 재전송하지 않는다. 새 계정/화면에 오래된 응답을 반영하지 않는다. 이는 BE가 저장됐는지 조회하는 별도 트랜잭션 상태 API나 입력값 일치 보장을 뜻하지 않는다.

### U03. 다른 사용자 공개 프로필

- **GET `/api/users/{id}/profile` / 인증 필요 / 구현 완료**
- Path: `id` Long, 대상 회원 ID. Query/Request Body: 없음.
- 설명: 대상 회원이 존재하고 프로필이 완료되어야 한다.
- Response: 위 `PublicProfileResponse`(공개 4필드+횟수 2필드) JSON, `Cache-Control: no-store`. 주요 상태: 200, 대상 없음 404, 대상 프로필 미완료 403, 인증 401.

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
- provider/email/joinedAt을 포함한 현행 본인 모델을 반환한다. 업로드 profile은 이후 공개 프로필/랭킹에서 U07 버전 URL로도 조회할 수 있지만 background는 본인 전용이다.
- 주요 상태: 200, 비어 있거나 유효하지 않은 이미지/해상도 검증 400, 업로드 용량 초과 413, kind 오류 404, 인증 401, CSRF 403.

### U06. 내 이미지 삭제

- **DELETE `/api/users/me/images/{kind}` / 인증 필요 / 구현 완료**
- Path: `kind`는 U04와 같음. Query/Request Body: 없음.
- 설명: 본인 이미지 삭제. 이미지가 없어도 204이며 프로필 완료는 필요하지 않다. 프로필 업로드 삭제 후 본인 조회는 provider 사진으로 돌아갈 수 있다.
- Response Body: 없음. 주요 상태: 204, kind 오류 404, 인증 401, CSRF 403.

### U07. 공개 업로드 프로필 이미지

- **GET `/api/users/{userId}/images/profile` / 인증 필요 / 구현 완료**
- Path: 숫자 정규식 `[0-9]+`에 매칭되는 userId(Long). Query: `v` String **필수**, Body 없음.
- 해당 프로필 완료 회원의 현재 업로드 프로필 bytes만 반환한다. 공개 배경 이미지 API는 없다. 업로드가 없을 때 provider URL로 redirect하지 않는다.
- Response: image/png 또는 image/jpeg binary, 200, Cache-Control: no-store, X-Content-Type-Options: nosniff.
- v 누락은 Spring 필수 query 검증 400, 빈 v/현재 revision 불일치/이미지 없음/대상 없음은 404, 대상 프로필 미완료는 403, 인증은 401. 기본 framework 400 body에 11장의 Map 형태를 보장하지 않는다.
- FE 사용: 공개 프로필·랭킹·일지 작성자 avatar가 DTO의 URL을 binary로 요청한다. URL의 공개성은 비로그인 허용을 뜻하지 않는다.

### U08. 회원탈퇴

- **DELETE `/api/users/me` / 인증·CSRF 필요 / 구현 완료**. 프로필 완료는 별도 필수 조건이 아니다.
- Path/Query 없음. Content-Type: application/json. Body `WithdrawalRequest`:

| 필드 | Java 자료형 | 필수/검증 |
| --- | --- | --- |
| expectedUserId | Long | 필수 양수, 현재 인증 회원과 일치 |
| confirmation | String | 필수, 공백 없이 정확히 `탈퇴` |

```json
{
  "expectedUserId": 7,
  "confirmation": "탈퇴"
}
```

- 성공 204, 빈 body, Cache-Control: no-store, 현재 세션 무효화 및 JSESSIONID 삭제. 입력 오류 400, 인증 401, CSRF 403, 예상 ID 불일치 409(`request_failed`, **profile_account_mismatch 아님**). DB 실패는 성공 204로 숨기지 않는다.
- 같은 트랜잭션에서 본인 점수 이벤트·일별 인증·활동 점수·공개 보너스·일지·활동·이미지 BYTEA·월 목표와 회원을 삭제한다. 다른 회원/공통 산·코스/외부 Google·Kakao 계정은 삭제하지 않는다. 공급자 동의 철회 API도 호출하지 않는다.
- 커밋 뒤 현재 OAuth authorized-client/세션을 정리한다. 다른 기기의 이전 세션은 다음 API에서 account_unavailable로 거절한다. 같은 소셜 계정의 재가입은 새 내부 회원 ID다.
- FE는 동의 체크 후 최종 버튼에서 confirmation을 채워 전송한다(사용자가 문구를 타이핑해야 한다는 UI 계약 아님). 성공 시 본인 찜/GPS 저장 키를 정리하되, 서버 탈퇴 성공과 브라우저 정리 실패를 구분한다. 실패/응답 유실은 완료로 단정하지 않는다.
- 백업/WAL/물리 저장소 삭제 완료까지 API가 보장하는 것은 아니다. 정책·이력은 [회원탈퇴](account-withdrawal.md)를 참조한다.

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

FE 실등산 선택 화면은 이 목록을 받은 뒤 이름/시·군을 필터링한다. 현재 탐색·검색·산/코스 선택은 서버 산 ID/목록을 사용한다. 지도 권역 배치·행정 경계는 FE의 정적 데이터이며 별도 산 API 응답이 아니다. 현재 규모의 단순 필터를 위해 별도 검색/지역 API를 추가 설계하지 않는다.

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
- FE 연결: 산 목록 상세와 GPS 시작 준비 화면의 `MountainWeather`가 호출한다. FE는 산별 5분 메모리 cache, 12초 timeout, 실패 재조회/오류/빈 예보를 구분한다. 기상청 외부 URL을 FE가 직접 호출하지 않는다.

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

**실제 산행 활동은 `HikingActivity`/`hiking_activities`**, **후기 일지는 `HikingRecord`/`hiking_records`**다. HTTP `hiking-records`는 활동이지만 Java `HikingRecordResponse`는 일지이므로 이름만으로 대응시키지 않는다.

회원은 Principal로 결정한다. 산/코스는 scalar ID와 DB FK로 연결되고 산 이름은 저장 시 DB snapshot이다. hikingDate는 startedAt의 Asia/Seoul 날짜다. FE의 completed는 기대값이고 **저장 completed는 서버가 제출 경로와 DB 정상 좌표를 대조한 결과**다. 실제 기기가 생성한 좌표/현장 이동의 진위까지 보증하지 않는다. GPS 원본 경로는 요청 검증에 쓰며 DB에는 요약·검증 버전·지문/해시만 저장한다.

### Request: HikingActivityRequest

| 필드 | Java 자료형 | 필수/검증 |
| --- | --- | --- |
| mountainId | Long | 필수 양수, 실제 산 존재 |
| courseId | Long | 선택/null 가능, 있으면 양수이고 해당 산에 속해야 함 |
| startedAt / endedAt | Instant | 둘 다 필수, 미래 불가, 종료≥시작 |
| distanceMeters | Double | 필수 유한한 0 이상, 단위 m |
| elapsedMs | Long | 필수 0 이상 정수, 단위 ms, 종료-시작 간격 이하; 일시정지 제외 활동 시간 |
| completed | Boolean | 필수; 클라이언트 정상 도달 기대값, 서버 결과와 달라도 정상 저장 가능 |
| clientRequestId | String | 필수, 8-4-4-4-12 hex UUID 형식, 저장 시 소문자 |
| gpsEvidence | GpsEvidence | 선택/null; 제공 시 중첩 검증, 전체 경로를 대체하지 못함 |
| gpsSegments | List<List<GpsPointRequest>> | 신규 저장 필수, 명시적 [] 허용. 구검증 버전 없는 기존 활동의 UUID 재조회만 생략 가능 |

| 경로 표본 필드 | Java 자료형 | 검증 |
| --- | --- | --- |
| lat / lng | Double | 필수 유한한 도 단위 위도 [-90,90]/경도 [-180,180] |
| measuredAt | Instant | 필수, 원본 시작~종료 사이(양 끝 포함), 모든 구간을 통틀어 비감소 |
| accuracyMeters | Double | 필수 유한한 0 이상 m; 일반 경로 형식 검증에는 50m 상한 없음 |

선택적 gpsEvidence는 `latitude`, `longitude`, `accuracyMeters`, `recordedAt` 네 필드가 모두 필수다. 위경도 범위는 동일, 정확도는 0~50m, recordedAt은 미래가 아니고 산행 구간 안이어야 한다. 이 단일 표본만 보내거나 전체 경로를 비워 일반 방문 점수 근거를 대신할 수 없다.

```json
{
  "mountainId": 1,
  "courseId": null,
  "startedAt": "2026-10-01T00:00:00Z",
  "endedAt": "2026-10-01T03:00:00Z",
  "distanceMeters": 600.0,
  "elapsedMs": 8040000,
  "completed": false,
  "clientRequestId": "123e4567-e89b-12d3-a456-426614174000",
  "gpsSegments": [
    [
      {
        "lat": 37.694604070364484,
        "lng": 127.3,
        "measuredAt": "2026-10-01T01:00:00Z",
        "accuracyMeters": 10.0
      },
      {
        "lat": 37.7,
        "lng": 127.3,
        "measuredAt": "2026-10-01T01:30:00Z",
        "accuracyMeters": 10.0
      }
    ]
  ]
}
```

위 예시는 M01의 가상 정상 좌표를 사용하는 형식 설명이다. 회원 ID·산 이름·hikingDate를 보내지 않으며 예시의 정상 도달이 실제 DB에서도 성립한다고 보장하지 않는다.

경로는 총 10,000표본·10,000구간 이하이며 null 구간/표본은 거절한다. 빈 구간은 허용한다. H01 JSON body는 unknown-length/chunked 및 알 수 없는 필드까지 포함한 실제 UTF-8 2MiB 이하이다.

모든 표본 정확도가≤50m이고 구간 내 양의 이동 간선이 있으면 거리 증거가 성립한다. 이때 반지름 6,371,000m 구면 거리의 구간 내 인접점 합을 제출 거리와 대조하며 차이가 max(1m, 경로합×0.000001)보다 크면 400이다. 구간 사이를 잇지 않으며 거리 증거가 없으면 대조를 생략한다. 서버 정상 도달은 정확도≤50m 표본이 DB 정상 반경≤30m에 있으면 true다. 최소 표본수/거리/시간은 저장의 필수 문턱이 아니며, 단일 유효 정상 표본도 정상 예외를 만들 수 있다.

### Response: HikingActivityResponse

| 필드 | Java 자료형 | null | 의미 |
| --- | --- | --- | --- |
| id / mountainId | Long / Long | 아니오 | 활동 ID / 산 ID |
| mountainName | String | 아니오 | 저장 시 산 이름 |
| courseId | Long | 가능 | 코스 ID |
| startedAt / endedAt | Instant | 아니오 | PostgreSQL JDBC와 같은 마이크로초 반올림으로 정규화한 시작/종료 |
| hikingDate | LocalDate | 아니오 | 서울 기준 시작 날짜 |
| distanceMeters | double | 아니오 | 저장 거리 m |
| elapsedMs | long | 아니오 | 저장된 일시정지 제외 활동 시간 ms |
| completed | boolean | 아니오 | 서버 제출 경로 대조 결과. 구기록은 기존 값 보존 |
| journalId | Long | 가능 | 연결된 일지 ID |
| gpsValidationVersion | Short | 가능 | 현행 검증 1, 과거 근거 없는 기록 null |
| scoring | ActivityScoringResponse | 가능 | 저장된 개별 활동 점수, 원장 없음은 null |

| scoring 필드 | Java 자료형 | null/의미 |
| --- | --- | --- |
| status | String | non-null; CERTIFIED/GPS_REQUIRED/BELOW_MINIMUM, DAILY_LIMIT는 과거 v1 호환 |
| certificationDate | LocalDate | non-null; 인증 실패에도 시작 시각의 서울 날짜 |
| visitPoints / summitPoints | int / int | non-null; 방문 0/100/200, 정상 0/70 |
| policyVersion | String | non-null; 해당 원장에 저장된 정책 버전 |

```json
{
  "id": 42,
  "mountainId": 1,
  "mountainName": "예시산",
  "courseId": null,
  "startedAt": "2026-10-01T00:00:00Z",
  "endedAt": "2026-10-01T03:00:00Z",
  "hikingDate": "2026-10-01",
  "distanceMeters": 600.0,
  "elapsedMs": 8040000,
  "completed": true,
  "journalId": null,
  "gpsValidationVersion": 1,
  "scoring": {
    "status": "CERTIFIED",
    "certificationDate": "2026-10-01",
    "visitPoints": 200,
    "summitPoints": 70,
    "policyVersion": "visit-general-or-verified-summit-multi-v2"
  }
}
```

요청 record를 서버 내부에서 Jackson으로 직렬화하면 검증 getter의 `gpsWithinSession`/`sessionValid`가 파생되어 나타날 수 있지만, FE가 제출하는 요청 필드나 활동 응답 필드는 아니다. 같은 방식으로 프로필 요청의 `birthYearValid`도 클라이언트 입력이 아니다. 검증 결과를 클라이언트가 지정하여 서버 검증을 대체할 수 없다.

elapsedMs는 TRACKING 활동 시간이다. 일시정지 시간은 제외하지만 움직이지 않고 TRACKING인 시간은 포함될 수 있다. BE는 실제 활동 시계를 복원하지 않고 제출값과 시각 간격을 검증한다. response에는 GPS 좌표, 지문, gpsEvidence 해시나 clientRequestId를 반환하지 않는다. 회원 총점·공개 보너스는 scoring 개별 항목에 포함되지 않는다.

### H01. 실제 산행 결과 저장

- **POST `/api/users/me/hiking-records` / 인증·프로필 완료·CSRF 필요 / 구현 완료**.
- Path/Query 없음. Request: 위 JSON, Content-Type: application/json.
- 신규 저장도 201이 아닌 **200**, Response: HikingActivityResponse. 산/코스/GPS/거리/시간 검증 400, 산 없음404, 프로필/CSRF403, 인증401, 충돌409, 본문 초과413.
- 회원 행 FOR UPDATE 아래 활동 저장→활동 점수·보너스·변경 이력·members.score 갱신을 같은 트랜잭션에서 수행한다. 점수 계산/DB 런타임 실패는 활동까지 롤백하며 성공으로 처리하지 않는다. 특정 DB 중복 제약은 서비스 롤백 후 409로 변환한다. 기타 DB 장애 전체를 특정 오류 Map으로 보장하지 않는다.
- 같은 회원의 UUID는 대소문자 구분 없이 대조한다. 현행 기록의 재전송은 정규화된 요약·원래 completed 기대값·전체 GPS의 버전 지문 및 선택적 표본 해시가 일치해야 한다. 같은 UUID/같은 본문은 기존 활동을 200으로 반환하고 점수는 재산정하되 누적 가산하지 않는다. 정상 도달 판정은 저장 결과를 재사용한다. 내용 불일치/구기록 UUID 대소문자 다중 충돌은 `409 client_request_conflict`.
- 다른 UUID라도 같은 회원·산·정규화 시작·종료 조합이면 `409 duplicate_hiking_activity`. 다른 시간의 별도 재방문은 허용한다. 구기록은 기존 요약/선택적 해시 호환 비교를 유지하고 GPS를 새로 추정해 인증하지 않는다.
- FE는 입력·UUID·증거를 고정하고 저장 확인 전 초안을 보존한다. 응답 유실은 결과 불명확이며 같은 요청으로만 재시도한다. 서버 completed가 기대값과 달라도 boolean으로 수용한다. 시작/진행/일시정지 전용 서버 API는 없다.

### H02. 내 등산기록 목록

- **GET `/api/users/me/hiking-records` / 인증 필요 / 구현 완료**. Path/Query/Body 없음.
- 본인 모든 활동을 startedAt DESC, id DESC로 반환. 프로필 완료 불필요, pagination 없음.
- Response: 위 HikingActivityResponse의 배열(없으면 []), 200/인증401. 각 원장·검증 버전·journalId도 포함한다. 목록 GET은 점수를 재계산하지 않는다.
- FE: 기록 탭·일지 후보·마이페이지 통계·검증 완등 목록에 실제 사용한다.

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
    "distanceMeters": 600.0,
    "elapsedMs": 8040000,
    "completed": true,
    "journalId": 10,
    "gpsValidationVersion": 1,
    "scoring": {
      "status": "CERTIFIED",
      "certificationDate": "2026-10-01",
      "visitPoints": 200,
      "summitPoints": 70,
      "policyVersion": "visit-general-or-verified-summit-multi-v2"
    }
  }
]
```

### H03. 내 등산기록 상세

- **GET `/api/users/me/hiking-records/{id}` / 인증 필요 / 구현 완료**.
- Path id(Long, 활동 ID), Query/Body 없음. Response: H01과 같은 객체, 200/없음·타인 활동404/인증401. 프로필 완료 불필요. 현재 FE 호출은 없다.

### H04. 본인 점수 재산정

- **POST `/api/users/me/hiking-records/recalculate-score` / 인증·프로필 완료·CSRF 필요 / 구현 완료**.
- Path/Query/Body 없음. 200 `ScoreSummaryResponse`, 인증401/프로필·CSRF403. 일반 DB 실패는 전파한다. 현재 FE 화면/함수 호출은 없다.
- 본인 회원 잠금 아래 현재 원천을 재산정하며 v1 원장/금액과 원장 없는 과거 활동을 소급 변경하지 않는다. GET 조회는 이 endpoint를 자동 실행하지 않는다.

| 응답 필드 | Java 자료형 | null/단위 |
| --- | --- | --- |
| totalPoints / visitPoints / summitPoints / publicBonusPoints | long | non-null, 점수 정수; 전체/방문/정상/공개 보너스 합계 |
| policyVersion | String | non-null, 현행 실행 정책 버전; 모든 구성 원장이 v2라는 뜻은 아님 |

```json
{
  "totalPoints": 320,
  "visitPoints": 200,
  "summitPoints": 70,
  "publicBonusPoints": 50,
  "policyVersion": "visit-general-or-verified-summit-multi-v2"
}
```

활동 수정·삭제·타인 활동 공개·원시 GPS 조회 API는 없다.

## 8. 등산일지 API

### 관계 / legacy 호환

`HikingRecord`는 회원과 연결된 일지이며 `HikingActivity`를 optional OneToOne로 참조한다. 원칙은 **활동 1 : 일지 0..1**이다. V4에서 `hiking_records.hiking_activity_id` nullable FK와 UNIQUE를 추가했다. 기존 일지는 값을 null로 남겨 보존하며 활동을 임의 생성하거나 억지로 연결하지 않는다.

새 일지 생성 시 회원 행 다음 본인 활동을 비관적 쓰기 lock으로 조회하고 이미 일지가 있는지 검사한다. 서비스 검사와 DB UNIQUE 모두 중복을 막는다. 새 일지의 산 이름과 날짜는 활동에서 복사하며 최초 생성 이후 수정하지 않는다. 정상 도달 `completed=true`만 후보로 제한하는 조건은 없다. 생성/수정/삭제는 일지 변경과 점수 재산정을 같은 트랜잭션에서 수행한다. 현재 정책 산의 마지막 공개 일지를 비공개/삭제하면 보너스 50을 회수하고 재공개하면 복원한다. 같은 산의 다른 공개 일지가 남으면 유지하며 중복 지급하지 않는다. v1 보너스의 저장 금액·버전은 공개 상태 변경으로도 바꾸지 않는다. 일지 사진 첨부/업로드 API와 이미지 필드는 없다. U05는 회원 프로필·배경 사진이며 일지 사진 API가 아니다.

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
- 공개 일지라면 회원·산별 보너스 재산정을 같은 트랜잭션에서 수행한다. 기존 v1 보너스 금액·버전은 보존한다.
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
- 공개 상태 변경에 따른 현재 정책 보너스 지급·회수도 함께 재산정한다. 같은 산의 공개 일지가 남으면50을 유지하며 v1 보너스는 소급 변경하지 않는다.
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
- 삭제 후 현재 정책 보너스를 재산정하되 활동의 방문·정상 점수와 v1 보너스의 저장 금액·버전을 보존한다.
- Response Body: 없음. 주요 상태 204, 없음/타인 일지 404, 프로필 미완료/CSRF 403, 인증 401.

## 9. 랭킹 / 점수 API

### R01. 전체 랭킹

- **GET `/api/rankings` / 인증 필요 / 구현 완료**.
- Path/Query/Body 없음. 프로필 완료 회원 최대 100명, score non-null 우선 → score DESC → id ASC. null도 참여하며 기간/지역/페이지 파라미터는 없다.
- Response: `List<PublicMemberResponse>`, 200, Cache-Control: no-store, 인증401. rank 필드는 없다. FE에 getRankings helper는 있으나 현재 랭킹 화면은 R02를 사용한다.

```json
[
  {
    "userId": 7,
    "nickname": "산사람",
    "profileImageUrl": "/api/users/7/images/profile?v=123e4567-e89b-12d3-a456-426614174000",
    "score": 270
  },
  {
    "userId": 8,
    "nickname": "걷는사람",
    "profileImageUrl": null,
    "score": null
  }
]
```

### R02. 상위 목록·본인 전체 순위

- **GET `/api/rankings/summary` / 인증 필요 / 구현 완료**.
- Path/Query/Body 없음. 200 RankingSummary, Cache-Control: no-store, 인증401. 현재 인증 회원의 profileCompleted는 호출 필수 조건이 아니지만 미완료면 mine=null이다.
- 단일 SQL에서 전체 참여자를 먼저 정렬해 ROW_NUMBER와 참여자 수를 계산한다. 동점은 회원 ID 오름차순의 서로 다른 순번이다. 상위 100명 밖의 본인 순위도 mine에 포함한다. 100위 밖 회원을 members에 추가하지 않는다. null score는 마지막이며 0점과 다르다.

| 필드 | Java 자료형 | null/의미 |
| --- | --- | --- |
| members | List<RankingEntry> | non-null, 상위 최대100, 없으면 [] |
| mine | RankingEntry | 본인이 참여 대상이 아니면 null |
| totalMembers | long | non-null, null score를 포함한 프로필 완료 참여자 수 |
| RankingEntry.userId / nickname / profileImageUrl / score | Long / String / String / Long | userId non-null, 나머지 공개 R01 모델과 같음 |
| RankingEntry.rank | long | non-null, 전체 기준 1부터 순번 |

```json
{
  "members": [
    {
      "userId": 7,
      "nickname": "산사람",
      "profileImageUrl": null,
      "score": 270,
      "rank": 1
    }
  ],
  "mine": {
    "userId": 7,
    "nickname": "산사람",
    "profileImageUrl": null,
    "score": 270,
    "rank": 1
  },
  "totalMembers": 1
}
```

FE는 이 결과를 임의 점수 재계산/재정렬 없이 사용하고 null은 '점수 미산정', 0은 '0점'으로 표시한다. 새로고침 버튼·화면 재진입·foreground 복귀로 재조회하며 계정 범위 변경/abort 뒤 오래된 결과를 버린다. 오류와 빈 결과·인증 실패를 구분한다.

### 현재 점수 저장과 v1/v2 호환

신규 산행 저장→활동 점수→공개 보너스→변경 이력→회원 총점 갱신은 동일 트랜잭션이다. 일지 생성/공개 변경/삭제와 H04도 재산정을 수행한다. 회원 총점은 원천 합계로 교체해 members.score에 저장하며 U01/U03/R01/R02는 저장된 값을 읽는다. 랭킹 조회 자체는 재산정하지 않는다. 커밋된 변경은 다음 조회에 반영한다.

현행 `visit-general-or-verified-summit-multi-v2`: 일반 인증은 거리≥500m AND elapsedMs≥1,200,000 AND gpsRecorded(전체 경로의 거리 근거)이고, 서버 검증 completed/gpsValidationVersion=1 정상은 거리·시간 예외다. 첫 인정 산 방문200, 같은 산 재방문100, 검증 정상70, 회원·산별 공개 일지50. v2 일일 제한·속도·높이 가산점 없음. 같은 날 별도 활동도 점수 대상이다.

v1 활동 원장·DAILY_LIMIT·일별 인증은 보존하고 소급 v2로 재인증하지 않는다. 과거 원장 없는 활동에도 점수를 새로 추정하지 않는다. v1 공개 보너스는 저장된 금액(0/50)·버전을 보존하며 일지 비공개/삭제/재공개에도 바뀌지 않는다. 같은 산에 v2 보너스를 중복 생성하지 않는다. 현재 보너스 합계는 보존된 구정책 금액 + 현재 정책 산별 0/50이다. 상세 원본은 [점수·통계 정책](SCORE_POLICY.md)을 따른다.

### 산행·완등·거리 통계는 점수와 독립

- 산행 수: `(distanceMeters >= 500 && elapsedMs >= 1200000) || (completed === true && gpsValidationVersion === 1)`.
- 완등 활동 수: `completed === true && gpsValidationVersion === 1`만 센다. 과거 completed만으로 자동 인정하지 않는다.
- 같은 날·산의 서로 다른 활동은 각각 인정한다. FE 고유 완등 산 수/목록만 산 ID를 중복 제거한다.
- 일반 횟수에는 GPS 점수 근거·scoring 상태·점수 원장 존재를 추가 조건으로 넣지 않는다. DAILY_LIMIT/0점/원장 없음도 거리·시간을 만족하면 횟수에 포함한다.
- U03은 전체 산행/완등 활동 수를 서버 SQL로 제공한다. 마이페이지 월/전체 횟수·완등 산 수·거리는 H02의 활동들을 FE에서 집계한다. 거리 합계는 인정 활동만이 아닌 **저장된 모든 활동 거리**이며 월 구분은 startedAt의 한국 시간이다. 총 m 합계 후 km로 표시한다. 소요 시간은 저장 elapsedMs를 표시하며 별도 서버 평균속도/월 통계 API는 없다.

### G01. 월 목표 조회

- **GET `/api/users/me/monthly-goals/{month}` / 인증·프로필 완료 필요 / 구현 완료**.
- Path month(String): 1900년 이상 네 자리 연도와 실제 두 자리 월의 YYYY-MM. Query/Body 없음. 잘못된 월400, 프로필403, 인증401.
- 200 MonthlyGoalResponse, Cache-Control: no-store. 미설정은 404가 아닌 두 목표 null로 응답한다. FE 마이페이지에서 사용한다.

### G02. 월 목표 교체/해제

- **PUT `/api/users/me/monthly-goals/{month}` / 인증·프로필 완료·CSRF 필요 / 구현 완료**.
- Path는 G01과 같음. Request application/json `MonthlyGoalRequest`. 거리 BigDecimal 1~1000km·소수점 최대3자리, hikeCount Integer 1~1000회. 각 필드는 선택/null 가능하며 둘 다 null/생략({})이면 행 삭제. 한 필드만 제공하면 다른 필드는 기존 값을 유지하지 않고 null로 교체한다.

```json
{
  "distanceKm": 12.345,
  "hikeCount": 3
}
```

- Response 200 MonthlyGoalResponse, Cache-Control: no-store. 입력/월400, 인증401, 프로필/CSRF403. 회원별 월 잠금 아래 저장하며 점수·활동은 수정하지 않는다.

| 응답 필드 | Java 자료형 | null/의미 |
| --- | --- | --- |
| userId | long | non-null, 현재 회원 |
| month | String | non-null, YYYY-MM |
| distanceKm | BigDecimal | 선택/null, km |
| hikeCount | Integer | 선택/null, 회 |

```json
{
  "userId": 7,
  "month": "2026-11",
  "distanceKm": 12.345,
  "hikeCount": 3
}
```

목표는 서버에 저장하며 FE localStorage가 원본이 아니다. FE는 회원·월 응답을 검증하고 PUT 직전 U01로 계정을 다시 확인한다. BE의 월 목표 body에는 expectedUserId가 없고 소유자는 Principal로 결정한다. 월 목표 진행률은 H02 통계와 이 목표를 FE에서 비교하며, 목표 설정이 산행 횟수나 점수를 만들지는 않는다.

## 10. 구현 예정 API 및 향후 개선

### 구현 예정 API

이번 확인에서 추가가 확정된 별도 API는 0개다. 산 상세/검색/필터는 M01~M03과 FE 로직을 사용한다. 업로드 프로필 공개(U07), 점수 재산정(H04), 월 목표(G01/G02), 회원탈퇴(U08), 본인 순위(R02)는 이미 구현되어 예정 목록에 두지 않는다.

### 현재 구현과 추가 검토의 경계

점수 자동 계산·원장·members.score 갱신 및 제출 경로의 거리/정상 검증은 구현되어 있다. 서버는 기기 좌표 진위·실제 현장 이동을 완전히 증명하지 않으며 원본 GPS 경로를 DB에서 보관/조회하는 API도 없다. 활동 수정/삭제·일지 사진·즐겨찾기 서버 저장·공동 순위/자유 pagination·지역 챌린지는 현행 계약이 아니다. 이런 기능의 정책을 확정된 신규 endpoint처럼 작성하지 않는다.

## 11. 실제 오류 응답

`GlobalExceptionHandler`는 특정 예외를 Map으로 반환한다. 별도 공통 `ErrorResponse` DTO는 없고 `ErrorCode`는 비어 있는 enum이다. `BusinessException` 선언만으로 공통 business 오류 처리 체계가 있는 것으로 간주하지 않는다.

| 발생 위치/조건 | 상태 | error | message |
| --- | --- | --- | --- |
| Security 인증 진입점 | 401 | unauthorized | 로그인이 필요합니다. 다시 로그인해주세요. |
| Security CSRF 실패 | 403 | invalid_csrf | 인증 토큰이 없거나 만료되었습니다. /api/csrf 조회 후 반환된 헤더와 토큰으로 다시 요청해주세요. |
| Security 기타 접근 거부 | 403 | forbidden | 이 요청에 대한 권한이 없습니다. |
| OAuth 실패 Handler | 302 | JSON 없음 | FE `/?oauth=failed`로 redirect |
| ActiveMemberSessionFilter 구세션 | 401 | session_upgrade_required | 다시 로그인 안내 |
| ActiveMemberSessionFilter 회원 없음/탈퇴 | 401 | account_unavailable | userId 추가 필드 포함, 본 요청의 이전 내부 회원 ID |
| ActiveMemberSessionFilter DB 조회 장애 | 503 | session_check_unavailable | 세션을 유지하고 재시도 안내 |
| 프로필 인증 계정 불일치 | 409 | profile_account_mismatch | 회원을 바꾸지 않고 거절 |
| 활동 동일 UUID의 본문 충돌 | 409 | client_request_conflict | 기존 요청 내용 충돌 안내 |
| 다른 UUID의 동일 활동 | 409 | duplicate_hiking_activity | 중복 산행 안내 |
| 활동 body 2MiB 초과 | 413 | hiking_request_too_large | 산행 요청은 2MiB 이하여야 합니다. |
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

위 예시는 각각 J01의 404, J01의 409에서 사용하는 reason이다. H03의 404는 reason 없이 발생하므로 message는 `요청을 처리할 수 없습니다.`다. U08 탈퇴 계정 불일치는 일반 ResponseStatusException의 request_failed다. 프로필 전용 profile_account_mismatch와 구분한다. 모든 예외를 포괄하는 Handler는 아니다. DB 무결성 예외, path 타입 변환, multipart 필수 part 누락 등 명시적으로 처리하지 않은 오류에 위 두 필드 형식을 일괄 보장하지 않는다. 프레임워크 기본 오류 응답은 별도 실행 환경에 따라 확인해야 한다.

## 12. FE-BE 연결과 현재 사용자 흐름

API helper의 존재와 실제 화면 호출을 구분한다. 계정 전환 확인용 GET/로컬 expectedUserId 변수는 해당 BE body 필드가 있다는 의미가 아니다.

| 기능 | 연결 상태 | 실제 흐름 |
| --- | --- | --- |
| Google/Kakao·본인 조회·CSRF·로그아웃 | 연결 | 브라우저 redirect, credentials include, 변경 요청마다 세션 CSRF |
| 프로필 최초 설정/수정 | 연결 | expectedUserId/nickname/birthYear PATCH, 이후 GET 확인, 미확정 상태 구분 |
| 탈퇴 | 연결 | 동의 후 명시적 body/CSRF, 204 확인, 계정별 저장소 정리 |
| 본인/공개 프로필 사진 | 연결 | 본인 PUT/GET/DELETE, 공개 avatar는 U07; 배경은 본인 전용 |
| 산·코스·경로/탐색 | 연결 | M01~M03, 서버 산 ID, 이름/시군 검색·정렬은 FE; 지도 권역/경계는 정적 자산 |
| 날씨 | 연결 | MountainWeather→W01, FE 산별 5분 cache/12초 timeout |
| GPS 시작/진행/일시정지/복구 | FE/네이티브 내부 | 웹 저장소 체크포인트, iOS 네이티브 Core Location/SQLite. 서버 실시간 세션 API 없음 |
| 산행 종료 저장/재시도 | 연결 | 고정 UUID·gpsSegments·요약, 2xx 본문 확인 후 서버 completed 수용 |
| 활동 목록/기록/일지 후보/통계 | 연결 | H02, 활동 단위 통계·서버 검증 완등 산 목록 |
| 활동 상세 H03 / 재산정 H04 | 미연결 | 현재 FE 호출 없음; 활동 저장 시 BE 자동 점수 갱신은 별개 |
| 일지 CRUD·공개 피드·사용자 목록 | 연결 | J01~J07, 상세 author를 통해 공개 프로필 이동 |
| 랭킹 | 연결 | R02 사용, R01 helper는 현재 화면 미사용 |
| 월 목표 | 연결 | G01/G02 서버 저장, 진행률은 FE 집계 |
| 즐겨찾기 | FE localStorage | 인증 회원 ID별 키, 서버 즐겨찾기 API 없음 |

### 등산 종료부터 일지 저장까지

1. 서버 산/코스 ID를 선택하고 FE에서 시작 시각·UUID와 활동 세션을 만든다.
2. 웹 GPS 또는 iOS 네이티브 수집 자료로 구간·거리·일시정지 제외 시간을 관리한다. 플랫폼/복구 상태별 세부 절차는 FE의 [iOS 문서](https://github.com/csie2026/FE/blob/dev/docs/ios-background-gps.md)를 따른다.
3. 종료된 요약과 전체 gpsSegments를 고정해 H01로 전송한다. 임시 기록·네이티브 세션의 소유 회원과 현재 계정을 확인한다.
4. BE가 경로/거리/정상·중복을 검사하고 활동과 점수를 같은 트랜잭션에서 저장한다. FE는 응답을 확인한 뒤에만 초안 저장 완료 처리한다. 실패/유실은 초안을 보존하고 같은 요청으로 재시도한다.
5. FE 일지 작성은 본인의 journalId=null 활동을 선택한다. 서버는 J01에서 활동 소유권·중복을 재확인하고 산/날짜를 활동에서 결정한다. 중복409 시 FE가 후보를 재조회한다.
6. J06은 제목/내용/공개 여부만 교체한다. J07은 활동을 남긴다. legacy null 연결 일지는 기존 산/날짜를 보존하며 수정/삭제할 수 있다.

FE 공통 API는 기본20초 제한, credentials include/cache no-store이며 CSRF 준비 후 변경 요청을 보낸다. 자동 재전송은 없다. 클라이언트의 ApiError status=0, network_error/request_timeout 및 invalid_response는 **BE가 반환한 HTTP 상태·오류 코드가 아닐 수 있다**. 204는 JSON으로 파싱하지 않는다. 모든 API에 동일한 계정 원자성 body가 있는 것은 아니며, 이미지·목표·활동은 FE 사전 확인과 Principal 권한 검증을 사용한다.

## 13. 개인정보 / 권한 확인

- 본인 전용 프로필·이미지·활동·일지 API는 인증 principal에서 회원을 결정한다. 클라이언트가 userId를 지정하여 소유자를 바꾸는 요청 필드는 없다.
- H03은 타인 활동 ID를 404로 숨긴다. H02는 본인만 반환한다. 원본 GPS 경로를 서버 DB에서 보관/조회하는 API는 없다. H01은 원본을 전송받아 요청 중 검증한다.
- J01은 활동 ID와 현재 회원을 함께 lock 조회하므로 타인 활동으로 일지를 만들 수 없다. 존재하지 않는 ID도 404다.
- 비공개 일지는 J03/J04에 포함되지 않고 J05에서는 작성자 외 404다. J06/J07은 작성자만 가능하며 타인 ID는 404다.
- 공개 DTO에는 birthYear/age/email/provider/providerId/인증정보가 없다. 본인 U01 응답의 birthYear/age가 공개 응답에 그대로 재사용되지 않는다.
- 본인 이미지 endpoint는 본인만 읽는다. U07은 프로필 완료 회원의 업로드 프로필을 다른 로그인 회원에게 제공한다. 공개 배경 이미지는 없고 revision이 달라지면 이전 URL은404다.
- 공개 일지의 활동 ID 노출은 활동 상세 접근 권한을 부여하지 않는다. 연결 ID까지 숨길 필요가 있는지는 별도 제품 정책이다.
- 프로필이 미완료여도 기존 활동/본인 일지/접근 가능한 상세 읽기는 가능하지만 활동·일지 쓰기는 완료가 필요하다. 공개 사용자 프로필과 사용자별 공개 일지 목록은 대상의 완료 상태도 검사한다.

## 14. 코드 대조 근거와 검토 필요 사항

### 주요 근거 파일

BE 경로는 `src/main/java/com/ggmount/` 기준이다.

| 범위 | 근거 |
| --- | --- |
| Method/URI/헤더 | global/auth/oauth/SessionController, member/controller/*, mountain/controller/MountainController, weather/WeatherController, hiking/controller/*, ranking/controller/RankingController |
| 인증/오류 | global/config/SecurityConfig, global/auth/oauth/ActiveMemberSessionFilter·OAuth2LoginSuccessHandler·OAuth2LoginFailureHandler, global/exception/GlobalExceptionHandler |
| 회원/이미지/탈퇴/목표 | member/dto/*, member/service/*, member/repository/PersonalDataRepository |
| GPS/활동/일지 | hiking/dto/*, hiking/service/GpsEvidence·HikingActivityService·HikingRecordService·HikingRequestFingerprint |
| 점수/랭킹 | hiking/service/HikingScorePolicy·HikingScoreService, ranking/service/RankingService, member/repository/MemberRepository |
| FE 계약/사용 | src/api.ts, src/services/responseValidation.ts·profileUpdate.ts·withdrawal.ts·monthlyGoals.ts·ranking.ts·hikingStatistics.ts·completedHikes.ts·mountainWeather.ts, App.tsx 및 관련 components/hooks |
| DB | BE/src/main/resources/db/migration/V1~V8(3.1 포함), application.yaml 및 local/prod/mobile 설정 |

### Migration 및 기존 데이터

V1 회원/일지, V2 산, V3 코스, V3.1 이미지/월 목표, V4 활동 연결, V5 점수 원장, V6 선택적 월 목표, V7 GPS 지문/동일 세션 제약, V8 가입 시각으로 총9개다. 적용된 SQL checksum/이력을 임의 변경하거나 repair/초기화하지 않는다. V4는 legacy 일지 null 연결을 보존하고 V7은 기존 GPS metadata를 null로 남긴다. V8은 기존 회원의 가입 시각을 과거 활동으로 추정하지 않는다. 기록 없는 활동/가입일을 자동 backfill하지 않는다.

### API별 FE 사용·테스트 대조

아래는 코드 대조 및 선행 동일 HEAD 테스트 결과의 범위다. `PG`는 신규 분리 PostgreSQL 14.24 검증, `H2`는 기존 메모리 DB suite, `FE`는 fetch 대체 단위 테스트를 포함하며 실제 OAuth/실기기 E2E가 아니다.

| API ID | FE 사용 | 검증 근거/한계 |
| --- | --- | --- |
| A01 | 공통 변경 요청 | Security/프로필/활동 PG·H2, FE api/CSRF |
| A02/A03 | 브라우저 로그인 | OAuth mapper/회원 생성/handler H2·단위, 최초 동시 로그인 PG; 제공자 실제 왕복 미실시 |
| A04/A05 | provider callback | OAuth2LoginHandlerTests; 실제 provider callback 운영 검증 아님 |
| A06 | 로그아웃 버튼 | ProductionSessionTests, FE logout/session 테스트 |
| U01 | 진입·프로필·계정 재확인 | ProfileJournal/AccountSafety H2, PG 보조 및 FE |
| U02 | 최초 설정/수정 | ProfileAccountSafetyIntegrationTests 7, PG 보조(expectedUserId/CSRF), FE profileUpdate |
| U03 | 공개 프로필 | PublicProfileStatistics/Image H2, PG 보조 통계 및 FE publicProfile |
| U04/U05/U06 | 본인 avatar/background | MemberImage/PersonalData 관련 H2, FE memberImages |
| U07 | 공개 avatar binary | PublicProfileImageIntegrationTests H2, FE 이미지 URL 사용 |
| U08 | 설정의 탈퇴 | WithdrawalIntegrationTests 9 H2, ActiveMemberSessionFilter 단위, FE withdrawal |
| M01/M02/M03 | 탐색/코스 선택/지도 | MountainDataContract/SeedData H2, FE courseApi/explore/응답검증 |
| W01 | 산 상세/시작 준비 날씨 | KmaForecastClient/HikingWeatherRules 단위, FE mountainWeather; live 1개는 키 미제공 스킵 |
| H01 | 종료/복구 저장 | 기존 PG 32 중 GPS/멱등/원장/롤백/동시성, HikingScoring H2 및 FE activitySave |
| H02 | 기록·통계·완등·일지 후보 | HikingStatisticsSource/PublicProfileStatistics H2, PG 보조 계정 분리, FE 통계 |
| H03 | 현재 미사용 | ProfileJournal H2와 PG의 본인/타인 활동 접근 검증 |
| H04 | 현재 미사용 | HikingScoring H2, PG 보조 v1 금액 보존 |
| J01~J07 | 작성·피드·내/타인 목록·상세·수정·삭제 | ProfileJournal H2, PG 생성/공개/삭제/보너스 및 FE hikingRecords/publicProfile |
| R01 | helper만 존재 | RankingIntegrationTests H2, FE ranking DTO helper |
| R02 | 랭킹 화면 | RankingIntegrationTests 6 H2, PG 보조 원장·총점·순위, FE ranking/rankingState |
| G01/G02 | 마이페이지 목표 | MonthlyGoalIntegrationTests 10 H2, FE monthlyGoals/응답검증. PG 전용 목표 CRUD suite는 미실시 |

선행 2026-11-13 같은 BE HEAD의 검증: 표준 javac 전체 verify **216개 중215통과·실패0·스킵1**(외부 날씨). 이는 저장소 기존 기본179개 + 활성 PG32개 + 저장소 밖 PG보조4개 + weather1개다. 기존 코드만의 활성 집계는212개다. FE 동일 코드 HEAD의 선행 테스트는352개 통과 및 build 성공이다. 이번 문서 수정 자체로 테스트를 새 실행한 수치와 혼동하지 않는다. 운영 DB migration 완료나 실기기 위치 진위까지 증명하는 결과는 아니다.

### 이번 2차 문서 검증 (2026-11-13)

실제 실행한 Spring HandlerMapping에서 Controller 28개를 다시 추출하고 Security 5개를 합해 Method/URI 33개를 확인했다. 소스 추출·문서 목록·FE 호출 경로를 대조했고 누락/중복 endpoint는 없었다. 문서 JSON 28개를 다시 파싱했으며 그중 DTO 객체 예시 25개는 현재 Spring Jackson으로 역직렬화·재직렬화하고 Request Bean Validation을 실행했다. 프로필 ID 누락/null/0/음수/소수, 계정 불일치409, 미인증401, CSRF403, 활동404, 목표 해제200 및 이미지 필수 query400을 실제 MockMvc로 확인했다.

이번 실행은 BE 기존 테스트95개와 저장소 밖 문서 검증3개 총98개가 모두 통과했다. 표준 javac와 새로운 H2 메모리 DB만 사용했으며 PostgreSQL/AWS는 사용하지 않았다. FE 테스트352개와 TypeScript/Vite build도 새로 통과했다. 이 결과를 위 선행 PostgreSQL verify와 구분한다. 외부 OAuth 왕복·실기기·모든 오류 조합의 E2E 검증을 뜻하지 않는다.

DOCS fetch와 원격 HTTP 접근은 실패했다. 캐시된 origin/dev `01c5013`는 로컬 dev `df7063e`보다 문서 커밋1개 앞서지만 최신 원격 HEAD인지는 확인하지 못했다. 해당 커밋의 기존 API 본문은 로컬 HEAD 원본과 같고 추가 FE 이력3개는 내용 보존/공통 원본 연결을 확인했다. Git merge/pull 없이 문서 내용만 통합했으며 상세 근거는 [검증 보고서](FINAL_DOCUMENTATION_AUDIT.md#2차-통합-및-전체-markdown-내용-검증-2026-11-13)에 기록한다.

### 남은 확인/개선 항목

1. OAuth 제공자 실제 로그인과 실시간 기상청/현장 GPS는 별도 환경 검증 대상이다. 원시 경로 저장·활동 수정/삭제·추가 챌린지는 미구현이다.
2. FE weather 타입은 일부 필드를 nullable/optional로 넓게 허용하지만 현행 BE는 days/notes/hours 및 강수·적설을 각각 배열/숫자로 반환한다. 명세는 BE 실제 계약을 따른다. 임의의 추가 필드를 보내도 값이 반영된다고 보장하지 않는다.
3. FE 이미지/목표/활동의 사전 계정 GET은 프로필 PATCH의 expectedUserId 원자 비교와 동일한 BE 계약이 아니다. 목적에 맞는 권한 경계를 구분한다.
4. 문서의 endpoint는 로컬 소스에서 확인했고 운영 DB 데이터·배포 URL·권한 있는 GitHub 접근은 별개다. 배포 TLS/cookie/proxy는 설정된 profile에 따라 검증해야 한다. WebConfig는 빈 클래스이며 활성 CORS 정책이 아니다.
5. API 문서에서 산 상세 API/일지 사진/서버 즐겨찾기 API/공동 순위를 새로 설계하지 않는다. 검증 중 코드 의심 사항과 문서 배치·링크·PDF 상태는 [문서 정합성 결과](FINAL_DOCUMENTATION_AUDIT.md)에 기록한다.

## 15. 최종 요약

| 구분 | 개수 |
| --- | --- |
| 현재 구현 API | **33**: Controller 28 + Security 5 |
| 종전 명세 누락 → 추가 | **6**: U07/U08/H04/R02/G01/G02 |
| 종전 명세에만 있는 미구현 endpoint | **0** |
| 별도 추가가 확정된 예정 API | **0** |

로그인·프로필 계정 보호·탈퇴·본인/공개 프로필 이미지·산/코스·날씨·활동/GPS·일지·점수·랭킹·통계·월 목표를 최신 코드와 대조했다. 기존 endpoint ID는 유지했고 새 경로만 추가했다. 점수 v2와 통계는 독립이며 v1 금액/버전은 보존한다. FE 미사용 경로(H03/H04/R01 화면)를 구현 누락으로 오인하지 않는다. 현재 계약·기획 확장·선행 테스트 실적과 실행 불가 범위를 분리한다.
