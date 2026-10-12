# Answer exports

## cyclamen-qa-live.json — the primary file, read this first

`ClaryfingQuestions.html` can write here **automatically**, on every answer, once the user
clicks "Connect auto-save…" on the page (one-time grant, Chromium browsers only, via the
File System Access API). While connected, `answers/cyclamen-qa-live.json` is overwritten
in place after each save — it is always current. **Junam should read this file first, if
it exists.** Its own `exportedAt` / `stats` fields say how fresh it is; git history on this
file is the audit trail now, so there is no need for dated snapshots of it.

## cyclamen-qa-\<YYYY-MM-DD\>.json — the fallback

Produced by the **Export for Junam** button, one file per manual click. This still exists
for browsers without File System Access support, or for a point-in-time snapshot. **Read
this only if `cyclamen-qa-live.json` is absent, or you have reason to think it's stale**
(e.g. auto-save was never connected in the browser that holds the newest answers) — in that
case, read the newest dated file instead.

Nothing in this folder is written by hand, and nothing here is a source of truth:
`ClaryfingQuestions.html` is.
