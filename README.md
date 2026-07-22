# BFG Kitchen Closeout Checklist

Google Apps Script backend for the end-of-semester kitchen closeout submission. Chapters submit a checklist plus photos; this script logs the submission, generates a signed-off Google Doc, files photos into the chapter's Drive folder, requests HD/chef signatures, and notifies the ops team in Google Chat once everything is in.

## What it does

1. Receives the checklist submission via `doPost(e)` (JSON, or a `text_data` wrapper for form POSTs).
2. Photos are uploaded separately as `type: 'file_upload'` requests (one call per photo) and saved into a per-chapter, per-semester Drive folder.
3. Generates a formatted Google Doc recording what was checked off, who submitted, and photo count.
4. Sends a signature request (`sendSignatureRequest`) and a notification to the House Director (`sendHDNotification`).
5. Once all expected photos are in (matched by `submissionId`), posts a summary to the university's Google Chat space — flags a mismatch if the photo count doesn't match what was reported.
6. Marks the corresponding "School Year Operations" Airtable record complete/signed for that semester (`markCloseoutComplete`, `markCloseoutSigned`).
7. `checkSignatureCompletions` — a time-based trigger (`installSignatureTrigger`) that polls for completed signature requests.

## Stack

- Google Apps Script (V8 runtime), deployed as a web app (not a Chat webhook receiver — it *sends* to Chat)
- Airtable (base `appnsTUTkpXhgycnG`) — Houses and School Year Operations tables
- Google Docs + Drive for the closeout doc and photo storage
- Per-university Google Chat webhooks for notifications
- `CacheService` used to dedupe notifications and correlate the async photo-upload calls back to the original submission

## Files

| File | Purpose |
|---|---|
| `Code.js` | Entire backend — submission handling, file uploads, doc generation, signatures, per-university Chat notifications |
| `appsscript.json` | Manifest — Gmail, Drive, Docs, Sheets OAuth scopes; web app deploy settings |

## Secrets

Stored in Script Properties, not in code:

- `AIRTABLE_TOKEN` — Airtable API token
- `CHAT_WEBHOOKS` — JSON object mapping university name → Google Chat webhook URL (18 universities as of this migration)

See `DEPLOYMENT.md` for details on updating these.
