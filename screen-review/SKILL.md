---
name: screen-review
description: Captures every screen and state of a feature or of the current branch's changes on a simulator/device, publishes them as a screenshot review board (draw + comment per card), then watches the board and acts on feedback. Use when the user says "screen review", "capture all screens", "screenshot every screen", "review the branch screens", or "screenshot review board".
---

# Screen review

Board = one Artifact page of screenshot cards backed by a shared `feedback` db. The user draws and comments per card; a cron loop answers, designs, or ships changes and recaptures.

Work folder `$W` = `<session scratchpad>/shots/` holding `review.html`, `manifest.json`, `img/`, `processed.json`.

## 1. Scope

- **Feature** ("shared lists"): grep the feature's views/routers/view models.
- **Branch**: `git diff <base>...HEAD --stat` (base = develop/main, ask only if neither exists) and read every changed view file.

Write a numbered **capture plan** in chat before capturing: every screen × state. Walk this checklist for each screen: owner vs member, empty, loaded, error, premium vs free, filters, sheets, menus, confirmations, toasts, RTL/Arabic, dark mode (if the app supports it). Group into sections; those become manifest `section`s.

Done when: every changed/feature view appears in the plan at least once.

## 2. Capture (one background agent)

Spawn ONE background agent. Its brief contains:
- the numbered plan, `$W`, and the device: ask the user which device only if unclear (simulator by default).
- read the repo's `CLAUDE.md` for launch args, UI-test hosts, mock servers. A local mock server runs on a **private port** for captures unless the user wants the device they test on.
- capture commands and shrink step from [reference.md](reference.md#capture).
- output: `img/NN-name.jpg` (NN = 2-digit display order) + `manifest.json` = `[{file, section, title, state}]` in display order (`file` is `img/NN-name.jpg`).
- leave the device and app as found (language, flags, logged-in account, mock server stopped).
- report any plan item it could not reach, and why.

Done when: manifest lists every reachable plan item and each `file` exists.

## 3. Publish the board

1. `cp ~/.claude/skills/screen-review/assets/review.html $W/review.html`, then replace the placeholders (plain string replace, e.g. `sed -i ''`):

   | Placeholder | Example |
   |---|---|
   | `{{TITLE}}` | `Shared Lists Review` (2–4 words, tab title) |
   | `{{EYEBROW}}` | `PROJ-123 · iPhone 16 Pro · 19 Nov 2026` |
   | `{{HEADING}}` | `Shared lists: screen review` |
   | `{{LEDE}}` | `Every shared-list screen and state, captured on the iPhone 16 Pro.` |

   Card aspect ratio comes from the first image automatically.
2. Artifact publish: `file_path: $W/review.html`, `root: $W`, `files: [{path:"manifest.json"}, {path:"img/01-…jpg"}, …every image]`, `capabilities: {"db": {}}`, `icon: "phone"`.
3. Functional check: ArtifactData `list` on collection `feedback` (empty is fine; an error is not).
4. Give the user the link.

## 4. Watch & act

1. Write `$W/processed.json` = `{}` (doc id → last handled `submitted_at`).
2. CronCreate every 3 minutes with [assets/cron-prompt.md](assets/cron-prompt.md), placeholders filled. Tell the user: session-only, expires after 7 days.

The loop, per new doc (schema in [reference.md](reference.md#feedback-doc)), acts on the user's messages after the last `claude` message:
- **Question** → answer. **Unclear** → ask.
- **Options wanted** → publish a small variants artifact with 3–5 variants and a "My pick", post its link.
- **Change** → implement. Small: inline. Bigger: background agent whose brief includes build, install on the device the user tests on, recapture that exact screen to `img/<name>-v2.jpg`.

When a change ships:
1. `cp img/<name>.jpg img/<name>-v1.jpg`; new capture over `img/<name>.jpg` (same path keeps the card's thread).
2. Republish `review.html` with `root: $W`, `files: [{path:"img/<name>.jpg"}]`.
3. ArtifactData `update` the doc: full `messages` array + `{from:"claude", text, at}`, `reply` = same text, pinned with `if_version`.
4. Record in `processed.json`; give the user a 1–2 line summary.

**Builds are serialized**: one agent builds at a time, in a worktree. Queue follow-ups to the running agent with SendMessage, or have a new agent wait until `pgrep -x xcodebuild` (or gradle) is empty.

## Rules

- Replies on cards: short, one or two sentences.
- Recapture after every shipped change.
- Commit only files you changed; pushing is the user's call.
