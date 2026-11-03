---
cairn: log
change: bounded-blob-download
landed: 2026-11-03
---

# Blob downloads are bounded, with a caller-set limit

io-jmap moved to io-http 0.6, whose readers bound what they read (e181530 there): a head over 64 KiB, a body over 256 MiB and a connection closed mid-body now fail every request-response coroutine through its `Send` variant, with `HeadersTooLarge`, `BodyTooLarge` and `Eof`, and the event source on a head over 64 KiB or a chunk over 256 MiB. **Breaking**, the variants being new on exhaustive enums the coroutines carry. The SSE frames assembled across chunks stay unbounded: io-http's `SseFrameParser` returns `Infallible`, so a bound waits for its next breaking release.

Blobs got their own limit, since 256 MiB is a default for a file, not a ceiling: `JmapBlobDownload::max_len` (rfc8620/blob_download.rs) is handed to `Http11Send::max_body_len` on each resume, so a caller sets it after `new`, and `JmapClientStd::max_blob_len` (client.rs) is handed to every `blob_download`. Both default to `HTTP_DEFAULT_MAX_BODY_LEN`.

Same release: minimum Rust version 1.89, the unused `coroutine::*` import of the event source tests dropped, and tests/common.rs silenced the way io-imap and io-smtp do (`#![allow(dead_code)]`, the TLS stream boxed).

Capabilities moved: responses (new: "Responses are bounded", "Blob downloads take their own limit").

Verified: `blob_over_max_len_fails` fails with the hand-over to `Http11Send` commented out and passes with it; `cargo clippy --all-features --all-targets` clean, `cargo test --all-features` green (107 lib tests), and the build passes with no default features and with `client` alone.
