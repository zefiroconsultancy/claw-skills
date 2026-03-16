---
name: telegram-delivery
description: >
  Deliver files to Telegram chats. Covers path mapping from sandbox to host,
  supported file types, and delivery patterns. Use whenever sending files
  (images, documents, audio, video) to a Telegram user.
triggers:
  - send file telegram
  - deliver file
  - send image
  - send document
  - send report
  - MEDIA telegram
version: 1
---

# Telegram File Delivery

## Core Rule

Files must use **host-side paths**, not sandbox paths. The sandbox filesystem
is isolated — Telegram can only access files via the host mount points.

## Path Mapping

| Sandbox Path | Host Path (use this in MEDIA:) |
|---|---|
| `/workspace/ledger/` | `/var/lib/claw/hermes/test/.hermes/ledger/` |
| `/workspace/browser_screenshots/` | `/var/lib/claw/hermes/test/.hermes/browser_screenshots/` |

### Pattern
```
# Wrong — won't deliver
MEDIA:/workspace/ledger/reports/chart.png

# Correct — delivers
MEDIA:/var/lib/claw/hermes/test/.hermes/ledger/reports/chart.png
```

## Supported File Types

All file types work when using host paths:

- **Images**: .png, .jpg, .webp — displayed inline as photos
- **Documents**: .pdf, .csv, .txt, .json — sent as document attachments
- **Audio**: .ogg — plays as voice bubble
- **Video**: .mp4 — plays inline

## Delivery Syntax

Include in your response text:
```
MEDIA:/var/lib/claw/hermes/test/.hermes/path/to/file.ext
```

Multiple files: put each MEDIA: on its own line.

## Procedures

### Sending a generated file
1. Generate the file to a path under `/workspace/ledger/` or `/workspace/browser_screenshots/`
2. Translate the path: replace `/workspace/ledger/` with `/var/lib/claw/hermes/test/.hermes/ledger/`
3. Include `MEDIA:<host_path>` in your response

### Sending browser screenshots
Browser screenshots are automatically saved to `/workspace/browser_screenshots/` in sandbox
which maps to `/var/lib/claw/hermes/test/.hermes/browser_screenshots/` on host.
The `browser_vision` tool returns the host path directly — use it as-is.

### Sending multiple files
```
Here are your reports:

MEDIA:/var/lib/claw/hermes/test/.hermes/ledger/reports/chart1.png
MEDIA:/var/lib/claw/hermes/test/.hermes/ledger/reports/chart2.png
MEDIA:/var/lib/claw/hermes/test/.hermes/ledger/reports/data.csv
```

## Pitfalls

- **Never use sandbox paths** — `/workspace/...` paths silently fail to deliver
- **No error on failure** — if the path is wrong, the file just doesn't arrive; no error message
- **Browser tool paths** — `browser_vision` returns host paths directly, safe to use as-is
- **New persistent volumes** — if new volumes are mounted, their host path mapping must be discovered and added here
