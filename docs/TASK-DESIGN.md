# Task node design

Status: applied. The first claim (a task claimed through the generated GraphQL update mutation) succeeded. The losing-claim path has not yet been exercised.

## Purpose

Let two agents divide work without a central coordinator, with the database refusing double claims instead of relying on message conventions.

## Model

`Task` node:

| Property | Type | Meaning |
| --- | --- | --- |
| `id` | string, unique | `task-` plus a zero-padded number, for example `task-0001` |
| `title` | string | What the task is |
| `status` | string | `OPEN`, `CLAIMED` or `DONE` |
| `createdSeq` | integer | The seq of the message that created it |

Claim edge: `(Task)-[:CLAIMED_BY]->(Agent)`, carrying two properties:

| Edge property | Meaning |
| --- | --- |
| `taskId` | The id of the task being claimed |
| `claimedSeq` | The seq of the message announcing the claim |

## Constraints

```
CREATE CONSTRAINT task_id IF NOT EXISTS FOR (t:Task) REQUIRE t.id IS UNIQUE;
CREATE CONSTRAINT claim_once IF NOT EXISTS FOR ()-[c:CLAIMED_BY]-() REQUIRE c.taskId IS UNIQUE;
```

The second constraint is the central idea: every claim edge carries its task's id, so the database itself refuses a second claim on the same task, whichever agent sends it. It requires relationship property uniqueness constraints (Neo4j 5.7 or later).

## Conditional claim write

```
MATCH (t:Task {id: $taskId}), (a:Agent {id: $agentId})
WHERE t.status = 'OPEN'
CREATE (t)-[:CLAIMED_BY {taskId: $taskId, claimedSeq: $seq}]->(a)
SET t.status = 'CLAIMED'
RETURN t.id AS id, t.status AS status;
```

Through the generated GraphQL API, the same claim is one update with a filter on status `OPEN`, setting status to `CLAIMED` and connecting the claim edge with its two properties.

A claim is lost when the write either matches zero tasks (the status is no longer `OPEN`) or fails on the uniqueness constraint. Both outcomes mean the same thing: re-read the task and move on.

## Rules

- A task is append-only except for its status and its claim edge.
- Only the claimant changes the status of a claimed task, and only the claimant sets it to `DONE`.
- Tasks are created by the Cypher path first, so that the schema generator sees the label, its properties and the edge properties. Generated type names are then read from the schema, not assumed.
- After creating new labels, two refresh calls were needed before the new types appeared in introspection.
- A claim message in the conversation is an announcement; the claim edge is authoritative.
