# reference-archive — 프로젝트 메모리

이 파일은 Claude Code가 이 폴더에서 세션을 시작할 때 자동으로 읽는 "외부 기억 장치"입니다.
새 대화를 시작하기 전에 이 파일부터 읽고, 지금까지 뭘 했는지 파악한 뒤 이어서 작업하세요.
**중요한 결정이나 새 PR이 병합될 때마다 이 파일을 최신 상태로 갱신할 것.**

## 저장소 정보
- GitHub: https://github.com/luseuss/reference-archive (소유자 계정: `luseuss`)
- 배포(GitHub Pages): https://luseuss.github.io/reference-archive/app.html
- 로컬 clone: `E:\code\reference-archive`

## 프로젝트가 뭔지
"레퍼런스 아카이브" — 이미지·디자인 레퍼런스와 유튜브 영상을 로컬 파일에 자동 저장하며 모으는
개인용 로컬 우선 PWA. 서버 없음, 구독 없음, File System Access API로 브라우저가 직접 로컬 파일에 저장.

- `index.html` — 설치/소개용 랜딩페이지
- `app.html` — 실제 아카이브 앱 (전부 여기 있음: 단일 파일 vanilla HTML/CSS/JS, 빌드 스텝 없음)
- `manifest.json`, `sw.js`, `icon-*.png` — PWA 관련 파일

## 배포 전략
**PWA가 메인 배포 채널.** `app.html`을 고치고 push하면 서비스워커(네트워크 우선)가 자동으로
최신화됨 — 별도 빌드/배포 불필요.

Tauri로 Windows exe도 한 번 만들어봤지만(독립 실행 파일 + NSIS 설치 프로그램), 부수적인 실험이었음:
- 자동 업데이트 인프라는 설정 안 함
- **`src-tauri/` 폴더는 의도적으로 git에 커밋하지 않고 이 컴퓨터에만 남겨둠** (`.gitignore`에
  `src-tauri/target/`만 등록되어 있고, `src-tauri/` 자체는 그냥 git add를 안 함).
  다른 컴퓨터에는 이 폴더가 없음 — exe를 다시 만들려면 Rust부터 새로 설치해야 함.

## 지금까지 병합된 작업 (PR #1~#16, 전부 main에 병합됨)

**PR #1~#5** (초기 단계):
1. README.md 추가 (첫 PR 연습)
2. 랜딩페이지 리서치(Eagle/PureRef/Milanote/Are.na/refern/Pinterest 비교) + 카피 5섹션
   (Hero/Problem/Benefits/Testimonials/CTA) + Variant.com 목업 참고한 다크 터미널/네오브루탈리즘
   리디자인 (세이지 그린 accent `#719f89`, Pretendard + IBM Plex Mono, 마퀴 티커, 세그먼트 네비게이션)
3. `app.html` 코드 리뷰 + 수정: 업로드 이미지 리사이즈(긴 변 1600px, JPEG q0.85), `safeHref()`로
   `javascript:` 등 위험 URL 스킴 차단, 검색 디바운스 + `render()`/`renderGrid()` 분리(타이핑마다
   전체 재렌더 제거), 폴더/카테고리 로직을 `makeTaxonomy()` 팩토리로 통합
4. 일괄 선택(체크박스 → 폴더 이동/태그 추가/일괄 삭제), 북마클릿(가장 큰 이미지 캡처 → 추가 모달
   자동 채움, GitHub Pages 주소 고정), 카드 핀 고정(⭐, 정렬 무관 항상 상단) + 즐겨찾기 필터

**PR #6~#11** (사용자가 노트북에서 별도 세션으로 작업):
- 플로팅 PIP 보드, 프로젝트 기반 정리, 씬 무드보드, 보드 줌/캡처/스티키노트, PiP 무드보드 창,
  북마클릿 복사 버튼

**PR #12~#16** (이번 세션, 유사도/무드보드 정리 기능 스펙을 0→2→1→3-4→3-3 순서로 진행):
- **PR #12 (0. 유사도 엔진)**: AI 호출 없이 태그 Jaccard + 카테고리/폴더/프로젝트 보너스 + dHash
  퍼셉추얼 해시(캔버스로 9×8 그레이스케일 다운스케일 → 64비트 해시, CORS 막히면 조용히 null로
  degrade). `item.pHash`에 캐싱, `similarityScore()`/`similarItems()` 핵심 함수.
- **PR #13 (2. 프리뷰 추천)**: 라이트박스 하단에 "비슷한 레퍼런스" 그리드 (`PREVIEW_SIMILAR_THRESHOLD = 0.12`)
- **PR #14 (1. 그리드 정렬)**: 정렬 드롭다운에 "유사한 것끼리" 옵션, 최신 아이템을 앵커로 유사도순 정렬
- **PR #15 (3-4. 보드 클릭 확장)**: 무드보드 카드를 드래그 아닌 클릭으로 프리뷰 열기
  (`BOARD_CLICK_THRESHOLD = 4`), PiP 창 안에서도 동작하도록 `previewContextDoc`/`pDoc()`/`$p()` 패턴 도입
- **PR #16 (3-3. 스냅)**: 보드 드래그/리사이즈 시 카드 엣지 8px 스냅 + 선택적 격자(20px) 스냅 토글,
  스냅 가이드라인 표시

로컬 `main`은 위 PR들 병합 후 `c10f80f`까지 동기화 확인됨.

## 중요한 결정/맥락 (git 로그엔 안 보이는 부분)
- Windows Smart App Control이 Tauri/Rust 빌드를 막은 적 있음(사용자가 직접 설정 껐음 — 보안 설정은
  내가 손대지 않음).
- Document PiP(`documentPictureInPicture.requestWindow()`)는 진짜 사용자 클릭이 필요해서 샌드박스
  자동화 테스트로는 검증 불가 — 사용자가 수동으로 직접 확인해줘야 함.
- 좌표 기반 클릭(`mcp__Claude_Browser__computer`)은 스케일링 문제로 불안정 → `ref` 기반 클릭이나
  `javascript_tool`로 synthetic DOM 이벤트 디스패치를 선호.
- `file://`나 실제 Chrome의 localhost 내비게이션이 막혀서, 테스트할 땐 PowerShell
  `System.Net.HttpListener`로 임시 로컬 서버를 띄워서 씀(끝나면 종료).

## 작업 워크플로우 (매번 이렇게 진행)
1. `main`에서 새 브랜치 생성
2. `app.html`에 기능 구현
3. 실제/근사 브라우저 환경에서 꼼꼼히 테스트
4. 상세한 커밋 메시지로 커밋 → push
5. `gh pr create`로 Summary + Test plan 포함해 PR 오픈
6. **사용자의 명시적 "병합해줘" 전까지 병합하지 않고 대기**
7. 병합 후 로컬 `main` 동기화(`git checkout main && git pull`)
8. 다음 계획 항목으로 이동

## 남은 작업 (사용자가 승인한 순서: 0 → 2 → 1 → 3-4 → 3-3 → 3-1 → 3-2)
아직 시작 안 함:
- **3-1. 다중 선택 & 그룹화**: 보드 캔버스 빈 공간 드래그로 선택 사각형(겹치는 카드 선택),
  Shift+클릭으로 개별 추가/제거. 선택된 카드들을 Ctrl+G로 그룹화 — `boardPlacements`에 `groupId`
  필드 추가, 그룹 내 카드 하나 드래그하면 전체가 같이 이동. 그룹 해제 기능도 필요.
- **3-2. 정렬/분배 툴바**: 2개 이상 선택 시 나타나는 툴바 — 좌/우/상/하 정렬, 가로/세로 중앙 정렬,
  크기 맞춤(match size).

이 두 항목이 끝나면 사용자가 요청한 전체 기능 스펙(0~3)이 완료됨.
