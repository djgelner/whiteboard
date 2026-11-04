# Cowork hands: how an architect gets a browser without a person in the loop

An architect session runs in the cloud. Some pages refuse it: a court site that blocks automated
fetches, a page that only renders in the owner's own signed-in browser, a PDF behind a click. The
lane could stop and ask the owner. This pattern asks a scheduled Cowork task instead, and the owner
sets that task up exactly once.

## The pieces

1. **One inbox folder per lane** in the owner's Google Drive, named `<Lane> inbox`, holding one file,
   `QUEUE.md`. The architect writes jobs into it with the Google Drive connector. The job form is in
   [queue/QUEUE.example.md](../queue/QUEUE.example.md).
2. **One index file**, `Cowork inbox/LANES.md`, listing every lane's queue file. The chief of staff
   edits it when a lane starts or retires; architects never touch it.
   [queue/LANES.example.md](../queue/LANES.example.md) is the form.
3. **One Cowork scheduled task** on the owner's computer, three times a day, with the prompt in
   [prompts/cowork-scheduled-task.md](../prompts/cowork-scheduled-task.md). It opens LANES.md, then
   every queue it lists, does each job marked `status: ready`, saves the results in the job's folder
   inside that lane's inbox, and marks the job `done <date>` or `blocked <date>: <why>`.
4. **The architect reads the results** on a later run with the same Drive connector, and writes what it
   learned into its repo under its usual truth gate.

## The rule that keeps it safe

Only read-only work goes in a queue: open public pages, download public files, read and summarize.
The task prompt says so, every queue file says so at the top, and a job that asks for more is marked
blocked and skipped. Anything that would submit a form, sign in, buy, post or send is a different
path: the architect puts it on its whiteboard as an item with a `prompt` field (the exact text to
paste) and a `promptFor` field (`Cowork` or `Claude`). The board shows a **Copy prompt** button, and
the owner fires it by hand, with the same hard gates that apply to everything else.

## What the architect's tick says

One paragraph, added to each lane's tick prompt (see [prompts/architect-tick.md](../prompts/architect-tick.md)):
when a step needs a browser the session does not have, try the headless browser once; if that fails,
write a read-only job to the lane's queue file (folder id and file id named in the tick, so no search
is needed); if the Drive connector is missing from the session, write the job in the same form as a
line in the log and the chief of staff copies it into the queue. Everything else goes to the whiteboard
as a prompt item.

## Why it works

- The owner does one setup, not one paste per job.
- A new lane needs one new line in LANES.md, nothing on the owner's side.
- The queue file is the contract: a job's status line is the only state, and both sides can read it.
- Results land in a folder the architect already has access to, so nothing is pasted back by hand.
- The read-only boundary is written in three places (the task prompt, every queue file, the tick), and
  the only thing that crosses it is a prompt the owner copies and fires themself.
