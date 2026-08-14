# reference-archive — 업데이트 로그

`app.html`에 반영된 변경사항을 PR 단위·시간순으로 기록합니다. 새 PR이 병합될 때마다 이 파일을
갱신하세요(프로젝트 구조·워크플로우 같은 절차적인 내용은 `CLAUDE.md` 참고).

## PR #1~#5 (초기 단계)
1. README.md 추가 (첫 PR 연습)
2. 랜딩페이지 리서치(Eagle/PureRef/Milanote/Are.na/refern/Pinterest 비교) + 카피 5섹션
   (Hero/Problem/Benefits/Testimonials/CTA) + Variant.com 목업 참고한 다크 터미널/네오브루탈리즘
   리디자인 (세이지 그린 accent `#719f89`, Pretendard + IBM Plex Mono, 마퀴 티커, 세그먼트 네비게이션)
3. `app.html` 코드 리뷰 + 수정: 업로드 이미지 리사이즈(긴 변 1600px, JPEG q0.85), `safeHref()`로
   `javascript:` 등 위험 URL 스킴 차단, 검색 디바운스 + `render()`/`renderGrid()` 분리(타이핑마다
   전체 재렌더 제거), 폴더/카테고리 로직을 `makeTaxonomy()` 팩토리로 통합
4. 일괄 선택(체크박스 → 폴더 이동/태그 추가/일괄 삭제), 북마클릿(가장 큰 이미지 캡처 → 추가 모달
   자동 채움, GitHub Pages 주소 고정), 카드 핀 고정(⭐, 정렬 무관 항상 상단) + 즐겨찾기 필터

## PR #6~#11 (사용자가 노트북에서 별도 세션으로 작업)
- 플로팅 PIP 보드, 프로젝트 기반 정리, 씬 무드보드, 보드 줌/캡처/스티키노트, PiP 무드보드 창,
  북마클릿 복사 버튼

## PR #12~#16 (유사도/무드보드 정리 기능 스펙을 0→2→1→3-4→3-3 순서로 진행)
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

## PR #19 (무드보드 우클릭 컨텍스트 메뉴 — 사용자가 스펙 완료 후 추가 요청)
상단 툴바 버튼으로만 쓸 수 있던 동작들을 우클릭으로도 쓸 수 있게 함. 빈 캔버스 우클릭은 메모/레퍼런스
추가(메모는 카스케이드 대신 클릭 지점에 중심 맞춰 배치)·격자 스냅·캡처, 카드 우클릭은 제거·그룹·정렬
6종+크기맞춤(3-2 함수 재사용, `n<2` 비활성화 패턴도 동일)을 보여줌. 선택 안 된 카드 우클릭은 그
카드만 선택, 이미 다중선택된 카드 중 하나를 우클릭하면 선택 유지. 바깥클릭/Escape(선택해제·보드닫기보다
먼저)/드래그·마퀴 시작 시 닫힘. **부수 버그 수정**: 카드 mousedown 핸들러가 `e.button`을 안 봐서
우클릭도 좌클릭 취급되던 것(선택+드래그 진입 → mouseup이 "클릭"으로 잡혀 프리뷰가 메뉴 위에 열림)을
`startBoardMarquee`에 이미 있던 것과 같은 `if(e.button !== 0) return;` 가드로 카드/리사이즈 핸들
mousedown에도 추가해서 고침.

## PR #20 (무드보드 드롭 위치/메모 폰트/폴더·카테고리 즉시 생성/팬-줌 — 사용자가 4가지를 한 번에 요청)
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

## `bookmarklet-copy-button` 브랜치 병합 (`0a1133f`, 사용자가 노트북 세션에서 직접 작업 후 PR 없이 `git merge`로 main에 병합, 커밋 `9350b14`)
이 브랜치는 PR #20이 나오기 전의 오래된 main에서 갈라져 있었음. (1) 무드보드 휠클릭(가운데 버튼)+드래그로
캔버스 이동(`wireBoardPan()`) — 이 세션이 PR #20에서 만든 스페이스바+드래그 팬과는 별개의 트리거라
병합 시 둘 다 유지됨(같은 `.panning` 커서 클래스 공유). (2) 유튜브 아이템을 그리드 카드/프리뷰
라이트박스에서 정적 썸네일 대신 실제 iframe으로 인라인 재생 — 카드에 호버하면 음소거 자동재생,
벗어나면 정지 후 처음으로 리셋(`ytPost()`가 `postMessage`로 유튜브 플레이어에 명령). `file://`로
직접 열면 유튜브가 임베드를 막아서(오류 153) 그때만 기존 썸네일로 폴백. 병합 커밋 메시지에 따르면
카드 드래그/리사이즈의 `e.button !== 0`·`boardSpaceDown` 가드는 main 쪽(PR #19/#20)이 이미 다
처리하고 있어서 그대로 유지됨. 병합 후 이 세션에서 synthetic 이벤트로 재검증: 가운데클릭 팬·스페이스+드래그
팬·PR #21 포인터 기준 줌이 서로 안 부딪히고 전부 정상 동작 확인.

## PR #21 (무드보드 휠 줌을 마우스 포인터 기준으로 — 사용자 요청)
휠로 확대/축소할 때 항상 캔버스 왼쪽 위가 기준이던 걸, 마우스 포인터 아래 콘텐츠가 줌 전후로 같은
화면 위치에 남도록 `setBoardZoom(z, anchor)`에 `{clientX, clientY}` anchor 파라미터를 추가해서 고침.
anchor가 있으면 줌 전 그 지점의 콘텐츠 좌표(줌 100% 기준)를 구해두고 줌 이후 `board-canvas-wrap`의
스크롤을 보정. 줌 인/아웃/리셋 버튼은 anchor 없이 호출되니 기존 동작(스크롤 그대로) 유지. synthetic
wheel 이벤트로 스크롤 보정값이 독립적으로 계산한 기대값과 서브픽셀 오차 이내로 일치함을 확인.

## PR #22 (무드보드 카드 드래그 화면 가장자리 자동 스크롤 — 사용자 버그 리포트: "일정 범위 이상 벗어나면 드래그가 안된다")
카드를 넓게 펼쳐놓고 화면 가장자리 너머로 옮기려 하면 마우스가 창 가장자리에 닿아 더 움직일 물리적
공간이 없어져 드래그가 멈춘 것처럼 보이던 문제. 드래그 중 포인터가 `board-canvas-wrap` 가장자리
36px 이내면 `requestAnimationFrame`으로 매 프레임 그 방향으로 계속 스크롤(추가 mousemove 없이도
계속 진행, Figma/Miro류 드래그-스크롤과 동일), 스크롤된 만큼 카드도 커서 아래 계속 붙어 따라옴,
놓으면 즉시 멈춤. `onDragMove(e)`를 "최신 포인터 좌표 기록"과 "실제 위치/스냅 계산"(`applyDragMove()`)으로
나눠서 실제 mousemove와 자동 스크롤 틱이 같은 계산을 공유하게 리팩터링. 캔버스 고정 크기(2400×1600)는
안 건드렸음 — 절대 위치 자식의 overflow가 스크롤 가능 영역에 포함되는 CSS 동작 덕분에 카드가 그
크기를 넘어가도 `board-canvas-wrap.scrollWidth`가 알아서 늘어나는 것을 synthetic 이벤트로 확인함
(가장자리에 오래 붙잡아 카드가 4392px까지, scrollWidth가 4592px까지 늘어나는 것도 검증).

**작업 중 실수**: 이 PR은 브랜치를 안 만들고 실수로 로컬 main에 바로 커밋했다가, 원격 main은 안
건드려진 것을 확인한 뒤 `git branch -m`으로 로컬 브랜치 이름만 바꾸고 origin/main에서 새 main을
다시 체크아웃해서 바로잡음(원격에는 영향 없었음).

## PR #23 (무드보드 캔버스 범위를 눈에 보이게 + 배치에 맞춰 유동적으로 — PR #22 후속 요청)
PR #22의 가장자리 자동 스크롤로 고정 캔버스(2400×1600) 밖으로도 드래그는 이어졌지만 그 "밖"이
안쪽과 똑같아 보여서 범위가 안 보이던 문제. `.board-canvas`에 `var(--surface)` 배경 + 안쪽
테두리(box-shadow — 레이아웃 크기 안 건드림)를 줘서 드래그 범위가 뚜렷한 "페이지"로 보이게 함.
`updateBoardCanvasSize()`가 현재 씬 배치 바운딩 박스(최대 x+w, y+h) + 600px 여유로 캔버스 크기를
계산(최소 2400×1600은 그대로 바닥), `renderBoard()`뿐 아니라 `applyDragMove()`/`onResizeMove()`에서도
호출해서 드래그·리사이즈 도중 프레임마다 실시간으로 커지고(가장자리 자동 스크롤과 맞물려 드래그하는
만큼 범위가 바로 늘어나 보임), 카드를 원점 쪽으로 옮기면 다시 최소값으로 줄어드는 것까지 synthetic
이벤트로 확인함.

## PR #24 (메모 폰트: 컴퓨터에 설치된 폰트 불러오기 + px 단위 세밀 조정 — 사용자 요청)
글자 크기는 3단계 select 대신 `<input type="number">`(6~200px, `input` 이벤트로 즉시 적용)로
바꿔서 원하는 px 값을 직접 입력 가능. 폰트는 기존 4개 프리셋 select 옆에 "💻 내 컴퓨터에 설치된
폰트 불러오기" 버튼 추가 — `window.queryLocalFonts()`(Local Font Access API, File System Access와
같은 급의 Chromium 전용·사용자 제스처 필요·origin당 1회 권한요청 API)로 시스템 폰트 이름을 가져와
"내 컴퓨터 폰트" optgroup으로 select에 추가. `note.fontFamily`는 이제 프리셋 id 또는 실제 로컬 폰트
이름 문자열 둘 다 저장 가능(`noteFontStack()`이 둘 다 처리).

**부수 버그 수정**: `parseAndValidate()`가 프리셋 id만 허용해서 시스템 폰트 선택이 파일을 다시 열
때마다 기본값으로 리셋되던 걸, 임의의 비어있지 않은 문자열을 허용하도록 완화해서 고침. 이 sandbox
환경엔 실제 폰트 열거 백엔드가 없어서 `queryLocalFonts()`가 에러 없이 폰트 0개를 반환함(Document
PiP가 항상 실패하는 것과 비슷한 제약) — 그 경로의 버튼 상태 복구·메뉴 재렌더는 확인했지만, select에
목업 폰트 이름을 직접 주입해 선택→적용→내보내기/불러오기 왕복까지는 검증 완료. 실제 폰트 목록이
뜨는지는 사용자가 실제 크롬/엣지에서 버튼을 눌러 권한 프롬프트를 직접 확인해야 함.

## PR #25 (메모에 이미지와 동일한 리치 텍스트 툴바 — 사용자가 참고 이미지 주면서 요청, "이미지와 동일하게 전부" 선택)
메모 본문을 `<textarea>`에서 `contenteditable` div로 바꾸고, 카드 위쪽에 항상 떠 있는 도구모음
추가: A(글자색)/✏(형광펜)/🪣(카드 배경색, 각각 숨긴 `<input type=color>`를 버튼이 대신 열어줌) ·
B/I/U · 🔗(링크, `safeHref()` 검증 + `target=_blank rel=noopener` 강제) · ☰(목록) · 정렬(클릭마다
왼쪽→가운데→오른쪽 순환, 라벨이 현재 상태 표시) · 글자크기(px 숫자 입력)/폰트(PR #24의
`queryLocalFonts()` 재사용) — 텍스트 선택 있으면 그 부분만 span으로 감싸고 없으면(커서만) 메모
전체 기본 스타일을 바꿈(Google Docs 방식, `execCommand('fontSize','7')`로 일단 감싼 뒤 실제
px/폰트 스택의 span으로 바꿔치기하는 표준 우회법 사용) · 🧹(서식 지우기) · ⤢(크게 보기 토글,
1.6배, 원래 크기는 메모리 Map에만 잠깐 들고 있고 저장 파일엔 안 남음).

placement에 `html`(정제된 리치 콘텐츠)·`bgColor` 필드 추가, `fontSize`/`fontFamily`는 이제 "선택
없을 때의 기본값" 의미로 재해석됨. 허용 목록 기반 `sanitizeNoteHtml()`을 새로 만들어서(태그:
b/strong/i/em/u/s/strike/a/ul/ol/li/span/br/div/p, 스타일: color/background-color/font-family/
font-size/text-align, href는 `safeHref()` 통과) `DOMParser`(문서에 안 붙는 inert 파싱이라 script
실행·리소스 요청 없음)로 파싱해서 불러오기(`parseAndValidate`)와 매 렌더링 시점 둘 다에서 돌림 —
렌더링 시점이 실제 보안 경계. PR #24의 우클릭 메뉴 폰트 컨트롤은 이 툴바가 완전히 대체해서 제거함
(우클릭 메뉴는 다시 구조적 동작만).

**테스트 중 발견한 실제 버그**: `execCommand('styleWithCSS')`는 명령마다 따로가 아니라 문서 전체에
계속 남아있는 상태라서, 색상 명령이 켜둔 styleWithCSS=true가 남은 채로 글자 크기를 적용하면
`<font size=7>` 감싸기 트릭이 아예 동작을 안 하고(`style="font-size:xxx-large"`가 기존 태그에
바로 발라짐) 정확한 px 지정이 깨졌음 — `applyInlineStyleToSelection`에서 styleWithCSS를 명시적으로
끄도록 고침. 악성 payload(`<script>`, `onerror=` 이미지, `javascript:` href, 허용 안 된 스타일/태그)를
실제 내보내기→불러오기 경로로 통과시켜서 데이터·렌더링 DOM 양쪽에서 다 걸러지는 것까지 synthetic
이벤트로 검증 완료.

## PR #26 (무드보드 유튜브 인라인 재생 + 채널정보 오버레이 제거 — 사용자 요청)
무드보드 유튜브 카드도 메인 화면 카드처럼 호버하면 음소거 재생·벗어나면 정지(정적 썸네일 대신).
`pointer-events:none`을 iframe에 줘서(이미지 카드에 이미 있던 패턴과 동일) 카드 위 어디를 눌러도
드래그·클릭 미리보기가 그대로 됨. `addToBoard()`가 유튜브는 썸네일 원본 비율(보통 4:3) 대신 실제
재생 비율 16:9로 카드 크기를 잡도록 바꿈. 세 군데(그리드 호버 미리보기·무드보드 호버 미리보기·프리뷰
라이트박스)에서 각자 만들던 embed URL을 `youtubeEmbedSrc(vid, {autoplayHover})` 공통 함수로 정리 —
호버 미리보기(음소거 자동재생이라 컨트롤 조작 불필요)는 `controls=0`으로 유튜브 기본 UI(채널명
오버레이 포함)를 통째로 꺼서 사용자가 불평했던 "채널 정보가 영상을 가리는" 문제를 해결, 실제 조작이
필요한 라이트박스는 `controls`는 유지하되 `modestbranding=1&rel=0&iv_load_policy=3`로 곁가지 UI만
최대한 줄임(유튜브가 옛 `showinfo` 파라미터를 없애서 컨트롤 유지한 채 제목바만 완전히 없애는 건
공식적으로 더 이상 불가능).

## PR #27 (무드보드 PiP 창 안에서는 유튜브 임베드 대신 썸네일로 폴백, 오류 153 — 사용자 버그 리포트, GitHub Pages에서 재현)
무드보드가 실제 Document PiP 창으로 뜨면 그 창은 `win.document.body.innerHTML`로만 채워지고 실제
URL로 이동한 적이 없어서 문서가 계속 `about:blank`임 — `enablejsapi=1` iframe이 그 안에서 유튜브에
요청하면 리퍼러/오리진이 비어있는 걸로 보여 오류 153이 뜨는 것으로 추정(메인 화면 카드는 진짜 페이지
문서라 무관, 클릭해서 여는 프리뷰 라이트박스는 `enablejsapi` 자체를 안 써서 무관 — 둘 다 문제없다는
사용자 확인과 일치). 보드 카드의 `canEmbed` 조건에 `&& !boardWindow`를 추가해서 PiP 창 안에서는
file:// 때처럼 정적 썸네일로 폴백, PiP 미지원 시 뜨는 정적 모달(메인 화면과 같은 문서)은 그대로
인라인 재생.

## PR #28 (무드보드 그룹에 시각적 영역 표시 — 사용자 요청: "그룹만의 영역이 생겼으면")
그룹으로 묶은 카드들 하나하나에 얇은 인셋 테두리만 붙던 걸(`.board-card.grouped`), 그룹 전체를
감싸는 점선+옅은 배경 프레임(`.board-group-frame`)으로 보이게 함. 카드보다 뒤에 깔리고(z-index:0,
카드는 항상 1 이상) `pointer-events:none`이라 드래그·클릭·마퀴에 전혀 관여 안 함.
`groupBoundingBox(groupId)`가 그룹 카드들의 바운딩 박스(여유 14px)를 계산, `renderBoard()`가 씬의
그룹마다 하나씩 그림. 카드 위치/크기가 실시간으로 바뀌는 드래그·리사이즈 도중엔 전체 재계산 대신
최소한만 갱신: 드래그는 그룹이 항상 통째로 같이 움직이므로(`dragSetFor`) 시작 시점 프레임 위치에서
같은 dx/dy만큼만 옮기고, 리사이즈는 그 카드가 속한 그룹의 바운딩 박스를 다시 계산해서 즉시 반영.

## PR #29 (PiP 창 안 프리뷰 라이트박스에도 유튜브 폴백 적용 — PR #27 후속, 사용자가 라이트박스에서도 153 재현·보고)
PR #27은 "`enablejsapi` 쓰는 호버 미리보기만 문제"라고 가정했는데, `enablejsapi` 안 쓰는 일반
프리뷰 라이트박스도 PiP 창(`about:blank` 문서) 안에서는 똑같이 153이 뜨는 걸 확인 — PiP 문서
안에서는 임베드 자체가 다 막히는 것으로 결론. `isFloatingPipDoc(doc)` 공통 함수를 만들어 보드 카드
렌더링(PR #27의 `!boardWindow`를 같은 이름으로 정리)과 `openPreview()`(`previewContextDoc`/`pDoc()`
기준이라 무관한 PiP 보드가 떠 있어도 메인 그리드에서 연 프리뷰는 안 막힘) 양쪽에 적용. 안내 문구도
file://·PiP·일반 세 가지로 나눠 PiP일 땐 "원본 열기"로 유튜브에서 보라고 명확히 안내. **중요**:
PiP로 띄운 동안은 유튜브가 임베드를 아예 막는 게 이 앱이 고칠 수 있는 부분이 아니라서, 클릭해서
들어가도 재생 대신 "원본 열기" 안내만 뜨는 게 현재 최종 동작.

## PR #30 (무드보드 유튜브 호버 재생 안 되는 문제 수정 — 사용자 버그 리포트, 일반 모달에서 재현)
iframe(`enablejsapi=1`) 자체는 잘 뜨는데 `mouseenter` 즉시 `playVideo`를 보내서, 유튜브 플레이어
내부 스크립트가 아직 부팅 중(postMessage 리스너 붙기 전)이면 조용히 무시되던 문제. 플레이어가
스스로 보내는 `{event:"onReady"}` 메시지를 기다리는 준비 상태 추적 레이어 추가: `window`에 origin
검증된 `message` 리스너 하나로 `contentWindow` 기준 어떤 iframe이 준비됐는지 표시,
`ytPlayOnHover()`/`ytStopOnHoverLeave()`가 준비 안 됐으면 재생 의도만 큐에 담아뒀다가 `onReady`
오면 보내고 그 전에 마우스 떠나면 예약 취소.

## PR #31 (유튜브 호버 재생을 postMessage 대신 src 교체 방식으로 — PR #30 후속, 사용자가 실제 배포 사이트에서 재보고)
이번엔 synthetic 이벤트가 아니라 실제 배포 사이트(GitHub Pages)와 로컬 재빌드본에 진짜 브라우저
클릭/호버로 직접 붙어서 조사함. 모든 origin의 postMessage를 로깅하는 진단 리스너로, `enablejsapi`
iframe에서 `onReady`는커녕 실제 재생이 시작된 뒤에도 메시지가 단 하나도 안 오는 것 확인(PR #30의
대기 로직 자체는 맞았지만 애초에 채널이 안 열리고 있었음 — `origin=` 파라미터를 추가해도 안 됨,
원인 특정은 못 함, 광고차단류 확장이나 브라우저 정책일 가능성). 별도로 메인 그리드 카드는 iframe에
`pointer-events:none`이 없어서 실제 마우스 호버 자체가 부모 요소까지 전달이 안 되는 것도 발견.
postMessage를 걷어내고 호버 시 iframe src를 `&autoplay=1` 붙인 것으로 바꿔치기(mute=1이라 브라우저
정책상 허용), 벗어나면 저장해둔 원래 src로 되돌리는(자연스럽게 리셋됨) 방식으로 교체. 그리드
iframe에도 `pointer-events:none` 추가. **이번엔 실제 배포 사이트에 진짜 마우스로 호버해서 그리드·보드
카드 둘 다 실제로(움직이는 영상으로) 재생되는 것까지 직접 확인함.**

**테스트 방법론 참고**: `mcp__Claude_Browser__preview_start`로 실제 `https://` 배포 URL을 열면
localhost로 서빙할 때와 달리 진짜 외부 네트워크(youtube.com 등)에 접근되는 걸 확인함 — 외부
API/임베드가 관련된 버그는 로컬 HttpListener보다 실제 배포 사이트에 직접 접속해서 검증하는 게 더 정확함.

## PR #32 (PiP 창 다이얼로그/캡처가 엉뚱한 창으로 가는 문제 수정 + 전체 코드 감사 — 사용자 요청: "PIP로 뜨면서 패치 적용이 안되는 느낌, 전체 코드 훑고 최적화")
PiP 템플릿(`ensureBoardWindow`/`wirePipBoardControls`)과 정적 모달의 마크업·하단 wiring을 한 줄씩
대조했지만 완전히 일치 — 빠진 wiring은 없었음. **핵심 버그**: 이 앱 코드는 전부 메인 페이지 스크립트
하나에서 돌고 `boardWindow`의 DOM을 밖에서만 조작하기 때문에, board 관련 함수에서 그냥
`alert()`/`confirm()`/`prompt()`를 부르면 지금 다루는 DOM이 어느 창에 있든 상관없이 항상 메인
창에 뜸 — PiP로 띄워놓고 메모 툴바의 링크/폰트 불러오기/캡처를 누르면 사용자가 안 보고 있는 메인
창에 다이얼로그가 떠서 "패치가 안 먹힌다"처럼 느껴졌을 것으로 추정. `applyNoteLink`/
`loadFontsForNoteToolbar`/`captureBoardAsImage`의 alert/prompt를 전부 `boardDoc().defaultView`
기준으로 고침. **두 번째 관련 버그**: `loadHtml2Canvas()`가 항상 메인 창에만 로드했는데 캡처 대상
캔버스는 PiP 창 문서에 있을 수 있어서 창 간 라이브러리/DOM 불일치로 캡처가 깨질 수 있었음 —
창(window)별 WeakMap 캐싱으로 고침. 그 외: `sanitizeNoteElement`/메모 폰트 select의 임시 `<option>`이
문서 소유권 안 맞추고 만들던 걸 `ownerDocument` 기준으로 고침, `exportData()`/`writeToFile()`의
중복 JSON 생성을 `serializeAppData()`로 통합. 전체 ~200개 함수 사용 횟수·CSS 클래스 참조를 훑어서
죽은 코드/미사용 CSS도 찾아봤는데 발견된 건 없음(코드베이스가 이미 깨끗했음).

---

로컬 `main`은 PR #32 병합 후 `92c0277`까지 동기화 확인됨.
