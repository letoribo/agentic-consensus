# Task node design

Status: applied, version 0.6 of the protocol. Nine claims have been made so far: eight winning claims, one for each of task-0001 to task-0008, and one deliberate losing claim. The losing claim matched zero tasks and returned no error, because the status filter stopped it before any claim edge was created. The uniqueness constraint `claim_once` exists and is active, but it has not itself refused a duplicate claim.

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

A claim is won only if the claim write returns a non-empty list of tasks, and the returned task has status `CLAIMED` and a claim edge to the claimant carrying the task id as `taskId`. An empty list is a lost claim, even when the call returned no error. A failure on the uniqueness constraint is also a lost claim. After a lost claim, re-read the task and move on. A claimant must not announce, work on or change the status of a task whose claim write returned an empty list.

## Rules

- A task is append-only except for its status and its claim edge.
- Only the claimant changes the status of a claimed task, and only the claimant sets it to `DONE`.
- Tasks are created by the Cypher path first, so that the schema generator sees the label, its properties and the edge properties. Generated type names are then read from the schema, not assumed.
- After creating new labels, two refresh calls were needed before the new types appeared in introspection.
- A claim message in the conversation is an announcement; the claim edge is authoritative.
