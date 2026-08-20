---
name: work-journal
description: Generate or append a dated daily work journal from GitHub, Slack, Linear, Calendar, and optional Notion and Figma activity.
disable-model-invocation: true
---

# Work journal

Generate or append one daily markdown entry answering "what did I work on?".

## Date and path

1. Default to today in the Mac's local timezone when no date is given.
2. Accept explicit past dates such as `August 17, 2026`, `2026-08-17`, or `26.08.17`. Treat bare month and day numbers as ambiguous and ask which date they mean.
3. Reject future dates. Stop before gathering data.
4. Resolve the target as `/Users/skull/Vercel/Journal/YYYY/YY.MM.DD.md`, where `YYYY` is the four-digit year and `YY.MM.DD` uses the last two year digits.
5. Create only the final dated file's missing parent directories. Read the existing file before writing and preserve its content exactly.

## Required sources

GitHub, Slack, Linear, and Calendar must each succeed through a live tool or CLI, or through pasted results supplied for the exact date. Notion and Figma are optional. Stop before writing if any required source is unavailable and no matching pasted data has been supplied.

When a required source is missing, give one exact retrieval instruction, then wait. Do not use a source's absence as evidence of no activity.

- **Slack:** Ask for the day's authored channel messages using a date-bounded Slack AI prompt: `Summarize my Slack activity for YYYY-MM-DD in channels only, including private channels but excluding DMs and group DMs. Include my substantive messages, decisions, investigations, feedback, coordination, and links to noteworthy threads. Do not summarize what other people did.`
- **Linear:** Install or authenticate `linear-cli`, or ask for the day's issue activity export. With `linear-cli`, query issue changes and comments for the authenticated user within the local day; use `--me`, date filters, and `--json` where the installed version supports them.
- **Calendar:** Ask for a date-bounded list from the primary work calendar, or use a connected calendar tool. Request non-declined events with titles, times, attendees, descriptions, and response status for `YYYY-MM-DD`.
- **GitHub:** Authenticate with `gh auth login`, then use the `gh` commands below.

If a required connector exists but returns no data, treat that as success and continue. If it errors, returns partial data, or cannot establish auth, stop before writing and ask for the exact fallback export.

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

Write only after required-source preflight and calendar attendance confirmation:

1. If no meaningful new activity remains, leave the journal unchanged and report that nothing was appended.
2. If the file does not exist, write the new bullets with a trailing newline.
3. If the file exists, append the new bullets directly after its existing content. Preserve the file byte-for-byte except for the separator needed to start a bullet and the appended bullets.
4. If existing content does not end with a newline, add exactly one newline before the bullets. If existing content ends with prose rather than a markdown list item, add one blank line before the bullets so the list renders cleanly.
5. After writing, report the file path and the number of bullets appended.

## Guardrails

- Never overwrite or normalize existing journal text.
- Never create an entry before all required sources have succeeded or supplied matching pasted data.
- Never treat Notion or Figma as required.
- Never include DMs or group DMs.
- Never list an unconfirmed meeting as attended.
- Never use a raw commit message as a bullet.
- Never fabricate authorship, links, meeting attendance, or missing evidence.
- Never replace the existing `what-did-i-get-done` skill; this skill runs only when invoked explicitly.
