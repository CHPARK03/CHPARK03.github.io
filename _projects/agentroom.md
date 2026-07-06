---
layout: project
title: AgentRoom
headline: 에이전트의 "완료"를 믿지 않는 오케스트레이션
status: OPEN · MIT
dim: false
order: 1
blurb: 사람이 게이트를 쥐는 Claude Code 멀티에이전트 오케스트레이션 — planner·dev·qa를 핑퐁시키고, 되돌리기 어려운 건 사람이 승인.
meta: [Claude Code, 2026]
steps:
  - k: "문제 / Problem"
    h: 자동화는 좋다 — 에이전트의 "됐습니다"를 믿기 전까지는
    p: 완전 자동 오케스트레이터는 방향이 틀어져도 끝까지 달리고, 코드를 쓴 에이전트가 완료 판정까지 스스로 내린다. 검증 없는 신뢰가 문제였다.
  - k: "접근 / Approach"
    h: 주장을 믿지 말고 상태를 검증한다
    p: 읽기 전용 qa가 실제 파일을 직접 읽어 dev의 주장을 반박하고, qa의 APPROVED 없이는 종료 불가. 되돌리기 어려운 일(push·삭제·배포)은 사람 게이트.
  - k: "결과 / Result"
    h: 코드 0줄, 구독만으로 도는 도구
    p: 슬래시 커맨드 1개 + 마크다운 에이전트 정의 3개. 첫 출시 테스트에서 자기 인프라 버그를 스스로 잡아냈다. MIT로 공개.
  - k: "교훈 / Lesson"
    h: 독립 검증이 신뢰를 만든다
    p: 글쓴이가 자기 글을 검수하면 편향이 생긴다. 가장 위험한 지점에만 독립 검증자를 두는 게 토큰과 안전의 균형점이었다.
links:
  - { label: "GitHub (MIT)", url: "https://github.com/CHPARK03/agentroom" }
  - { label: "Dev Log: 만든 이야기", url: "/2026/07/05/agentroom-human-gated-orchestration/" }
---
