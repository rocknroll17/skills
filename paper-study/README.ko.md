# paper-study (한국어)

Claude Code plugin. 빈 폴더 하나를 **논문 한 편 전용 브리핑 에이전트**로 만들어준다.

설치 후 빈 폴더에서 `/paper-study:new-paper <arxiv_id>` 한 줄이면, Claude가 PDF 받고 `CLAUDE.md`·`notes/`·`glossary/` skeleton을 만들고, 브리핑 모드로 첫 질문을 기다린다.

---

## 설치

```
/plugin marketplace add rocknroll17/skills
/plugin install paper-study@rocknroll17-skills
```

그 다음 빈 폴더에서:

```
/paper-study:new-paper 2508.00298
```

로컬 PDF:

```
/paper-study:new-paper ./mypaper.pdf
```

## 요구 사항

- [Claude Code CLI](https://code.claude.com/docs) ≥ 1.0 (plugin 지원)
- `curl`
- `pdfinfo` (선택, `poppler-utils` 패키지 — 제목·저자 추출용)

## `/paper-study:new-paper`가 하는 일

1. PDF를 `./pdf/paper.pdf`로 다운로드
2. 현재 폴더에 `CLAUDE.md`·`notes/` (6개)·`glossary/terms.md` 생성
3. 제목·저자·venue 추출해 `CLAUDE.md` §1 채움
4. PDF 첫 4쪽 정도 읽고 `notes/00-overview.md`에 한 줄 요약 쓰기
5. "어디부터 브리핑할까요?" 1줄로 보고

`CLAUDE.md`는 매 세션 자동 로드. 이제 Claude가 그 논문 전담 브리핑 모드로 바뀐다.

## 슬래시 커맨드 (전부 `paper-study:` 스코프)

| 커맨드 | 용도 |
| --- | --- |
| `new-paper <arxiv_id \| pdf_path \| url>` | 현재 폴더를 논문 환경으로 부트스트랩 |
| `summarize-section <섹션>` | 섹션 5층 브리핑 (한줄·기술·상세·비판·계보) |
| `explain-equation <수식>` | 수식 3단 해설 (기호·직관·한 줄 예시) |
| `compare-prior <선행작>` | 선행작 비교표 |
| `glossary <용어 \| list>` | 용어집 추가·조회 |
| `quiz <주제>` | 이해 확인 퀴즈 *(사용자 명시 요청 시만)* |

## 부트스트랩 후 폴더 구조

```
my-paper/
├── CLAUDE.md                 # 튜터 성격 정의, 매 세션 자동 로드
├── pdf/paper.pdf             # 논문
├── notes/                    # 00-overview.md ~ 05-limitations.md
└── glossary/terms.md         # 용어 누적
```

## 본인 스타일로 커스터마이즈

`CLAUDE.md` 기본값은 다음을 가정:

- 사용자는 **해당 분야 초심자** (용어를 한 줄씩 설명).
- 답변은 **한국어, 브리핑 모드** — 짧게, 결론 먼저, 자발적 Socratic 질문 금지.

수정 대상:

- `§2 사용자` — 본인 배경·선호 언어
- `§3~§7` — 말투, 용어 처리, 길이, 금지 사항
- `§11` — Claude가 먼저 질문해도 되는지 여부

파일은 세션마다 다시 로드되므로 저장 즉시 반영.

## Plugin 시스템 없이 쓰기

Claude Code CLI가 plugin 지원 전이거나, 템플릿만 복사해서 본인 스타일로 고치고 싶다면:

```bash
git clone https://github.com/rocknroll17/skills.git /tmp/skills-src
cp -r /tmp/skills-src/paper-study/scaffold ~/my-paper
cp -r /tmp/skills-src/paper-study/skills ~/my-paper/.claude/skills
cd ~/my-paper
claude
```

`scaffold/` 폴더에 `/paper-study:new-paper`가 만드는 파일과 **동일한 내용**이 들어있다. 직접 복사해서 바로 쓸 수 있음.

## 업데이트

```
/plugin marketplace update rocknroll17-skills
```

기존 논문 폴더는 건드리지 않음. 다음 `/paper-study:new-paper`부터 새 scaffold 반영.

## 라이선스

MIT — repo 최상위 [LICENSE](../LICENSE).
