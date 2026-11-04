# Cowork inbox: the lanes' queues

Read by the owner's one Cowork scheduled task. Each line below names one architect lane's queue file.
The task opens every queue listed here, does each job whose status line reads exactly "status: ready"
under that file's own rules (read only, always), saves the results in the job's folder inside that
lane's inbox folder, and changes the job's status line to "done <date>" or "blocked <date>: <why>".

The chief of staff adds a line when a lane starts and removes it when a lane retires. The architects
never edit this file; each writes only its own QUEUE.md.

## Queues

- <Lane A> inbox/QUEUE.md (file id <id>; results go in "<Lane A> inbox/<folder>")
- <Lane B> inbox/QUEUE.md (file id <id>; results go in "<Lane B> inbox/<folder>")
