---
cairn: spec
capability: responses
status: current
---

# Responses

How io-jmap reads what a JMAP server answers over HTTP/1.1, through io-http's readers, their failures surfacing as a variant of each coroutine's own error.

### Requirement: Responses are bounded
Every request-response coroutine (`JmapSend` and the method coroutines built on it, `JmapSessionGet`, `JmapBlobUpload`, `JmapBlobDownload`) SHALL read its response through io-http's `Http11Send` with its default bounds: a response head over 64 KiB SHALL fail with `Send(HeadersTooLarge)`, a body over 256 MiB with `Send(BodyTooLarge)`, and a connection closed before the response ends with `Send(Eof)`.

`JmapEventSource` SHALL read its head through `Http11HeadersRead`, failing with `ReadHeaders(TooLarge)` past 64 KiB, and its chunks through `Http11ChunksReadStream`, failing with `ReadChunks(ChunkTooLarge)` on a chunk over 256 MiB. The SSE frames its `SseFrameParser` assembles across chunks carry no bound of their own.

### Requirement: Blob downloads take their own limit
`JmapBlobDownload` SHALL fail with `Send(BodyTooLarge(max_len))` once the blob exceeds its `max_len`, 256 MiB by default and settable after `new`. `JmapClientStd::blob_download` SHALL apply the client's `max_blob_len`, same default.

#### Scenario: A blob over the limit
- GIVEN a `JmapBlobDownload` whose `max_len` is 4
- WHEN the server answers 200 with a 10-byte body
- THEN the coroutine completes with `Send(BodyTooLarge(4))`
