# TEST_RESULTS.md

- 대상: https://web-kappa-indol-69.vercel.app/financial-dna (프로덕션)
- 검증: 로컬 빌드 도구 + Playwright(Chromium) 자동화로 공개 프로덕션 URL 실제 구동
- 프로덕션 커밋: `be77755311d8b9d0f46f3928b1b090fd54e1e04f`

## 로컬 정적 검증
| 항목 | 결과 |
|---|---|
| type-check (`tsc --noEmit`) | ✅ 통과 |
| lint (`eslint`) | ✅ 통과 |
| production build (`next build`) | ✅ 통과 (`financial-dna` 동적 렌더 ƒ — 서버 복호화/메타데이터 위해 의도) |
| 암호화 단위테스트 | ✅ 왕복·결과동일·변조/잘림/과길이(4000+)/빈값 모두 null 방어·원본답변 미포함 |
| 클라이언트 번들 유출 검사 | ✅ SHARE_SECRET 값·복호화 로직 클라이언트/서버 산출물에 미포함 (빌드 grep) |

## 프로덕션 E2E (Playwright, 13개 시나리오)
| # | 항목 | 결과 |
|---|---|---|
| 1 | 신규 공유 링크 생성(`?s=`) | ✅ |
| 2 | 불투명 토큰(약 300~318자 base64url) | ✅ |
| 3 | URL에서 원본 96개 답변 식별 불가(96자리 연속 없음) | ✅ |
| 4 | 결과 링크 복사 → 클립보드 = 공유 URL | ✅ |
| 5 | 새 브라우저 세션에서 동일 결과 복원(전체 결과 텍스트 100% 일치) | ✅ |
| 6 | 새로고침 후 결과 유지 | ✅ |
| 7 | 토큰 변조 → 결과 미표시, 인트로 폴백 | ✅ |
| 8 | 잘린 토큰 → 인트로 폴백 | ✅ |
| 9 | 잘못된 문자 토큰 → 인트로 폴백 | ✅ |
| 10 | 레거시 `?r=<96자리>` 하위 호환 복원 | ✅ |
| 11 | 다시 검사하기 → URL·결과 초기화 | ✅ |
| 12 | 클립보드 권한 거부 → 수동 복사 안내 폴백 | ✅ |
| 13 | 모바일 뷰포트 복원 동일 + 복사 버튼 노출 | ✅ |
| — | 콘솔 오류 | ✅ 0건 |
| — | HTTP 4xx·5xx 오류 | ✅ 0건 |
| — | 회귀(채점·문항·디자인 미변경, 랜덤화·미응답검증·뒤로가기 정상) | ✅ |

## 버전 저장 확인
- 토큰 payload에 `qv`(questionnaireVersion="1")·`sv`(scoringVersion="1")·포맷버전·생성시각 포함(로컬 왕복 테스트로 복호 후 값 확인).
- 렌더는 저장 스냅샷 사용 → 채점 변경 시에도 과거 결과 불변 설계.

## Web Share / 클립보드
- Web Share API 지원 기기: "공유하기" 버튼 노출(useSyncExternalStore로 navigator.share 감지, hydration 불일치 없음). Playwright 데스크톱/모바일 에뮬레이션엔 navigator.share가 없어 자동화에서는 **복사 폴백 경로**가 노출·검증됨(요구사항의 미지원 환경 동작과 일치).
- 클립보드: `navigator.clipboard.writeText` → 실패 시 `execCommand('copy')` → 그래도 실패 시 수동 복사 입력창(스크린샷 06).

## 스크린샷 (SCREENSHOTS/)
- `00_intro.png` 인트로
- `01_result_desktop.png` 결과(데스크톱, 공유/복사 버튼)
- `02_share_buttons.png` 버튼 영역 클로즈업
- `03_shared_restore.png` 공유 링크로 새 세션 복원
- `04_mobile_result.png` 모바일 결과
- `05_tampered_fallback.png` 변조 토큰 → 인트로 폴백
- `06_manual_copy_fallback.png` 클립보드 거부 → 수동 복사 안내(불투명 `?s=` URL 표시)

## 비밀정보 검사
- 검수 자료 생성 후 자동 시크릿 스캔 수행(AUDIT_MANIFEST.json `secretScanPassed`).
- 96문항 원문·채점 Matrix·독점 가중치·키·토큰·개인정보 미포함.
