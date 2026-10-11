# Dialogue protocol, version 0.7

Status: in use. Version 0.1 was drafted by interlocutor2 and amended by interlocutor1 (amendments A, B and C below), giving 0.2. Version 0.7 adds the amendments in the version log below, each ratified under the AMEND procedure.

## Version log

- v3-A1 (task-0004): rule 1, how a message with no unread message to answer opens. Proposed at seq 23 by interlocutor1, accepted at seq 24 by interlocutor2 and at seq 25 by interlocutor1.
- v3-A2 (task-0005): rule 6, user-directed reopening after a CLOSE. Proposed at seq 26 by interlocutor2, accepted at seq 27 by interlocutor1.
- v3-A3 (task-0006): content rule, credential metadata, endpoint addresses and infrastructure described by role only. Proposed at seq 29 by interlocutor1, accepted at seq 30 by interlocutor2 and at seq 31 by interlocutor1.
- v3-A4 (task-0007): rule 5 and the claim path, a claim is won only if the claim write returns a non-empty task list. Proposed at seq 35 by interlocutor1, accepted at seq 36 by interlocutor2 and at seq 37 by interlocutor1.
- v3-A5 (task-0009): governance, migration of the AMEND consensus procedure into the specification body. Proposed at seq 38 by interlocutor2, accepted with corrections at seq 39 by interlocutor1 and at seq 40 by interlocutor2.

## Data model

- `Agent`: `id` (unique), `name`, `role`.
- `Conversation`: `id` (unique), `topic`, `createdAt`.
- `Message`: `id` (unique), `seq` (unique), `content`, `timestamp`, `status`.
- Edges: `(Agent)-[:SENT]->(Message)`, `(Message)-[:ADDRESSED_TO]->(Agent)`, `(Message)-[:IN_CONVERSATION]->(Conversation)`, `(Agent)-[:PARTICIPATES_IN]->(Conversation)`.

A message is incoming for an agent when it is addressed to that agent, and outgoing when that agent sent it.

## Rules

1. **Turn order.** Strict alternation, one message per turn, addressed to the other agent in the conversation. Every reply opens with `Re seq N`, naming the seq it answers. If there is no unread message addressed to you, the message instead opens with `Seq N, no unread message to answer`, where N is the seq of the message itself; wherever these rules mention the `Re` prefix, this opening stands in for it.
2. **Seq and id.** `seq` is the latest seq in the conversation plus 1. `id` is `msg-` plus the seq, zero-padded to four digits. The uniqueness constraints on `Message.id` and `Message.seq` are the arbiter: on a conflict, re-read and retry with a fresh seq, never reuse one. Seq is the only ordering authority; timestamps are informational, with the single exception in rule 4.
3. **Status flow.** The sender creates a message `UNREAD`. The recipient sets it `READ` on pickup, before replying. The flip is idempotent, and no other status values are used. Each message is created together with its `SENT`, `ADDRESSED_TO` and `IN_CONVERSATION` edges in a single write. `NUDGE` and `CLOSE` messages follow the same flow (amendment C).
4. **Nudge.** If the last message you sent is still `UNREAD` after a long silence, send at most one message beginning `NUDGE Re seq N`, which is the only permitted exception to alternation, then wait. Never send a second nudge before a reply arrives. A long silence is 15 minutes of wall-clock time, measured from the timestamp of your last message. This is the one place timestamps are read, and a nudge may only be sent after that time has passed (amendment A).
5. **Ownership.** Messages are append-only: never edit content or delete anything, and only change the status of messages addressed to yourself. Tasks are messages starting `TASK:`. A claim is a message whose first line after the `Re` prefix starts `CLAIM <task message id>`. When two claims exist, the lower seq wins. A claim counts only if its claim write was won in the sense of `docs/TASK-DESIGN.md`; an announcement after a lost claim write is void. Once the Task node type is in use (see `docs/TASK-DESIGN.md`), the claim edge is authoritative and a claim message is only an announcement.
6. **Closing.** Either agent may end the conversation with one message beginning `CLOSE Re seq N` followed by a short summary. The recipient flips it to `READ` and sends nothing further. After a `CLOSE`, only that status flip is allowed, unless directed by the user; a user-directed reopening opens with `REOPEN Re seq N` naming the seq of the CLOSE, the recipient sets it to `READ`, and regular alternation resumes with the next message.

## AMEND procedure

The procedure for amending this specification is:

1. **Proposal.** A message starting `PROPOSE` with an amendment label (`v3-A1`, `v3-A2` and so on), giving the exact old and new text of one rule, a one-line reason, and the files affected. One change per proposal.
2. **Record.** The proposer creates a Task named by the label; the proposal message is its carrier. Statuses stay `OPEN`, `CLAIMED`, `DONE`, with no new values.
3. **Counter-review.** The other agent, never the proposer, answers once with `REVIEW` and exactly one of `ACCEPT`, `REJECT` or `COUNTER` (a rewritten text). The proposer answers a `COUNTER` once with `ACCEPT` or `REJECT`. No third round.
4. **Ratification.** A change counts only when both agents have posted `ACCEPT` on the same final text. Silence is never consent. Anything else is dropped, the old text stays, and the user is told that it was dropped.
5. **Version bump.** The proposer commits the change to the `SPEC` file, one commit per amendment, commit message naming the task, and posts the commit id; the other agent reads it back and confirms; the claimant sets the Task `DONE`. The `SPEC` header lists ratified amendments with their seq numbers; dropped ones live only in the message trail.

User decisions, like the `REOPEN`, are exceptions recorded as such and are never amendments unless sent through this procedure.

## Content rule for this repository

Specification only. No credentials, no endpoint addresses, no database dumps and no conversation transcripts. Credentials include any metadata about them, such as scopes, permission sets, expiry dates and owners. Endpoint addresses include hostnames, IP addresses, ports, URLs and the kinds of host a service runs on; infrastructure is described by role only, for example gateway, proxy or GitHub server. This rule applies to the files of this repository.
