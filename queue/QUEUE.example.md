# <Lane> Cowork queue

Written by the <Lane> Architect (the lane in <owner>/<repo>). Read by the owner's Cowork scheduled
task, which runs each job below whose status is "ready", saves the results in the job's folder inside
"<Lane> inbox", and changes the status line to "done <date>" (or "blocked <date>: <why>").

Only READ-ONLY jobs go in this file: open public pages, download public files, read and summarize.
Cowork never submits a form, signs in to anything, buys, posts, sends a message or emails anyone for a
job in this file. A job that needs any of that goes on the whiteboard for the owner to fire by hand.

A job is a section in this form (copy it, change the slug, folder and steps):

    ## Job <slug>

    status: ready
    folder: <slug>
    added: <date>

    <one line on what the job is for and why the lane cannot do it itself>

    1. <step, with the exact URL>
    2. <step>

    Save everything in the folder "<Lane> inbox/<slug>". Add RESULTS.md there with: each file's
    name and URL; <the facts the lane needs, named one by one>.

---

## Job example-official-forms

status: ready
folder: example-official-forms
added: 2026-11-04

The state court site returns 403 to the lane's server, even from a headless browser. Read only.

1. Open <the public page> and download every PDF it links.
2. Save the page's own text as a text file.

Save everything in the folder "<Lane> inbox/example-official-forms". Add RESULTS.md there with: each
file's name, URL and SHA-256 if you can compute it; the exact sentence that states the form's
effective date.
