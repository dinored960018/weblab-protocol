# TOOLING

모작·창작에 쓰는 도구와 **언제 무엇을 쓰는가**.
깔아만 두면 안 쓴다. 이 문서는 절차의 각 단계에 도구를 못 박기 위한 것이다.

참고 가이드: https://lazyowen.com/guides/claude-designer-kill

---

## 설치

**홈 디렉터리에서 유저 스코프로 설치한다.** 프로젝트 스코프로 깔지 않는다.

```bash
cd ~

npx --yes skills@1.7.4 add emilkowalski/skills --skill "emil-design-eng" --agent claude-code --yes --copy
npx --yes skills@1.7.4 add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend" --agent claude-code --yes --copy
npx --yes impeccable@4.7.0 install --providers=claude --user --yes

claude mcp add --scope user playwright npx @playwright/mcp@0.0.83
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
```

### 왜 유저 스코프인가 — 실제로 걸렸던 문제

2026-09-21에 프로젝트 스코프(`--scope=project`)로 깔았다가 둘 다 안 됐다.

- **스킬이 안 뜬다.** 프로젝트 스코프 스킬은 Claude Code를 **그 폴더에서 열었을 때만** 로드된다.
  홈에서 열면 목록에 없다.
- **MCP가 계속 승인 대기.** 프로젝트 `.mcp.json`은 레포에서 딸려올 수 있는 것이라
  매번 승인을 요구한다. 유저 스코프는 안 묻는다.

유저 스코프로 옮기니 스킬은 즉시 잡혔고 Playwright는 바로 `Connected`가 됐다.

설치 결과는 `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude.json`에 남는다.
**작업 레포에는 아무것도 커밋하지 않는다.** 재현은 위 명령으로 한다.

### 버전

위 명령은 **2026-10-23에 확인한 버전으로 고정**했다. `@latest`로 깔면 규칙 수·명령 이름이 말없이 바뀐다.
올릴 때는 버전을 바꾸고 `impeccable detect`를 지난 작업 하나에 돌려 결과가 크게 달라지지 않는지 본 뒤
이 문서를 같이 고친다.

### 클라우드 세션 (claude.ai/code)

클라우드 세션은 **매번 새 컨테이너**라 홈 디렉터리 설치가 남지 않는다.

- 스킬·MCP는 세션마다 위 명령을 다시 돌리거나, 환경 설정의 설치 스크립트에 넣는다
- `impeccable detect`는 설치 없이 `npx --yes impeccable@4.7.0 detect …`로 바로 돈다. 게이트는 이걸로 충분하다
- Playwright MCP가 없으면 Playwright 스크립트로 같은 클릭 검증을 한다(브라우저는 미리 깔려 있다)
- Figma MCP는 OAuth가 필요해서 클라우드 세션에서는 쓰지 않는다

### 설치 후

1. **Claude Code를 재시작한다.** 스킬은 재시작해야 등록된다.
2. `claude mcp list`로 확인한다. Playwright는 `✔ Connected`가 떠야 한다.
3. **Figma MCP는 OAuth 로그인**이 따로 필요하다. 처음엔 `! Needs authentication`으로 뜬다.
4. `/impeccable init`을 한 번 돌려 `PRODUCT.md`를 만든다.

---

## 무엇이 무엇인가

| 도구 | 출처 | 하는 일 |
|---|---|---|
| **emil-design-eng** | [emilkowalski/skills](https://github.com/emilkowalski/skills) | 타이포·간격·레이아웃·모션의 기준. 이징·지속시간 같은 작은 선택을 틀리지 않게 함 |
| **design-taste-frontend** | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | 참조 모델을 "기본 템플릿"에서 떼어냄. 뻔한 결과가 나오는 걸 막음 |
| **impeccable** | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | 디자인 명령 모음 + **결정론적 탐지 규칙**(`detect`). AI 티 나는 패턴을 LLM 없이 잡아냄 |
| **Playwright MCP** | [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) | 브라우저를 열고 **실제로 클릭·입력**하고 스크린샷을 찍음 |
| **Figma MCP** | `mcp.figma.com/mcp` | Figma 파일에서 컬러·간격 토큰을 읽음 |

---

## 모작 절차에 박아 넣기

PROTOCOL 4절의 각 단계에서 쓸 것.

### 4-1. 원본 확보

- **`scripts/capture.mjs`** — 전체·섹션별 스크린샷과 computed style 실측. 기본 도구다.
- **Playwright MCP** — `capture.mjs`로 못 잡는 것에 쓴다.
  - **기능의 상태별 캡처**: 드롭다운을 실제로 열고 찍기, 탭 2번을 누르고 찍기, 모달을 띄우고 찍기
  - 쿠키 배너·프리로더·게이트를 통과해야 보이는 화면
  - 멀티페이지의 하위 페이지를 돌며 찍기

  > `capture.mjs`는 스크롤만 한다. **클릭이 필요하면 Playwright MCP를 쓴다.**

### 4-2. 원본 실측

- **`scripts/extract-origin.mjs`** — 라이브러리·스크롤 방식·문서 높이·이징·`@keyframes`·셰이더를 잰다.
  해부(4-3)의 수치는 여기서 나온다.

### 4-3. 해부

- **Figma MCP** — 원본의 Figma 파일이 공개돼 있는 경우에만. 대부분은 없으므로 건너뛴다.
  실무 작업에서 디자인 파일을 받았을 때가 주 용도다.
- 수치는 `capture.mjs`의 `measure.json`과 4-2의 `origin.json`이 이미 뽑아준다. 추측하지 않는다.

### 4-4. 재현

- **emil-design-eng** — 원본 수치를 옮길 때 판단이 필요한 지점에 쓴다.
  이징 방향(들어올 때 `ease-out`, 나갈 때 `ease-in`), 지속시간, 그림자 대 보더 같은 선택.
- **design-taste-frontend** — 원본에 없는 부분을 메울 때. 더미 이미지·플레이스홀더를
  뻔하게 만들지 않기 위해.

  > **주의**: 모작은 창작이 아니다. 이 두 스킬이 원본과 다른 방향을 제안하면
  > **원본을 따른다.** 스킬은 원본이 침묵하는 자리에서만 쓴다.

### 4-5. 기능 구현

- **Playwright MCP** — 만든 기능을 **실제로 눌러서 확인한다.**
  - 드롭다운 열고 → 바깥 클릭 → 닫히는지
  - `Tab`으로 이동, `Enter`로 열림, `ESC`로 닫힘
  - 탭 클릭 후 내용이 바뀌는지, `aria-selected`가 따라가는지
  - 폼에 입력이 되는지

  CHECKLIST의 기능 항목은 **눈으로 본 게 아니라 눌러본 결과로** 체크한다.

### 4-6. 대조

- **Playwright MCP** — 원본과 재현물을 같은 조건(뷰포트·스크롤 위치·상태)에서 찍어 비교.
- **`record-motion.mjs` → `motion-compare.mjs`** — 모션 대조. `extract-origin.mjs` — 실측 대조.
- **커밋 전 게이트 3종** — `verify-site.mjs` · `impeccable detect` · `check-korean.mjs`(한국어 화면).

  ```bash
  npx --yes impeccable@4.7.0 detect works/<주차>/<폴더>/*.html
  ```

  규칙이 LLM 없이 돈다. AI가 자주 흘리는 흔적(모든 것에 Inter, 보라-파랑 그라디언트,
  카드 안의 카드, 색 배경 위 회색 글자, 제목 위 둥근 사각 아이콘 타일)을 잡는다.
  **모작에서 이게 잡히면 원본이 아니라 내가 만든 흔적이다.** 고친다.

- **`/impeccable audit <대상>`** — 접근성·성능·반응형 기술 점검.

### 4-7. 분석

- **`/impeccable critique <대상>`** — 위계·명료성 관점의 리뷰. `REVIEW.md` 초안에 쓴다.
  다만 **최종 판단은 사람이 한다.** 스킬 출력을 그대로 붙여넣지 않는다.

---

## 창작 절차에 박아 넣기

PROTOCOL 4-8절. 창작은 모작에서 실측한 **조각을 골라 맞추는** 일이다. 조각의 값은 SPEC 에서 옮기고, 도구는 조각이 침묵하는 자리에서만 쓴다. 기획서가 먼저다.

| 단계 | 쓸 것 | 주의 |
|---|---|---|
| 조각 고르기 | `research/notes/parts.md` · `works/**/SPEC.md` · `REVIEW.md` | 값은 SPEC 의 실측값만. 스킬로 조각을 만들어 내지 않는다 |
| 기획서 | 사람 · 조각 표 | 스킬로 기획서를 쓰지 않는다. 소재와 방향은 사람이 정한다 |
| 조각 맞추기(통일) | **emil-design-eng** | 조각 사이를 잇는 간격·이징의 작은 선택. 조각의 실측값을 바꾸지 않는다 |
| 뻔한 결과 막기 | **design-taste-frontend** | 조각이 없는 자리(더미 내용·사진 고르기)에만. "지난 창작과 무엇이 다른가" 표와 같이 본다 |
| 사진 | `scripts/fetch-stock.mjs` | 조각을 가져온 모작의 사진 자리와 명도·톤을 맞춘다 |
| 다듬기 | `/quieter` · `/distill` · `/bolder` · `/colorize` 등 | 기획서의 방향을 바꾸는 제안은 버린다 |
| 기능 검증 | **Playwright MCP** | 모작과 같다. 눌러서 확인. AI 상주 기능 10가지도 하나씩 |
| 조각 대조 | **Playwright MCP** · `capture.mjs` | 조각을 가져온 모작과 같은 폭으로 찍어 나란히 본다 |
| 게이트 | `verify-site.mjs` · `impeccable detect` · `check-korean.mjs` | 모작과 같다 |
| 리뷰 | `/impeccable critique` · `audit` | `README.md` 검수 결과의 초안. 최종 판단은 사람 |

`generate` · `craft`처럼 페이지를 통째로 만드는 명령은 쓰지 않는다. 창작의 재료는 모작에서 검증된 조각이다.

---

## 모작에서 쓰지 않는 impeccable 명령

impeccable은 **디자인을 만드는** 도구다. 모작은 만드는 게 아니라 따라가는 것이라
아래는 쓰지 않는다.

`bolder` · `quieter` · `distill` · `delight` · `overdrive` · `colorize` · `animate` ·
`shape` · `craft` · `generate`

이걸 쓰면 원본에서 멀어진다. **모작에서 쓰는 건 `detect` · `audit` · `critique` 셋뿐이다.**

---

## 요약표

| 단계 | 주 도구 | 보조 |
|---|---|---|
| 4-1 원본 확보 | `capture.mjs` · **`record-motion.mjs`** | **Playwright MCP** (상태별·하위페이지) |
| 4-2 원본 실측 | **`extract-origin.mjs`** | — |
| 4-3 해부 | `measure.json` · `origin.json` | Figma MCP (파일 있을 때만) |
| 4-4 재현 | 원본 수치 | emil-design-eng, design-taste-frontend |
| 4-5 기능 | 직접 구현 | **Playwright MCP** (눌러서 검증) |
| 4-6 대조 | `capture.mjs` · **`motion-compare.mjs`** · `extract-origin.mjs` | **Playwright MCP**, 게이트 3종 |
| 4-7 분석 | 사람 | `/impeccable critique` |
| 4-8 창작 | 조각 목록 · 기획서(조각 표) | 위 「창작 절차에 박아 넣기」 |

---

## 원칙

1. **도구가 원본을 이기지 않는다.** 스킬 제안과 원본이 다르면 원본을 따른다. 창작에서는 조각의 실측값이 원본이다.
2. **기능은 눌러서 확인한다.** Playwright MCP 없이 "될 것이다"로 체크하지 않는다.
3. **게이트 3종(`verify-site` · `impeccable detect` · `check-korean`)을 통과 못 하면 커밋하지 않는다.**
4. **스킬 출력을 그대로 붙여넣지 않는다.** 특히 `REVIEW.md`. 그건 내 분석이어야 한다.
