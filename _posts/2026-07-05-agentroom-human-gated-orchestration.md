---
layout: post
title: "에이전트의 \"완료했습니다\"를 못 믿어서, 에이전트를 신뢰하지 않는 오케스트레이션을 만들었다"
date: 2026-07-05
tags: [claude-code, agents, agentroom, build-in-public]
---

*🇬🇧 [English version below](#english-version)*

나는 모든 걸 혼자 만든다. 한국 슈퍼앱에 올리는 단어 게임, 내 컴퓨터에서만 도는 로컬 LLM 개인 에이전트, 그리고 1인 운영을 지탱하는 온갖 도구들. 멀티에이전트 오케스트레이션은 나 같은 사람에게 꿈의 도구여야 한다 — AI 에이전트 팀을 띄워놓고 작업을 던진 뒤, 돌아와서 완성된 결과물을 받는 것.

그런데 완전 자동 오케스트레이터 방식을 들여다볼수록 두 가지가 계속 걸렸다.

첫째, **방향 이탈**. 에이전트가 2단계에서 방향을 잘못 잡아도 10단계까지 전력 질주한다. 돌아와 보면 확신에 찬 오답 더미가 쌓여 있고, 이걸 되돌리는 비용이 직접 하는 것보다 크다.

둘째, 더 나쁜 것: **"완료했습니다. 빌드 통과합니다."** — 실제로는 아니다. 파이프라인의 누구도 진짜로 확인한 적이 없다. 코드를 쓴 에이전트가 완료 판정까지 스스로 내리는 구조이기 때문이다.

## 문제는 자동화가 아니라, 검증 없는 신뢰다

대부분의 오케스트레이터는 한 가지 질문에 최적화돼 있다: *사람 없이 에이전트가 얼마나 멀리 갈 수 있는가?*

내가 원한 질문은 달랐다: *에이전트가 나에게 거짓말할 수 없는 상태로, 얼마나 멀리 갈 수 있는가?*

그래서 [AgentRoom](https://github.com/CHPARK03/agentroom)을 만들었다. Claude Code 세션 하나가 **총괄(director)**이 되어 세 서브에이전트 — **planner**(설계), **dev**(구현), **qa**(검수) — 를 핑퐁시키는 작은 오케스트레이션 레이어다. 설계는 통상적인 "자율성 우선" 가정을 뒤집는다:

1. **dev는 스스로 "완료"를 선언할 수 없다.** 완료는 반드시 qa를 거친다. 총괄은 qa가 `APPROVED`를 반환한 뒤에만 작업을 종료한다.
2. **qa는 물리적으로 읽기 전용이다.** 도구 권한에 Write/Edit 자체가 없어서 "조용히 고쳐놓기"가 불가능하다. 디스크의 실제 파일을 직접 읽고, dev의 *주장*과 대조한 뒤 판정을 내린다. 주장은 단서일 뿐, 증거는 파일이다.
3. **되돌리기 어려운 일은 멈추고 사람에게 묻는다.** 방향 전환, git push, 파일 삭제, 배포 — 총괄은 반드시 멈추고 나에게 물어야 한다. 되돌리기 쉬운 일(코드 작성, 검수)만 무인으로 달린다.

매 턴은 재개 필드와 함께 transcript 파일에 기록된다. 세션이 중간에 죽어도 새 세션이 마지막 상태를 디스크에서 — 누군가의 기억이 아니라 — 복원해 이어간다. 그리고 실행 시작 시 역할별로 어떤 모델을 쓸지 묻는다. 문제를 잡아내기엔 너무 약한 qa는 이 설계 전체를 무너뜨리기 때문이다.

## 출시 당일, 내 도구에 설득당한 순간

이 도구를 공개하기로 결심하게 만든 사건이 있다.

첫 엔드투엔드 테스트에서 첫 dev 에이전트가 **소리 없이 증발했다**. 서브에이전트 호출이 기본적으로 백그라운드로 돌아가는데, 총괄이 dev가 "아직 작업 중"인 상태로 자기 턴을 끝내버린 것이다. 에러도, 출력도, 아무것도 없었다.

그런데 시스템이 정확히 설계대로 움직였다. 총괄은 dev가 만들었어야 할 파일을 디스크에서 확인했고, 없다는 걸 발견했고, 에이전트 유실로 판정한 뒤 재호출했고, 결과를 qa로 라우팅했다 — qa는 실제 파일을 읽고 승인했다. 실행은 자기 인프라의 실패를 *우회해서* 올바르게 끝났다.

그 실패는 나에게 거짓 "완료"로 도착하지 않았다. 복구 과정이 담긴 transcript 한 줄로 도착했다. 근본 원인은 그날 바로 고쳤고(총괄 규칙에 동기 호출 의무화), v1.0을 공개했다.

"주장을 믿지 말고 상태를 검증하라"가 테제인 도구가, 주장을 믿지 않고 상태를 검증해서 자기 자신의 출시 버그를 잡았다. 이 정도면 됐다 싶었다.

## 실체는 이렇다

코드 0줄. 슬래시 커맨드 1개 + 마크다운 에이전트 정의 3개가 전부고, Claude Code에 내장된 서브에이전트 기능 위에서 돌아간다. 이미 쓰고 있는 구독으로 동작한다. API 키도, 서버도, 프레임워크도 없다.

- 레포: [github.com/CHPARK03/agentroom](https://github.com/CHPARK03/agentroom) (MIT, 한국어/영어 문서)
- 설치: `.claude/` 폴더를 프로젝트에 복사 → 세션 재시작 → `/agentroom <작업>`
- 정직한 고지: Opus + 높은 reasoning effort 기준으로 설계·튜닝됐다. 약한 모델은 약한 검수자가 된다.

에이전트의 "완료했습니다"에 데인 적이 있다면, 어떤 상황이었는지 듣고 싶다 — 그리고 읽기 전용 검수자가 있었다면 잡을 수 있었을지도.

---

## English version

I build everything solo: a word game for a Korean super-app, a local-LLM personal agent, and all the tooling that keeps a one-person operation alive. Multi-agent orchestration should be a dream for someone like me — spin up a team of AI agents, throw a task at them, come back to finished work.

But the more I looked into fully-automated orchestrators, the more two things kept bothering me.

First, **drift**. The agents pick a wrong direction at step 2 and still sprint all the way to step 10. You come back to a mountain of confidently wrong work — which costs more to unwind than doing it yourself.

Second, and worse: **"Done. Build passes."** — and it doesn't. Nobody in the pipeline ever actually checked. The agent that wrote the code was also the one declaring it finished.

### The problem isn't automation. It's unverified trust.

Most orchestrators optimize for one question: *how far can agents run without you?*

The question I actually cared about was different: *how far can agents run without being able to lie to me?*

So I built [AgentRoom](https://github.com/CHPARK03/agentroom) — a small orchestration layer for Claude Code where one session becomes the **director** and ping-pongs three subagents: **planner** (design), **dev** (implementation), **qa** (review). The design inverts the usual autonomy-first assumptions:

1. **dev can never declare itself done.** Completion always routes through qa. The director terminates the run only after qa returns `APPROVED`.
2. **qa is read-only — physically.** Its tool permissions don't include Write or Edit, so it can't "fix things quietly". It has to read the actual files on disk, compare them against what dev *claimed*, and return a verdict. Claims are leads; files are evidence.
3. **Irreversible things stop and ask the human.** Direction changes, git pushes, deletions, deploys — the director must stop and ask me. Reversible work (writing code, reviewing it) runs unattended.

Every turn gets appended to a transcript file with resume fields, so if the session dies mid-run, a fresh session picks up exactly where the last one stopped — from disk, not from anyone's memory. And at the start of each run it asks which model to use for each role, because a QA agent that's too weak to catch problems quietly defeats the whole design.

### The launch-day bug that sold me on my own tool

Here's the part that convinced me to publish it.

During the very first end-to-end test, the first dev agent **silently vanished**. Subagent calls run in the background by default, and the director ended its turn while dev was "still working" — the spawned agent was simply lost. No error, no output, nothing.

And then the system did exactly what it was designed to do: the director checked the disk for the file dev was supposed to create, found nothing, declared the agent lost, re-spawned it, and routed the result through qa — which read the actual file and approved. The run finished correctly *around* its own infrastructure failure.

The failure never reached me as a false "done". It reached me as a transcript line showing a recovery. I fixed the root cause the same day (synchronous agent calls are now mandatory in the director's rules) and shipped v1.0.

A tool whose whole thesis is "don't trust claims, verify state" caught its own launch bug by not trusting a claim and verifying state. I'll take that.

### What it literally is

Zero lines of code. One slash command plus three markdown agent definitions — the whole thing runs on Claude Code's built-in subagent machinery, on the subscription you already have. No API keys, no server, no framework.

- Repo: [github.com/CHPARK03/agentroom](https://github.com/CHPARK03/agentroom) (MIT, docs in English and Korean)
- Install: copy `.claude/` into your project, restart your session, run `/agentroom <task>`
- Honest caveat: it was designed and tuned on Opus with high reasoning effort. Weaker models make weaker reviewers.

If you've ever been burned by an agent's "done", I'd like to hear how — and whether a read-only reviewer would have caught it.
