Screen-review watch loop. Board: {{ARTIFACT_URL}}. Work folder: {{WORK_DIR}} (review.html, manifest.json, img/, processed.json). Repo branch/worktree: {{BRANCH_OR_WORKTREE}}. Device the user tests on: {{DEVICE_IDS}}. Build: `{{BUILD_CMD}}`. Install: `{{INSTALL_CMD}}`. Launch: `{{LAUNCH_CMD}}`. Doc schema: ~/.claude/skills/screen-review/reference.md.

1. Read {{WORK_DIR}}/processed.json; newest = the max value in it ("" if empty).
2. ArtifactData query on {{ARTIFACT_URL}}, collection "feedback", submitted_at > newest. Nothing new → end the turn with no output.
3. For each doc, take the user's messages after the last "claude" message and act:
   - question → answer; unclear → ask one question;
   - options wanted → publish a small HTML variants artifact (3–5 variants, one marked "My pick") and post its link;
   - change → small: edit in {{BRANCH_OR_WORKTREE}}, build, install, launch, recapture. Bigger: background agent whose brief includes the build, install on {{DEVICE_IDS}}, and recapture of that exact screen to img/<name>-v2.jpg. One build at a time: if an agent is already building, SendMessage it the follow-up instead, or brief the new agent to wait until `pgrep -x xcodebuild` (or gradle) is empty.
4. When a change ships: cp img/<name>.jpg img/<name>-v1.jpg; put the new capture at img/<name>.jpg; Artifact publish file_path {{WORK_DIR}}/review.html, root {{WORK_DIR}}, files [{path:"img/<name>.jpg"}].
5. Reply on the card: ArtifactData get the doc, then update with the full messages array plus {from:"claude", text, at:<now ISO>}, reply = text, if_version = the version read. Keep text to 1–2 sentences.
6. Set processed.json[docId] = that doc's submitted_at. Give the user a 1–2 line summary per card.

Commit only files you changed; pushing is the user's call.
