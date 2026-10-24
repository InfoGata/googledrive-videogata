# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A VideoGata cloud sync plugin for Google Drive. It is a storage backend only:
VideoGata decides when to sync, merges documents with automerge and hands this
plugin opaque bytes. The plugin never reads playlists itself.

It implements, from `@infogata/videogata-plugin-typings`:
- `onSyncUpload({ docUrl, data })` / `onSyncDownload({ docUrl })` -- `data` is a
  base64 automerge document; download returns `{ data: null }` when there is no
  file yet (first sync).
- `onLogin` / `onLoginCallback` / `onLogout` / `onIsLoggedIn` -- the app opens a
  blank popup, `onLogin` returns the OAuth url, and the app relays the callback
  url to `onLoginCallback`. The auth url carries `state={"pluginId": ...}` so
  the Android and iOS apps can route the callback deep link back here.

This is a port of `googledrive-audiogata` (itself a port of
`googledrive-socialgata`); keep them in step.

## Build Commands

```bash
npm run build          # tsc, then both vite builds
npm run build:options  # options page (Preact) -> dist/options.html
npm run build:plugin   # plugin script -> dist/index.js
```

`dist/` is committed: jsdelivr serves the plugin from the repo.

## Google Drive details

- Files live in the hidden `appDataFolder` (`drive.appdata` scope), named
  `<docUrl>.automerge` (VideoGata uses `videogata-library`).
- Drive addresses files by id; ids are looked up by name once and cached in
  localStorage. Lookups order by `createdTime` so devices that each created a
  copy settle on the oldest. A 404 on a cached id drops the cache and retries.
- Tokens: the default OAuth client's secret lives in the token service
  (`TOKEN_SERVER` in `src/shared.ts`), so code exchange and refresh go through
  it. A user's own client is used only when both id and secret are set.
  Access tokens are refreshed a minute before expiry and once more on a 401;
  concurrent refreshes share one request.
- All Drive calls go through `application.networkRequest`.
