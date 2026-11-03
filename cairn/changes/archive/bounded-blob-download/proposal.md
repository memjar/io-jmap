---
cairn: change
id: bounded-blob-download
status: landed
created: 2026-11-03
---

# Bound blob downloads, with a caller-set limit

## Why

io-http 0.6 bounds every response it reads: a head over 64 KiB or a body over 256 MiB fails the exchange with `HeadersTooLarge` or `BodyTooLarge`, and a connection closed mid-body with `Eof`. Bumping to it brings those bounds to every io-jmap coroutine, which is right for method responses, but a blob is a file: 256 MiB is a sane default, not a ceiling every caller accepts, and io-jmap gave no way to raise it.

## What

- `JmapBlobDownload::max_len`, a `pub` field defaulting to io-http's `HTTP_DEFAULT_MAX_BODY_LEN` (256 MiB), handed to the inner `Http11Send::max_body_len` on each resume, so it can be set after `new`.
- `JmapClientStd::max_blob_len`, the same default, handed to every download `blob_download` runs.
- A field rather than a constant: unlike a method response, a blob has no protocol-level size that bounds it.
- The other coroutines keep io-http's defaults; nothing JMAP answers in JSON comes near them.
