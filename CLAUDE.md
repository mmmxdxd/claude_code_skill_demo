# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A single-file frontend prototype (`import.html`) for a "知识库文件导入" (knowledge-base file import) UI. It has no build system, no package manager, no linter, and no test suite. Everything — HTML structure, CSS, and JavaScript — lives inline in that one file using only native browser APIs (`fetch`, `FormData`, `setInterval`). There are no third-party or CDN dependencies.

## Running / viewing

There is no build step. Open `import.html` directly in a browser, or serve it locally:

```bash
python -m http.server 8001
```

Note: the page's JS points at `http://127.0.0.1:8000` for its backend (see below), so **the backend is a separate, not-yet-implemented piece**. The frontend renders and the UI works, but uploads will fail until that backend exists.

## The backend contract (the key cross-file interface)

The frontend depends on a REST backend that is currently **unimplemented** — it's a mock/prototype placeholder, not part of this repo. Two endpoints are required:

- `POST http://127.0.0.1:8000/upload` (multipart `FormData`, field `file`) → returns `{ "task_id": string }`
- `GET http://127.0.0.1:8000/status/{task_id}` → returns:

```json
{
  "status": "processing" | "completed" | "failed",
  "done_list": ["node_name", ...],
  "running_list": ["node_name", ...],
  "durations": { "node_name": seconds, ... }
}
```

The frontend never hardcodes the specific workflow node names; it consumes whatever `done_list`/`running_list` the backend returns. The only client-side assumption is the **node count** (see below).

## Architecture that spans the whole file

- **Three layers in one file**: view (DOM in `<body>`), styling (inline `<style>`), logic (inline `<script>`). There are no module boundaries — the 8 JS functions are separated by naming convention only.
- **Data flow is upload → poll**: `handleFiles` → `uploadFile` (posts `/upload`, gets `task_id`) → `pollStatus` (loops every 1500 ms on `/status/{task_id}`) → `renderLogsAndCalculateProgress` (computes progress % and redraws the log timeline). `formatDuration` and `normalizeLogText` are leaf helpers; `updateProgressUI` is a leaf UI helper.
- **Progress is a client-side node-based estimate, not a real percentage from the server.** `totalNodes` is computed from file type: PDF = 8 steps, MD = 7 steps (upload counts as 1 step). Progress = `done.length / totalNodes`, plus a half-node while a node is running, capped at 95% until `completed`. This `totalNodes` value is threaded explicitly from `uploadFile` → `pollStatus` → `renderLogsAndCalculateProgress`.
- **Per-file state is scattered, not centralized.** Each `.file-item` card carries its own state through DOM + closures: `statusBadge` class encodes the lifecycle (`uploading` → `processing` → `completed`/`failed`), and a `stopped` boolean closure flag guards against the async race where a poll response arrives after the interval has been cleared.

## Known issues (do not reintroduce these patterns)

- **XSS via `innerHTML`**: `uploadFile` interpolates `file.name` directly into an HTML template string (and `textContent` into log items from backend node names) with no escaping. Filenames or node names containing `<` will inject markup.
- **No timeout/cancel/retry**: polling runs indefinitely if the backend stalls; there is no way to abort an in-flight upload.
- **Backend missing**: any work that makes uploads actually function needs to create the backend first (FastAPI/Flask/etc.) matching the contract above.
