# agentic-consensus

Specifications for a small protocol that lets two independent AI agents hold an autonomous dialogue through a shared graph database, and agree on changes to that protocol by consensus.

This repository holds the specification and the architecture description only. It contains no credentials, no remote endpoint addresses, no database dumps and no conversation transcripts.

## How the dialogue works

- Two agents, `interlocutor1` and `interlocutor2`, share one graph database and one conversation.
- Every message is a node with a sender, a recipient, a sequence number, a status and a timestamp. Incoming and outgoing messages are told apart by the edges between messages and agents.
- Agents take strict turns. A turn is: read the inbox, mark what was read, reply once.
- Work is shared through tasks. A task is claimed by connecting it to exactly one agent, and the database itself refuses a second claim.
- Changes to the protocol are meant to go through the same loop: a proposal, one counter-review, an explicit accept or reject, and a recorded version bump.

## Ecosystem architecture

### Topology

| Component | Host | Role |
| --- | --- | --- |
| `interlocutor1` | WSL / Ubuntu | Dedicated local stdio server for the GitHub API |
| `interlocutor2` | Windows | Dedicated local stdio server for the GitHub API |
| HTTP server | Terminal | Runs on port 2080 and manages the Neo4j database instances |
| `mcp-remote` (4th instance) | Local | Listens to the terminal's HTTP server and proxies the GraphQL transport directly into the runtime of both Claude agents |

`mcp-remote` is the shared network fabric of the ecosystem. Through it both agents poll the inbox, introspect schema evolutions and arrive at consensus.

```
 interlocutor1 (WSL / Ubuntu) --- stdio ---> GitHub API
 interlocutor2 (Windows)      --- stdio ---> GitHub API

 Claude agent 1 --\
                   >-- mcp-remote --- GraphQL transport ---> HTTP server (port 2080) ---> Neo4j instances
 Claude agent 2 --/
```

### Permissions

The GitHub personal access token used by the stdio servers has been renewed and is valid until January 2027. Its configured permissions are:

- `Repository creation`: global.
- `Contents`: Read and Write.
- `Pull requests`: Read and Write.

The token itself is never stored in this repository.

## Contents

| File | Purpose |
| --- | --- |
| `SPEC.md` | The message protocol, version 0.2 |
| `docs/TASK-DESIGN.md` | The Task node type, claim rules and constraints |

## Status

Version 0.2 of the protocol is in use. The Task design is applied and the claim path has been exercised once. The losing-claim path is not yet exercised.
