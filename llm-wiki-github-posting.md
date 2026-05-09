# Local LLM Wiki 설치 및 운영 정리

> 작성일: 2026-05-09  
> 목적: GitHub 게시용으로 local LLM Wiki 설치 방식, 구성, 운영 흐름, Lesson Learned를 한 문서로 정리

## 1. 개요

이 문서는 local 환경에 구축한 **LLM Wiki** 설치 및 운영 방식을 정리한 기록입니다.

LLM Wiki는 Andrej Karpathy가 제안한 "LLM Wiki" 패턴을 참고하여, LLM/AI 관련 지식과 프로젝트 메모를 단발성 질의응답으로 흩어지지 않게 **지속적으로 축적되는 Markdown 기반 지식베이스**로 관리하기 위한 구조입니다.

핵심 방향은 다음과 같습니다.

- 원본 자료는 `raw/` 아래에 보존
- 정리된 지식은 `concepts/`, `queries/`, `summaries/`, `comparisons/`, `entities/` 등에 저장
- 모든 문서는 Obsidian에서 열 수 있는 Markdown 형식으로 관리
- `index.md`로 전체 문서를 탐색
- `log.md`로 작업 이력을 추적
- NotebookLM, Obsidian, CLI workflow와 연결 가능한 local-first 구조 유지

## 2. 설치 위치

현재 환경의 주요 경로는 다음과 같습니다.

- LLM Wiki 실제 위치: `/home/ai/shared/obsidian/llm-wiki`
- 미러 복사본: `/home/ai/wiki`
- Obsidian vault: `/home/ai/shared/obsidian`
- NotebookLM workflow repo: `/home/ai/notebooklm-llm-wiki-flow`
- NotebookLM home: `/home/ai/.notebooklm`

## 3. 기본 디렉터리 구조

LLM Wiki는 별도 데이터베이스 없이 Markdown 파일과 폴더 구조만으로 운영합니다.

```text
llm-wiki/
├── SCHEMA.md
├── index.md
├── log.md
├── raw/
│   ├── articles/
│   ├── papers/
│   ├── transcripts/
│   ├── assets/
│   ├── datasets/
│   ├── books/
│   ├── web-clips/
│   └── notes/
├── concepts/
├── summaries/
├── comparisons/
├── queries/
└── entities/
```

각 파일/폴더의 역할은 다음과 같습니다.

- `SCHEMA.md`: 위키의 도메인, 문서 규칙, 태그 체계, 워크플로우 정의
- `index.md`: 전체 문서 카탈로그
- `log.md`: append-only 작업 이력
- `raw/`: 원본 자료 보관 영역. 원칙적으로 수정하지 않음
- `concepts/`: 개념 정리 문서
- `summaries/`: 원본 또는 작업 결과 요약
- `queries/`: 질의응답 중 보존 가치가 있는 답변
- `comparisons/`: 비교 분석 문서
- `entities/`: 사람, 조직, 제품, 모델 등 엔티티 문서

## 4. 설치 및 초기화 과정

초기 설치는 다음 흐름으로 진행했습니다.

### 4.1 Wiki root 선택

Obsidian과 함께 쓰기 위해 shared vault 아래에 LLM Wiki를 배치했습니다.

```bash
/home/ai/shared/obsidian/llm-wiki
```

필요 시 로컬 기본 복사본으로 다음 경로도 사용합니다.

```bash
/home/ai/wiki
```

### 4.2 Backbone 파일 생성

다음 세 파일을 먼저 생성했습니다.

- `SCHEMA.md`
- `index.md`
- `log.md`

`SCHEMA.md`에는 아래 내용을 정의했습니다.

- 도메인: LLM / AI research notes, source summaries, concepts, comparisons, project memory
- 파일명 규칙: lowercase, hyphen-separated, no spaces
- frontmatter 규칙
- tag taxonomy
- page threshold
- ingest / query / lint workflow

### 4.3 Raw source 폴더 생성

원본 보관을 위해 다음 폴더를 만들었습니다.

```text
raw/articles/
raw/papers/
raw/transcripts/
raw/assets/
raw/datasets/
raw/books/
raw/web-clips/
raw/notes/
```

원본 자료는 가능하면 이 영역에 먼저 저장하고, 실제 지식 정리는 별도 wiki layer 문서에서 수행합니다.

### 4.4 Karpathy LLM Wiki source ingest

초기 seed source로 Karpathy의 LLM Wiki gist를 캡처했습니다.

- Source: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Raw capture: `raw/articles/karpathy-llm-wiki-2026.md`
- Summary: `summaries/karpathy-llm-wiki-source.md`
- Concept: `concepts/persistent-wiki-pattern.md`

이후 `index.md`와 `log.md`를 갱신했습니다.

## 5. NotebookLM 연동

LLM Wiki를 NotebookLM 기반 리서치 워크플로우와 연결하기 위해 별도 repo를 사용했습니다.

### 5.1 Repo 위치

```bash
/home/ai/notebooklm-llm-wiki-flow
```

### 5.2 사용한 주요 명령

```bash
git clone https://github.com/reallygood83/notebooklm-llm-wiki-flow.git notebooklm-llm-wiki-flow
cd notebooklm-llm-wiki-flow
./scripts/bootstrap.sh
./.venv/bin/notebooklm login
./.venv/bin/nlwflow init-config
./.venv/bin/nlwflow doctor --json
```

### 5.3 환경 설정

`.env`에는 다음 값들을 설정했습니다.

```bash
NLWFLOW_PROJECT_NAME=llm-wiki-automation
NLWFLOW_OBSIDIAN_VAULT=/home/ai/shared/obsidian
NLWFLOW_WIKI_PATH=/home/ai/shared/obsidian/llm-wiki
NLWFLOW_QMD_COLLECTION=learningmaster
NLWFLOW_ARTIFACTS_ROOT=/home/ai/notebooklm-llm-wiki-flow/artifacts
NLWFLOW_NOTEBOOKLM_COMMAND=/home/ai/notebooklm-llm-wiki-flow/.venv/bin/notebooklm
NLWFLOW_QMD_COMMAND=qmd

WIKI_PATH=/home/ai/shared/obsidian/llm-wiki
OBSIDIAN_VAULT_PATH=/home/ai/shared/obsidian
NOTEBOOKLM_HOME=/home/ai/.notebooklm
```

Shell 환경에도 다음 값을 추가했습니다.

```bash
export WIKI_PATH=/home/ai/shared/obsidian/llm-wiki
export OBSIDIAN_VAULT_PATH=/home/ai/shared/obsidian
export NOTEBOOKLM_HOME=/home/ai/.notebooklm
```

### 5.4 검증

다음 명령으로 설정을 확인했습니다.

```bash
./.venv/bin/nlwflow doctor --json
./.venv/bin/notebooklm auth check
./.venv/bin/notebooklm list
./.venv/bin/notebooklm status
```

NotebookLM CLI 인증은 완료되어 있으며, 실제 workflow 실행 전에는 필요에 따라 notebook을 선택합니다.

```bash
notebooklm use <notebook-id>
```

## 6. 운영 방식

LLM Wiki는 다음 순서로 운영합니다.

### 6.1 새 자료 ingest

1. 원본 자료를 `raw/` 아래에 저장
2. 기존 `index.md`와 관련 문서를 검색
3. 새 개념/요약/질의 문서를 생성하거나 기존 문서를 업데이트
4. `index.md`에 문서 추가
5. `log.md`에 작업 이력 추가

### 6.2 질의 결과 저장

재사용 가치가 있는 답변은 `queries/` 아래에 저장합니다.

예시:

- `queries/how-to-install-llm-wiki.md`
- `queries/llm-wiki-notebooklm-installation-notes.md`
- `queries/notebooklm-cli-integration.md`

### 6.3 Lint / Maintenance

주기적으로 다음을 점검합니다.

- frontmatter 누락 여부
- broken wikilink 여부
- orphan page 여부
- `index.md` completeness
- raw source frontmatter 여부
- 문서 길이 과다 여부
- 태그 taxonomy 위반 여부

## 7. 현재 존재하는 관련 문서

설치와 운영 관련 문서는 다음과 같습니다.

- `queries/how-to-install-llm-wiki.md`  
  LLM Wiki vault와 folder 구조 설치 방법

- `queries/llm-wiki-notebooklm-installation-notes.md`  
  이 환경에서 LLM Wiki와 NotebookLM을 실제로 어떻게 설치했는지 기록

- `summaries/notebooklm-llm-wiki-flow-summary.md`  
  NotebookLM → LLM Wiki → Obsidian → qmd 워크플로우 요약

- `concepts/persistent-wiki-pattern.md`  
  LLM으로 지식을 계속 축적하는 지속형 위키 패턴

- `log.md`  
  설치, ingest, lint, query 등 전체 작업 이력

## 8. Lesson Learned

### 8.1 Markdown 기반 구조가 가장 단순하고 오래 간다

LLM Wiki는 별도 DB나 복잡한 서버 없이 Markdown만으로 운영됩니다. 덕분에 Obsidian, GitHub, CLI, LLM agent가 모두 같은 자료를 쉽게 읽고 수정할 수 있습니다.

### 8.2 `raw/`와 wiki layer를 분리해야 한다

원본 자료와 정리된 지식을 섞으면 나중에 출처 추적이 어려워집니다. 원본은 `raw/`에 보존하고, 요약/해석/비교는 `summaries/`, `concepts/`, `queries/` 등에 분리하는 방식이 좋습니다.

### 8.3 `index.md`와 `log.md`가 위키의 backbone이다

문서가 늘어나면 검색만으로는 전체 구조를 파악하기 어렵습니다. `index.md`는 탐색용 지도이고, `log.md`는 작업 히스토리입니다. 두 파일을 꾸준히 갱신해야 위키가 지식베이스로 유지됩니다.

### 8.4 질의응답도 보존 가치가 있으면 문서화해야 한다

LLM에게 한 번 물어보고 끝내면 같은 질문을 반복하게 됩니다. 유용한 답변은 `queries/` 아래에 저장하면 이후 다시 검색하고 개선할 수 있습니다.

### 8.5 Obsidian과 연결하면 사람이 읽고 고치기 쉽다

LLM agent가 문서를 만들고, 사람은 Obsidian에서 빠르게 읽고 수정할 수 있습니다. `[[wikilinks]]`를 사용하면 개념 간 연결도 자연스럽게 쌓입니다.

### 8.6 NotebookLM은 drafting/source gathering layer로 두는 것이 좋다

NotebookLM은 자료 기반 초안 생성이나 source gathering에 유용하지만, 장기 보존 지식은 LLM Wiki에 남기는 방식이 더 안정적입니다.

### 8.7 설치 노트 자체도 wiki에 남겨야 한다

설치 후 시간이 지나면 어떤 명령을 실행했는지, 어떤 환경 변수를 썼는지 잊기 쉽습니다. 이번 구성에서는 `queries/llm-wiki-notebooklm-installation-notes.md`에 실제 설치 방법을 남겨 재현 가능성을 확보했습니다.

## 9. 참고 링크

- Karpathy LLM Wiki gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- notebooklm-llm-wiki-flow: https://github.com/reallygood83/notebooklm-llm-wiki-flow

## 10. 요약

이 local LLM Wiki 구성은 다음 목적에 적합합니다.

- LLM/AI 리서치 메모의 지속적 축적
- NotebookLM 결과물의 장기 보존
- Obsidian 기반 개인 지식관리
- GitHub에 게시 가능한 Markdown 문서화
- Agent가 읽고 업데이트할 수 있는 local-first knowledge base

핵심은 단순합니다.

> 원본은 `raw/`에 보존하고, 정리된 지식은 Markdown wiki로 축적하며, 모든 변경은 `index.md`와 `log.md`에 남긴다.
