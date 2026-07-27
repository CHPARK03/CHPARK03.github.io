---
layout: post
title: "승리는 자기가 신고할 수 없다"
date: 2026-07-27 10:00:00 +0900
tags: [game-dev, multiplayer, firebase, build-in-public]
---

*🇬🇧 [English version below](#english-version)*

혼자 만드는 퍼즐 게임에 1대1 대전을 붙였다. 처음 상대는 봇이었다. 봇전은 크게 어렵지 않았다. 상대의 모든 것이 내 기기 안에서 돌아가니까, 누가 이겼는지도 결국 내 코드가 알고 있었다.

사람 대 사람으로 넘어가는 순간 그 전제가 통째로 무너졌다. 이제 상대는 남의 폰에서 돌아가고, 그 폰이 뭐라고 말하든 나는 그걸 확인할 방법이 없다. 지난 3주는 대부분 이 한 문장을 감당하는 일이었다.

## 이긴 사람이 이겼다고 말하면 안 된다

제일 먼저 정한 규칙은 단순하다. 자기 승리는 신고할 수 없고, 자기 패배만 신고할 수 있다.

이유는 뻔하다. "내가 이겼다"를 클라이언트가 보낼 수 있으면 그 메시지 하나가 곧 랭킹 조작 버튼이 된다. 반대로 "내가 졌다"는 굳이 위조할 이유가 없다. 그래서 각자는 자기 패배만 말하고, 승리는 그 반대편에서 파생된다.

그런데 이것만으로는 부족했다. 대전이 끝나는 경로가 하나가 아니기 때문이다. 한쪽 보드가 꽉 차서 끝날 수도 있고, 제한 시간이 다 될 수도 있고, 한쪽이 나가버릴 수도 있다. 경로마다 두 기기가 서로 조금씩 다른 시점에 결론을 낸다. 각자 자기 결론을 자기 화면에 그리면, 같은 경기를 두고 두 사람이 다른 결과를 보는 사태가 열린다.

그래서 결과를 각자 판단하지 않게 만들었다. 어느 경로로 끝나든 확정은 두 기기가 공유하는 결과 자리를 반드시 거친다. **먼저 그 자리에 남은 기록이 그 경기의 결론이고, 늦게 도착한 쪽은 자기 판단을 버리고 그 결론을 읽어서 표시한다.** 양쪽 화면이 어긋날 수 없는 구조가 된다.

나머지는 경로별 예외 처리였다. 두 보드가 사실상 동시에 끝나면 사망 시각을 비교해 다시 판정하고, 시간 종료는 상대 보고를 기다리되 앱이 백그라운드로 내려가 시계가 밀리는 경우를 대비해 기다림에 절대 상한을 뒀다. 한쪽이 나가면 잠깐 기다린 뒤 남은 사람의 승리로 넘겼다. 이 이탈 처리는 나중에 실기기 두 대에서 크게 무너져 전면 재설계하게 되는데, 그건 다음 글에서 따로 다룬다.

## 방은 누가 만드는가

다음은 자동 매칭이었다. 방 코드를 주고받는 사설 대전은 이미 돌고 있었지만, 아무나 눌러도 상대가 붙는 랭크 매치가 필요했다.

문제는 나에게 매치메이커 서버가 없다는 것이다. 대기열을 실시간 DB에 두고 클라이언트끼리 붙게 하면, 둘이 동시에 서로를 발견하는 순간이 온다. 둘 다 방을 만들면 방이 두 개 생기고 각자 빈 방에서 상대를 기다린다.

해결은 중재자를 두는 대신 결론이 하나로만 나오게 만드는 쪽이었다. **두 사람의 ID 중 작은 쪽이 방을 만든다.** 쌍마다 유일하게 정해지는 값이라 누가 먼저 상대를 발견했든 결론이 같다. 작은 쪽이 방을 만들고 자기 대기열 항목에 방 번호를 적으면, 큰 쪽이 그걸 읽고 들어가면서 입장 흔적을 남긴다. 경합할 여지 자체가 사라진다.

권한도 같이 잠갔다. 대기열에서는 자기 항목만 쓸 수 있다. 남의 항목을 건드릴 수 있으면 남의 매칭을 가로채거나 망가뜨릴 길이 열리니까.

사실 첫 설계는 상대 항목에 플래그를 심어 넘기는 방식이었는데, 방금 그 권한 규칙과 정면으로 부딪혔다. 검수에서 반려됐고, 그때 지금의 방식으로 전면 교체했다. 권한을 먼저 못 박아두니 설계 오류가 스스로 드러난 셈이다.

한 가지 더 있었다. 대기자는 실력 구간으로 나눠 조회하는데, 구간 경계를 사이에 두고 선 두 사람은 서로를 영영 못 본다. 실력이 거의 같은데도 그렇다. 자기 구간뿐 아니라 인접 구간까지 함께 조회하도록 넓혀서 메웠다.

## 마커와 보상이 다른 곳에 있으면

가장 오래 붙잡은 건 트로피 정산이었다.

랭크 경기가 끝나면 서버가 양쪽 트로피를 조정한다. 여기서 같은 경기가 두 번 정산되면 랭킹 전체가 부풀어버린다. 그래서 "이 경기는 정산됐다"는 마커를 남기고, 이미 있으면 아무것도 하지 않게 만들려 했다.

내가 처음 짠 건 마커를 실시간 DB에, 트로피 변경을 문서 DB에 두는 구조였다. 검수가 이걸 두 번 반려했다. 그때는 좀 답답했는데, 지적이 정확했다.

서로 다른 저장소라 하나의 트랜잭션으로 묶을 수가 없다. 두 쓰기 사이에서 함수가 죽으면 둘 중 하나가 벌어진다. 마커만 남고 트로피가 안 들어갔으면 그 경기는 재시도해도 "이미 처리됨"으로 걸러져 영영 미지급이다. 트로피만 들어가고 마커가 안 남았으면 재시도할 때마다 또 들어간다. 멱등성을 지키려고 넣은 장치가, 놓인 위치 때문에 미지급과 중복 지급을 동시에 열어둔 것이다.

고친 방식은 마커를 효과와 같은 곳으로 옮기는 것이었다. 정산 원장을 트로피와 같은 DB에 두고, 하나의 트랜잭션 안에서 원장을 먼저 읽는다. 이미 있으면 아무것도 하지 않고 끝낸다. 없으면 양쪽 트로피 변경과 원장 기록을 한 번에 커밋한다. 재시도해도 원장이 이미 있으니 조용히 아무 일도 일어나지 않는다.

위조 방어도 같은 트랜잭션 안에 넣었다. 두 참가자 모두 그 방에 들어온 흔적이 남아 있고, 각 흔적의 신원이 배정된 역할과 일치할 때만 정산이 유효하다. 방에 들어온 적도 없는 사람을 패자로 지목하는 요청은 통과하지 못한다.

## 관통하는 것

세 가지 다 결국 같은 자리에서 나왔다. 클라이언트가 하는 말은 단서일 뿐 증거가 아니라는 것.

전에 내 에이전트 도구를 만들면서 "주장을 믿지 말고 상태를 검증하라"를 원칙으로 삼았는데, 대전을 붙이면서 그 문장을 다시 만났다. 상대가 뭐라고 말하든, 끝나고 남는 건 서버에 원자적으로 기록된 것뿐이다.

혼자 만들면 서버 팀도 없고 매치메이커도 없다. 대신 가진 게 하나 있다. 어떤 값이 위조되면 곤란한지 처음부터 알고 있다는 것. 그 자리마다 원자적 기록을 하나씩 놓는 걸로 꽤 멀리 갈 수 있었다.

---

## English version

I added 1v1 matches to a puzzle game I build alone. The first opponent was a bot, and that part was easy. Everything about the opponent ran on my own device, so my code always knew who had won.

That assumption collapsed the moment two humans faced each other. Now the opponent runs on someone else's phone, and whatever that phone says, I have no way to check it. Most of the last three weeks went into absorbing that one sentence.

### You don't get to announce your own win

The first rule I settled on was simple: you can never report your own victory, only your own defeat.

The reasoning is obvious enough. If a client can send "I won", that message *is* the rank-manipulation button. Nobody has a reason to forge "I lost". So each side reports only its own loss, and the win is derived from the other end.

That alone wasn't enough, though, because a match can end through more than one path: a board fills up, the clock runs out, someone walks away. Each path has the two devices reaching a conclusion at slightly different moments. If each one renders its own conclusion, you get two players staring at contradictory results for the same match.

So I stopped letting them decide independently. However a match ends, finalizing has to pass through one shared result slot. **Whichever record lands there first is the outcome, and the late arrival discards its own verdict and displays what it reads.** The two screens can no longer disagree.

The rest was per-path handling. When both boards die at effectively the same instant, the tie is re-judged by comparing death timestamps. Timeouts wait for the opponent's report, but with a hard ceiling on that wait, since a backgrounded app can drift badly. If someone left, a brief wait ran and the match went to whoever stayed. That last piece fell apart badly once I tested on two real devices and had to be redesigned from scratch, which is a story for the next post.

### Who creates the room?

Automatic matchmaking came next. Private matches with a room code already worked, but ranked play needed strangers to pair up on a single tap.

The problem: I have no matchmaker server. Park a queue in the realtime DB and let clients pair themselves, and sooner or later two players spot each other in the same instant. If both create a room, you get two rooms and two people waiting alone.

Instead of adding an arbiter, I made the outcome impossible to disagree on. **Of the two IDs, the smaller one creates the room.** It's a value that resolves identically for a given pair, so it doesn't matter who noticed whom first. The smaller ID creates the room and writes the room number into its own queue entry; the larger one reads it, joins, and leaves a join marker. There's simply nothing left to race over.

I locked the permissions to match: in the queue, you can only write your own entry. Anything looser opens a path to hijacking or wrecking someone else's match.

My first design actually did pass state by planting a flag on the *opponent's* entry — which ran straight into that permission rule. Review rejected it, and that's when it was replaced wholesale with the handshake above. Nailing down the permission first is what made the design error visible.

One more wrinkle: waiting players are bucketed by rating, and two people sitting on opposite sides of a bucket boundary never see each other, even when their ratings are nearly identical. Widening the query to cover adjacent buckets closed that gap.

### When the marker and the reward live apart

Trophy settlement took the longest.

When a ranked match ends, the server adjusts both players' trophies. Settle the same match twice and the whole ladder inflates, so I wanted a marker saying "this match is settled" and a no-op whenever it already exists.

My first version put the marker in the realtime DB and the trophy change in the document DB. Review rejected that twice. It was frustrating at the time, and it was also right.

Two different stores can't be wrapped in a single transaction. If the function dies between the two writes, one of two things happens. Marker written, trophies not applied: every retry sees "already handled" and the match never pays out. Trophies applied, marker missing: every retry pays out again. The mechanism I added *for* idempotency was, by virtue of where it sat, holding both double-payment and permanent non-payment open at once.

The fix was to move the marker next to the effect. The settlement ledger now lives in the same DB as the trophies, and a single transaction reads the ledger first. If the entry exists, it does nothing. If it doesn't, both trophy deltas and the ledger entry commit together. A retry finds the ledger and quietly does nothing at all.

Forgery defense went into the same transaction. Settlement is only valid if both participants left a join marker in that room and each marker's identity matches the role it was assigned. A request naming someone who never entered the room doesn't get through.

### The thread running through it

All three came from the same place: what a client says is a lead, not evidence.

When I built my agent tooling earlier, I landed on "don't trust claims, verify state" as a principle. Wiring up multiplayer, I ran into that sentence again. Whatever the opponent's device says, what survives is only what was atomically recorded on the server.

Building solo means no backend team and no matchmaker. But it comes with one advantage: from day one you know exactly which values would hurt if forged. Putting a single atomic record at each of those spots got me surprisingly far.
