# screen-review

A Claude Code skill that captures every screen and state of a feature (or of the current branch's changes) on a simulator or device, publishes them as a screenshot review board where people can draw and comment on each card, then watches the board and ships the requested changes, recapturing each screen.

## Install

```bash
git clone https://github.com/ahmedezenys/screen-review-skill
cp -R screen-review-skill/screen-review ~/.claude/skills/
```

Then in Claude Code: `/screen-review`.
