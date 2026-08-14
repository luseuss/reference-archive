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

## 지금까지 병합된 작업 (PR #1~#22 + 노트북 세션의 브랜치 병합 1건, 전부 main에 병합됨)

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
- **PR #17 (3-1. 다중 선택 & 그룹화)**: 빈 캔버스 드래그로 마퀴 선택, Shift+클릭 토글, Ctrl+G로
  그룹화(`boardPlacements[].groupId`)/Ctrl+Shift+G 또는 버튼으로 해제, 그룹·다중선택 카드는 함께
  드래그(`dragSetFor()`), 선택 시 나타나는 선택 바, Esc는 선택 해제 후 보드 닫기 순서로 동작.
  부수적으로 `parseAndValidate()`가 `groupId`를 파일 재로드 때 조용히 지우던 버그도 같이 수정.

- **PR #18 (3-2. 정렬/분배 툴바 — 유사도/무드보드 정리 스펙의 마지막 항목)**: 선택 바
  (`boardSelectionBar`)에 좌/우/상/하/가로중앙/세로중앙 정렬 + 크기 맞춤 아이콘 버튼 7개 추가.
  `alignSelectedPlacements(mode)`는 선택된 카드 전체의 바운딩 박스를 기준으로 각 카드를
  독립적으로 옮김(그룹 여부 무시 — 디자인 툴의 일반적인 정렬 동작과 동일). `matchSizeSelectedPlacements()`는
  "마지막에 선택에 추가된" 카드(JS `Set`이 삽입 순서를 보존하는 걸 이용)를 기준으로 나머지 카드의
  w/h만 맞추고 위치는 그대로 둠. 둘 다 `n < 2`일 때 버튼 비활성화. 정적 모달 + PiP 템플릿 양쪽에
  마크업/wiring 다 붙임(3-1~3-3과 동일 패턴). 이 sandbox 환경엔 Node가 없어서 문법 검증은 못 했고,
  로컬 PowerShell `HttpListener` + `mcp__Claude_Browser__javascript_tool`로 synthetic mousedown/click
  이벤트를 실제 앱 코드에 디스패치해서 정렬 6종 + 크기맞춤 + 비활성화 가드까지 좌표 계산으로
  검증 완료(Document PiP는 이 환경에서 항상 실패해서 기존 모달 폴백 경로로 테스트함 — 예상된 동작).

**이걸로 유사도/무드보드 정리 기능 스펙(0~3, PR #12~#18)이 전부 완료됨.**

**PR #19 (무드보드 우클릭 컨텍스트 메뉴 — 사용자가 스펙 완료 후 추가 요청)**: 상단 툴바 버튼으로만
쓸 수 있던 동작들을 우클릭으로도 쓸 수 있게 함. 빈 캔버스 우클릭은 메모/레퍼런스 추가(메모는
카스케이드 대신 클릭 지점에 중심 맞춰 배치)·격자 스냅·캡처, 카드 우클릭은 제거·그룹·정렬
6종+크기맞춤(3-2 함수 재사용, `n<2` 비활성화 패턴도 동일)을 보여줌. 선택 안 된 카드 우클릭은 그
카드만 선택, 이미 다중선택된 카드 중 하나를 우클릭하면 선택 유지. 바깥클릭/Escape(선택해제·보드닫기보다
먼저)/드래그·마퀴 시작 시 닫힘. **부수 버그 수정**: 카드 mousedown 핸들러가 `e.button`을 안 봐서
우클릭도 좌클릭 취급되던 것(선택+드래그 진입 → mouseup이 "클릭"으로 잡혀 프리뷰가 메뉴 위에 열림)을
`startBoardMarquee`에 이미 있던 것과 같은 `if(e.button !== 0) return;` 가드로 카드/리사이즈 핸들
mousedown에도 추가해서 고침.

**PR #20 (무드보드 드롭 위치/메모 폰트/폴더·카테고리 즉시 생성/팬-줌 — 사용자가 4가지를 한 번에 요청)**:
1. 메인 화면 카드를 무드보드로 드래그해서 놓으면 이제 카스케이드가 아니라 실제로 놓은 위치(카드
   중심 기준)에 정확히 배치됨. 좌표 변환을 `screenToCanvasCoords(clientX, clientY)` 공통 함수로
   뽑아서 드롭 배치·마퀴 선택·우클릭 메뉴가 전부 공유(각자 따로 갖고 있던 rect/줌 계산 중복 제거).
   `addToBoard(itemId, atX, atY)`가 좌표를 받으면 그 지점을 카드 중심으로 배치, 없으면 기존
   카스케이드 유지(피커 클릭 추가는 그대로).
2. 메모(스티키노트)에 `fontSize`/`fontFamily` 필드 추가(기본값은 기존 12.5px 산세리프 그대로).
   메모 하나만 선택한 채 우클릭하면 컨텍스트 메뉴에 글자 크기(작게/보통/크게)·폰트(기본 고딕/명조/
   손글씨/고정폭 — 전부 시스템 폰트, 새 웹폰트 로드 없음) 드롭다운이 떠서 즉시 적용됨.
   `parseAndValidate()`에서 안전한 기본값으로 복원, `exportData`/`writeToFile`은 이미 placement를
   통째로 직렬화하고 있어서 별도 수정 불필요.
3. "레퍼런스 추가" 모달의 폴더/카테고리 select 옆에 "+" 버튼 추가 — 기존
   `folderTaxonomy.openModal()`/`categoryTaxonomy.openModal()`을 그대로 재사용(새 모달 안 만듦).
   `makeTaxonomy()`에 `onCreate`/`onClose` 훅을 추가해서, 이 버튼으로 만든 폴더/카테고리는 저장
   즉시 그 select에 자동 선택됨. "먼저 만들어주세요" 안내문 삭제. **부수 버그 수정**: 전역 Escape
   키다운 핸들러가 열려있는 모달을 전부 개별 `if`로 닫고 있어서, 중첩 모달(폴더/카테고리 생성
   모달이 레퍼런스 추가 모달 위에 뜬 상태)에서 Escape를 누르면 안쪽 모달만 닫혀야 하는데 바깥
   레퍼런스 추가 모달까지 같이 닫혔음. 우선순위 체인(가장 위 모달 하나만 처리하고 return)으로 재작성.
4. 무드보드 휠 줌에서 Ctrl 요구 제거(휠만으로 줌). 스페이스바를 누른 채 드래그하면 캔버스가
   팬(스크롤)되고, 커서가 grab/grabbing으로 바뀜. 스페이스 없이 드래그하면 기존처럼 마퀴 선택
   (빈 캔버스)·카드 드래그(카드 위)가 그대로 동작 — 카드 위에서 스페이스+드래그해도 카드는
   움직이지 않고 팬만 되도록 카드/리사이즈 mousedown 핸들러에 가드 추가.
   이 sandbox 환경엔 Node가 없어서 문법 검증은 못 했고, 로컬 PowerShell `HttpListener` +
   `mcp__Claude_Browser__javascript_tool`로 실제 UI를 조작해 프로젝트/장면/아이템을 만들고,
   70% 축소 상태에서 synthetic `DragEvent('drop')`으로 드롭 위치 계산을 검증하고, 메모 폰트/크기를
   바꾼 뒤 `btnExport`를 가로채 내보낸 JSON을 `fileImportFallback`에 넣어 실제
   `parseAndValidate`→`mergeOrReplace` 경로로 재로드해서 값이 복원되는지까지 확인했음. 폴더/카테고리
   중첩 모달의 Escape/바깥클릭 격리와 자동 선택, 휠 줌·스페이스+팬·마퀴의 상호 배제(카드 위
   스페이스+드래그 포함)도 synthetic 이벤트로 전부 확인 완료.

**`bookmarklet-copy-button` 브랜치 병합 (`0a1133f`, 사용자가 노트북 세션에서 직접 작업 후 PR 없이
`git merge`로 main에 병합, 커밋 `9350b14`)**: 이 브랜치는 PR #20이 나오기 전의 오래된 main에서
갈라져 있었음. (1) 무드보드 휠클릭(가운데 버튼)+드래그로 캔버스 이동(`wireBoardPan()`) — 이
세션이 PR #20에서 만든 스페이스바+드래그 팬과는 별개의 트리거라 병합 시 둘 다 유지됨(같은
`.panning` 커서 클래스 공유). (2) 유튜브 아이템을 그리드 카드/프리뷰 라이트박스에서 정적 썸네일
대신 실제 iframe으로 인라인 재생 — 카드에 호버하면 음소거 자동재생, 벗어나면 정지 후 처음으로
리셋(`ytPost()`가 `postMessage`로 유튜브 플레이어에 명령). `file://`로 직접 열면 유튜브가 임베드를
막아서(오류 153) 그때만 기존 썸네일로 폴백. 병합 커밋 메시지에 따르면 카드 드래그/리사이즈의
`e.button !== 0`·`boardSpaceDown` 가드는 main 쪽(PR #19/#20)이 이미 다 처리하고 있어서 그대로
유지됨. 병합 후 이 세션에서 synthetic 이벤트로 재검증: 가운데클릭 팬·스페이스+드래그 팬·PR #21
포인터 기준 줌이 서로 안 부딪히고 전부 정상 동작 확인.

**PR #21 (무드보드 휠 줌을 마우스 포인터 기준으로 — 사용자 요청)**: 휠로 확대/축소할 때 항상
캔버스 왼쪽 위가 기준이던 걸, 마우스 포인터 아래 콘텐츠가 줌 전후로 같은 화면 위치에 남도록
`setBoardZoom(z, anchor)`에 `{clientX, clientY}` anchor 파라미터를 추가해서 고침. anchor가 있으면
줌 전 그 지점의 콘텐츠 좌표(줌 100% 기준)를 구해두고 줌 이후 `board-canvas-wrap`의 스크롤을
보정. 줌 인/아웃/리셋 버튼은 anchor 없이 호출되니 기존 동작(스크롤 그대로) 유지. synthetic wheel
이벤트로 스크롤 보정값이 독립적으로 계산한 기대값과 서브픽셀 오차 이내로 일치함을 확인.

**PR #22 (무드보드 카드 드래그 화면 가장자리 자동 스크롤 — 사용자 버그 리포트: "일정 범위 이상
벗어나면 드래그가 안된다")**: 카드를 넓게 펼쳐놓고 화면 가장자리 너머로 옮기려 하면 마우스가
창 가장자리에 닿아 더 움직일 물리적 공간이 없어져 드래그가 멈춘 것처럼 보이던 문제. 드래그 중
포인터가 `board-canvas-wrap` 가장자리 36px 이내면 `requestAnimationFrame`으로 매 프레임 그
방향으로 계속 스크롤(추가 mousemove 없이도 계속 진행, Figma/Miro류 드래그-스크롤과 동일), 스크롤된
만큼 카드도 커서 아래 계속 붙어 따라옴, 놓으면 즉시 멈춤. `onDragMove(e)`를 "최신 포인터 좌표
기록"과 "실제 위치/스냅 계산"(`applyDragMove()`)으로 나눠서 실제 mousemove와 자동 스크롤 틱이
같은 계산을 공유하게 리팩터링. 캔버스 고정 크기(2400×1600)는 안 건드렸음 — 절대 위치 자식의
overflow가 스크롤 가능 영역에 포함되는 CSS 동작 덕분에 카드가 그 크기를 넘어가도
`board-canvas-wrap.scrollWidth`가 알아서 늘어나는 것을 synthetic 이벤트로 확인함(가장자리에
오래 붙잡아 카드가 4392px까지, scrollWidth가 4592px까지 늘어나는 것도 검증).
**작업 중 실수**: 이 PR은 브랜치를 안 만들고 실수로 로컬 main에 바로 커밋했다가, 원격 main은
안 건드려진 것을 확인한 뒤 `git branch -m`으로 로컬 브랜치 이름만 바꾸고 origin/main에서 새
main을 다시 체크아웃해서 바로잡음(원격에는 영향 없었음).

로컬 `main`은 PR #22 병합 후 `d888d31`까지 동기화 확인됨.

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
3. 실제/근사 브라우저 환경에서 꼼꼼히 테스트 — 이 환경은 File System Access 저장이나 Document
   PiP의 실제 사용자 제스처를 지원하지 않는 샌드박스라, PowerShell `System.Net.HttpListener`로
   임시 로컬 서버를 띄우고(`run_in_background: true`), `mcp__Claude_Browser__preview_start`를
   작업 디렉터리 루트의 `.claude/launch.json`(예: `E:\code\.claude\launch.json`)에
   `{"url": "http://localhost:PORT", "port": PORT}`로 연결해서 테스트. 테스트 끝나면 서버
   프로세스 반드시 종료하고, 테스트용 `launch.json`은 커밋하지 말 것(레포용이 아님).
4. 상세한 커밋 메시지로 커밋 → push
5. `gh pr create`로 Summary + Test plan 포함해 PR 오픈 (PiP 관련 기능은 실제 사용자 제스처가
   필요해 자동화로 검증 불가능하다는 점을 본문에 명시하고, 폴백 모달 경로로 검증)
6. **사용자의 명시적 "병합해줘" 전까지 병합하지 않고 대기**
7. 병합 후 로컬 `main` 동기화(`git checkout main && git pull origin main`)
8. **이 `CLAUDE.md`도 이번 PR 내용으로 갱신 → 커밋 → push**
9. 다음 계획 항목으로 이동

## 남은 작업
사용자가 승인했던 유사도/무드보드 정리 기능 스펙(0 → 2 → 1 → 3-4 → 3-3 → 3-1 → 3-2)은 PR #18
병합으로 전부 완료됨. 현재 정해진 다음 작업 없음 — 새 세션에서는 사용자에게 다음에 뭘 할지
먼저 물어볼 것.
