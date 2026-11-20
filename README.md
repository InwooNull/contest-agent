# 공모전 에이전트 (Claude Code용)

분야를 가리지 않고 공모전 준비를 돕는 Claude Code 에이전트입니다.
**Claude 구독(Pro/Max)만으로 동작**하며, 유료 API나 유료 MCP를 쓰지 않습니다.

## 구성

```
contest-agent/
├── CLAUDE.md                     # 에이전트 기본 지침 (6단계 흐름, 규칙)
├── .mcp.json                     # 무료 MCP: Playwright(웹), Notion(일정, 선택), hwp-mcp(한글 문서)
├── skills-lock.json              # 외부 스킬(Remotion) 설치 기록
├── .claude/
│   ├── settings.json             # Anthropic 공식 스킬 플러그인 자동 등록
│   ├── skills/
│   │   ├── contest-scout/        # 1. 공모전 탐색·추천
│   │   ├── rubric-parser/        # 2. 요강 분석 → rubric.json
│   │   ├── idea-generator/       # 3. 심사기준 역산형 아이디어 발상
│   │   └── proposal-writer/      # 4. 기획서·대본 등 초안 작성
│   └── agents/
│       ├── researcher.md         # 배경자료·수상작 조사 (출처 포함)
│       ├── judge.md              # 심사위원 1: Claude (읽기 전용, 배점대로 채점)
│       └── judge-codex.md        # 심사위원 2: OpenAI Codex (교차 채점, 선택)
└── contests/                     # 공모전별 작업 폴더가 여기 생김 (git에는 안 올라감)
```

공모전 폴더 하나의 구조:

```
contests/2026-관광데이터/
├── 00_진행상황.md        # 현재 단계, 내 결정, 다음 할 일
├── 00_공고/              # 공고문, 요강 PDF, 첨부 양식
├── rubric.json           # 요강 분석 결과
├── 01_리서치.md          # 배경자료, 지난 수상작 (출처 포함)
├── 02_아이디어.md        # 아이디어 후보와 선택
├── 03_초안.md            # 기획서/산출물 초안 (03_초안_v2.md …)
├── 04_심사결과.md        # judge 채점 + 교차 채점 요약
├── 04_심사결과_codex.md  # judge-codex 채점
├── 05_제출체크리스트.md
└── 06_영상/              # 영상 제출물이 있을 때만
```

## 연결되는 스킬과 MCP 한눈에 보기

| 종류 | 이름 | 하는 일 | 설치 |
|---|---|---|---|
| 직접 만든 스킬 | contest-scout, rubric-parser, idea-generator, proposal-writer | 공모전 6단계 | 폴더에 포함 |
| 서브에이전트 | researcher, judge | 조사, 모의 심사 | 폴더에 포함 |
| 서브에이전트 | judge-codex | 다른 회사 모델(Codex)로 교차 채점 | 선택 (Codex CLI 필요, 아래 참고) |
| 외부 스킬 | Remotion (remotion-dev/skills) | 코드 기반 영상 제작 | 선택 (`npx skills add remotion-dev/skills`) |
| 공식 스킬 | **document-skills** (pdf, docx, pptx, xlsx) | PDF 요강 읽기, Word·PPT·Excel 제출물 만들기 | 자동 (settings.json) |
| 공식 스킬 | **example-skills** (canvas-design, frontend-design, skill-creator 등) | 포스터·카드뉴스, 웹 데모, 스킬 개선 | 자동 (settings.json) |
| MCP | Playwright | JS로 그려지는 공모전 사이트 읽기 | 자동 (Node.js만 필요) |
| MCP | Notion | 일정·체크리스트 관리 | 선택 (`/mcp`에서 로그인) |
| MCP | **hwp-mcp** | 한글(HWP·HWPX) 요강 읽기, 표 추출, HWPX 양식 채우기·렌더링 확인 | 자동 (Node.js만 필요) |

공식 스킬(Anthropic `anthropics/skills`)은 라이선스상 폴더에 복사하지 않고, `settings.json`에서 플러그인으로 연결해 두었습니다. 처음 폴더를 신뢰(trust)하면 설치를 제안합니다.

## 설치 (처음 한 번)

1. **Node.js 설치** (Playwright MCP용): https://nodejs.org 에서 LTS 버전
2. 이 폴더를 원하는 위치에 압축 해제
3. 폴더를 Claude Code로 열기
   - 터미널: `cd contest-agent` → `claude`
   - 데스크톱 앱: Code 탭에서 이 폴더 선택
   - "이 폴더를 신뢰하시겠습니까?"에서 신뢰 → 공식 스킬 플러그인 설치 제안이 뜨면 설치
   - 제안이 안 뜨면 직접 입력:
     ```
     /plugin marketplace add anthropics/skills
     /plugin install document-skills@anthropic-agent-skills
     /plugin install example-skills@anthropic-agent-skills
     ```
4. **한글 문서 지원 (hwp-mcp)** – 공모전 요강·양식은 HWP/HWPX인 경우가 많습니다.
   - `.mcp.json`에 `hwp`(`npx -y hwp-mcp@latest`)로 등록되어 있어 따로 설치할 것이 없습니다. Node.js만 있으면 됩니다.
   - 한글 프로그램이 없어도 `.hwp`와 `.hwpx`를 둘 다 읽고, 심사기준·배점 표를 따로 추출합니다.
   - 지정 양식은 HWPX 복사본에 채우고(원본은 보존) 렌더링으로 페이지를 확인합니다. 양식이 구형 `.hwp`면 공모전 페이지에서 HWPX 양식을 먼저 찾고, 없으면 같은 목차로 새 HWPX를 만듭니다.
   - 처음 실행 때 패키지를 내려받느라 연결이 늦을 수 있습니다. `/mcp`에서 hwp가 실패로 보이면 재연결해 보세요.
5. **(선택) Notion** – Claude Code에서 `/mcp` 입력 → notion 선택 → 브라우저에서 로그인
   (안 쓰면 `.mcp.json`에서 notion 부분을 지워도 됩니다)
6. 확인: `/mcp`로 연결된 MCP, `/plugin`으로 설치된 플러그인 목록을 볼 수 있습니다.

## 사용 예시

그냥 말로 시키면 됩니다. 스킬과 서브에이전트는 상황에 맞게 자동으로 호출됩니다.

```
# 1. 탐색
IT나 아이디어 분야로 12월 안에 마감하는 대학생 공모전 찾아줘. 개인으로 할 거야.

# 2. 요강 분석
이 공모전 시작하자: https://www.wevity.com/...

# 3. 리서치 + 아이디어
researcher로 배경 조사하고 아이디어 후보 4개 뽑아줘.

# 4~5. 초안 + 모의 심사
2번 아이디어로 기획서 초안 써주고 judge랑 judge-codex한테 채점받아서 고쳐줘.

# 6. 제출 관리
제출 체크리스트랑 마감 역산 일정 만들어줘.
```

스킬을 직접 부르고 싶으면 `/rubric-parser`처럼 슬래시로 호출할 수도 있습니다.

## 사용량 아끼는 팁

- 서브에이전트는 한 번에 하나씩 (CLAUDE.md에 규칙으로 들어 있음)
- 작성→심사→수정 루프는 최대 2회
- 공고 PDF를 직접 `contests/<폴더>/00_공고/`에 넣어두면 웹 수집 비용이 줄어듭니다
- 한도가 자주 걸리면 judge·researcher의 `model:`을 `haiku`로 바꿔보세요

## 고쳐 쓰기

- 에이전트 성격·규칙 → `CLAUDE.md`
- 단계별 방법 → `.claude/skills/*/SKILL.md`
- 심사위원·리서처 → `.claude/agents/*.md`
- 분야 전문가 추가 (예: 디자인 공모전) → `.claude/agents/`에 새 md 파일 추가

## 추가 기능

### 교차 심사 (judge-codex)
Claude가 쓴 초안을 Claude만 채점하면 점수가 후해질 수 있어서, OpenAI Codex에게도 같은 기준으로 따로 채점을 맡깁니다.
- 필요한 것: [Codex CLI](https://github.com/openai/codex) 설치 후 `codex login` (ChatGPT 구독 계정으로 로그인 가능)
- 흐름: judge 채점 → judge-codex 채점 → 두 결과를 `04_심사결과.md` 맨 위에 요약 (두 심사위원이 같이 지적한 약점부터 고침)
- Codex가 없으면 judge-codex는 안내 한 줄만 남기고, judge 결과만으로 진행합니다.

### 영상 공모전 (Remotion)
홍보영상·UCC·데모영상 제출물이 있으면 Remotion으로 코드 기반 영상을 만듭니다.
- 설치 (contest-agent 폴더에서 한 번): `npx skills add remotion-dev/skills`
- 흐름: 규격 확인 → 스토리보드(사용자 확인) → 제작 → Studio 미리보기 → mp4 렌더링 → 출처 기록
- 개인·소규모 팀은 무료로 알려져 있지만, 처음 쓸 때 `/remotion-license`로 조건을 확인하세요.
- 공모전이 AI 생성 영상을 금지하면 쓰지 않습니다.

### 진행상황과 공모전 노트 (가벼운 기억)
- 공모전 폴더마다 `00_진행상황.md`: 현재 단계, 내 결정, 다음 할 일. 에이전트가 단계마다 갱신합니다.
- `contests/_나의_공모전_노트.md`: 공모전이 끝날 때마다 결과와 반복된 약점을 쌓고, 새 공모전에서 먼저 읽습니다.
- 별도 프로그램 없이 파일만 쓰므로 추가 사용량이 거의 없습니다.
