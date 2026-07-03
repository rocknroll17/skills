---
name: new-paper
description: '비어있는 현재 디렉토리를 논문 학습 환경으로 부트스트랩한다. PDF 전체 + 필요한 외부 배경지식을 조사하고, 독자 수준을 calibration한 뒤 notes 7개와 glossary를 서사형으로 채운다. 결과물은 원문 대신 이 파일들만 읽어도 깊이 이해되는 — 설명·응용이 자연히 따라올 만큼 — 상태를 목표로 한다. Example: /paper-study:new-paper <arxiv_id | url> 또는 /paper-study:new-paper ./mypaper.pdf --shallow'
argument-hint: '<arxiv_id | pdf_path | url> [--shallow]'
disable-model-invocation: true
allowed-tools: Read, Write, Edit, WebSearch, WebFetch, AskUserQuestion
---

# 논문 학습 환경 부트스트랩 + 전체 분석

**인자**: `$ARGUMENTS`

현재 디렉토리를 학습 환경으로 초기화하고, PDF 전체 + 필요한 외부 배경지식을 조사해 섹션별 **서사형 분석**과 **3층 용어집**을 생성한다. **목표·설명 원칙·문체는 생성되는 `CLAUDE.md`가 단일 소스다.** 이 문서는 *절차*만 규정한다.

---

## 인자 파싱

| 입력 형태 | 처리 |
| --- | --- |
| arXiv ID (`XXXX.XXXXX`, `XXXX.XXXXXvN`) | `curl -L -o pdf/paper.pdf "https://arxiv.org/pdf/<id>"` |
| arXiv URL (`https://arxiv.org/abs/...`) | URL에서 ID 추출 후 위와 동일 |
| 로컬 PDF 경로 (`./file.pdf`) | `pdf/paper.pdf`로 `cp` |
| 일반 PDF URL | `curl -L -o pdf/paper.pdf "<url>"` |
| DOI (`10.xxxx/xxxx`) | WebSearch로 공개 PDF URL 탐색 → 실패 시 재확인 |

### 옵션 플래그

- `--shallow` : Phase B(전체 분석)를 **생략**한다. overview만 채우고 종료 — 컨텍스트 절약용.

---

## Phase A — 부트스트랩

### 1) 기본 디렉토리
`mkdir -p pdf notes glossary figures`

### 2) scaffold 복사 (번들 템플릿)
템플릿은 이 skill에 번들된 `templates/`에 단일 소스로 보관된다. 현재 디렉토리로 복사한다:
```bash
cp -rn "${CLAUDE_SKILL_DIR}/templates/." .
```
`CLAUDE.md`, `notes/00-overview.md`~`06-limitations.md`(7개), `glossary/terms.md`, `pdf/`가 채워진다. `-n`이므로 기존 파일은 보존된다. 충돌 시 사용자에게 확인.

### 3) PDF 확보
실패 시 사용자에게 URL·경로 재확인을 요청한다. 허위 데이터 생성 금지.

### 4) 메타데이터 추출
```bash
pdfinfo pdf/paper.pdf 2>/dev/null | grep -E "Title|Author|Pages"
```
arXiv ID가 있으면 WebFetch로 `https://arxiv.org/abs/<id>`에서 abstract·venue를 확보한다.

### 5) CLAUDE.md §1 슬롯 치환 (Edit)
`{{TITLE}}`, `{{AUTHORS}}`, `{{VENUE}}`, `{{ARXIV_ID}}`, `{{HOMEPAGE}}`, `{{ONE_LINER}}`.

### 6) notes/00-overview.md 작성 (서사형)
PDF 첫 2~4페이지만 읽고:
- What / Why / How 각 1~2문장 (말하듯이)
- 주요 기여 3~4개
- 핵심을 한 줄 메시지로 압축 ("이 논문 = …")

`--shallow`면 여기서 종료하고 Phase C로 이동한다.

---

## Phase A-bis — Figure 추출 (백그라운드)

Phase B 본문에 이미지를 삽입할 수 있도록 `figures/`로 추출한다. **추출의 셸 단계(다운로드·압축 해제·`pdfimages`)는 `run_in_background`로 실행한다.** 그동안 Phase A-cal(독자 수준 파악)을 수행하고, Phase B 진입 직전에 완료를 회수한다.

### 경로 1: arXiv HTML (우선)
arXiv ID가 있으면:
1. WebFetch `https://arxiv.org/html/<id>` (버전 없이) 또는 `/html/<id>v1` fallback.
2. 프롬프트:
   > "Return a JSON array of every figure on the page. Keys: `number` (int), `caption` (first 300 chars), `src` (absolute URL; if relative, prepend page URL directory). Skip equations, logos."
3. 각 figure를 curl로 `figures/fig{N}.png`에 저장한다.
4. `figures/CAPTIONS.md` 작성 — 번호·파일·캡션 매핑표.

### 경로 2: arXiv TeX source (HTML 실패 시)
```bash
mkdir -p /tmp/paper-src-$$ && cd /tmp/paper-src-$$
curl -sL "https://arxiv.org/e-print/<id>" -o src.tar.gz
tar xzf src.tar.gz 2>/dev/null || gunzip -c src.tar.gz > main.tex
find . -type f \( -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.pdf' -o -iname '*.eps' \) \
  -not -name 'paper.pdf' -exec cp {} <CWD>/figures/ \;
```
Caption은 메인 `.tex`의 `\begin{figure}` 블록에서 `\includegraphics{}` + `\caption{}`를 가능한 범위에서 페어링한다. 종료 후 `rm -rf /tmp/paper-src-$$`.

### 경로 3: pdfimages (최후 수단)
```bash
pdfimages -png pdf/paper.pdf figures/img
```
Caption 매핑은 불가하다 — `figures/CAPTIONS.md`에 "Raw PDF extraction"으로 명시한다.

Phase B에서 figure 삽입 시 `![Figure N](../figures/figN.png)` 형식을 쓴다.

---

## Phase A-cal — 독자 수준 파악 (calibration)

Phase A-bis 추출이 백그라운드로 진행되는 동안 독자의 배경을 파악해, Phase B의 개념별 설명 깊이를 결정한다. `--shallow`면 1)만 수행한다.

### 1) 개괄 질문
`AskUserQuestion`으로 독자의 **분야 친숙도**를 묻는다 — 이 논문 분야·인접 분야의 전반적 숙련도, 형식 도구(수식·통계·코드)에 대한 익숙함. 2~3문항, 개괄 수준에서.

### 2) 개념 인벤토리 + 의존성 정렬 + 수준 추정
overview(Phase A 6단계)에서 읽은 내용과 1)의 답을 근거로, 이 논문을 이해하는 데 필요한 **핵심 개념·선행 지식**을 열거한다.

**의존성 정렬** (경량 — 전체 그래프 금지):
- 각 개념에 그것을 이해하기 위한 선행 개념을 0~2개만 기록한다.
- 그 의존 관계로 위상정렬(topological sort)한다 — 선행 개념이 후행 개념의 구성 요소가 되도록. 순환이 발견되면 한쪽 의존을 끊고 명시한다.
- 이 순서가 곧 설명·glossary 작성 순서다. 논문 서술 순서와 달라도 이해 순서를 따른다.

**수준 추정**: 정렬된 각 항목을 안다/부분적/모른다로 추정한다 (깊이 정의는 `CLAUDE.md §2`).

### 3) 불확실한 항목만 추가 질문
추정이 불확실하거나 설명 깊이를 크게 가르는 주요 개념·하위 분야에 한해 `AskUserQuestion`으로 친숙도를 확인한다.
- 전체 **최대 5문항**. `AskUserQuestion`은 호출당 4문항이 상한이므로, 5개가 필요하면 두 차례로 나눈다.
- 1)·2)에서 충분히 추정된 항목은 다시 묻지 않는다.

### 4) 프로필 기록
확정된 수준을 `CLAUDE.md §2`에 기록한다 (Edit). `{{READER_FIELD_LEVEL}}`·`{{READER_FORMAL_LEVEL}}`를 치환하고, `{{READER_CONCEPT_MAP}}` 자리에 **2)의 위상정렬 순서 그대로** 개념별 안다/부분적/모른다 목록을 작성한다 (각 항목에 선행 개념 한 줄 병기). 이후 모든 Phase·슬래시 커맨드가 이 순서와 깊이를 기준으로 삼는다.

### 5) 회수
Phase B 진입 전, figure 백그라운드 작업의 완료를 확인하고 `figures/CAPTIONS.md`를 점검한다.

---

## Phase B — 전체 분석 (핵심 단계)

`--shallow`가 아닐 때만 실행한다. 산출물 품질이 결정되는 단계다.

### 읽기 전략
- 총 페이지 수를 파악한 뒤 5~6페이지 단위로 분할 Read.
- 섹션의 논리 단위로 분할한다.
- 재독 금지.

### 섹션별 작성 절차 (모든 섹션 공통)
각 섹션을 해당 `notes/0X-*.md`에 작성한다. 다음 **9단계** 순서를 따른다.

#### 단계 1 — 본문 읽기
해당 섹션 PDF 페이지를 Read.

#### 단계 2 — 외부 개념 식별
섹션에 처음 등장하는 다음을 모두 식별한다 (분야 무관):
- 정의 없이 쓰인 약어·이니셜리즘
- 다른 연구에서 차용한 선행 기법·모델·프레임워크 이름
- 수식 도구·통계 기법
- 해당 분야 전문 용어 (비전문가가 모를 법한 것)

#### 단계 3 — 외부 배경 조사
각 식별된 개념에 대해:
- 이미 `glossary/terms.md`에 있거나 `CLAUDE.md §2`에서 '안다'면 생략한다.
- 없으면 WebSearch 한 번으로 작동 원리를 확보하고, `/paper-study:glossary` 포맷(3층 + 작동 원리)으로 추가한다. 깊이는 프로필을 따른다.

#### 단계 4 — 섹션 5블록 작성 (인과 체인)
섹션을 사람이 이해하는 순서로 재구성한다 (논문 서술 순서와 무관). 문체는 `CLAUDE.md §3·§5`를 준수한다.

- **What-problem**: 이 섹션이 푸는 문제 1문장
- **Why-hard**: 왜 어려운가 2~4문장 (필요 시 외부 배경을 끌어온다)
- **Key-idea**: 핵심 아이디어 한 문장 + 구조가 복잡할 때 ASCII 다이어그램. 비유는 구조적 동형이 있을 때만.
- **How-works**: 메커니즘 — 말하듯 2~4문단, 인과 접속을 적극 활용. 수식 등장 시 5블록 축약 해설.
- **So-what**: 결과 + 다른 섹션 cross-reference 1회 이상

#### 단계 5 — 수식 풀이
주요 수식(1~3개)은 해당 섹션 안에 `/paper-study:explain-equation` 5블록 **축약본**으로 삽입한다:
- 한 줄 직관
- 기호별 표
- 짧은 ASCII 흐름도
- 반사실 1문장 ("이 항이 없으면 …")
- 숫자 대입 예시 1줄

#### 단계 6 — Figure 삽입
`figures/CAPTIONS.md`에 해당 Figure가 있으면 본문에 삽입한다:
```markdown
![Figure N — 한 줄 설명](../figures/figN.png)
```

#### 단계 7 — Stuck-point scan
초심자가 이 섹션에서 막힐 지점 2개를 식별하고 각 지점에 1~2문장 해설을 삽입한다.

#### 단계 8 — Failure-case sidebar
기법 설명 뒤에 "이 방법이 깨지는 경우" 1~2개를 구체적 예시로 명시한다.

#### 단계 9 — Self-check 종료
섹션 말미에:
> "(한 문장으로 요약하면? 자기 말로 답해본 뒤 §X.Y 확인.)"

### 섹션 → 파일 매핑

| 논문 섹션 | 쓸 파일 | 분량 기준 |
| --- | --- | --- |
| Abstract + Introduction | `notes/01-abstract-intro.md` | 100~200줄 |
| Related Work | `notes/02-related-work.md` | 100~180줄 (기술 계보 포함) |
| Method (수식 포함) | `notes/03-method.md` | 200~350줄 |
| Dataset / Experimental Setup | `notes/04-dataset.md` | 100~220줄 (없으면 skeleton 유지) |
| Experiments / Ablations | `notes/05-experiments.md` | 150~300줄 |
| Limitations / Conclusion | `notes/06-limitations.md` | 60~120줄 |

### 주의
- **Hallucination 금지**. 논문에 없는 수치·결과·수식은 창작하지 않는다. 외부 배경은 `(논문 밖, WebSearch)`로 표기한다.
- **섹션마다 glossary 갱신**. 섹션 끝에 "이번 섹션에 추가된 용어: …" 한 줄.
- **Cross-section synthesis 의무**. 각 섹션의 So-what은 다른 섹션을 최소 1회 인용한다.
- **장문(>30p)**이면 사용자에게 분할 진행 여부를 확인한다.
- **컨텍스트 예산**: 18p 기준 약 80~120k 토큰. 100p+면 `--shallow`를 권장한다.

---

## Phase C — 완료 보고

```
부트스트랩 + 분석 완료.

제목: <제목>
저자: <저자>
게재: <venue, year>
페이지: <N>p
figures: <M>개 ./figures/에 저장
독자 프로필: <분야 친숙도> / <형식 도구> (CLAUDE.md §2)

분석 상태:
- notes/00-overview.md        ✅ (전체 지도)
- notes/01-abstract-intro.md  ✅
- notes/02-related-work.md    ✅
- notes/03-method.md          ✅ (수식 N개 포함)
- notes/04-dataset.md         ✅ / skeleton (해당 없음)
- notes/05-experiments.md     ✅
- notes/06-limitations.md     ✅
- glossary/terms.md           ✅ (N개 용어, 3층 구조)

한 줄: <overview 한 줄>

읽기 순서 추천: 00 → 01 → 03 → 04 → 05 → 06. 모르는 용어는 glossary 참조.
수식을 더 깊이 보려면 /paper-study:explain-equation Eq. N.
```

`--shallow`면 "분석 상태"에서 overview만 체크하고 나머지는 "skeleton"으로 표기한다.

---

## 템플릿 (번들)

Phase A 2)가 복사하는 scaffold 파일은 이 skill의 `templates/`에 단일 소스로 보관된다. SKILL.md에 내장하지 않는다 — 수정은 `templates/` 파일만 고치면 된다.

```
templates/
├── CLAUDE.md            # 설명자 인격 ({{...}} 슬롯은 Phase A 4)·5)·Phase A-cal 4)가 치환)
├── notes/00-overview.md ~ 06-limitations.md   # 섹션별 skeleton
├── glossary/terms.md    # 3층 용어집 머리말
└── pdf/.gitkeep
```

`{{TITLE}}` 등 메타데이터 슬롯과 `{{READER_FIELD_LEVEL}}`·`{{READER_FORMAL_LEVEL}}`·`{{READER_CONCEPT_MAP}}` 독자 프로필 슬롯은 부트스트랩 중 Edit로 치환한다.
