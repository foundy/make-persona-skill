[English](README.md) | [한국어](README.ko.md)

# make-persona-skill

자기 자신이 쓴 텍스트 말뭉치(Obsidian 노트, JIRA 코멘트, PR 리뷰, Slack 메시지)로부터 **개인 작성 스타일 스킬**을 생성하는 Claude Code 메타 스킬입니다.

생성된 스킬은 JIRA 코멘트, PR 리뷰, Slack 답변, 이메일 등 업무용 텍스트를 **당신의 말투**로 초안 작성해서 복붙할 수 있게 해줍니다. Claude의 전체 페르소나를 바꾸는 것이 아니라, 필요할 때 호출하는 대필 도구입니다.

## 왜 필요한가

업무 커뮤니케이션에서 톤은 중요합니다. 같은 "LGTM"이라도 시니어 엔지니어가 쓴 것과 주니어가 쓴 것은 뉘앙스가 다릅니다. 대부분의 AI 초안은 AI 티가 납니다. 이 스킬은 *당신의* 어휘, 문장 구조, 레지스터를 학습해서 초안이 당신이 쓴 것처럼 읽히게 만듭니다.

## 동작 원리

2-tier 구조:

1. **`make-persona-skill`** (이 저장소) — 메타 스킬. 1회만 설치.
2. **`persona-{닉네임}`** — 사용자별 생성. `~/.claude/skills/persona-{닉네임}/` 에 위치.

파이프라인:

```
corpus  →  patterns.md  →  style-card.md  →  blind test  →  SKILL.md
(필터링)   (분석 누적)     (20줄 증류)         (검증)         (자동 생성)
```

- `patterns.md` — 전체 분석 결과 (크기 제한 없음)
- `style-card.md` — 초안 작성 시 주입되는 20줄 요약
- Blind test 루프 — 재생성 결과가 원본 톤과 맞는지 검증

## 설치

### 방법 1 — Plugin Marketplace (권장)

```
/plugin marketplace add foundy/make-persona-skill
/plugin install make-persona-skill
```

### 방법 2 — 로컬 경로 (개발용)

```
/plugin marketplace add /path/to/cloned/make-persona-skill
/plugin install make-persona-skill
```

### 방법 3 — 수동 clone

```bash
git clone https://github.com/foundy/make-persona-skill.git /tmp/mps
cp -r /tmp/mps/skills/make-persona-skill ~/.claude/skills/
rm -rf /tmp/mps
```

## 사용법

### 1회 — 페르소나 스킬 생성

아무 Claude Code 세션에서:

```
make-persona-skill 실행해줘.
 - 닉네임: foundy
 - 데이터 소스: ~/Documents/Obsidian/vault
```

Claude 가 샘플을 수집하고, 본인이 직접 쓴 글만 필터링하고, 패턴을 추출하고, style card 로 증류하고, blind test 를 실행한 뒤 `~/.claude/skills/persona-foundy/` 에 스킬을 배치합니다.

### 생성된 스킬 사용

생성 직후부터 `/persona-foundy` 슬래시 커맨드가 바로 활성화됩니다:

```
/persona-foundy 이 JIRA 티켓에 대한 코멘트 초안 만들어줘: <내용 붙여넣기>
```

자연어 트리거도 동작합니다:

```
이 Slack 답변을 foundy 스타일로 다시 써줘: ...
```

### 새 데이터로 업데이트

```
persona-foundy 업데이트해줘. corpus/github-pr.md 에 새 PR 리뷰 추가했어.
```

메타 스킬이 패턴 추출을 재실행하고, 기존 패턴과 병합하고, style card 를 다시 증류하고, blind test 를 다시 실행합니다.

## 저장소 구조

```
make-persona-skill/
├── .claude-plugin/
│   └── marketplace.json
├── skills/
│   └── make-persona-skill/
│       ├── SKILL.md
│       └── templates/
│           ├── persona-skill.template.md
│           ├── patterns.template.md
│           └── style-card.template.md
├── docs/
│   └── architecture.md
├── README.md
├── README.ko.md
├── LICENSE
└── .gitignore
```

## 데이터 보안

- `persona-*/` 디렉토리는 **생성 결과물** 이며 로컬 머신(`~/.claude/skills/`) 에만 존재합니다.
- `persona-*/corpus/` 는 당신의 원본 텍스트입니다. 공유하지 않는 이상 외부로 나가지 않습니다.
- `.gitignore` 가 `persona-*/` 와 `corpus/` 를 제외하므로 실수로 커밋될 위험이 없습니다.
- 공용 머신에서 메타 스킬을 실행하기 전에 `corpus/` 를 검토하세요.

## 설계 원칙

1. 메타 스킬(생성기)과 생성된 스킬(소비자) 분리
2. 분석 산출물(`patterns.md`) 과 주입 산출물(`style-card.md`) 분리
3. 사용자 **본인이** 1인칭으로 쓴 글만 필터링
4. 자기 채점이 아닌 blind test 로 검증
5. Claude Code 2026 에서 Skills = Slash Commands — 파일 하나로 양쪽 트리거 확보
6. corpus 를 persona 디렉토리 내부에 둬서 업데이트 편의성 우선

## 기여

이슈와 PR 환영합니다. 메타 스킬은 작고 조합 가능하게 유지해 주세요.

## 라이선스

[MIT](LICENSE)
