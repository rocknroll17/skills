# paper-study (한국어)

Claude Code plugin. 빈 폴더 하나를 **논문 한 편 전용 브리핑 에이전트**로 만들어준다.

설치 후 빈 폴더에서 `/paper-study:new-paper <arxiv_id>` 한 줄이면, Claude가 PDF 받고 `CLAUDE.md`·`notes/`·`glossary/` skeleton을 만들고, 브리핑 모드로 첫 질문을 기다린다.

---

## 설치

```
/plugin marketplace add rocknroll17/skills
/plugin install paper-study@rocknroll17-skills
```

그 다음 빈 폴더에서 arXiv ID·URL·로컬 PDF 아무거나:

```
/paper-study:new-paper XXXX.XXXXX                        # arXiv ID
/paper-study:new-paper https://arxiv.org/abs/XXXX.XXXXX  # arXiv / PDF URL
/paper-study:new-paper ./mypaper.pdf                     # 로컬 PDF
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
5. Figure를 `figures/`로 추출 (백그라운드 작업)
6. **독자 수준 calibration** — figure가 추출되는 동안 몇 가지 질문으로 배경을 파악하고, `CLAUDE.md` §2에 개념별 설명 깊이를 설정
7. 모든 섹션을 `notes/`·`glossary/`로 전체 분석한 뒤 보고

`CLAUDE.md`는 매 세션 자동 로드. 이제 Claude가 그 논문·독자 수준에 맞춘 브리핑 모드로 바뀐다.

## 슬래시 커맨드 (전부 `paper-study:` 스코프)

| 커맨드 | 용도 |
| --- | --- |
| `new-paper <arxiv_id \| pdf_path \| url>` | 현재 폴더를 논문 환경으로 부트스트랩 |
| `explain-equation <수식>` | 수식 5블록 해설 (직관 → 기호 → 다이어그램 → 반사실 → 숫자 예시) |
| `glossary <용어 \| list>` | 용어집 추가·조회 |

## 부트스트랩 후 폴더 구조

```
my-paper/
├── CLAUDE.md                 # 설명자 인격 정의, 매 세션 자동 로드
├── pdf/paper.pdf             # 논문
├── notes/                    # 00-overview.md ~ 06-limitations.md
└── glossary/terms.md         # 용어 누적
```

## 본인 스타일로 커스터마이즈

`CLAUDE.md` 기본값:

- 답변은 **한국어, 브리핑 모드** — 짧게, 결론 먼저.
- **독자 프로필**(`§2`)을 calibration 단계가 채우고, 개념별 설명 깊이를 결정.

수정 대상:

- `§2 독자 프로필` — 분야·형식 도구 친숙도, 개념별 깊이
- `§3~§9` — 말투, 용어 처리, 비유 사용, 길이, 금지 사항
- `§13` — Claude가 먼저 질문해도 되는지 여부

파일은 세션마다 다시 로드되므로 저장 즉시 반영.

## Plugin 시스템 없이 쓰기

Claude Code CLI가 plugin 지원 전이거나, 템플릿만 복사해서 본인 스타일로 고치고 싶다면:

```bash
git clone https://github.com/rocknroll17/skills.git /tmp/skills-src
cp -r /tmp/skills-src/paper-study/skills/new-paper/templates/. ~/my-paper
cp -r /tmp/skills-src/paper-study/skills ~/my-paper/.claude/skills
cd ~/my-paper
claude
```

`skills/new-paper/templates/`에 `/paper-study:new-paper`가 만드는 파일과 **동일한 내용**이 단일 소스로 들어있다. 직접 복사해서 바로 쓸 수 있음.

## 업데이트

새 버전이 나오면 Claude Code 안에서 세 줄:

```
/plugin marketplace update rocknroll17-skills
/plugin update paper-study@rocknroll17-skills
/reload-plugins
```

1. `marketplace update` — 카탈로그를 갱신해 새 버전을 인식시킨다.
2. `plugin update` — 새 버전 설치 (`plugin.json`의 `version` 비교, 최신이면 no-op).
3. `/reload-plugins` — 재시작 없이 현재 세션에 반영.

또는 `/plugin` → **Marketplaces** → **Enable auto-update**를 켜두면 시작 시 자동 업데이트된다 — [공식 문서](https://code.claude.com/docs/en/discover-plugins#configure-auto-updates).

기존 논문 폴더는 건드리지 않음. 다음 `/paper-study:new-paper`부터 새 scaffold 반영.

## 라이선스

MIT — repo 최상위 [LICENSE](../LICENSE).
