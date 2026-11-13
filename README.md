# ToPeak 공통 문서

API·정책·서비스 기획의 원본을 관리합니다. 개발 환경과 코드별 기술 안내는 FE/BE 저장소에 유지합니다.

- [API_SPEC.md — 현행 API 전체 명세](API_SPEC.md)
- [서비스 기획안](ToPeak%20서비스%20기획안.md) — 장/연구·로드맵을 보존하고 현재 구현 정합성만 보완
- [점수·랭킹·산행 통계 정책](SCORE_POLICY.md)
- [GPS 활동 무결성·호환/롤아웃 이력](hiking-activity-integrity.md)
- [회원탈퇴 정책·구현 이력](account-withdrawal.md)
- [계정 정보·가입일 구현 이력](settings-account-details.md) — FE의 [기존 경로](fe-settings-account-details.md)도 동일 원본으로 안내
- [FE 출생연도 선택 이력](fe-profile-year-selection.md)
- [FE 설정·회원탈퇴 UI 이력](fe-settings-withdrawal.md)
- [최종 문서 정합성·배치·검증 결과](FINAL_DOCUMENTATION_AUDIT.md)

각 문서의 과거 테스트 수와 실행 대기 표현은 작성 당시 이력입니다. 현재 HTTP 계약은 API_SPEC, 현재 점수·횟수 기준은 SCORE_POLICY를 우선합니다. 기존 [한글 API 문서 경로](ToPeak%20API%20명세서.md)는 최신 원본으로 안내하는 호환 링크로 유지합니다.

저장소 간 링크는 GitHub의 `https://github.com/csie2026/{FE|BE|DOCS}/blob/dev/...` 구조를 사용합니다. 로컬에서 옆 저장소로 이어지는 상대 경로를 원격 링크로 사용하지 않습니다. 미커밋 문서가 원격에 게시됐다고 가정하지 않으며 반영 상태·PDF 유무는 검증 결과에 구분합니다.
