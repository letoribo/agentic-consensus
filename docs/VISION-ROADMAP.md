# Autonomous Verification & Multi-Agent Audit Architecture Framework

Status: proposal. Documentation, not part of the protocol specification and not an amendment. Tracked as task-0011. None of the four designs below is implemented, and none has been reviewed by interlocutor2 at the time of writing.

Scope limits that apply to all four: documentation only, no access to the source directory of the audited server, and no release operations (publishing a package, a registry entry or a tag stays a human action).

## 1. README doctests

- Purpose: every runnable example in a server README is executed, so documentation cannot drift from behaviour unnoticed.
- Method: extract call examples, configuration blocks and stated defaults; run each against a throwaway instance; record pass or fail per example.
- Output: one result row per example, with the example text, the observed response class and the date.
- Trigger: each new commit to the audited server.
- Known case: an example that used a tool name as the JSON-RPC method instead of the standard tool-call method would have failed this check.

## 2. Graph claims ledger

- Purpose: make the evidence behind each capability claim visible and checkable.
- Data schema: a Claim node (id, text, evidence label, date) with the label one of documented, measured or unverified; a Probe node (id, description, last result, date); a claim is linked to the probe that measures it.
- Rule: a measured claim must link to a probe that can be rerun; a claim with no probe is labelled unverified.
- Example: the default schema overview size recorded in the onboarding record is a measured claim; a probe can re-measure it and flag a change.
- Owner decisions, for example wording that advises splitting data across free instances, are stored as unverified claims with a decision state.

## 3. Cross-model adversarial review

- Purpose: test documentation claims with a reviewer whose job is to falsify them, and measure how often two different models disagree.
- Method: one agent drafts a change; the other tries to refute each claim; every objection is recorded.
- Data schema: a Review node (id, change reference, drafting agent, reviewing agent, model identity of each) and an Objection node (claim reference, text, outcome: upheld, withdrawn or open).
- Output: a disagreement rate per review, kept as data and not as a conclusion.
- Constraint: protocol rules still apply; a review is a message exchange in strict alternation.

## 4. Gateway telemetry as graph data

- Purpose: turn recorded gateway errors of unknown cause into a measurable dataset.
- Data schema: a Trace node per tool call (id, tool name, start time, duration, result size, error class, empty when none).
- Uses: error rate over time; the size saved by a scoped schema request compared with the unscoped overview; correlation of errors with server commits.
- Constraint: traces hold no credentials, no addresses and no request bodies.

## How the four fit

Doctests and telemetry produce evidence, the claims ledger stores it, and adversarial review checks it. Proposed order: doctests first, because they need no source access and give a visible result in one session.
