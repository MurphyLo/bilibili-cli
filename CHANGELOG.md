# Changelog

## Unreleased

### Fixed

- `bili login` no longer saves an empty credential. Bilibili's web QR poll may
  return a ticket URL whose query string carries no `SESSDATA`; the real cookies
  now arrive via `Set-Cookie` on the poll response. Login reads those headers
  (falling back to following the login URL), refuses to save a credential
  without `SESSDATA`, and fills in `buvid3`/`buvid4`
  (adapted from cestivan@8111162 and @9e99be3, upstream PR #29)
- `video` no longer fails with HTTP 412: the main `x/web-interface/view` request
  passed `credential=None` unconditionally, so it was always anonymous — the
  same defect fixed for `user-videos` in 0.7.0. Bilibili now rejects anonymous
  calls to this endpoint; a `buvid3`-only cookie is not enough, `SESSDATA` is
  required
- The test suite no longer deletes the developer's real
  `~/.bilibili-cli/credential.json`. `test_get_credential_write_rejects_missing_bili_jct`
  mocked `_validate_credential` into returning `False` without also mocking
  `clear_credential`, so `get_credential()` took the expired-credential branch
  and unlinked the actual file. A `conftest.py` fixture now redirects
  `CONFIG_DIR`/`CREDENTIAL_FILE` to `tmp_path` for every test

## 0.7.0

Fork release. Upstream `public-clis/bilibili-cli` has had no commits since
2026-03-14 (`dbe2855`); this release collects fixes from several forks plus
local work. Every commit records its origin as `Source: <fork>@<hash>`.

### Fixed

- `user-videos` no longer fails with HTTP 412: the command now passes the saved
  credential to the WBI-signed request
- `duration` no longer reports `00:00` and `owner` is no longer empty in
  video summary output
- `watch-later` uses the dedicated `x/v2/history/toview` endpoint instead of the
  navigation sidebar endpoint, which could return `count > 0` with empty items
  (from zwczwczwc, upstream PR #22)
- Network timeouts during credential validation are treated as indeterminate
  rather than as an invalid credential (from HawkW1027, upstream PR #13)
- Proxies are auto-detected from the environment for `bilibili-api-python`
  (from HawkW1027, upstream PR #13)
- `VideoCodecs.UNKNOWN` is excluded from audio stream detection
  (from Gqingbo, upstream issue #23)
- The default credential path no longer scans browser cookies, which spawned a
  `browser-cookie3` subprocess and stalled for 15s before reporting
  `Cookie extraction timed out`. The flow is now saved credential -> QR login
  (reimplemented from annoft@c150f96)

### Added

- Charging-exclusive markers in video summary output: `charging_exclusive`,
  `charging_type` (1 = 充电专属, 2 = 抢先看, 0 = none) and `charging_badge`.
  Two fields are exposed because a single boolean conflates permanently
  exclusive content with early access that later becomes free
- `bili cover` downloads a video cover image (from wjjsn, upstream PR #25)

### Changed

- Audio downloads default to a higher quality tier (`_192K` instead of `_64K`)
  (from wjjsn, upstream PR #25)

## 0.5.0

- Add subtitle timeline output via `bili video --subtitle-timeline` / `-st`
- Add `--subtitle-format timeline|srt`
- Keep subtitle timeline compatible with current `--yaml` / `--json` command surface
- Ensure subtitle timeline requests load optional credentials like plain subtitles
- Restore README badges and fix CI type-checking with `types-PyYAML`
