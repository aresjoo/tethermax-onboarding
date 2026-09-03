# DESIGN_SYSTEM_AUDIT.md — Gemini-grade 리팩터 (2026-09-03)
Reference: Google Gemini 웹 UI 스크린샷 4장 (rail·tooltip·settings popover·theme submenu).
원칙: **Gemini의 craft를 가져오되 비주얼 아이덴티티는 TetherMax 실측 토큰 유지** (사용자 확정).
브랜드 에셋(별 로고 등) 미복제. 모바일 퍼널 특성상 tooltip/desktop rail은 원리만 이식(시트=팝오버 대응).

## 스택 파악
단일 HTML SPA(프레임워크 없음) · CSS variables가 토큰 · 인라인 SVG 아이콘 · 자체 Spring 모션 엔진(rAF)
· hash 라우터 · localStorage 상태 · 시트/토스트 자체 구현. 외부 라이브러리 0.

## CURRENT → TARGET

| # | 영역 | CURRENT (문제) | TARGET |
|---|---|---|---|
| 1 | 토큰 공백 | spacing/motion/z-index/icon size/hover surface/elevation 토큰 없음 — 값이 인라인 산재 | `--sp-*`(4grid), `--mo-*`(dur 3+ease 3), `--z-*`, `--ic-*`, `--surface-hover/pressed`, `--elev-*` 신설 후 전 컴포넌트가 var()만 참조 |
| 2 | 모션 산재 | CSS transition 9종(.15/.22/.25/.28/.3/.35/.38/.65/.7s), easing 제각각 | fast 120 / normal 200 / slow 280ms + standard·enter·exit 3 ease로 통일 (Spring 물리 레이어는 유지 — 토스 스펙) |
| 3 | 아이콘 stroke 불일치 | 1.8/2/2.4 혼재, viewBox 24·16 혼재, cap 일부 미지정 | UI 아이콘 전부 viewBox 24·stroke 1.8·round cap/join. 예외: 10px 체크마크(2.4/16)는 optical 보정으로 유지, 로고(브랜드 마크)는 불변 |
| 4 | 상태 이중/누락 | `.btn:active{opacity:.85}`+JS 프레스 스프링 이중 적용 · btn-s/paste/tabbar/support/exc/vchip hover 없음 · input hover 없음 | opacity 핵 제거(스프링 단일화). `@media(hover:hover)`로 전 컨트롤에 surface hover — “새 버튼이 나타나는” 게 아니라 “지면이 은은히 드러나는” 톤(white 4~6%) |
| 5 | 서피스 위계 | hover/selected surface 미정의 · 딤 값 인라인 | base(#121721) < surface(#1D2028) < surface-2(#232732) < sheet, hover=white .045 / pressed=.07 / selected=brand-soft로 층 정의 |
| 6 | 컨트롤 높이 임의 | 36/38/40/44/48/56 혼재 (38=tg 아이콘 등 그리드 밖) | `--ctl-sm:36 / md:44 / lg:56` + 원형 40/48 스냅. 38→40 |
| 7 | 반복 인라인 스타일 | 작은 파란 버튼(36px) 인라인 재정의 · 로딩 뷰 2곳 중복 마크업 스타일 · 백버튼 아이콘버튼 스타일 산재 | `.btn-sm` 수정자 · `.loading-view` 클래스 · `.icon-btn` 프리미티브로 통합 |
| 8 | 시트 a11y | Escape 미지원 · 열릴 때 포커스 이동 없음 · 복원 없음 | Escape→top sheet 닫기, open 시 첫 버튼 focus, close 시 트리거 복원, dim은 aria-hidden |
| 9 | 탭바 선택 상태 | 색상만 변경(선택 surface 없음) · hover 없음 | Gemini rail 원리: 선택 아이콘 뒤 soft pill surface + hover surface. 색은 브랜드 블루 유지 |
| 10 | 라디우스 비토큰 | 토스트 8px·시트 16px 인라인 | `--r-toast`, `--r-sheet` 토큰화 (값 유지 — 실측 계열) |
| 11 | 스페이싱 리듬 | 6/10/14/18 등 4-grid 이탈 혼재 | 페이지 리듬은 4-grid 스냅(6→8, 14→16, 18→16/20). 단 실측 컴포넌트 내부값(배지 4×6 등)은 브랜드 실측이므로 유지 |
| 12 | 포커스 링 | 전역 outline 2px 있으나 라디우스 컨텍스트 무시 | focus-visible 토큰화 + 컨트롤별 radius 승계 |
| 13 | z-index 매직넘버 | 19/20/30/31/35/40/45 인라인 | `--z-nav/overlay/sheet/cele/toast/fly` 토큰 |
| 14 | 타이포 인라인 | 13/15/17px 반복 인라인 | 반복값만 토큰(--fs-label:13, --fs-btn:15, --fs-input:17), lh 명시 |

## Gemini에서 가져오는 것 / 다르게 하는 것
- 가져옴: 조용한 surface 위계, hover의 절제(밝기 4~6%), 상태 완비, 아이콘 단일 optical weight, 토큰 단일 출처, overlay 진입 모션(op+2~4px translate), a11y 키보드 경로.
- 다르게: 컬러·라디우스·타이포는 TetherMax 실측 유지(#2453F0/r4/r12/WantedSans). Toss 스프링 물리(오버슛·슬롯롤)는 이전 브리프의 확정 스펙이므로 제거하지 않고 성공 모먼트에 한정 유지 — Gemini의 "조용함"은 rest/hover 레이어에, Toss의 "물성"은 터치·보상 레이어에.
- 미적용: 데스크톱 tooltip/rail expand(모바일 퍼널에 해당 표면 없음 — 시트가 popover 역할).

## 롤백
`git checkout checkpoint-before-gemini-system -- tmx-onboarding/index.html`
