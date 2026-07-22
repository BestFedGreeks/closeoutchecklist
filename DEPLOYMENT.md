# Deployment — BFG Kitchen Closeout Checklist

## Making a change

```
cd ~/bfg-scripts/closeoutchecklist
clasp pull
# ...edit files...
node --check Code.js
clasp push
git add .
git commit -m "describe the change"
git push origin main
```

Deployed as a web app — after `clasp push`, if the change needs to reach the live URL immediately, create a new deployment version: Apps Script editor → Deploy → Manage deployments → edit active deployment → New version → Deploy.

## Secrets — Script Properties

Set in the Apps Script editor → gear icon (Project Settings) → Script Properties.

| Property | What it is |
|---|---|
| `AIRTABLE_TOKEN` | Airtable API token |
| `CHAT_WEBHOOKS` | JSON string — `{ "University Name": "https://chat.googleapis.com/...", ... }`. To add/change a university's Chat space, edit this JSON value directly (no code change needed). |

**Adding a new university:** get its Chat webhook URL (Google Chat → target space → Apps & integrations → Webhooks), then edit the `CHAT_WEBHOOKS` Script Property, adding `"University Name": "https://..."` to the JSON object. Keep it valid JSON (commas between entries, no trailing comma on the last one).

## .claspignore

This project's `.claspignore` only allows `appsscript.json` and `Code.js` to be pushed — any other local file (notes, scratch scripts) won't accidentally get deployed. If new script files are added going forward, they need to be explicitly un-ignored in `.claspignore` or they won't push.

## Script ID

`19yhXUAZAC-mh50agl0juGQmrajcs3HlwWniH1ksw-dg39K4agoOOdIp_`
