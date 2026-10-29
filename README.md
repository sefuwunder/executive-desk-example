# Executive desk — example workspace and tutorial

A worked example of the proposed acme tag verbs (Look, Approve, Done,
Note, Fanout) on a small fictional workspace, so the design can be
judged before anything is wired up.

- **[TUTORIAL.md](TUTORIAL.md)** — a basic daily workflow: morning
  (Today, approve one draft, reject a stale one), midday (a reply
  completes a step, Done by hand, Note), afternoon (Fanout on a quiet
  step), evening (Activity, close clean). Each step prints the tag
  row the window would carry and the file changes the verb would
  make.
- **`desk/`** — the example workspace, frozen at the morning state.
  It mirrors the real mounted layout with `desk/` standing in for
  `/n/`: `desk/cadence/` is `/n/cadence`, `desk/comms/relay/` is
  `/n/comms/relay`, `desk/comms/outpost/` is `/n/comms/outpost`.
  File shapes match the real 9P trees: staging drafts carry
  `template, thread, contact, excerpt, created` plus virtual
  `approve`/`reject` control files; run steps carry
  `title, role, week, done, links/<link>/{kind, ref, label}`;
  threads carry `name, is_group, members, member_names,
  messages/<msg>/{channel, direction, subject, body, created,
  participants}`.

All people, businesses, and addresses are fictional (`.example`
domains). No servers are needed — browse with plain tools or open
the files in acme.

Status: this is a mock of a proposal, not a feature. The built desk
today is mounts plus launch; the tag verbs are not implemented.
The tutorial names the three open design points the example
surfaced: Reject's missing tag word, Note's storage, and whether
Fanout confirms before sending.
