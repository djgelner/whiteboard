# Data model

The desk reads four things from the artifact's database. The chief of staff (or the ArtifactData tool acting for it) writes almost all of it. The page writes only the fields marked below. Access rule: `{ path: "", read: "view", write: "admin" }`, so everyone signed in reads, and only admins and the owner write.

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
| status | string | `open` or `done`. |
| createdAt | string | When it was raised. |
| doneAt | string or null | When it was closed. |

Written by the page: `status` and `doneAt`, when the owner presses "Mark done" or "Undo".

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
| issues | creates, updates, may delete old ones | `status`, `doneAt` |
| requests | handles pending ones, sets `declined` | creates new ones |
| meta/board | writes it every check-in | reads only |
