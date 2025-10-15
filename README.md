# pp

https://excalidraw.com/#room=6aed08aa7f45319c1053,jrEIXohncITvu-_MJSe5GQ


## Concept
1) Subscriber (the person who needs files)

Think: “I just want my files.”

What they can do:

See my groups (like “risk”, “finance”) and only the files for those groups.

Search & filter files by day / week / month / custom dates.

Open / download a file.

⭐ Favorite a file to find it faster next time.

Recent: see what I opened last time in one click.

Ask to join a group: “Please add me to risk” (sends a request to Admin).

What they cannot do:

Change other people’s access.

Move or upload files inside RW Tool (that happens automatically).

What the screen looks like:

Left side: groups I belong to.

Big list: files for the selected group, with Download buttons.

Small sections: Favorites, Recents, Date filter, Search box.

2) Admin (the gatekeeper)

Think: “I keep the house tidy and safe.”

What they can do:

Approve / reject requests like “Add me to risk group.”

Add / remove people from groups.

(Optional later) Tag / organize the file catalog so it’s easy to find.

What they cannot do (by default):

They don’t move files around; that’s the Ops robot’s job.

What the screen looks like:

Requests list with Approve / Reject buttons.

Members of each group.

(Optional) A simple Settings page for group labels and paths.

3) Ops (the robot helper + a tiny dashboard)

Think: “I make sure files arrive safely. Humans mostly just watch.”

There are two parts to Ops:

A) The robot helper (Java scheduler) — runs in the background

Listens for a message from outside systems (like “Risk”):
“Hey, file A.doc is ready!”
That message is a simple web call: POST /ingest/notify.

Also watches folders/FTP that we already know (“risk inbox folder”).

When it sees a new file, it:

Picks it up from the outside place.

Moves it into RW Tool’s safe storage.

Writes a note in the database:

file name, size, when it arrived, which group it belongs to

a short event like “UPLOAD”, “MOVED”, or “ERROR”

If something breaks (e.g., network hiccup), it tries again a few times.

This robot is the “smart Ops guy” who never sleeps.

B) The Ops dashboard (a simple web page for humans)

What you see:

KPIs (little boxes):
New Today, Failed Moves, Downloads Today, Total Stored

Storage connections: are AWS/Azure/Google storage working? (green = ok)

Recent syncs: a short list like:

“AWS S3 – Finance” → Completed

“Google Cloud – Reports” → In progress

“Local – Archive” → Failed (has a Retry button)

What humans can do here:

Mostly look and feel safe that the robot is working.

If a row says Failed, click Retry (rare).

That’s it—no manual file moving.

How Ops works end-to-end (super simple story)

Outside world (Risk) finishes a file and tells RW Tool:
“Hey boss, A.doc is uploaded.” (It calls the REST API, or the robot finds it in a watched folder.)

Robot helper picks up the file:

checks it’s real (non-empty),

gives it a fingerprint (checksum so we don’t double-process),

moves it into RW storage,

writes down what happened (events).

Subscriber in the risk group opens the app and sees the file.
They click Download → we add a DOWNLOAD event.

Admin only steps in if someone asks for access or if a policy says “approve this file”.

That’s all. No manual babysitting.

One-line data flow (words a child can read)
Outside system says “file is ready”  →  Robot stores it safely  →  App shows it

What if something goes wrong?

The robot marks the file with ERROR and writes the reason (like “can’t reach FTP”).

On the Ops dashboard, humans see Failed Moves > 0 and a Retry button.

HTTP codes stay standard (e.g., 403 means “you’re not allowed”).

If we need our own internal code too, we add it inside the JSON, like:

{ "ok": false, "error": "Permission denied", "code": "RW1403" }

The tiny set of tables we care about (in friendly words)

file_record — one row per file we store (name, where, status, when).

file_event — a running diary for each file (UPLOAD, MOVE, DOWNLOAD, ERROR).

group — the teams like “risk”, “finance”.

file_group — which files belong to which teams.

user — who you are (admin, subscriber, ops, or bot).

user_group — which teams you belong to.

group_access_request — “please add me to risk” + Admin’s decision.

intake_queue — the robot’s to-do list when outside systems notify us.

(You don’t need to remember the names—just know we keep tidy lists.)

What each role sees on their first day

Subscriber:
“I see Risk on the left, a list of files on the right, a Download button, and a filter for dates. I can click ⭐ to favorite.”

Admin:
“I see a list of requests. I press Approve or Reject. I can add people to Risk. Easy.”

Ops (dashboard):
“Four numbers at the top, storage health, and a short list of latest syncs. If something failed, I press Retry. Mostly, I relax.”

Why this is minimal & effective

The robot does the work, not people.

The screens are tiny and speak plain language.

Groups make sure people only see what they should.

Events give us a clear history without digging.

If you want, I can also give you:

a checklist to demo the flow (with example cURL for /ingest/notify),

or a one-pager diagram (PNG/SVG) you can paste in your README.
