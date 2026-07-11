# TEST_RESULTS.md

- 대상: https://web-kappa-indol-69.vercel.app/financial-dna (프로덕션)
- 검증 방식: 로컬 빌드 도구 + Playwright(Chromium) 자동화로 공개 프로덕션 URL을 실제 구동
- 프로덕션 커밋: `c30e1c2267863a6fda6ea04e9d139dcc82fdf0a6`

| # | 항목 | 결과 | 근거 |
|---|------|------|------|
| 1 | type-check (`tsc --noEmit`) | ✅ 통과 | exit 0, 오류 0 |
| 2 | lint (`eslint`) | ✅ 통과 | 경고·오류 0 |
| 3 | production build (`next build`) | ✅ 통과 | 8/8 페이지 생성, `financial-dna`는 동적 렌더링(ƒ) — 공유 링크별 메타데이터 생성을 위해 의도된 전환 |
| 4 | 브라우저 완주 테스트 | ✅ 통과 | 96문항 전체 응답 → 결과 산출(탐험가형/Explorer, IGFP, 종합점수 48) |
| 5 | 모바일 테스트 | ✅ 통과 | 390px 뷰포트에서 결과 전체 정상 렌더링, 12 Factor 점수 데스크톱과 동일 (SCREENSHOTS/03_mobile_result.png) |
| 6 | 콘솔 오류 | ✅ 0건 | Playwright `console`/`pageerror` 리스너 수집 결과 0건 |
| 7 | 네트워크 오류 | ✅ HTTP 4xx·5xx 0건 | Playwright `response` status ≥400 = 0건. 별도로 잡힌 `requestfailed` 5건은 전부 Next.js RSC 프리페치(`?_rsc=`)가 페이지 이동/컨텍스트 종료 시 취소(ERR_ABORTED)된 것으로 실제 오류 아님 |
| 8 | 회귀 테스트 | ✅ 통과 | 채점 로직·문항·유형·디자인 미변경. 문항 랜덤화·미응답 검증·이전(뒤로가기) 상태 보존 등 기존 기능 정상 |
| 9 | 잘못된 URL 예외 처리 | ✅ 통과 | `?r=abc`(형식 오류), 95자리(1자 잘림), 범위 밖 값(`9`·`0` 포함) 3종 모두 크래시 없이 인트로로 안전 폴백 |
| 10 | 공유 링크 복원 테스트 | ✅ 통과 | 결과 링크를 **새 클린 컨텍스트**에서 열었을 때 유형·종합점수·4축·12 Factor 점수·리포트가 원본과 100% 일치. 새로고침 후에도 유지 (SCREENSHOTS/05_shared_restore_desktop.png) |

## 원본 결과 = 공유 결과 동일성 (채점 무결성 간접 검증)
- 유형 코드: `IGFP` (원본 = 공유)
- 유형명: 탐험가형 (Explorer)
- 종합점수: 48
- 12 Factor: 계획성 28 · 금융지식 41 · 안전 28 · 기회 47 · 통제 47 · 성장 75 · 자유 72 · 가족 53 · 성취 56 · 현재 59 · 신뢰 19 · 위험 50
- 4축: 전략축 직관형 · 위험축 성장형 · 신뢰축 자유형 · 동기축 관계현재형
→ 동일 URL이 항상 동일 결과를 재현하므로, 인코딩/디코딩이 채점 입력을 무손실 보존함이 확인됨.

## 스크린샷 (SCREENSHOTS/)
- `00_intro.png` — 테스트 인트로 (진입 화면)
- `01_before_result_buttons.png` — **변경 전**(공유 버튼 제거 재현): "다시 검사하기"·"상담 신청하기"만 존재
- `02_after_result_buttons.png` — **변경 후**: "결과 링크 복사하기" 버튼 추가
- `03_mobile_result.png` — 모바일 결과 화면 (390px)
- `04_result_desktop.png` — 데스크톱 결과 화면 전체
- `05_shared_restore_desktop.png` — 공유 링크(`?r=`)로 클린 세션에서 복원한 결과 화면

## 비밀정보 탐지
- 검수 자료 생성 후 자동 시크릿 스캔 수행 (AUDIT_MANIFEST.json의 `secretScanPassed` 참고).
- 96문항 전문·전체 채점 Matrix·독점 가중치·토큰·키·개인정보는 본 저장소에 포함하지 않음.
