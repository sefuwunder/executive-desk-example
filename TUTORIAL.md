# A day on the executive desk — example workflow

## What this is

A worked example of the proposed acme tag verbs — **Look, Approve, Done,
Note, Fanout** — on a small fictional workspace. The data is invented
(Harbor Café, Northwind, Maya Chen, Priya Nair — all `.example`
addresses), and the verbs themselves are **not implemented yet**: this
shows what you would be presented with, and what each verb would do,
so the design can be judged before anything is wired up.

No servers are needed. `desk/` mirrors the real mounted layout, with
`desk/` standing in for `/n/`:

| Here | On the real desk |
| ---- | ---------------- |
| `desk/cadence/` | `/n/cadence` |
| `desk/comms/relay/` | `/n/comms/relay` |
| `desk/comms/outpost/` | `/n/comms/outpost` |

Browse it with plain tools (`cat`, `grep`) or open the files in acme.
In acme the verb words are not in the tags yet — that is the unbuilt
part. Each step below prints the tag row the window would carry, and
the file changes the verb would make. The workspace itself is frozen
at the morning state; everything after that is shown as before/after
text, so you can compare against the files.

The example day is Thursday, 2026-10-29. The one interaction rule
throughout, from the guide: **tags are verbs**. Middle-click a word
to run it. To act on something, select it first, then middle-click —
the selection is the argument, the tag word is the verb. Each window
shows only the verbs that apply to what it holds.

## Morning: Today, then approvals

### 1. Open Today

On the real desk this is what `desk acme` opens:
`/n/cadence/today`. Here, open `desk/cadence/today`.

```
Northwind renewal — Confirm renewal pricing (owner: you, week 44)
Northwind renewal — Collect signed renewal (owner: Priya Nair, week 43, quiet 6 days)
```

Tag row the window would carry:

```
today  Look Done Note Fanout
```

Two steps, one active run, nothing else due. The Harbor Café work has
not started yet — it is still a draft.

### 2. Check staging, approve the real one

Open the first draft, `desk/cadence/staging/relay-t1/`. Its field files
are exactly what the translator writes: `template`, `thread`,
`contact`, `excerpt`, `created`. There is no `status` file yet — a
draft has no status until you decide it.

```
template: Client onboarding
thread:   relay:t1
contact:  Maya Chen
excerpt:  Hi — we just signed the lease on our second location and
          would love to bring you on for the wholesale onboarding.
          Can we start this week?
created:  2026-10-28T16:40:00Z
```

Tag row the window would carry:

```
relay-t1  Look Approve Reject
```

Before approving, check the source. Select `relay:t1` in the draft
window and middle-click **Look** — the plumber routes the thread id
to its window, `desk/comms/relay/threads/t1/`, where Maya's message
sits in `messages/m1/body`. It matches the excerpt. Real conversation,
real person, first-seen thread. Approve it: middle-click **Approve**.

What Approve does, underneath: it writes to the draft's virtual
`approve` control file (relay and outpost trees are read-only, so
every cadence write goes through a named control file, never a blind
update). The run is created from the template, an event is logged,
and the draft gains two files:

```
desk/cadence/staging/relay-t1/status   →  approved
desk/cadence/staging/relay-t1/result   →  harbor-onboarding
```

And the new run appears in the tree:

```
desk/cadence/runs/harbor-onboarding/
  name        →  Harbor Café — Client onboarding
  playbook    →  Client onboarding
  start_date  →  2026-10-29
  status      →  active
  steps/h1/   →  title: Send welcome packet, role: you, week: 44, done: 0
  steps/h2/   →  title: Confirm tasting date, role: Maya Chen, week: 44, done: 0
                 links/l1: kind: thread, ref: relay:t1
  steps/h3/   →  title: Set up wholesale billing, role: you, week: 45, done: 0
```

The draft directory stays put after approval. That is deliberate:
the decision itself is the audit record.

### 3. Reject the stale one

Open `desk/cadence/staging/outpost-t9/`. Same shape, different story:
a renewal thread from Maya Chen — but Harbor Café has no renewal,
because Harbor Café is not a client yet. The onboarding draft you
just approved is the real work; this one is stale. Middle-click
**Reject**.

```
desk/cadence/staging/outpost-t9/status  →  rejected
```

The draft stays as an audit record, and — this is the rule that
makes rejection safe — a rejected draft does **not** re-draft on the
next message. The translator only drafts first-seen threads, so
rejecting is final unless you change your mind and approve it later
from the same directory.

One honest gap in the proposal, visible right here: the guide's tag
list is Look, Approve, Done, Note, Fanout, but its morning flow says
"reject the rest." Reject has a control file and a rule, but no tag
word in the guide's list. This example puts Reject on draft windows
because the flow cannot work without it — that word is the example's
addition, not the guide's.

Staging is now decided: one approved, one rejected, nothing left you
have not looked at. Morning done — go code.

## Midday: replies move the work

### 4. A confirming reply completes a step by itself

During the day a new message lands in the Harbor thread,
`desk/comms/relay/threads/t1/messages/m2/body`:

```
Thursday works for the tasting — see you at 10.
```

Step h2, "Confirm tasting date," is tied to that thread (its
`links/l1` says so). The desk reads the confirmation, and the step
completes on its own — no hand-marking. The message itself becomes
the proof:

```
desk/cadence/runs/harbor-onboarding/steps/h2/done       →  1
desk/cadence/runs/harbor-onboarding/steps/h2/links/l2/
  kind   →  message
  ref    →  relay:t1/m2
  label  →  Maya confirms Thursday tasting
```

If you ever wonder "why is this step done?", the answer is one Look
away, in the link.

### 5. Done, by hand

Not everything arrives with proof. You confirm the Northwind pricing
on a phone call — no message, nothing for the desk to read. Open the
step window, `desk/cadence/runs/northwind-renewal/steps/n1/`, and
middle-click **Done**:

```
desk/cadence/runs/northwind-renewal/steps/n1/done  →  1
```

Tag row the step window would carry:

```
n1  Look Done Note Fanout
```

Your word wins over any automatic reading: a step you mark Done
stays Done, and a step the desk completed can be corrected the same
way.

### 6. Note, correcting the record

Step n2, "Collect signed renewal," is Priya's, and it has gone quiet
— see the next step for that. First, what you know: Priya is on leave
until Monday. Select that sentence (or type it in the step window)
and middle-click **Note**:

```
desk/cadence/runs/northwind-renewal/steps/n2/links/l2/
  kind   →  note
  ref    →  (your words, stored with the step)
  label  →  Priya is on leave until Monday — re-time to next week
```

plus an Activity line (see the evening section). A note from you
overrules an automatic reading, and notes are always editable.

Second honest gap: the proposal never says where a Note is stored.
A step link of kind `note` is this example's chosen mapping, because
links are the step's existing "proof attached" mechanism. If Note
should live somewhere else — its own file, or Activity only — that is
still an open design point.

## Afternoon: Fanout on the quiet step

### 7. Fanout

Step n2 is a week 43 step, six days quiet, tied to
`desk/comms/outpost/threads/t5/` (its `links/l1` says
`outpost:t5`), where Priya's last message — "I will send the signed
copy by end of week" — is dated 2026-10-23. Nothing has arrived
since. Open the step window and middle-click **Fanout**.

Fanout sends a follow-up **in the thread the step is tied to**, not
in a separate tool:

```
Hi Priya — checking in on the signed renewal copy. No rush if you're
still working through it, just let me know where it stands.
```

Two rules from the guide, both visible in what Fanout does not do:
follow-ups are staggered, so three overdue steps do not become three
simultaneous messages; and nothing goes out between 10 p.m. and
7 a.m. Fanout is itself the approval — the guide's rule is that
nothing sends without your approval, and middle-clicking Fanout is
that approval — so it sends on the click, with no second confirmation
dialog. If that should instead confirm first, that is a third open
design point.

## Evening: Activity, then close clean

### 8. Skim Activity

`desk/cadence/activity`, newest first, at the end of the example day:

```
2026-10-29T15:05:00Z  fanout sent on n2 (Collect signed renewal) in outpost:t5
2026-10-29T13:20:00Z  note added on n2: Priya is on leave until Monday — re-time to next week
2026-10-29T11:47:00Z  step n1 done by hand (Confirm renewal pricing)
2026-10-29T10:12:00Z  step h2 done by message relay:t1/m2 (Confirm tasting date)
2026-10-29T09:03:00Z  run harbor-onboarding started from draft relay-t1
2026-10-29T09:02:00Z  draft outpost-t9 rejected
2026-10-28T16:40:00Z  draft relay-t1 staged (Harbor Café onboarding)
2026-10-28T16:40:00Z  draft outpost-t9 staged (Harbor Café renewal)
2026-10-27T09:15:00Z  step n1 opened on Northwind renewal (Confirm renewal pricing)
2026-10-26T11:02:00Z  run northwind-renewal started from template Renewal
```

The morning's `desk/cadence/activity` in this workspace is the
before-state: it ends at the 2026-10-28 staging lines. Everything
above those lines is what the day's verbs added.

Close clean: code pushed, drafts decided, nothing left in staging
you have not looked at. The rejected draft staying in its directory
is not unfinished business — it is the audit trail.

## What the verbs touch, in one table

| Verb | Window it belongs to | What it writes | What it never does |
| ---- | -------------------- | -------------- | ------------------ |
| Look | any | nothing — routes an id, path, or URL to its window | alters a file |
| Approve | staging draft | draft `approve` control file → run created, `status` = approved, `result` = run id, event logged | starts anything unapproved |
| Reject | staging draft | draft `reject` control file → `status` = rejected, kept as audit | re-drafts later |
| Done | step (or a selected step line in Today) | step `done` = 1, event logged | deletes or moves the step |
| Note | step, thread | step link (kind `note`) + Activity line | overrules nothing but your own earlier note |
| Fanout | step, thread | a message in the tied thread, staggered, never 22:00–07:00 | sends without your click |

Relay and outpost trees take no writes at all in this design — their
servers refuse every write — so a verb can never alter a thread.
Cadence's only writable files are step `done` flags and the staging
shelf.

## Status of this example

This is a mock of a proposal, not a feature. The built desk today is
mounts plus launch (`desk acme` opens `/n/cadence/today`); the tag
verbs, the plumber routing for desk ids, and the control-file
wiring behind Approve/Done/Note/Fanout are not built, and nothing
here asks for them to be. What this example settles is the
presentation: the tag rows, the click order, and the file effects.
The three open points it surfaced — Reject's missing tag word,
Note's storage, Fanout's confirmation — are the decisions still owed
before any implementation starts.
