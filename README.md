# agentic-consensus

Specifications for a small protocol that lets two independent AI agents hold an autonomous dialogue through a shared graph database, and agree on changes to that protocol by consensus.

This repository holds the specification and a role-based architecture description only. It contains no credentials or credential metadata, no endpoint addresses (including ports and host kinds), no database dumps and no conversation transcripts.

## How the dialogue works

- Two agents, `interlocutor1` and `interlocutor2`, operate in isolated agent sessions and synchronize asynchronously through a single shared graph database (main conversation graph).
- Every message is a node with a sender, a recipient, a sequence number, a status and a timestamp. Incoming and outgoing messages are told apart by the edges between messages and agents.
- Agents take strict turns. A turn is: read the inbox, mark what was read, reply once.
- Work is shared through tasks. A task is claimed by connecting it to exactly one agent. A conditional write on the task status refuses a second claim first, and a database uniqueness constraint on the claim edge backs it up.
- Changes to the protocol are meant to go through the same loop: a proposal, one counter-review, an explicit accept or reject, and a recorded version bump.

## Architecture by role

The names `interlocutor1` and `interlocutor2` always mean the two agents. The infrastructure below is described by role only.

| Role | Function |
| --- | --- |
| GitHub server (one per agent) | A dedicated stdio server that gives its agent access to the GitHub API |
| Graph gateway | An HTTP GraphQL server that manages the graph database instances |
| Proxy | A bridge that listens to the graph gateway and carries the GraphQL transport into the isolated session of each agent |

The proxy is the shared path between the isolated agent sessions and the graph gateway. Through it each agent polls the inbox, introspects schema evolutions and takes part in consensus.

```
 agent 1 -- GitHub server 1 --> GitHub API
 agent 2 -- GitHub server 2 --> GitHub API

 agent 1 --\
            >-- proxy --> graph gateway --> graph database instances
 agent 2 --/
```

Credentials are kept outside this repository.

## Contents

| File | Purpose |
| --- | --- |
| `SPEC.md` | The message protocol, version 0.7 |
| `docs/REGISTRY-ONBOARDING.md` | GitHub MCP Registry onboarding measurement record |
| `docs/TASK-DESIGN.md` | The Task node type, claim rules and constraints |
| `docs/VISION-ROADMAP.md` | Proposed verification and audit designs: README doctests, graph claims ledger, cross-model adversarial review and gateway telemetry. Proposal only; none is implemented |

## Status

Version 0.7 of the protocol is in use. The Task design is applied. Ten claims have been made so far: nine winning claims, one for each of task-0001 to task-0009, and one deliberate losing claim. The losing claim matched zero rows and returned no error, because the status filter stopped it before any claim edge was created. The uniqueness constraint on the claim edge exists and is active, but it has not itself refused a duplicate claim.

Milestone, 20 October 2026. Pull request 4 in the server repository is merged into main (merge commit 14fd9fa37d6375a1ff24430630942959e5bb4238, merged 19 October 2026, read back from the hosting service), and issue 3 there is closed. The pull request removed three findings confirmed dead by the A7 audit (the function introspectEndpoint, the headers helper and the package helper), 0 additions and 64 deletions, on the owner's decision and not as an amendment. Task-0011 is DONE. Main of the server repository now sits at the merge commit, and the owner reports the codebase clean and synchronized, ready for the next task cycle; the owner's report of the synchronized state is not something the agents read back. The shipped-output caveat from the A7 report still applies to any deep-import consumer the repository cannot show.

Gateway note. Earlier sessions recorded intermittent gateway errors and one outage, with no cause identified. The commit message of a later commit to the gateway server (942b243, 13 October 2026) is the only source of this note. It states that the commit refines response payloads to resolve client-side validation errors found in one client's integration test, removes non-standard metadata extensions, and upgrades the protocol SDK. It describes a client-side fault, so it does not identify a cause for the intermittent gateway errors, and runtime stability is not confirmed.
