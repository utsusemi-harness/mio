# mio — Graph

Nodes, edges, and states of the traceability graph.

## Diagram

```mermaid
graph TD
  P[project]
  R[requirement]
  E[evidence]
  D[external document]
  EI[evidence implementation]
  I[implementation]

  R -->|"refines"| R
  R -->|"refines"| P
  E -->|"verifies"| R
  P -->|"references"| D
  R -->|"references"| D
  E -->|"references"| D
  EI -->|"implements"| E
  EI -.->|"exercises"| I
```

## Representation

The graph is computed from the nodes: each node's attributes hold the edges it is the subject of, and states are computed on top of them.

### Nodes

Every node is a Markdown file. The frontmatter holds the attributes below:

- `node`: its kind
- `id`: assigned by mio
- `target`: the artifact it stands for
- the edges it is the subject of (see [Edges](#edges))
- `created`, `updated`: when the node was first and last written
- `commit`: the commit the node is current as of; optional

Only implementation has no representation of its own: it is expressed as the `target`s an evidence implementation lists under `exercises`.

A `target` is the artifact the node stands for, either a `file` or a `uri`. A `file` has a `path` (repo-root-relative, `/`-separated, a file or a directory) and may have a `range` narrowing it to a `symbol` or to `lines` (`from` and `to`, inclusive).

The body is free prose; its first line is the node's subject, as in a git commit message, and a blank line separates it from the rest.

| Node | Is |
| --- | --- |
| project | the system; exactly one; what every requirement ultimately refines |
| requirement | a statement of intent |
| evidence | how one requirement is verified; belongs to exactly one requirement |
| external document | what a node references |
| evidence implementation | what implements an evidence: a test function, a checklist, an alert rule |
| implementation | what evidence implementations exercise: code, schema, configuration |

### Edges

An edge is written in the frontmatter of its subject, under the edge's name.

| Edge | Subject → object | Cardinality | Written as |
| --- | --- | --- | --- |
| refines | requirement → requirement \| project | exactly one per requirement; acyclic, reaching the project | one `id` |
| verifies | evidence → requirement | exactly one per evidence | one `id` |
| implements | evidence implementation → evidence | many-to-many | a list of `id` |
| exercises | evidence implementation → implementation | one-to-many | a list of `target` |
| references | project \| requirement \| evidence → external document | many-to-many | a list of `id` |

### Examples

<details>
<summary>project</summary>

```markdown
---
node: project
id: P-4hd9w0sz
created: 2026-09-01T09:00:00Z
updated: 2026-09-06T01:10:00Z
commit: a1c9e2f
---
example-service

Issues and validates session tokens for the storefront.
```

</details>

<details>
<summary>requirement</summary>

```markdown
---
node: requirement
id: R-7f3kq2m8
refines: P-4hd9w0sz
references:
  - D-9sk2mq7e
created: 2026-09-01T09:00:00Z
updated: 2026-09-06T01:10:00Z
commit: a1c9e2f
---
Expired session tokens are rejected

A session token whose expiry has passed is rejected on every authenticated
endpoint. No session is created and no refresh is issued.
```

</details>

<details>
<summary>evidence</summary>

```markdown
---
node: evidence
id: E-q2m8x1y2
verifies: R-7f3kq2m8
created: 2026-09-01T09:00:00Z
updated: 2026-09-06T01:10:00Z
commit: a1c9e2f
---
A request with an expired token receives 401

Send a request carrying a token whose exp is in the past. Expect 401, no
session row, and no refresh token in the response.
```

</details>

<details>
<summary>external document</summary>

```markdown
---
node: external document
id: D-9sk2mq7e
target:
  uri: https://wiki.example/security-review-2026-07
created: 2026-09-01T09:00:00Z
updated: 2026-09-06T01:10:00Z
commit: a1c9e2f
---
Security review, July 2026

Section 3 covers session expiry.
```

`target` is `file` or `uri`.

</details>

<details>
<summary>evidence implementation</summary>

```markdown
---
node: evidence implementation
id: T-3nq8vz5c
target:
  file:
    path: tests/auth_test.rs
    range:
      symbol: rejects_expired_token
implements:
  - E-q2m8x1y2
exercises:
  - file:
      path: src/auth/session.rs
      range:
        symbol: validate
  - file:
      path: src/auth/session.rs
      range:
        symbol: is_expired
created: 2026-09-01T09:00:00Z
updated: 2026-09-06T01:10:00Z
commit: a1c9e2f
---
rejects_expired_token

Builds a token with exp = now - 1s, calls /me, asserts 401 and an empty
sessions table.
```

`range` is optional; without it the target is the whole file.

</details>

## States

States are computed, never stored. Subject side: present participle. Object side: past participle. Negation: `not`.

| Node | State | Holds when |
| --- | --- | --- |
| any node with a target | resolving | the artifact exists at HEAD; `uri` is not checked |
| evidence implementation | implementing | some implements edge has it as subject |
| | exercising | some exercises edge has it as subject |
| evidence | verifying | the requirement it verifies exists |
| | implemented | some implements edge has it as object, and every such subject is resolving |
| requirement | refining | the node it refines exists; the project alone is not refining |
| | refined | some requirement refines it |
| | verified | some evidence verifies it |
| | traced | every evidence verifying it is implemented and every requirement refining it is traced |
| implementation | exercised | some exercises edge has it as object |

`check` reports the cause of a negative state.

Artifact-caused changes only lower a state.
