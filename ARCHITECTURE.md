# Architecture — BFG Kitchen Closeout Checklist

## Two request types hit the same `doPost`

```
doPost(e)
  │
  ├─ data.type === 'file_upload'  ──► handleFileUpload(data)
  │                                     saves one photo to Drive,
  │                                     and if it's the LAST expected
  │                                     photo, calls finalizeSubmissionNotification
  │
  ├─ data.type === 'text_data'    ──► unwraps data.payload
  │                                     (form-encoded POST fallback)
  │
  └─ (default)                    ──► processSubmission(data)
                                        the main checklist submission
```

`doGet` exists mainly as a health check ("BFG Closeout API running") and a fallback path that accepts `?payload=` for `processSubmission`.

## Why the photo flow is separate from the checklist

The front end submits the checklist once, then uploads each photo as its own request (likely to keep individual payloads small / avoid timeouts on large multi-photo submissions). Because these arrive as separate, unordered HTTP calls:

- `processSubmission` caches its result under `closeout_<submissionId>` (`CacheService`, `ScriptCache`) — this is how photo count expectations and submitter info survive across calls.
- `handleFileUpload` checks `photoIndex === totalPhotos` to detect the last photo, then calls `finalizeSubmissionNotification`, which:
  - dedupes with a `closeout_notified_<submissionId>` cache key (6 hour TTL) so simultaneous/retried "last photo" calls don't double-notify
  - re-counts actual files in the folder vs. the reported count and flags a mismatch in the Chat message if they don't match
  - only *then* calls `sendChatNotification` and `markCloseoutComplete`

This means: if a photo upload silently fails or arrives out of order, the notification/completion step may never fire for that submission — worth checking `CacheService` state or Drive folder contents directly if a chapter's closeout doesn't appear to complete.

## School year / period logic

`getSchoolYearAndPrefix()` derives both the current school year label (e.g. `2026-27`) and whether it's a "Winter" or "Spring" closeout, purely from today's date (Aug–Dec = Winter of the year starting; Jan–Jul = Spring of the year that started the previous fall). This drives which Airtable field (`Winter Close-Out Complete` / `Spring Close-Out Complete`, same pattern for `...Signed`) gets updated on the House's School Year Operations record. If this script is ever run to backfill a past semester, this date-based logic will get the *current* period, not necessarily the one being backfilled — worth checking against real dates for backfills.

## Per-university Chat routing

`CHAT_WEBHOOKS` (Script Property, JSON) maps university name → Chat webhook URL. Several universities intentionally share the same webhook (e.g. Colorado/Colorado State/Denver/Colorado School of Mines all post to the same space) — this is expected, not a data error. `sendChatNotification` looks up `CHAT_WEBHOOKS[data.university]`; if a university isn't in the map, no notification is sent (check `matchedUni` logic near line 770 for the exact fallback/logging behavior).

## Key data model (Airtable base `appnsTUTkpXhgycnG`)

- **Houses** (`tbl7Cbf4tX6BtF4WX`)
- **School Year Operations** (`tbl7v4tnvVE1Qwcn0`) — one record per house per school year; this is what gets patched with `Winter/Spring Close-Out Complete` and `...Signed` flags.
