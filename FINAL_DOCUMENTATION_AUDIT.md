# API·FE·BE·DOCS 문서 정합성 정비 결과

검토일: 2026-11-13. 변경은 문서만이며 기능·기존 테스트·마이그레이션은 수정하지 않았다.

> 아래 A~D는 1차 작업 이력이다. 원격 동기화 판단과 이번에 새로 실행한 검증은 문서 하단의 [2차 통합 결과](#2차-통합-및-전체-markdown-내용-검증-2026-11-13)를 우선한다. 1차의 링크 복구 판단을 원격 최신 상태 확인 결과로 해석하지 않는다.

## A. API 명세서

원래 `ToPeak API 명세서.md`의 1~15장 구성·기존 API ID·표·유효한 설명과 예시를 바탕으로 [API_SPEC.md](API_SPEC.md)를 최신 원본으로 정리했다. 기존 한글 경로는 호환 링크로 유지한다. 새로운 명세를 원문과 무관하게 처음부터 작성한 작업이 아니다.

- BE 코드 기준: `bd102653758f104cb9369d288ce1f1ebda35c337`.
- FE 코드 기준: `66ceb1fe288d9047cd821b986b4c33e4f030e75b`.
- 전체33개 = Controller28 + Security5. 기존 명세27개에서 누락6개를 추가했다.
- 누락 추가: U07 공개 프로필 이미지, U08 탈퇴, H04 점수 재산정, R02 랭킹 summary, G01/G02 월 목표.
- 기존 API 상세15개 수정: A04/A05/U01/U02/U03/U05/M01/W01/H01/H02/H03/J01/J06/J07/R01. 공통 인증·응답 모델·정책의 영향 범위는 이 개수에 중복 합산하지 않는다. 신규6개 포함 추가·수정 상세는21개다.
- 명세에만 있던 미구현 Method/URI는0개였다. 수정 후 실제 BE 대비 누락/과잉 endpoint도0개다.

### 주요 정정

- OAuth 실패는401 JSON이 아니라 FE oauth=failed로302 redirect.
- 프로필 expectedUserId 필수 양수, 사전 원자 비교와409/profile_account_mismatch. 탈퇴의409/request_failed와 구분.
- FE 프로필 PATCH 성공 뒤 GET 재확인, 네트워크/timeout 미확정 상태, GET-only 저장 확인은 BE의 새로운 endpoint와 구분.
- 본인 응답 provider/email/joinedAt, 공개 프로필의 활동 횟수2개, 공개 업로드 이미지 revision 경로, 개인정보 경계 반영.
- 신규 gpsSegments, 선택적 gpsEvidence, 크기·좌표·시각·거리·정상 검증, 지문/UUID/동일 세션 충돌, 서버 completed·gpsValidationVersion·scoring 반영.
- 활동 저장/일지 변경과 점수 원장·총점의 동일 트랜잭션, v2 배점·일일 제한 없음, v1 활동/일별 인증/공개 보너스 금액·버전 보존 반영.
- 랭킹 TOP100·본인 전체 순위·null/0·동점 id 순서, 서버 월 목표·FE 활동 통계/고유 산·거리 및 로컬 찜/GPS 저장소 구분.
- 날씨는 실제 FE 화면 연결이며 기상청 외부 API와 ToPeak W01·메모리 cache/시간대·결측 응답을 구분.
- 프레임워크 기본 오류와 전용 error Map, FE 자체 network_error/request_timeout/invalid_response를 구분.

## B. 문서 전수 조사·조치

Git 추적/미추적/ignored를 구분해 .md/.mdx/.pdf/.docx/.txt 및 문서성 HTML을 조사했다. node_modules/target/.git/dist 및 앱 public 번들 같은 산출물은 제외했다. 별도 AGENTS.md/독립 LICENSE, PDF/docx/mdx 문서는 조사 범위에서 발견하지 못했다. 표의 상태는 작업 시작 기준이며 신규 DOCS 문서는 이번 작업의 미추적 파일이다.

| 저장소 | 기존 경로 | 조치 | 최종 경로 | 사유 |
| --- | --- | --- | --- | --- |
| DOCS | ToPeak API 명세서.md(추적) | 이전·수정 | API_SPEC.md + 기존 경로 안내 | 원문 구조 보존, 요청한 표준 원본명, 기존 링크 유지 |
| DOCS | ToPeak 서비스 기획안.md(추적) | 수정 | 기존 경로 | 장·연구·표·로드맵 보존, 현재 점수/검증/목표/날씨와 모순되는 부분만 보완 |
| DOCS | account-withdrawal.md(추적) | 유지 | 기존 경로 | 유효한 공통 탈퇴·데이터 삭제 정책/과거 검증 기록 |
| DOCS | settings-account-details.md(추적) | 유지·통합 | 기존 경로 | FE Git 원문과 본문이 동일해 중복 FE 사본을 만들지 않음 |
| DOCS | 없음 | 신규 | SCORE_POLICY.md | 현재 v2/과거 v1 보존/통계의 공통 정책 원본 |
| DOCS | 없음 | 신규 | README.md | 단일 원본과 이력·검증 결과 탐색 |
| DOCS | 없음 | 신규 | FINAL_DOCUMENTATION_AUDIT.md | 이번 조치·제한·검증 근거 |
| FE | README.md(추적) | 수정 | 기존 경로 | API 원본 링크/공유 이력/현재 웹 복구·서버 월 목표 안내 |
| FE | docs/profile-year-selection.md(추적 안내) | 유지·복구 | FE 안내 + DOCS/fe-profile-year-selection.md | 이전 전 3b54a38^ Git 원문 전체를 DOCS로 복구, 현재 문서와 구분 |
| FE | docs/settings-withdrawal.md(추적 안내) | 유지·복구 | FE 안내 + DOCS/fe-settings-withdrawal.md | 같은 Git 이력에서 유효한 FE UI/정리·검증 기록 복구 |
| FE | docs/settings-account-details.md(추적 안내) | 수정·통합 | FE 안내 → DOCS/settings-account-details.md | 존재하지 않는 fe- 파일 링크를 동일한 기존 공통 원본으로 정정 |
| FE | docs/ios-background-gps.md(추적) | 수정·유지 | 기존 경로 | 네이티브 코드/Mac 검증 직접 연결, 웹 숨김/자동 복구 설명만 현행화 |
| FE | src/data/README.md(추적) | 수정·유지 | 기존 경로 | 데이터 출처 보존, 실제 SVG400×350·fitSize360×320와 일치 |
| FE | src/data/MAP_SOURCE.md(추적) | 수정·유지 | 기존 경로 | 다른 지도 자산의 출처·생성 절차 보존, 현재 사용 데이터셋과 구분 |
| FE | ios/App/CapApp-SPM/README.md(추적) | 유지 | 기존 경로 | 생성 SPM 개발 지침, 공통 정책 문서가 아님 |
| BE | README.md(추적) | 수정 | 기존 경로 | 실행/환경 유지, 현행 API·점수/통계·V8과 dev 문서 링크 보완 |
| BE | docs/hiking-activity-integrity.md(추적) | 이전·수정 | DOCS/hiking-activity-integrity.md + BE 안내/테스트 | FE·BE 공통 지문·계약·롤아웃 원문 보존, BE 실행 절차는 유지 |
| BE | docs/account-withdrawal.md(추적 안내) | 유지 | 기존 경로 → DOCS 원본 | 이미 정상 이전된 링크 |
| BE | docs/settings-account-details.md(추적 안내) | 유지 | 기존 경로 → DOCS 원본 | 이미 정상 이전된 링크 |
| BE | HELP.md(ignored) | 유지 | 기존 경로 | Spring Initializr 개발 도움말 |
| BE | gps-script-check.txt(미추적) | 보류 | 기존 경로 | 문서가 아닌 기존 점검 스크립트 작업물, 수정/이전하지 않음 |
| FE | index.html(추적) | 유지 | 기존 경로 | 웹 앱 진입점이며 문서가 아님 |
| FE | ios/App/App/public/index.html(ignored 산출물) | 보류·제외 | 기존 경로 | 앱 번들이며 문서 정리 대상으로 편집하지 않음 |

과거 GPS 문서의 binary fingerprint·배포 전 중복 검사·legacy 호환·제약 설명 전체를 새 공통 문서의 '과거 구현·배포 이력 원문' 절에 보존했다. 하루1건/옛 검증 결과는 현재 정책으로 읽지 않도록 경계를 명시했다. 복구한 FE 이력의 당시 테스트/브랜치/대기 문구도 임의로 최신 결과로 바꾸지 않았다.

## C. Git·보존 원칙

- DOCS는 dev에서 직접 작업하며 별도 브랜치 생성 없음.
- FE/BE는 읽기 조사 후 실제 문서 수정 **전에** 각각 docs/final-documentation-alignment 브랜치를 생성했다.
- 시작 FE/DOCS는 clean, BE는 기존 mvnw.cmd 수정 및 gps-script-check.txt 미추적이었다. 이 두 파일은 작업 전 해시를 별도로 보관해 종료 시 비교한다.
- 기능/테스트 소스, 기존 별도 worktree는 변경하지 않는다. git add/commit/push/PR/merge는 수행하지 않는다.
- 최종 FE 변경5개: README.md, docs/ios-background-gps.md, docs/settings-account-details.md, src/data/README.md, src/data/MAP_SOURCE.md.
- 최종 BE 문서 변경2개: README.md, docs/hiking-activity-integrity.md. 기존 mvnw.cmd 수정과 gps-script-check.txt 미추적은 그대로이며 두 파일의 SHA-256도 작업 전과 동일하다.
- DOCS dev: 기존 ToPeak API 명세서.md·ToPeak 서비스 기획안.md 수정, API_SPEC.md·SCORE_POLICY.md·README.md·FINAL_DOCUMENTATION_AUDIT.md·fe-profile-year-selection.md·fe-settings-withdrawal.md·hiking-activity-integrity.md 신규7개.
- FE HEAD 66ceb1f, BE HEAD bd10265, DOCS HEAD df7063e는 시작과 같다. staged 변경 없음, add/commit/push/PR/merge 미수행. BE-wrapper-fix도 8874361·clean을 유지한다.

## D. 검증·남은 사항

### 검증 방식

- BE Controller method/URI 자동 추출과 Security5 경로를 명세 전체 목록/상세 API ID와 비교.
- FE literal API 경로·helper·화면 사용, Principal/CSRF/오류/DTO 타입과 서비스·테스트를 정적으로 교차 검토.
- JSON fence 문법, 실제 DTO response 필드, Spring Jackson 직렬화 및 요청 DTO 역직렬화/Bean Validation을 별도 DB 없는 진단 프로그램에서 확인.
- 로컬 Markdown 파일·앵커와 GitHub csie2026 FE/BE/DOCS의 dev 링크를 해당 저장소 파일·앵커에 매핑해 검사. 저장소 간 ../ 상대 경로를 새로 쓰지 않음.

### 최종 검사 결과

| 검사 | 결과 |
| --- | --- |
| Controller28 + Security5 ↔ 명세 전체33 및 상세 ID | 누락0·과잉0·ID 불일치0 |
| 기존27개 대비 | 새 경로6개·기존 상세15개 수정 |
| FE literal API 경로20개 | BE/명세 대응됨, 동적 image/provider 경로 및 실제 화면 사용은 별도 수동 대조 |
| JSON 예시28개 | 문법 오류0, 원문과 같은2칸 들여쓰기 유지 |
| Spring Jackson 직렬화 | 실제 현재 응답 모델 등12개 직렬화 성공, 응답 예시16개 필드 집합 일치 |
| 문서 DTO 객체25개 | Spring Jackson 역직렬화 성공, 요청 객체 Bean Validation 실패0 |
| 로컬 파일/앵커 및 저장소 매핑 링크64개 | 대상 없음0·앵커 불일치0 |
| git diff --check | FE·BE·DOCS 모두 통과 |
| 원본 보존 | API 1~15장 제목 동일, 기획안 전체 heading 동일, BE GPS 원문 전체 및 복구 FE 이력2개 전체 보존 확인 |

별도 진단은 JacksonAutoConfiguration만 올린 DB 없는 ApplicationContextRunner와 표준 javac를 사용했다. 요청 record의 검증용 getter를 요청 body 필드로 잘못 추가하지 않고 실제 문서 JSON을 역직렬화하여 확인했다. 제품 코드의 재빌드·서버 실행·DB 쓰기는 이번 문서 검증에서 수행하지 않았다. 진단에서는 동일 코드 HEAD의 선행 컴파일 산출물을 사용하고 기존 소스/테스트는 수정하지 않았다.

기계 판독 목록·링크 결과·실제 DTO JSON·진단 로그는 로컬 workspace의 `ToPeak/.audit-docs-alignment-20261113/`에 보관했다. 이는 DOCS 원격 링크로 접근 가능한 게시 산출물이 아니다. 이번 문서 확인에 불필요한 전체 앱 테스트를 반복 실행하지 않았다.

### 원격 링크와 게시 상태

GitHub 저장소 및 누락3개 URL의 실제 HTTP 열기를 시도했으나 이 환경에서 Cache miss로 내용을 가져오지 못했다. 이를404나 접근 성공으로 단정하지 않는다. 소유권·인증 상태도 추정하지 않는다. GitHub blob/dev 구조와 로컬 대상/앵커는 검증하되 **새로 생성/변경한 문서는 미커밋·미게시 상태**다. 사용자가 이후 반영하기 전 원격에서 새 문서가 이미 열린다고 보장하지 않는다. 기존 외부 출처 링크는 보존했으며 모든 외부 사이트의 HTTP 상태를 재검증한 결과는 아니다.

### Markdown/PDF

작업 시작 DOCS에는 API_SPEC.md/SERVICE_PLAN.md 및 대응 PDF가 없었고 한글 명칭의 Markdown2개가 있었다. API 원문은 API_SPEC.md로 이전하고 서비스 기획안의 기존 파일명과 문서 구성을 유지했다. PDF는 세 저장소의 조사 범위에서 발견하지 못했다. 기존 디자인을 가진 PDF를 임의 재생성하지 않았다. 별도 제출 PDF가 저장소 밖에 있다면 API·기획안의 이번 Markdown 변경과 동기화가 필요하며 해당 PDF의 디자인/내용 일치 여부는 확인 불가다.

### 코드 수정 없이 남긴 경계

현재 FE 호출과 BE Method/URI 사이의 누락/과잉 API는 발견하지 못했다. 이미지·목표·산행의 FE 계정 사전 GET을 프로필 PATCH expectedUserId의 BE 원자 비교와 같다고 표현하지 않는다. FE weather optional/nullable 타입이 BE보다 넓은 것은 수용 범위 차이로 문서에 표시했다. 서버의 GPS 좌표 진위 보증·활동 삭제·일지 사진·서버 찜 API는 구현되어 있다고 적지 않았다.

운영 DB/외부 OAuth/실시간 날씨/현장 GPS를 이번 문서 검사에서 실행한 것은 아니다. 선행 동일 HEAD의 PostgreSQL/javac·FE 결과와 이번 문서 진단을 구분한다.

## 2차 통합 및 전체 Markdown 내용 검증 (2026-11-13)

### A. DOCS 동기화와 보존

- FE: `docs/final-documentation-alignment`, HEAD `66ceb1fe288d9047cd821b986b4c33e4f030e75b`.
- BE: `docs/final-documentation-alignment`, HEAD `bd102653758f104cb9369d288ce1f1ebda35c337`.
- DOCS: `dev`, HEAD `df7063eed1da7c4b6142a8428fd1039c08ff7210`. 브랜치와 HEAD를 바꾸지 않았다.
- 수정 전에 현재 Markdown24개를 `ToPeak/.audit-docs-round2-20261113/backup/{FE,BE,DOCS}/`에 원본 bytes로 보존했다. 현재 API_SPEC, 이전 미커밋 문서와 시작 diff/해시도 보존했다. 이 디렉터리는 로컬 검증 산출물이며 원격 게시 문서가 아니다.
- `git fetch --no-tags origin dev`는 GitHub TCP443 연결 실패로 완료되지 않았다. raw GitHub/branch HTTP 조회도 실패했다. **최신 원격 dev 확인은 미완료**이며 최신 상태라고 선언하지 않는다.
- 확인 가능한 캐시 `origin/dev`는 `01c5013b2994803fd0f567dc6ecbc787903ae8ac`다. 로컬 dev와의 left/right는 `0 / 1`, 즉 확인 가능한 이력에서 로컬은 문서 커밋1개 뒤에 있다. 해당 커밋은 FE 이력3개만 추가했고 기존 API 파일 내용은 로컬 HEAD와 동일하다. 이 비교는 fetch 성공이나 현재 서버 HEAD 확인을 의미하지 않는다.
- 캐시된 원격의 Markdown7개도 `cached-origin-snapshot/`에 별도 보존했다. 여기에는 API_SPEC.md가 없고 `ToPeak API 명세서.md`가 원본이다. 이를 현재 미커밋 API_SPEC와 실제 구현을 기준으로 비교했다.
- 캐시의 `fe-profile-year-selection.md`, `fe-settings-withdrawal.md`는 머리말을 제외한 전체 본문이 현재 로컬과 동일함을 확인했다. `fe-settings-account-details.md` 본문은 공통 `settings-account-details.md`와 전체 동일하다. 누락된 해당 경로에는 공통 원본 안내만 복원해 중복 원본을 만들지 않았다. 캐시의 고유한 유효 본문을 누락하지 않았다.
- 기존 API의1~15장·27개 ID·표 형식·유효한 상세는 유지하고 이미 추가한6개도 보존했다. OAuth 실패401, 옛 프로필/응답 필드, FE 날씨 미연결, 현재 일일 제한 등 코드와 다른 설명은 현행 계약에서 제외/정정했다. 실제 호환 중인 v1 원장·금액·버전 설명은 유지했다. 기획 목표나 역사 문서의 당시 기록을 현재 구현으로 덮어쓰지 않았다.
- pull/reset/checkout 폐기/clean/stash/merge를 사용하지 않았다. 원격 접근 복구 후 fetch와 문서 비교를 다시 해야 하며 이번 통합을 Git 이력 동기화 완료로 보고하지 않는다.

### B. API 재추출·요청·응답 검증

| 검증 항목 | 이번 결과 | 근거와 범위 |
| --- | --- | --- |
| 실제 Controller | 28개 Method/URI | 전체 Controller 소스 추출과 실행한 RequestMappingHandlerMapping 결과 일치 |
| Security 경로 | 5개 | 공급자별 로그인2/callback2 및 POST 로그아웃1, SecurityConfig·설정·Handler 확인 |
| 실제 API / 문서 API | 33 / 33 | 이름·query를 정규화해 집합 대조, index/상세 ID 대조 |
| 문서 누락 / 과잉 / 중복 | 0 / 0 / 0 | 숫자를33으로 고정하지 않고 추출 결과에서 계산 |
| 캐시 기존 API 대비 추가 | 6개 | U07/U08/H04/R02/G01/G02, 구현 근거 유지 |
| 캐시 기존 API 상세 변경 | 15개 | A04/A05/U01/U02/U03/U05/M01/W01/H01/H02/H03/J01/J06/J07/R01, 공통 모델 변경 중복 합산 제외 |
| 런타임 record 구조 | 24종 | Request/Response 및 중첩 모델의 필드·타입·enum을 추출해 코드/문서 대조 |
| JSON 예시 | 28개 파싱 성공 | 현재 최종 API_SPEC의 모든 json fence |
| DTO 예시 객체 | 25개 | 실제 Spring Jackson 역직렬화/재직렬화, Request Bean Validation; 배열 원소 포함 |
| Response 예시 | 19개 일치 | 중첩 객체 포함 입력 예시와 실제 직렬화 결과 대조, null 필드 보존 |
| FE literal 경로 | 20가지 | 중복 제거/템플릿 정규화 후 BE·문서 경로 대조, API helper 메서드·동적 이미지/OAuth는 소스 교차 검토 |
| 실제 HTTP 오류·정상 경계 | 11회 확인 | 프로필5종 ID오류400, 불일치409, 미인증401, CSRF403, 활동404, 목표{}200, 공개이미지 v누락400 |

`DocumentationContractTests`3개는 저장소 밖 복사본에만 추가하여 실행했다. 실제 Spring Jackson/Validator/보안 필터/본문 제한 필터를 사용했다. 프로필 ID 소수는 invalid_request, 누락/null/0/음수는 validation_failed였으며 불일치는 profile_account_mismatch다. 공개 이미지 필수 v 누락의 MockMvc400은 body가 비어 있었다. 따라서 모든 framework 오류에 error/message Map을 보장하지 않는다. 프로필 충돌의 정확한 문구와 no-store, 탈퇴의 request_failed 구분도 유지했다.

새 직렬화 확인에서 요청 record의 검증 getter가 파생 JSON 필드로 보일 수 있었다. `birthYearValid`, `gpsWithinSession`, `sessionValid`는 FE 입력 필수 필드나 활동 Response가 아니며 서버 검증을 대신하지 못한다는 설명을 API에 추가했다. 선택 필드 생략과 응답 null 필드, Instant/LocalDate/LocalDateTime·거리m/시간ms·검증 Short1을 구분했다. getter를 새 API 필드로 요구하지 않았다.

프로필·활동/GPS·일지·이미지·공개 프로필·산/코스·목표·점수·랭킹·날씨를 각 Controller→DTO→서비스/예외→FE 호출/응답검사와 대조했다. 회원 총점은 저장값, 랭킹은 null마지막/점수내림차순/id오름차순, 월 목표는 서버 저장, 찜/GPS 초안은 브라우저 저장이다. stats의500m AND20분 OR 검증완등과 completed/검증버전1은 점수 원장과 독립이다. 최근 HikingScoreService의 비현행 버전 보너스 보존 분기와 기존 v1의0/50점·버전 보존 테스트를 이번에 다시 실행했다.

FE·BE Method/URI·필수 body 사이에 새로 확인된 호출 계약 오류는 없다. FE weather의 optional/nullable 허용 범위가 BE보다 넓고, 산행/이미지/목표의 FE 사전 계정 GET은 원자적 expectedUserId 계약과 다르다. 이를 코드에서 해결한 것처럼 서술하지 않았다. Java Long 전체 범위와 JS safe integer, 제출 GPS의 실제 이동 진위, 모든 외부 payload 오류의 통일되지 않은 처리도 기존 한계로 남긴다.

### C. 이번 실행 결과와 안전 범위

| 실행 | 통과 | 실패/오류 | 스킵 | 비고 |
| --- | --- | --- | --- | --- |
| BE 선정 기존 테스트 | 95 | 0 | 0 | 프로필 계정 보호7, 점수29, 랭킹6, 공개통계10, 목표10, 프로필/일지17, 공개이미지8, OAuth Handler2, 산계약2, GPS4 |
| 외부 문서 계약 검증 | 3 | 0 | 0 | 런타임 매핑/24record, 현재 예시, 실제HTTP 경계 |
| 위 BE 합계 | 98 | 0 | 0 | 표준 javac/Maven test BUILD SUCCESS |
| FE npm test | 352 | 0 | 0 | 현재 소스로 새 실행 |
| FE npm run build | 성공 | 0 | 해당없음 | tsc -b 및 Vite8.3.1 빌드 |

BE는 현재 POM/src의 복사본과 독립 H2 메모리 DB만 사용했다. audit 빌드에 `-Dtest=DocumentationContractTests,ProfileAccountSafetyIntegrationTests,HikingScoringIntegrationTests,RankingIntegrationTests,PublicProfileStatisticsIntegrationTests,MonthlyGoalIntegrationTests,ProfileJournalIntegrationTests,PublicProfileImageIntegrationTests,OAuth2LoginHandlerTests,MountainDataContractTests,GpsEvidenceTests`를 지정했다. 캐시는 기존 감사용 복제 Maven cache를 사용하고 Java 경로 제한은 실행 중에만 Z: 매핑으로 우회 후 해제했다. 프로젝트 POM/JDK/Wrapper/파일 권한은 바꾸지 않았다.

이번에는 PostgreSQL 테스트나 전체 verify를 재실행하지 않았다. 위98개는 이전 PostgreSQL216개 verify와 별개다. 기존 DB·AWS·외부 OAuth 로그인·실시간 날씨 API는 사용하지 않았다. H2 seed를 새로 조회한 값은 산140/코스420/코스가 있는 산134로 기획안과 일치한다. Mac 명령은 파일·설정·script 존재를 정적으로 확인했으며 Swift/Xcode/실기기 성공을 의미하지 않는다.

### D. 전체 Markdown별 내용 검증·조치

빌드/의존성/캐시·임시 감사 디렉터리를 제외하고 세 저장소의 Markdown을 다시 열거했다. 신규 호환 안내1개를 포함해25개이며, 그중 일반 내용 검토 대상22개와 생성/감사 문서3개를 구분한다. AGENTS.md, mdx, 별도 SERVICE_PLAN.md 및 PDF는 조사 범위에서 발견하지 못했다. 공통 기획안은 기존 한글 파일명 그대로다. 아래의 유지는 이번 회차 조치이며 1차 미커밋 수정이 없다는 뜻이 아니다.

| 저장소 | 문서 경로 | 검증 결과 | 조치 | 근거 |
| --- | --- | --- | --- | --- |
| FE | README.md | 구현/stack/script 대조, 구조 오류 수정 | 수정 | package.json·vite.config.ts·실제 components; 없는 mountain/ 제거, Capacitor/settings 추가 |
| FE | docs/ios-background-gps.md | TS/Swift·프로젝트/SQL 정적·자동 테스트 확인, Mac/실기기 일부 미검증 | 수정 | recoveryPolicy/webSessionLifecycle·native 소스·352테스트; 자동pause 문구/브랜치/과거번들 비교 정정 |
| FE | docs/profile-year-selection.md | 공통 이력 안내 정확 | 유지 | DOCS 대상 본문과 ProfileForm/연도 테스트 대조 |
| FE | docs/settings-account-details.md | 공통 원본 안내 정확 | 유지 | settings-account-details와 계정 정보 UI·Member DTO |
| FE | docs/settings-withdrawal.md | 공통 이력 안내 정확 | 유지 | withdrawal API/확인창·FE 테스트 |
| FE | src/data/README.md | 현재 지도 입력·투영/크기 일치, 외부 라이선스 원문 미확인 | 유지 | GyeonggiMap fitSize360×320/viewBox400×350 및 GeoJSON |
| FE | src/data/MAP_SOURCE.md | 보존 중인 별도 생성자산 설명 일치, 외부 출처 최신성 미확인 | 유지 | generator·gyeonggiMap/경계 데이터; 현재 컴포넌트 원본과 구분 |
| FE | ios/App/CapApp-SPM/README.md | CLI 생성 문서 | 유지(일반 검토 제외) | SPM 관리 산출물, 수동 수정하지 않음 |
| BE | README.md | POM/설정·migration·실행/테스트 대조 | 수정 | V8 누락 운영 순서 정정, local none/prod validate, PG opt-in·verify 구분 |
| BE | docs/account-withdrawal.md | 공통 이력 안내 정확 | 유지 | WithdrawalService/세션 필터·공통 본문 |
| BE | docs/settings-account-details.md | 공통 이력 안내 정확 | 유지 | Member/MemberResponse·V8·공통 본문 |
| BE | docs/hiking-activity-integrity.md | 현행 계약·테스트 속성/URL 제한 일치 | 유지 | HikingActivityPostgresTests의 활성 조건/검사/쓰기·정리, GPS/점수 코드 |
| BE | HELP.md | ignored Spring Initializr 생성 안내 | 유지(일반 검토 제외) | 제품 계약 문서가 아니며 기존 생성 내용 유지 |
| DOCS | API_SPEC.md | 전체33 API·24record·예시·보안/정책 재대조 | 수정 | 이번 런타임 매핑/직렬화/HTTP98개·FE352개,1~15장 유지/목차 추가 |
| DOCS | ToPeak API 명세서.md | 단일 원본 안내 정확 | 유지 | API_SPEC 링크, 중복 계약 본문 없음 |
| DOCS | ToPeak 서비스 기획안.md | 구현 관련 설명 대조·오류 정정, 외부 조사/경쟁사 사실 일부 미검증 | 수정 | 11.4의 서버검증 부정 정정;10.2/GpsEvidence와 일치; PDF 존재 단정 제거 |
| DOCS | SCORE_POLICY.md | 현재 v2/v1 보존·통계 일치 | 유지 | HikingScoreService/Policy·통계SQL·FE activityCertification/hikingStatistics 및 점수29개 재실행 |
| DOCS | hiking-activity-integrity.md | 현행 요약 일치, 옛본문은 역사로 분리 | 유지 | 현재 GPS·지문·V7/V8·원장; 옛 하루1건/Java17/대기 결과는 당시 기록 유지 |
| DOCS | account-withdrawal.md | 기능 일치·과거 검증 기록 구분, 정책 출처 최신성 미검증 | 유지 | 삭제 순서/트랜잭션·세션코드와 FE 동의 확인; 법률 적합성 검증 완료로 표시하지 않음 |
| DOCS | settings-account-details.md | 현재 기능 일치·당시 테스트대기 기록 보존 | 유지 | V8·MemberResponse·계정 UI; 캐시 FE 이력과 전체 본문 동일 |
| DOCS | fe-profile-year-selection.md | 캐시 전체 본문 보존·현재 폼 일치 | 유지 | 캐시/로컬 본문 비교,1950~현재 및 기존1900~1949 값 예외 |
| DOCS | fe-settings-withdrawal.md | 캐시 전체 본문 보존·현재 흐름 일치 | 유지 | cached diff·withdrawal/cleanup/계정 전환 테스트;161개는 당시 기록 |
| DOCS | fe-settings-account-details.md | 캐시의 기존 경로 보존·중복 본문 제거 | 통합(안내 추가) | 공통 settings-account-details와 전체 본문 동일, 유효 정보 손실 없음 |
| DOCS | README.md | 원본/역사/저장소 경계 정확 | 수정 | 계정 정보 이력의 기존 FE 경로도 공통 원본으로 안내 |
| DOCS | FINAL_DOCUMENTATION_AUDIT.md | 이전 감사 기록을 참고하되 일반 제품 내용 검토 제외 | 수정(검증 기록) | 1차 결과 보존, 이번 결과/제약/전체 목록 추가 |

삭제·이동은 하지 않았다. 폐기 후보로 새로 판정한 문서는 없다. 중복되는 계정 정보 이력은 기존 공통 본문 하나를 유지한다. 학술제 기획안의 연구 근거·제품 목표·우선순위·로드맵과15장 구성은 보존하고, 외부 통계/경쟁 서비스의 최신 내용까지 재검증했다고 보고하지 않는다. 별도 원문/원격 접근을 확보한 뒤 제출 담당자가 확인할 항목이다.

### E. 발견한 불일치와 이번 변경 파일

1. FE README가 실제 없는 `components/mountain/`을 표시했다. 실제 GyeonggiMap/CityMountainList/settings로 정정하고 이미 사용 중인 Capacitor8을 stack에 표시했다.
2. BE README 운영 순서가 V7에서 끝났다. 실제 최신 V8 및 common/prod validate와 local none 차이를 반영하고 기본 H2/명시적 PG 테스트·verify 방법을 구분했다.
3. iOS 문서 시험표의 웹 자동pause는 현행 생명주기 코드와 달랐다. 숨김 체크포인트/진행·수동pause 보존으로 수정했다. 과거 브랜치·CLI 오류·번들9개 동일 비교·189/194개 결과를 현재 실적으로 오해하지 않게 명시했다. cap sync와 Mac/실기기 검증은 보류다.
4. 기획안11.4의 서버 경로/정상검증 부정은10.2/API/실제 코드와 충돌했다. 제출경로 검증은 구현, 원본 DB미보관·현장진위 미보증으로 정정했다. 저장소에 없는 PDF의 이미지 포함 여부는 확정하지 않았다.
5. 캐시된 원격 문서 추가분이 로컬에 전부 반영된 상태는 아니었다. 고유한 본문 동일성을 비교하고 계정정보 FE 옛 경로 안내를 복원했다. 원격 최신은 여전히 미확인이다.
6. API의 검증 getter/실제 요청 필드를 구분하고 내부 목차 및 이번 검증 근거를 추가했다. 새로 누락된 endpoint나 배점을 임의 추가하지 않았다.

이번 회차 변경은 **FE2개**(README, ios-background-gps), **BE1개**(README), **DOCS5개**(API_SPEC, README, 서비스 기획안, 신규 계정정보 FE 안내, 본 보고서)다. 기존 미커밋 FE의 다른3개 문서·BE 산행 문서·DOCS 나머지 문서는 유지했다. 전체 누적 Git 변경 목록은 1차 수정도 포함한다.

BE/mvnw.cmd와 gps-script-check.txt는 시작 SHA-256과 같으며 기존 별도 worktree도 수정하지 않았다. 제품 코드·기존 테스트·migration/POM·환경 파일을 수정하지 않았다. add/commit/push/PR/merge는 수행하지 않았다. 백업/캐시 원문·runtime-endpoints.json·runtime-record-schemas.json·dto-roundtrips.json·actual-http-responses.json·endpoint-matrix.json·link-results.json·current-contract-tests.log와 audit용 복사본은 `.audit-docs-round2-20261113/`에 남긴다. DB 서버/컨테이너를 생성하거나 가동하지 않았다.

### F. 최종 정합성 평가와 남은 확인

| 항목 | 평가 | 남은 확인 |
| --- | --- | --- |
| API 명세서 정확성 | 현재 로컬 구현 대조 검증 완료 | 원격 최신 재비교, 외부 provider/live·실기기 E2E와 모든 오류 조합은 미검증 |
| 기능 설명 정확성 | 현재 구현 설명 정정 완료, 일부 미검증 | 네이티브 실제 동작·기획안의 외부 조사/경쟁사 사실 |
| 기술 스택/실행 안내 | 선언/설정/명령 정적 대조·FE build/BE javac test 검증 완료 | 신규 환경 설치·Mac/Xcode/cap sync·실제 배포 기동은 미실행 |
| 점수·통계 일관성 | 검증 완료 | v1/v2·통계 독립 기준 유지, 운영 원장 검사/변경 없음 |
| FE·BE·DOCS 일관성 | 로컬 및 확인 가능한 캐시 기준 검증 완료 | GitHub 최신 내용/실제 HTTP 링크 접근·미커밋 문서 게시 미완료 |
| 학술제 제출 준비 | Markdown 원본 정리 완료, 일부 미검증 | PDF 미생성; 원격 재비교·대외 출처와 최종 조판/이미지 확인 필요 |

최종 API의1~15장과JSON/표 구조를 유지하고 내부 목차를 연결했다. 로컬 링크/앵커 및 GitHub blob/dev를 로컬 파일에 대응한85개 검사가 모두 통과했다. Markdown25개는 Prettier Markdown 파서로 쓰기 없이 파싱했고 fence 짝과 git diff --check도 통과했다. 시작 백업24개와 보호 파일2개의 해시, 기존 BE src/POM의 복사본과 현재 파일 동일성을 확인했다. 임시 Z: 매핑도 남아 있지 않다. 이는 원격 HTTP 접근 성공과 다르다. 외부22개 URL의 HTTP 상태/출처 원문은 확인하지 못했다. 별도 PDF가 있다면 이번 API/기획안 변경을 반영해야 하며, 이번 작업에서는 PDF를 생성하거나 기존 디자인을 재구성하지 않았다.
