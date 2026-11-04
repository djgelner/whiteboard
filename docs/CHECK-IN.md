# Check-in procedure

The chief of staff runs this at every scheduled check-in. Keep a call cap in the prompt and stop when the steps are done. A check-in with nothing to act on ends within a few calls.

1. **Arm the next check-in first.** Create or update the one-shot trigger for the next check-in time, then write its id and time into `meta/board` (`fable.nextCheckinTriggerId`, `fable.nextCheckin`). Do this before anything else so a failed run does not break the chain.

2. **Read the board's pending requests.** List the `requests` collection for `status = pending`. For each, check the architect's trigger first: the page may have reached it before failing, so the tick might already read as requested. If it does not, apply it with the trigger tools: `cadence` sets the cron and enables the tick, `deactivate` disables it, `activate` enables it with the stored cron. Then update the architect document (`status`, `cron`, `cadence`, `updatedAt`) to match and mark the request `applied` with `result` and `appliedAt`. If you will not apply one, mark it `declined` and say why in `result`.

3. **Confirm the applied requests.** For requests marked `applied` since the last check-in, read the trigger and confirm it matches the architect document. Fix the document if they differ, and note the fix in the record.

4. **Read every lane's log.** For each architect, read the newest lines of its log file (the URL is `logUrl`). Note what shipped, what is open in a PR, what failed, and what the lane is waiting on. Read only what changed since the last check-in. Do not read the architect sessions themselves unless a log points at a problem.

5. **Prioritize.** Decide what each lane should do next. Move a lane to `as-needed` when its queue is thin, to `inactive` (and pause its tick) when it is done, and back to `scheduled` when work returns. Adjust cadences against the cost of a run: a lane with nothing to do should not fire eight times a day.

6. **Merge.** Merge pull requests whose checks pass and that fall under your merge rule. Leave a PR that needs the owner's word, and raise it as an issue.

7. **Refresh the board.**
   - `architects`: update `version`, `lastRun`, `lastRunAt`, `notes`, `status`, `cron`, `cadence` and `updatedAt` for every lane that changed.
   - `issues`: add what now waits on the owner as a decision, go-ahead or blocker, each with a recommendation. Close issues that were resolved. Remove old done ones so the list stays short.
   - `meta/board`: set `updatedAt`, and refresh the `fable` object (check-in schedule, next check-in, record link).
   Write each document only when it changed, one write at a time per document. Use a batch for several documents.

8. **Write the record.** Append a short entry to the continuity file: what shipped, what was merged, what changed on the board, what is open, and the cost of this check-in (subagents launched and their models, tool calls).

9. **Report to the owner.** Plain language, short. Say what shipped, what you changed, what it cost, and end with what waits on them, naming each issue by title with your recommendation. If nothing waits on them, say so.
