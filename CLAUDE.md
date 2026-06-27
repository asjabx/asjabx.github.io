# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Pages static site hosting a shared photo album web app (共享相册) — a Chinese-language single-page application supporting photo and video browsing, uploading (simulated), album management, and user administration.

## Architecture

The entire application lives in a single file: `index.html`. There is no build system, no bundler, no package manager, and no backend. All CSS, HTML, and JavaScript are inline.

### State model

All application state is held in four in-memory JavaScript arrays declared at the top of the `<script>` block:

- `users` — registered accounts with `id`, `username`, `nickname`, `role` (`admin`/`user`), `status` (`approved`/`pending`/`rejected`)
- `albums` — photo albums with `id`, `name`, `description`
- `photos` — media items with `id`, `filename`, `original_filename`, `media_type` (`photo`/`video`/`live`), `thumbnail_url`, `url`, `uploader_id`, `album_id`
- `activities` — audit log entries with `user_id`, `action_type`, `description`, `created_at`

State resets on every page reload — there is no persistence layer.

### UI structure

Three screen states toggled via `display` style: `#login-screen`, `#register-screen`, `#main-screen`.

Within the main screen, tab-based navigation switches between four panels: `#photos-tab`, `#albums-tab`, `#upload-tab`, `#admin-tab`. The admin tab is hidden from non-admin users.

Two modal overlays: `#viewer-modal` (full-screen media viewer with zoom/nav) and `#album-modal` (create/edit albums).

### Authentication

Login is entirely client-side: credentials are hardcoded in the `login()` function and compared directly in JavaScript. The admin account has elevated privileges that reveal the admin tab. New registrations via `register()` add to the in-memory `users` array and are lost on reload.

**Security note**: Credentials are visible in page source. Do not treat this as a real authentication system.

### Upload simulation

`uploadFiles()` simulates a progress bar with `setInterval` and appends new entries to the in-memory `photos` array using placeholder URLs from `picsum.photos`. No files are actually stored anywhere.

### Photo viewer

`openViewer(index)` indexes into `currentPhotos` (the currently filtered subset). Ctrl+click on a photo toggles multi-select mode (`selectedPhotos` Set); regular click opens the viewer. Navigation with `prevPhoto()`/`nextPhoto()` moves through `currentPhotos`.

## Development workflow

No build step is needed. Edit `index.html` directly and open it in a browser to preview changes.

To preview via a local server:
```
python3 -m http.server 8080
# then open http://localhost:8080
```

Deployment is automatic: pushing to `main` publishes to GitHub Pages at `https://asjabx.github.io`.

## Key conventions

- All user-visible text is Simplified Chinese (zh-CN).
- Dates are rendered with `formatDate()` using `toLocaleString('zh-CN')`.
- CSS uses a flat, component-style naming pattern (`.photo-item`, `.album-card`, `.stat-card`) — no utility framework.
- DOM manipulation uses `innerHTML` for rendering lists and `style.display` for show/hide toggling.
- `getElementById` / `querySelector` are used directly; no DOM library.
- When adding new media types beyond `photo`/`video`/`live`, update the type filter `<select>` in `#photos-tab`, the media-type badge rendering in `loadPhotos()`, and the viewer branch in `openViewer()`.
