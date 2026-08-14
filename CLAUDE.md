# reference-archive — 프로젝트 메모리

이 파일은 Claude Code가 이 폴더에서 세션을 시작할 때 자동으로 읽는 "외부 기억 장치"입니다.
새 대화를 시작하기 전에 이 파일부터 읽고, `update.md`(PR 단위 변경 이력)도 같이 확인해서
지금까지 뭘 했는지 파악한 뒤 이어서 작업하세요.
**중요한 결정이 생기면 이 파일을, 새 PR이 병합되면 `update.md`를 최신 상태로 갱신할 것.**

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

Tauri로 Windows exe도 몇 번 만들어봤음(독립 실행 파일 + NSIS 설치 프로그램), 부수적인 실험:
- 자동 업데이트 인프라는 설정 안 함
- **`src-tauri/` 폴더는 의도적으로 git에 커밋하지 않고 이 컴퓨터에만 남겨둠** (`.gitignore`에
  `src-tauri/target/`만 등록되어 있고, `src-tauri/` 자체는 그냥 git add를 안 함).
  다른 컴퓨터에는 이 폴더가 없음 — exe를 다시 만들려면 Rust부터 새로 설치해야 함.
- exe를 다시 빌드할 땐: `app.html`/`manifest.json`/`sw.js`/아이콘을 `src-tauri/dist/`로 복사(특히
  `app.html`이 자주 바뀌므로 빌드 전 항상 최신인지 확인) → `cargo tauri build`(release 프로필,
  50초 안팎) → 결과물은 `src-tauri/target/release/reference-archive.exe`(독립 실행 파일)와
  `src-tauri/target/release/bundle/nsis/레퍼런스 아카이브_0.1.0_x64-setup.exe`(설치 프로그램).

## 업데이트 로그
PR 단위 변경 이력(무엇을 왜 어떻게 고쳤는지, 테스트 방법과 한계)은 `update.md`에 기록합니다.
지금까지 PR #1~#32 + 노트북 세션의 브랜치 병합 1건이 전부 main에 병합됨. 로컬 `main`은
PR #32 병합 후 `92c0277`까지 동기화 확인됨 — 최신 상태는 `update.md` 맨 아래 줄 참고.

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
8. **`update.md`에 이번 PR 내용 추가 → 커밋 → push** (CLAUDE.md는 구조/워크플로우 같은 절차적인
   내용이 바뀔 때만 갱신)
9. 다음 계획 항목으로 이동

## 남은 작업
사용자가 승인했던 유사도/무드보드 정리 기능 스펙(0 → 2 → 1 → 3-4 → 3-3 → 3-1 → 3-2)은 PR #18
병합으로 전부 완료됨. 현재 정해진 다음 작업 없음 — 새 세션에서는 사용자에게 다음에 뭘 할지
먼저 물어볼 것.
