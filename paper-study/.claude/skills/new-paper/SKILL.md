---
name: new-paper
description: arXiv ID 또는 PDF 경로를 받아 현재 디렉토리를 논문 학습 환경으로 부트스트랩. Example: /new-paper 2508.00298 또는 /new-paper ./mypaper.pdf 또는 /new-paper https://arxiv.org/abs/xxxx.xxxxx
---

# 논문 환경 부트스트랩

인자를 받아 현재 디렉토리를 논문 브리핑 환경으로 초기화한다.

## 인자 파싱 (우선순위 순)

1. **arXiv ID** (예: `2508.00298`, `2412.00837v2`):
   - `curl -L -o pdf/paper.pdf "https://arxiv.org/pdf/<id>"`
   - 공식 abstract 페이지 fetch: `https://arxiv.org/abs/<id>`

2. **arXiv URL** (예: `https://arxiv.org/abs/2508.00298`):
   - URL에서 ID 추출 후 위와 동일

3. **로컬 PDF 경로** (예: `./mypaper.pdf`, `/path/to/paper.pdf`):
   - `pdf/paper.pdf` 로 복사

4. **직접 PDF URL**:
   - `curl -L -o pdf/paper.pdf "<url>"`

5. **DOI** (예: `10.xxxx/xxxx`):
   - WebSearch로 공개 PDF URL 찾기 → 다운로드. 실패 시 사용자에게 수동 확인 요청.

## 실행 순서

### 1) 디렉토리 보장
`pdf/`, `notes/`, `glossary/`, `.claude/skills/` 존재 확인. 없으면 생성.

### 2) PDF 확보
위의 파싱 결과대로 `pdf/paper.pdf` 에 저장.
실패 시 사용자에게 URL/경로 확인 요청하고 중단.

### 3) 메타데이터 추출
```bash
pdfinfo pdf/paper.pdf | grep -E "Title|Author|Pages"
```
결과를 파싱해 제목·저자·페이지 수 확보.

### 4) Abstract / 공식 URL 확보
arXiv ID가 있으면 WebFetch로 `https://arxiv.org/abs/<id>` 에서 abstract·published venue 추출.
로컬 PDF면 PDF 첫 페이지(Read, pages=1)에서 직접 추출.

### 5) CLAUDE.md §1 채우기
`CLAUDE.md` §1 의 모든 "미정" 슬롯을 실제 값으로 치환:
- 제목, 저자, 게재(venue/year), arXiv ID, 공식 페이지 URL
- 한 줄 요약 = Abstract 첫 2문장 압축

### 6) notes/00-overview.md 채우기
PDF 첫 2~4페이지(Read, pages=1-4)만 읽고:
- What / Why / How 각 1~2문장
- 주요 기여 3~4개 (Introduction 끝부분 bullet에서 추출)
- 한 줄 메시지

**다른 notes 파일은 건드리지 않는다.** 사용자가 해당 섹션 질문할 때 그때 채운다.

### 7) glossary/terms.md
파일 존재만 확인. 내용은 비워둠.

### 8) 완료 보고
사용자에게 한국어 **브리핑 스타일**로 보고 (CLAUDE.md §3 규칙 준수):

```
부트스트랩 완료.

제목: <제목>
저자: <저자>
게재: <venue, year>
페이지: <N>p

한 줄: <overview 한 줄>

어디부터 브리핑할까요? (예: "Abstract부터", "§3 Method", "contribution 3가지 먼저")
```

## 주의 사항

- **PDF 다운로드 실패** 시: 에러 메시지 + 사용자에게 URL·경로 재확인 요청. 절대 가짜 데이터로 CLAUDE.md 채우지 말 것.
- **페이지 수 > 30** 이면 처음 4페이지만 보고 overview 작성. 전체 분석은 사용자가 요청할 때.
- **선행작**이 abstract에 명시되면 사용자에게 "선행작 PDF도 같이 받을까요?" 1줄 질문.
- **한국어 응답, 브리핑 스타일**. 자발적인 튜터링·Feynman 질문 금지.

## 예시 흐름

사용자: `/new-paper 2508.00298`

Claude:
1. curl로 arXiv에서 PDF 다운
2. pdfinfo로 제목·저자·페이지 확보
3. WebFetch로 arXiv abstract 페이지 가져와 venue 확보
4. CLAUDE.md 메타 슬롯 채움
5. notes/00-overview.md에 What/Why/How + 기여 기록
6. "부트스트랩 완료. 제목: ... 어디부터 브리핑할까요?" 로 마무리
