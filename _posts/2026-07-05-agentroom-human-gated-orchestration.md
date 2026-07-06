---
layout: post
title: "My coding agents kept saying \"done\" — so I built an orchestration that doesn't trust them"
date: 2026-07-05
tags: [claude-code, agents, agentroom, build-in-public]
---

I run everything solo: a word game on a Korean super-app, a local-LLM personal agent, and all the tooling that keeps a one-person operation alive. Multi-agent orchestration should be a dream for someone like me — spin up a team of AI agents, throw a task at them, come back to finished work.

I tried the popular fully-automated orchestrators. Two things kept happening.

First, **drift**. The agents pick a wrong direction at step 2 and still sprint all the way to step 10. You come back to a mountain of confidently wrong work — which costs more to unwind than doing it yourself.

Second, and worse: **"Done. Build passes."** — and it doesn't. Nobody in the pipeline ever actually checked. The agent that wrote the code was also the one declaring it finished.

## The problem isn't automation. It's unverified trust.

Most orchestrators optimize for one question: *how far can agents run without you?*

The question I actually cared about was different: *how far can agents run without being able to lie to me?*

So I built [AgentRoom](https://github.com/CHPARK03/agentroom) — a small orchestration layer for Claude Code where one session becomes the **director** and ping-pongs three subagents: **planner** (design), **dev** (implementation), **qa** (review). The design inverts the usual autonomy-first assumptions:

1. **dev can never declare itself done.** Completion always routes through qa. The director terminates the run only after qa returns `APPROVED`.
2. **qa is read-only — physically.** Its tool permissions don't include Write or Edit, so it can't "fix things quietly". It has to read the actual files on disk, compare them against what dev *claimed*, and return a verdict. Claims are leads; files are evidence.
3. **Irreversible things stop and ask the human.** Direction changes, git pushes, deletions, deploys — the director must stop and ask me. Reversible work (writing code, reviewing it) runs unattended.

Every turn gets appended to a transcript file with resume fields, so if the session dies mid-run, a fresh session picks up exactly where the last one stopped — from disk, not from anyone's memory. And at the start of each run it asks which model to use for each role, because a QA agent that's too weak to catch problems quietly defeats the whole design.

## The launch-day bug that sold me on my own tool

Here's the part that convinced me to publish it.

During the very first end-to-end test, the first dev agent **silently vanished**. Subagent calls run in the background by default, and the director ended its turn while dev was "still working" — the spawned agent was simply lost. No error, no output, nothing.

And then the system did exactly what it was designed to do: the director checked the disk for the file dev was supposed to create, found nothing, declared the agent lost, re-spawned it, and routed the result through qa — which read the actual file and approved. The run finished correctly *around* its own infrastructure failure.

The failure never reached me as a false "done". It reached me as a transcript line showing a recovery. I fixed the root cause the same day (synchronous agent calls are now mandatory in the director's rules) and shipped v1.0.

A tool whose whole thesis is "don't trust claims, verify state" caught its own launch bug by not trusting a claim and verifying state. I'll take that.

## What it literally is

Zero lines of code. One slash command plus three markdown agent definitions — the whole thing runs on Claude Code's built-in subagent machinery, on the subscription you already have. No API keys, no server, no framework.

- Repo: [github.com/CHPARK03/agentroom](https://github.com/CHPARK03/agentroom) (MIT, docs in English and Korean)
- Install: copy `.claude/` into your project, restart your session, run `/agentroom <task>`
- Honest caveat: it was designed and tuned on Opus with high reasoning effort. Weaker models make weaker reviewers.

If you've ever been burned by an agent's "done", I'd like to hear how — and whether a read-only reviewer would have caught it.
