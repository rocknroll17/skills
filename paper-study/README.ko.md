# paper-study (한국어)

Claude Code로 **논문 한 편을 빠르게 이해하기 위한 프로젝트 템플릿**.

새 폴더 만들고 arXiv ID만 넘기면, Claude가 그 논문 전담 브리핑 에이전트로 변한다. 섹션 요약, 수식 해설, 선행작 비교, 용어집 축적이 슬래시 커맨드로 미리 설정돼 있다.

---

## 뭐가 들어있나

빈 폴더에 이 템플릿을 복사하면:

- `CLAUDE.md` — 매 세션 자동 로드. 논문 메타데이터, 사용자 프로필, Claude의 브리핑 모드 규칙(짧게, 결론 먼저, 용어 첫 등장만 괄호로 한 줄 설명).
- `notes/` — skeleton 파일들 (`00-overview.md` ~ `05-limitations.md`). 사용자가 섹션별로 질문하면 Claude가 하나씩 채움.
- `glossary/terms.md` — 새 용어 등장할 때마다 누적.
- `.claude/skills/` — 6개 슬래시 커맨드, 논문 종류 무관하게 범용.

Claude가 PDF를 읽고 `CLAUDE.md` 메타를 채운 뒤, `notes/00-overview.md`에 한 줄 요약 쓰고, 첫 질문을 기다린다.

## 요구 사항

- [Claude Code CLI](https://code.claude.com/docs) ≥ 1.0
- `git`, `curl`
- `pdfinfo` (선택, `poppler-utils` 패키지) — `/new-paper`가 제목·저자 추출할 때 씀

## 3단계로 시작

```bash
# 1) 새 폴더에 템플릿 복사
git clone https://github.com/rocknroll17/skills.git /tmp/skills-src
cp -r /tmp/skills-src/paper-study ~/my-paper
cd ~/my-paper

# 2) Claude Code 실행
claude
```

Claude 안에서:

```
/new-paper 2508.00298
```

끝. Claude가 arXiv에서 PDF 받고, 메타데이터 뽑고, `notes/00-overview.md`에 한 줄 요약 쓰고, "어디부터 브리핑할까요?" 물어본다.

로컬 PDF가 있으면:

```
/new-paper ./mypaper.pdf
```

### repo 전체 clone 안 하고 바로 쓰기

```bash
mkdir ~/my-paper && cd ~/my-paper
curl -sL https://api.github.com/repos/rocknroll17/skills/tarball \
  | tar xz --strip=2 --wildcards '*/paper-study/*'
```

## 슬래시 커맨드

| 커맨드 | 용도 |
| --- | --- |
| `/new-paper <arxiv_id | pdf_path | url>` | 새 논문 환경 부트스트랩 |
| `/summarize-section <섹션>` | 섹션 5층 브리핑 (한줄·기술·상세·비판·계보) |
| `/explain-equation <수식>` | 수식 3단 해설 (기호·직관·한 줄 예시) |
| `/compare-prior <선행작>` | 선행작 비교표 |
| `/glossary <용어 | list>` | 용어집 추가·조회 |
| `/quiz <주제>` | 이해 확인 퀴즈 *(사용자 명시 요청 시만)* |

## 폴더 구조

```
my-paper/
├── CLAUDE.md
├── pdf/              # 논문 PDF
├── notes/            # 00-overview.md ~ 05-limitations.md
├── glossary/terms.md
└── .claude/skills/   # 6개 슬래시 커맨드
```

## 본인 스타일로 커스터마이즈

기본 `CLAUDE.md`는 다음을 가정한다:

- 사용자는 **해당 분야 초심자** (용어를 항상 한 줄씩 설명함).
- 답변은 **브리핑 모드 한국어** — 짧게, 결론 먼저, 자발적 Socratic 퀴즈 금지.

언어·톤·설명 깊이를 바꾸려면 `CLAUDE.md`를 그냥 수정하면 된다:

- **§2 사용자** — 본인 배경·선호 언어
- **§3~§7 스타일 규칙** — 말투, 용어 처리, 길이, 금지 사항
- **§11 작업 원칙** — Claude가 먼저 질문해도 되는지 여부

파일은 세션마다 다시 로드되므로 저장 즉시 반영.

## 업데이트

```bash
cd /tmp/skills-src && git pull
```

기존 작업 폴더는 그대로. 다음 `/new-paper`부터 최신 템플릿 반영.

## 라이선스

MIT — repo 최상위 [LICENSE](../LICENSE) 참조.
