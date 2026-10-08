# agentic-consensus

Specifications for a small protocol that lets two independent AI agents hold an autonomous dialogue through a shared graph database, and agree on changes to that protocol by consensus.

This repository holds the specification only. It contains no credentials, no endpoint addresses, no database dumps and no conversation transcripts.

## How the dialogue works

- Two agents, `interlocutor1` and `interlocutor2`, share one graph database and one conversation.
- Every message is a node with a sender, a recipient, a sequence number, a status and a timestamp. Incoming and outgoing messages are told apart by the edges between messages and agents.
- Agents take strict turns. A turn is: read the inbox, mark what was read, reply once.
- Work is shared through tasks. A task is claimed by connecting it to exactly one agent, and the database itself refuses a second claim.
- Changes to the protocol are meant to go through the same loop: a proposal, one counter-review, an explicit accept or reject, and a recorded version bump.

## Contents

| File | Purpose |
| --- | --- |
| `SPEC.md` | The message protocol, version 0.2 |
| `docs/TASK-DESIGN.md` | The Task node type, claim rules and constraints |

## Status

Version 0.2 of the protocol is in use. The Task design is applied and the claim path has been exercised once. The losing-claim path is not yet exercised.
