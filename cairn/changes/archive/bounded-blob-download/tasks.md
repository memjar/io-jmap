---
cairn: tasks
change: bounded-blob-download
---

# Tasks

- [x] Bump io-http to 0.6
- [x] Add `JmapBlobDownload::max_len` and hand it to `Http11Send::max_body_len`
- [x] Add `JmapClientStd::max_blob_len` and hand it to `blob_download`
- [x] Test a blob over the limit, failing without the hand-over
- [x] CHANGELOG, spec, log
