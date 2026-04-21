---
name: glossary
description: 용어를 "비유 → 논문 맥락 → 형식 정의" 3층으로 설명하고, 외부 개념은 작동 원리까지 포함. 단순 정의 금지. Example: /glossary attention 또는 /glossary MoE 또는 /glossary list
---

# 용어집 (3층 + 작동 원리)

`glossary/terms.md`에 누적 추가·갱신. 단순 "한 문장 정의"로 끝내지 말 것. 사용자가 **머릿속에 원리를 그릴 수 있게** 만드는 것이 목표.

---

## 기본 포맷

```markdown
## <용어 (English)>

**L1 — 한 줄 직관**
_중립적이고 간결한 한 문장 요약. 기술 용어 최소화. **비유는 개념 이해에 결정적일 때만**. 동형 관계가 없는 장식성 비유(사무실·자석·레고 등) 금지. 직접 설명이 더 명료하면 그 쪽을 택할 것._

**L2 — 논문 맥락**
_이 논문에서 이 용어가 어디서·왜 쓰이는지 1~2문장._

**L3 — 형식 정의**
_원저의 정식 정의. 필요하면 기호·수식 포함, 1~2문장._

**작동 원리** _(외부 모델·기법인 경우 필수)_
_한 문단(3~5줄)으로 "어떻게 돌아가는지" 풀이. 입력 → 내부 연산 → 출력 순서. 필요 시 짧은 ASCII 다이어그램._

**왜 중요**
_이 논문에서 빠지면 뭐가 무너지나? 1문장._

**관련 용어** _(선택)_

**출처**
_논문 §X.Y / 외부 논문 URL_
```

---

## 예시 — 나쁨 vs 좋음 (일반 ML 개념으로 demo)

### 나쁨 (피해야 할 수준)
```markdown
## Self-attention
- 정의: Transformer의 핵심 연산. Query·Key·Value로 가중합.
- 출처: Vaswani et al., 2017.
```
→ 사용자는 여전히 "어떻게 돌아가는 애인지" 모름.

### 좋음
```markdown
## Self-attention

**L1 — 한 줄 비유**
문장 안의 **각 단어가 다른 단어들에게 "너 얼마나 관련 있어?"라고 투표**해서, 받은 표를 가중치로 삼아 자기 표현을 업데이트하는 메커니즘.

**L2 — 논문 맥락**
이 논문은 기존 CNN 대신 self-attention을 써서 입력의 먼 부분 사이 관계도 한 번에 본다. §Method에서 아키텍처의 주축으로 등장.

**L3 — 형식 정의**
$\text{Attention}(Q, K, V) = \text{softmax}(QK^T/\sqrt{d_k}) V$
 — Query·Key·Value는 입력을 서로 다른 가중행렬로 투영한 것.

**작동 원리**
1. 입력 토큰 시퀀스를 **세 개의 선형 레이어**에 각각 통과 — Query, Key, Value 세 벌의 표현이 나옴.
2. 각 토큰의 Query를 모든 토큰의 Key와 내적 → "내가 너와 얼마나 관련 있나" 점수(score).
3. score를 `softmax`로 확률화 — 모든 토큰에 대한 가중치 분포.
4. 각 토큰의 Value를 이 가중치로 가중합 → 그 토큰의 새 표현.

```
  Query ─┐
  Key   ─┼─→ dot product ─→ softmax ─→ attention weights
  Value ─┼─────────────────────────────────┐
                                            ▼
                                         weighted sum → 새 표현
```

**왜 중요**
문장·이미지 전체에서 **먼 거리 의존성**을 한 번에 본다. CNN처럼 여러 레이어 거쳐야 멀리 있는 정보가 닿는 단점 제거.

**관련 용어**: Transformer, multi-head attention, Q/K/V

**출처**: Vaswani et al., "Attention Is All You Need", NeurIPS 2017. 이 논문 §X.Y.
```

→ "이게 어떻게 돌아가?" 질문에 자체적으로 답됨.

---

## 언제 **작동 원리** 섹션을 꼭 쓰나

| 용어 종류 | 작동 원리 필요? |
| --- | --- |
| 외부 모델·아키텍처 (Transformer, ResNet, Diffusion, GAN 등) | ✅ 필수 |
| 학습 기법 (contrastive, distillation, RL의 PPO 등) | ✅ 필수 |
| 기본 수식·기호 ($\beta$, $\tau$, $\lambda$) | ❌ 생략 (L3에 shape/의미만) |
| 데이터셋 이름 | ❌ L2에 생성 방식 한 문단이면 충분 |
| 평가 메트릭 (BLEU, FID, mAP, PCK) | ✅ 짧게 — 어떻게 계산되는지 한 문단 |

---

## 외부 배경 조사 의무

L2·작동 원리에 쓸 정보가 논문 본문에 없으면 **WebSearch 한 번**으로 끌어와라. 예:
- "transformer self-attention explained"
- "linear blend skinning how it works"
- "controlnet conditioning mechanism"
- "diffusion model denoising steps"

가져온 배경은 `(논문 밖, WebSearch)` 꼬리표와 출처 URL 병기.

---

## 동작 유형

### 추가 · 갱신 — `/glossary <용어>`

1. 이미 있으면 기존 엔트리 보여주고 **어느 층이 부족한지** 묻고 그 층만 보강.
2. 없으면 위 포맷으로 새 엔트리. **L1은 반드시 비유** — 정의 반복 금지.

### 조회 — `/glossary <용어>` 또는 `/glossary list`

- `list`: 알파벳 순으로 용어 목록만 나열.
- 단일 용어: 해당 엔트리 그대로 출력.

---

## 파일 머리말

`glossary/terms.md` 상단에 다음 안내 주석을 유지:

```markdown
# 용어집

> 각 엔트리는 3층 구조:
> - **L1 (비유)**: 일상 말로 1문장
> - **L2 (논문 맥락)**: 이 논문에서 어디 쓰이나
> - **L3 (형식 정의)**: 원저의 정식 정의
>
> 외부 모델·기법은 **작동 원리** 문단이 추가.
> 알파벳 순 유지. 새 용어는 `/paper-study:glossary <용어>` 로 추가.
```

---

## 주의

- **연속 나열 금지**: 한 줄에 여러 용어 짬뽕 금지.
- **수학 기호 최소화**: L1은 기호 0개, L3에서만 필요한 만큼.
- **Hallucination 금지**: 작동 원리를 모르면 "외부 조사 필요"로 남기고 사용자에게 알릴 것.
