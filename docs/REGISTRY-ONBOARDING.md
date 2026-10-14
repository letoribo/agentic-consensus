# GitHub MCP Registry onboarding record

Status: record of one onboarding request and one verification. Documentation, not part of the protocol specification and not an amendment. Tracked as task-0008.

## The request

| Field | Value |
| --- | --- |
| Request | GitHub discussion number 2987, the onboarding request for the GitHub MCP Registry |
| Server | mcp-graphql-enhanced, a federated GraphQL server for the Model Context Protocol |
| Address of the discussion | Omitted on purpose, because the content rule of this repository excludes URLs |
| Endorsement link | Confirmed posted; explicit URLs omitted to satisfy the v3-A3 Content Rule |
| Deployment confirmation state | Confirmed |

## What was verified

| Check | Result |
| --- | --- |
| Subject | The default schema overview of the server, called against the GitHub GraphQL API node |
| Date | 2026-10-11 |
| Verified by | interlocutor1 |
| Overview size | 54,935 characters |
| Type names in the overview | 1,829, counted as the comma-separated entries of its entity line |
| Layout | A flat list of capability names followed by a flat list of type names. No type bodies are included |
| Per-type detail | Fetched separately by naming the types, with a depth limit |

Notes on scope:

- The count is a point-in-time figure for the schema as served that day. It changes when the schema changes.
- The unscoped overview exceeded the result-size limit of the agent client that called it, and the client saved it to a file. This is a limit of that client, not of the server, and it is the reason scoped calls are preferred.
- Only the figures in the table were measured. Entity counts quoted for this server elsewhere were not reproduced, and no timing or comparison with other servers was measured here.
- During the same session the graph gateway behind the server returned generic connector errors on repeated calls and needed restarts. It is not known whether the cause was the server or its backend. A later server commit changed its response payloads; whether it affects these errors has not been tested.

## Related records

- task-0008 holds this record.
- The endorsement text itself is not stored in this repository.
