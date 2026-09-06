---
name: atlassian-compass-push-metric-value
description: Push a numeric metric value onto a Compass component's metric source so it shows on the
  component and can be scored by a scorecard criterion.
api: Atlassian Compass REST API
version: v1
operations:
  - insertMetricValue
spec: openapi/atlassian-compass-compass-rest-api-openapi.json
generated: '2026-09-06'
method: generated
source: openapi/atlassian-compass-compass-rest-api-openapi.json,
  https://developer.atlassian.com/cloud/compass/components/push-metric-values-using-a-curl-command/
---

# Push a metric value to Compass

## Before you start

You need a **metric source ARI**, not a component id. A metric source is a metric definition bound
to one component. Compass surfaces a ready-made cURL command on the metric tile of the component
page — copy it rather than assembling the ARI by hand. Metric sources are created through GraphQL
(`compass.createMetricSource`), not through REST.

Authentication is HTTP Basic: your Atlassian account email plus an API token from
https://id.atlassian.com/manage/api-tokens.

## Call

`POST {base}/compass/v1/metrics` — operationId `insertMetricValue`.

```bash
curl --request POST \
  --url https://<your-site>.atlassian.net/gateway/api/compass/v1/metrics \
  --user "$USER_EMAIL:$USER_API_TOKEN" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --data '{
    "metricSourceId": "ari:cloud:compass:<cloudId>:metric-source/<...>/<...>",
    "value": 32,
    "timestamp": "2026-09-06T00:00:00Z"
  }'
```

The spec offers three request shapes — `InsertMetricRequestDto`,
`InsertMetricByMetricSourceRequestDto` and `InsertMetricByMetricDefinitionRequestDto` — depending on
whether you address the value by metric source or by metric definition plus component. Pick the one
whose required fields you actually hold.

`200` returns `InsertMetricResponseDto`: `metricSourceId`, `value`, `timestamp`, `annotation`.

## Rules that will bite you

- **Rate limit: 100 requests per user per minute**, `429` on exhaustion, and **no rate-limit
  response headers are published**. If you are backfilling history, pace yourself to under 100/min
  and handle the 429 by sleeping — there is no header telling you how long.
- **No idempotency and no undo.** There is no delete-metric-value operation. A wrong value can only
  be superseded by a later one, and it stays in the series for the tier's retention window (one year
  on Standard, two on Premium). Validate `value` before you send.
- **`timestamp` is yours to set.** Backfilling means sending historical timestamps, one call per
  point, against the same rate limit.
- `403` means the user lacks permission to insert metrics; `404` means the metric source, metric
  definition or component was not found. Both are user/ARI problems, not payload problems.
- Errors use `{"errors":[{"type","message"}]}`; branch on `type`.
