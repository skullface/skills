---
name: daily-work-journal
description: Generate or append a dated daily work journal from GitHub activity, with optional Slack, Linear, Calendar, Notion, and Figma sources.
disable-model-invocation: true
---

# Daily work journal

Generate or append one daily markdown entry answering "what did I work on?".

## Date and path

1. Default to today in the Mac's local timezone when no date is given.
2. Accept explicit past dates such as `2026-08-20`. Treat bare month and day numbers as ambiguous and ask which date they mean.
3. Reject future dates. Stop before gathering data.
4. Resolve the target as `/Users/skull/personal/vournal/YYYY/MonthLongName/YYYY-MM-DD.md`, where `MonthLongName` is the full English month name. For example, `/Users/skull/personal/vournal/2026/August/2026-08-20.md`.
5. Look in the month subdirectory to see whether the dated file already exists. Read the existing file before writing and preserve its content exactly. Create only the dated file's missing parent directories.

## Sources

GitHub is the only required source. It must succeed through the live `gh` CLI or through pasted results supplied for the exact date. Authenticate with `gh auth login` if needed, then use the `gh` commands below. Stop before writing if GitHub is unavailable and no matching pasted data has been supplied; give that one retrieval instruction, then wait.

Slack, Linear, Calendar, Notion, and Figma are optional. Use each one when a live tool, an authenticated CLI such as `linear-cli`, or a pasted export for the exact date is available, and skip it otherwise. Never stop or wait on an optional source, and never treat a source's absence as evidence of no activity.

If a connected source returns no data, treat that as success and continue. If GitHub errors, returns partial data, or cannot establish auth, stop before writing and ask for the exact fallback export. If an optional source errors, skip it and continue.

## Collection

Collect activity in parallel where tooling allows. Keep the user's identity, timestamps, source links, and attribution attached to every item.

### GitHub

Include activity from all repositories visible to the authenticated `gh` account:

- Authored PRs opened, merged, or closed during the day, regardless of who changed the state.
- PRs authored by others that the user personally opened, merged, or closed during the day.
- PRs that received commits authored by the user during the day.
- Standalone commits authored by the user during the day.
- PR reviews submitted by the user during the day.

Use `gh api` and search queries with the authenticated login and exact local day bounds. Start with `gh api user --jq .login`. Search PR lifecycle, authored commits, and reviews separately, then use repository, pull-request, commit, and review endpoints for detailed state, bodies, files, timestamps, and URLs. Useful search patterns include `gh search prs --author @me --created YYYY-MM-DD..YYYY-MM-DD`, `gh search prs --reviewed-by @me --updated YYYY-MM-DD..YYYY-MM-DD`, and `gh search commits --author @me --author-date YYYY-MM-DD..YYYY-MM-DD`. Inspect diffs for standalone commits. Group related reviews by project or change area instead of listing routine approvals one by one.

### Slack

Include substantive channel messages authored by the user, including private channels. Exclude DMs and group DMs. A standalone Slack discussion counts when the user's own messages show concrete decisions, investigation, feedback, debugging, coordination, or customer-facing work. Keep thread links when available.

### Linear

Include only actions attributable to the user: issue creation or edits, comments, assignment actions, and state changes. Exclude changes made by other people on issues assigned to the user.

### Calendar

List non-declined meetings with at least one other attendee. Exclude declined events, focus blocks, reminders, out-of-office holds, and solo events. Ask exactly one question listing candidate meetings and asking which the user attended. Wait for the answer before writing. Include an attended meeting only when its title or description supplies concrete work context; fold it into the related work bullet rather than listing meetings separately.

### Notion and Figma

Use these sources only when a live tool or pasted export is available. Include attributable page or file creation, content edits, and substantive comments. Exclude views, opens, timestamps that name no actor, and other unattributed events.

## Deduplication and grouping

1. Associate commits with PRs when the same change appears in both.
2. Merge PR, Linear, Slack, document, and meeting evidence for the same work into one bullet.
3. Group related reviews by project or change area.
4. Include substantive Slack-only work without forcing corroboration.
5. Preserve each distinct meaningful item. Do not impose a bullet limit.
6. Read existing journal content before drafting and treat bullets already represented there, including differently worded but functionally identical lines, as duplicates.

## Writing

- Append bullets only. Add no generated heading, timestamp, divider, or surrounding narrative.
- Start every bullet with `- ` and use verb-led past-tense fragments.
- Use the user's own wording when it is concrete. Use PR titles when useful. Describe standalone commits functionally from their diffs.
- State what changed. Do not infer intent, motivation, business value, impact, or sentiment.
- Include compact links such as PR links, Linear issue identifiers, Slack thread links, and document links when available.
- Keep bullets information-dense. Omit cosmetic-only changes, routine approvals, views, and activity not attributable to the user.

## Append

Write only after the GitHub preflight and, when calendar data was collected, the calendar attendance confirmation:

1. If no meaningful new activity remains, leave the journal unchanged and report that nothing was appended.
2. If the file does not exist, write the new bullets with a trailing newline.
3. If the file exists, append the new bullets directly after its existing content. Preserve the file byte-for-byte except for the separator needed to start a bullet and the appended bullets.
4. If existing content does not end with a newline, add exactly one newline before the bullets. If existing content ends with prose rather than a markdown list item, add one blank line before the bullets so the list renders cleanly.
5. After writing, report the file path and the number of bullets appended.

## Guardrails

- Never overwrite or normalize existing journal text.
- Never create an entry before GitHub has succeeded or supplied matching pasted data.
- Never treat Slack, Linear, Calendar, Notion, or Figma as required.
- Never include DMs or group DMs.
- Never list an unconfirmed meeting as attended.
- Never use a raw commit message as a bullet.
- Never fabricate authorship, links, meeting attendance, or missing evidence.
- Never replace the existing `what-did-i-get-done` skill; this skill runs only when invoked explicitly.
