# API Design Decisions

## Versioning

I used path-based versioning with `/v1/` because it is easy for partner customers to see, test, and debug in normal HTTP tools. Header-based versioning is cleaner visually, but it is easier to miss during manual testing and support, so path-based versioning is safer for a public partner-facing API.

## Batch ordering and partial failures

The batch response returns one result object per caller-supplied `id`, and clients must match results by `id` instead of relying only on array order. If one image in a batch is corrupt, the whole HTTP request still returns `200` when the batch envelope is valid, but the corrupt image receives `status: "failed"` with a structured error object. This allows the client to use the successful predictions immediately while showing or retrying only the failed item.

## Async lifecycle

An async job moves through `queued → running → done` when successful, or `queued → running → failed` when processing fails. The `POST /v1/predict-async` endpoint returns `202 Accepted` with a `job_id`, and the client checks progress with `GET /v1/predictions/{job_id}`. Completed and failed job results are retained for 24 hours, after which the status endpoint returns `404` so stored results do not remain indefinitely.
