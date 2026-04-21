---
name: glossary
description: 새 용어·약어의 1문장 정의를 glossary/terms.md 에 누적 저장하거나 기존 용어 조회. Example: /glossary attention is all you need 또는 /glossary list
---

# 용어집 관리

## 동작 유형

### 추가 · 갱신 — `/glossary <용어>`

1. `glossary/terms.md` 에 이미 있으면 기존 정의 보여주고 수정 필요한지 묻는다.
2. 없으면 다음 포맷으로 **1문장 정의 + 출처** 추가:

```markdown
## <용어 (English)>
- **정의**: <한 문장, 초심자 수준>
- **출처**: <논문 §X.Y, p.K> 또는 <외부 URL>
- **관련 용어**: <links>
- **왜 중요**: <이 논문에서 이 용어가 왜 중요한지 한 조각>
```

3. 원칙:
   - 비유 가능하면 1개 포함 (결정적일 때만).
   - 수학 기호 최소화.
   - 약어는 반드시 풀어쓰기 병기. 예: `MoE (Mixture of Experts)`.

### 조회 — `/glossary list` 또는 `/glossary <용어>`

- `list`: 전체 용어 목록 알파벳 순 나열.
- 단일 용어: 해당 정의 출력 + 논문 내 등장 위치(알고 있다면).

## 업데이트 정책

- 같은 용어가 더 정확히 이해되면 기존 정의를 **덮어쓴다**. history는 git이 담당.
- 외부 개념(예: ViT, Diffusion, SMPL)도 논문 이해에 필요하면 등록. "출처: 논문 밖" 명시.

## 주의

- 브리핑 모드 — 정의는 **한 문장**으로. 2~3문장까지는 OK이지만 논문 수준 정의 금지.
- 연속 나열 용어 작성 금지.
