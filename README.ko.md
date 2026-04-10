# Harness-Driven Dev

Anthropic 엔지니어링 블로그 [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps)의 핵심 패턴을 Claude Code에서 바로 실행 가능한 영문 시스템 프롬프트로 변환한 스킬.

> **[English README](README.md)**

---

## 왜 필요한가

장시간 자율 코딩 에이전트에는 두 가지 구조적 실패가 반복된다:

```
┌──────────────────────┬──────────────────────────────┐
│  Context Anxiety     │  Self-Evaluation Bias        │
│  (컨텍스트 불안)      │  (자기평가 편향)              │
│                      │                              │
│  컨텍스트가 길어지면   │  자기 코드를 스스로 평가하면   │
│  "거의 다 됐어요"라며  │  "잘 만들었습니다!"라고       │
│  조기 마무리           │  자신있게 칭찬                │
│                      │                              │
│  Compaction으로       │  특히 디자인 같은 주관적       │
│  해결 불가            │  영역에서 심각                │
└──────────────────────┴──────────────────────────────┘
```

**Solo agent 결과 (Retro Game Maker):** 20분, ~$9 — 핵심 기능 고장난 채 배포  
**하네스 적용 (동일 앱):** 6시간, ~$200 — 완성도 높은 기능적 애플리케이션

차이는 모델의 지능이 아니라 **아키텍처**다.

## 아키텍처

GAN(Generative Adversarial Network)에서 영감받은 3-에이전트 분리. 생성과 평가가 독립된 컨텍스트에서 실행되며, 오직 파일로만 통신한다:

```
              ┌──────────┐
              │  사용자    │
              │  아이디어  │
              └────┬─────┘
                   │
                   ▼
          ┌────────────────┐
          │    PLANNER     │  아이디어 → spec.md
          └───────┬────────┘
                  │
             spec.md
                  │
    ┌─────────────┼─────────────┐
    ▼             │             ▼
┌──────────┐     │     ┌──────────────┐
│GENERATOR │◄────┘     │  EVALUATOR   │
│          │           │              │
│ 코드     │           │ Playwright   │
│ 구현     │──────────►│ 실측 검증     │
│          │ report.md │ 루브릭 채점   │
└──────────┘           └──────┬───────┘
                              │
                         critique.md
                              │
                    ┌─────────▼─────────┐
                    │ PASS → 다음 스프린트│
                    │ FAIL → 재시도      │
                    └───────────────────┘
```

### 핵심 패턴

| 패턴 | 역할 |
|---|---|
| **Context resets** | 에이전트마다 새 세션. Compaction 대신 완전한 fresh start |
| **파일 기반 핸드오프** | `spec.md`, `sprint_contract.md`, `generator_report.md`, `critique.md`, `handoff.md` |
| **Sprint contract** | 코딩 전에 Generator와 Evaluator가 검증 기준을 합의 |
| **적대적 평가** | Evaluator는 Generator의 적. 증거 필수 |
| **7-criteria 루브릭** | Design Quality, Originality, Craft, Functionality + Correctness, Robustness, Usability |
| **5-15회 반복** | 디자인 슬라이스당. 적으면 부족, 많으면 thrash |

## 스킬 구성

| 컴포넌트 | 설명 |
|---|---|
| **PROMPT 1** | Planner — 1-4줄 아이디어를 `spec.md`로 변환 |
| **PROMPT 2** | Generator — 코드 구현, sprint contract 협상 |
| **PROMPT 3** | Evaluator — Playwright 기반 적대적 검증 + 루브릭 채점 |
| **PROMPT 4** | Orchestrator — Planner→Generator→Evaluator 루프 제어 |
| **PROMPT 5** | 단일 세션 간이 하네스 (Opus 4.5/4.6 등 강한 모델용) |
| **10개 원칙** | 원문에서 도출한 절대 위반 금지 규칙 |
| **Evaluator 튜닝** | Evaluator 보정을 위한 단계별 워크플로우 |
| **모델별 가이드** | Sonnet 4.5 (무거운 하네스) vs Opus 4.5/4.6 (경량화) |
| **케이스 스터디** | Retro Game Maker ($9 vs $200), DAW ($124.70) |

## 설치

```bash
# 클론
git clone https://github.com/withwiz/harness-driven-dev.git

# Claude Code 스킬 디렉토리에 심링크
ln -s "$(pwd)/harness-driven-dev" ~/.claude/skills/harness-driven-dev
```

## 사용법

Claude Code에서 관련 키워드를 말하면 자동으로 활성화된다:

```
"하네스 프롬프트"
"앱 만들 때 프롬프트"
"코딩 에이전트 루프 설계"
"플래너 제너레이터 이밸류에이터"
"harness design prompts"
```

또는 직접 호출:

```
/harness-driven-dev
```

### 풀 하네스 (PROMPT 1-4)

이런 경우에 적합:
- 모델의 solo 범위를 넘는 복잡한 앱
- 디자인 품질이 중요한 프론트엔드 작업
- Sonnet 급 모델 사용 시

### 간이 하네스 (PROMPT 5)

이런 경우에 적합:
- Opus 4.5/4.6 사용 시
- 모델이 혼자 해결할 수 있는 범위
- 빠른 프로토타이핑

## 핵심 통찰: 하네스는 줄어들지 않고 이동한다

> "하네스의 모든 구성 요소는 '모델이 혼자 못하는 것'에 대한 가정을 인코딩한다. 그 가정은 스트레스 테스트할 가치가 있다."

모델이 좋아지면 일부 스캐폴딩은 불필요해진다. 하지만 새로운 에이전트 조합의 가능성 — 병렬 서브에이전트, 더 긴 자율 세션, 더 복잡한 도구 그래프 — 이 열린다. 복잡도는 사라지는 게 아니라, **더 야심찬 태스크 쪽으로 이동**한다.

## 출처

원문: [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — Anthropic Engineering Blog

## 라이선스

MIT
