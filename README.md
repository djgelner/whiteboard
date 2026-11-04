# Architect Desk

A one-screen control board for a chief-of-staff AI session that runs several architect AI sessions. It is a single HTML page you publish as a claude.ai artifact. The owner opens it, sees every lane at a glance, reads what waits on them, and changes a schedule or pauses a lane with two clicks.

## The pattern

One **chief of staff** session plans, prioritizes, merges and rotates the work. It is the only session that talks to the owner. Beneath it sit **architect** sessions, one per lane (a product, a site, a newsletter, a research thread). Each architect is woken by a **scheduled tick**, a cron routine that starts one run. A run does one deliverable, opens a pull request, writes a line in the lane's **log file**, and stops. The log is the only channel up: the architect never pings anyone, and the chief of staff reads the logs at its own check-ins.

Architects are PR-only. They do not merge, send, publish or pay. The chief of staff merges what passes, or the owner does, in the lane's own chat. When a lane has run its course, the chief of staff pauses its tick and the session stays on the shelf.

The desk page is the owner's one screen onto all of this. The chief of staff rewrites its data at every check-in.

## What the board shows and does

- **Home.** The chief of staff card (live session link, check-in schedule, next check-in, the record, the co-session). Everything waiting on the owner, grouped into decisions, go-aheads and blockers, each with the chief of staff's recommendation and the three decision buttons (Accept recommendation, Something else, Decline) and a collapsed "Decided" list with Undo. A strip of the owner's most used links. A table of current architects with live state (working, idle or blocked), context used, cadence, next run and links. A list of pending requests.
- **One tab per active architect.** Header with version, model, status and live state, recent action and cost. Links to the live session, whiteboard, repo, log and live site. The cadence section with a preset-or-custom cron editor ("Apply cadence"), and a "Pause lane" button that asks for confirmation in the page. Notes and the merge rule.
- **Links.** Grouped links the owner asks for repeatedly, with an owner-only form to add one.
- **Ideas.** Workflows for repeated tasks, systems and revenue ideas, each with a recommendation and the same decision buttons.
- **Inactive.** A card for every paused or finished lane, with its last run, stored cron and links, and a "Resume lane" button that confirms first.

Everything the owner does is recorded as a **request** in the database. If the live connection worked, the request is marked applied. If not, it is marked pending and the chief of staff applies it at the next check-in. Nothing is lost either way.

## Set up in Claude

1. **Publish `board.html` as a claude.ai artifact.** Declare these capabilities when you publish:
   - `db`: `{ rules: [ { path: "", read: "view", write: "admin" } ] }` so everyone signed in reads and only admins and the owner write.
   - `user`: `{}`
   - `mcp`: `{ servers: [ { server: "Claude Code Remote", tools: ["get_trigger", "update_trigger", "get_session", "send_message"] } ] }`
2. **Seed the database.** Copy `seed.example.json`, replace the example architects, issues and meta with your own, and write each document into the artifact's database with the ArtifactData tool: one document per architect in `architects`, one per issue in `issues`, and the single `meta/board` document. Leave `requests` empty. The page never contains your data; it only reads the database.
3. **Pin the artifact** so it sits in your sidebar.
4. **Give every new chief-of-staff session one rule:** open the desk on its first turn, and refresh it at every check-in. `docs/CHECK-IN.md` is the procedure, and `prompts/chief-of-staff-checkin.md` is the prompt to arm each check-in with.

## One-click decisions and the notify path

Every open decision card, on Home and on the Ideas tab, has three buttons. **Accept recommendation** records that the owner agrees and tells the chief of staff to act. **Something else** opens a small dialog where the owner writes what should happen instead, in their own words. **Decline** closes the item. Decided items collapse under "Decided", each with its status, any note, and an Undo link.

Each decision updates the issue or idea and adds a record to `decisions`. The page then tries to wake the chief of staff at once: if the connector is available, it sends one message to the chief of staff's session with `send_message`, in the form `DESK: <kind> <collection>/<id> <status> by the owner at <time>. Title: <title>. Note: <note or none>.` The same message goes out after a cadence, pause or resume request. The toast says whether the chief of staff was notified. If the connector is missing or the call fails, the toast says the decision was recorded and will be read at the next check-in. The page never retries a message on its own. The page sends nothing else and pays nothing; a decision is only a record the chief of staff reads.

## Data model

Collections: `architects`, `issues`, `requests`, `links`, `ideas`, `decisions` and the single document `meta/board`. Field tables, status values and who writes what are in [docs/DATA-MODEL.md](docs/DATA-MODEL.md). The chief of staff writes almost everything. The page writes only an architect's status, cron, cadence and update time, the decision fields of an issue or idea, and new requests, links and decision records.

## What needs the Claude Code Remote connector

The live parts: session state and context use, the real next-run time, and the apply buttons that change a tick directly. They use four tools of the Claude Code Remote connector, `get_session`, `get_trigger`, `update_trigger` and `send_message`, with the viewer's own credentials. Without the connector (not granted, not connected, or a viewer outside your organization) the page still works fully from the database. Live values show a "from the record" marker, and cadence, pause and resume requests are queued as pending for the chief of staff to apply. If the connector fails mid-session, the page keeps the last good values and offers one "Try again" for reads. For writes it never retries on its own, because a failed call may still have run.

## The two prompts

`prompts/chief-of-staff-checkin.md` is what the chief of staff runs at each check-in; it re-arms the next one first, so a crashed run does not end the chain. `prompts/architect-tick.md` is what each architect's tick says: one deliverable, a PR, a log line, stop. Both use `<angle-bracket>` placeholders.

## Why this works

- **Tokens are money.** Every tool call re-reads the whole context. A tick that does one thing, in a small context, with a stated call cap, costs a known amount.
- **One deliverable per run.** One reviewed pull request is easy to check and easy to roll back.
- **The owner decides from one screen.** Decisions, go-aheads and blockers are written once, with a recommendation, and decided with one click. Cadence and pause are on the same screen, so cost control does not need a conversation.
- **Nothing is sent or paid by machine.** Architects draft and open PRs. A person sends the email, approves the spend and merges the risky change. The desk reflects that: it holds no money and sends nothing.

## License

MIT. See [LICENSE](LICENSE).

## Lane whiteboards

Each architect can have its own board, `lane-board.html`: one column, driven entirely by its database (a `meta/board` document with the lane's name, session link, repo and log; `items` with the same Accept, Something else and Decline buttons; `links` and `files`). Publish it once per lane with the `db` and `user` capabilities, seed `meta/board`, and give the lane's scheduled prompt two lines: open the board first, and at the end of the run add anything the owner needs to it. The Chief of Staff's desk links to each lane board from that architect's tab.
