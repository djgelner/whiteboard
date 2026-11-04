# Data model

The desk reads these collections from the artifact's database: `architects`, `issues`, `requests`, `links`, `ideas`, `decisions` and the single document `meta/board`. The chief of staff (or the ArtifactData tool acting for it) writes almost all of it. The page writes only the fields marked below. Access rule: `{ path: "", read: "view", write: "admin" }`, so everyone signed in reads, and only admins and the owner write.

Timestamps are ISO 8601 strings in UTC. Document ids use letters, digits and `- _ . ~ : @ +` only.

## architects/{id}

One document per lane.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. Used in the tab hash `#a-<id>`. |
| order | number | Sort order in the tab bar and table. |
| name | string | Display name, for example "Newsletter Architect". |
| lane | string | One line on what the lane does. |
| status | string | `scheduled`, `as-needed` or `inactive`. See below. |
| version | string | The architect's current version label, for example "v6". |
| model | string | The model the lane runs on, as text. |
| sessionId | string | The architect's session id. Empty hides live state. |
| sessionUrl | string | Link that opens the live session. |
| repo | string | Repository name, as text. |
| repoUrl | string | Link to the repository. |
| logUrl | string | Link to the lane's log file. |
| tickId | string | The scheduled tick's trigger id. Empty disables cadence and resume controls. |
| cron | string | The tick's cron, five fields, UTC. Kept when paused so it can be resumed. |
| cadence | string | The cron in words, or a note such as "Paused 2026-01-10 by the owner from the Desk". |
| whiteboardUrl | string | The lane's own whiteboard page, or empty. |
| whiteboardTitle | string | Its title. |
| liveUrl | string | The lane's public site, or empty. |
| lastRun | string | One or two sentences on the last run. |
| lastRunAt | string | When it finished. |
| notes | string | Standing notes for the owner. |
| merges | string | The merge rule for this lane's PRs. |
| updatedAt | string | When this document last changed. |

Written by the page: `status`, `cron`, `cadence`, `updatedAt`, after a successful tick change.

Status values:

- `scheduled`: the tick fires on its cron. Shown as a tab.
- `as-needed`: the tick exists and runs, but the chief of staff expects it to be quiet at times. Shown as a tab.
- `inactive`: the tick is paused or the lane is finished. Listed on the Inactive tab.

## issues/{id}

What waits on the owner.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. |
| order | number | Sort order within its group. |
| kind | string | `decision`, `go-ahead` or `blocker`. |
| lane | string | Which lane it concerns, shown as a chip. |
| title | string | The short ask. |
| body | string | The facts the owner needs. |
| recommendation | string | The chief of staff's recommendation. |
| status | string | `open`, `accepted`, `custom`, `declined` or `done`. See below. |
| createdAt | string | When it was raised. |
| decidedAt | string or null | When the owner decided. |
| decidedBy | string or null | `owner` when the page wrote the decision. |
| note | string or null | The owner's own words, when the status is `custom`. |
| doneAt | string or null | When the chief of staff closed it as done. |

Status values:

- `open`: waiting on the owner. Shown with the buttons "Accept recommendation", "Something else" and "Decline".
- `accepted`: the owner accepted the recommendation. The chief of staff acts on it and sets `done`.
- `custom`: the owner wants something else and wrote it in `note`. The chief of staff reads the note, acts or asks, and sets `done`.
- `declined`: the owner said no. The chief of staff closes it at the next check-in.
- `done`: handled. Shown under "Decided".

Written by the page: `status`, `decidedAt`, `decidedBy` and `note` when the owner decides, and all four cleared back to `open` on "Undo". The chief of staff writes `done` and `doneAt`.

## requests/{id}

One document per action the owner takes on the page. The id is `<type>-<architectId>-<epoch ms>`.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. |
| architectId | string | The lane the request is about. |
| type | string | `cadence`, `deactivate` or `activate`. |
| value | string | The cron for `cadence` and `activate`; empty for `deactivate`. |
| note | string | A readable summary of the ask. |
| requestedAt | string | When the owner pressed the button. |
| status | string | `applied`, `pending` or `declined`. |
| result | string | What the tick now reads (applied), or why it is queued (pending). |
| appliedAt | string | Present when status is `applied`. |

Status values:

- `applied`: the page changed the tick through the connector and updated the architect document.
- `pending`: the connector was unavailable or the call failed. The chief of staff applies it at the next check-in. A failed call may still have run, so check the trigger before repeating it.
- `declined`: set by the chief of staff when it decides not to apply a request. Put the reason in `result`.

Written by the page: creates the document. The chief of staff updates `status`, `result` and `appliedAt` after handling a pending one.

## links/{id}

The owner's frequently used links, shown on the Links tab and as a strip of the first six on Home. The chief of staff adds what the owner asks for repeatedly. The owner can also add one on the Links tab.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. The page makes it from the label plus a short random suffix. |
| group | string | Free text heading, for example `Record`, `Sites`, `Sales`, `Money`, `Lanes`, `Tools`. |
| label | string | The link text. |
| url | string | An https address. Anything else is not rendered as a link. |
| note | string | One line under the label. |
| order | number | Sort order across the whole tab; groups appear in the order of their first link. |
| askedFor | number | Optional. How often the owner asked for it. |

Written by the page: creates documents from the "Add link" form (owner and admins). The chief of staff edits and removes them.

## ideas/{id}

Proposals the chief of staff wants a decision on, shown on the Ideas tab in three groups.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. |
| kind | string | `workflow` (a repeated task worth saving), `system` or `revenue`. |
| lane | string | Which lane it concerns, shown as a chip. |
| title | string | The short proposal. |
| body | string | The facts. |
| recommendation | string | The chief of staff's recommendation. |
| status | string | `open`, `accepted`, `custom`, `declined` or `done`, with the same meanings as for issues. |
| note | string | The owner's words when `custom`. |
| decidedAt, decidedBy | string | Written by the page when the owner decides. |
| order | number | Sort order within its group. |
| createdAt | string | When it was raised. |
| result | string | Optional. What the chief of staff did, written when it marks the idea `done`. |

Written by the page: the same decision fields as issues. The chief of staff creates ideas and writes `done` and `result`.

## decisions/{id}

An audit record of every decision the owner makes on the page. The id is `<collection>-<docId>-<epoch ms>`.

| Field | Type | Meaning |
|---|---|---|
| id | string | Same as the document id. |
| collection | string | `issues` or `ideas`. |
| docId | string | The id of the issue or idea. |
| title | string | Its title at the time. |
| status | string | `accepted`, `custom` or `declined`. |
| note | string | The owner's words for `custom`, otherwise empty. |
| decidedAt | string | When the owner pressed the button. |
| decidedBy | string | `owner`. |
| notified | boolean | True when the page managed to message the chief of staff's session at once. |

Written by the page only. The chief of staff reads it at check-ins. The page shows only a count on Home: decisions made since `meta/board.updatedAt`.

## meta/board

A single document with the board header. Read only for the page.

| Field | Type | Meaning |
|---|---|---|
| updatedAt | string | When the chief of staff last refreshed the board. |
| fable | object | The chief of staff. The key is named `fable` for historical reasons; keep it. |

The `fable` object:

| Field | Type | Meaning |
|---|---|---|
| name | string | Display name. |
| role | string | One line on its job. |
| sessionId | string | Its session id. |
| sessionUrl | string | Link that opens the live session. |
| checkins | string | The check-in schedule in words. |
| nextCheckin | string | When the next check-in fires. |
| nextCheckinTriggerId | string | The trigger that will fire it. |
| recordUrl | string | Link to the continuity record. |
| coSession | object | Optional: `{ name, sessionId, sessionUrl }` for a helper session that merges some lanes. |

## Who writes what

| Data | Chief of staff | Page (owner or admin) |
|---|---|---|
| architects | creates, updates everything | `status`, `cron`, `cadence`, `updatedAt` |
| issues | creates, updates, sets `done`, may delete old ones | `status`, `decidedAt`, `decidedBy`, `note` |
| requests | handles pending ones, sets `declined` | creates new ones |
| links | creates, edits, removes | creates new ones |
| ideas | creates, sets `done` and `result` | `status`, `decidedAt`, `decidedBy`, `note` |
| decisions | reads at check-ins | creates one per decision |
| meta/board | writes it every check-in | reads only |
