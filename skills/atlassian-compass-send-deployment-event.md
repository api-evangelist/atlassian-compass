---
name: atlassian-compass-send-deployment-event
description: Record a deployment (or build, incident, alert, flag, push, pull-request, vulnerability
  or custom event) against a Compass component so it lands in that component's activity feed and
  feeds DORA metrics.
api: Atlassian Compass REST API
version: v1
operations:
  - createCompassEvent
spec: openapi/atlassian-compass-compass-rest-api-openapi.json
generated: '2026-09-06'
method: generated
source: openapi/atlassian-compass-compass-rest-api-openapi.json,
  https://developer.atlassian.com/cloud/compass/components/send-events-using-rest-api/
---

# Send an event to a Compass component

Compass is an event **sink**. This skill covers the one operation that gets toolchain activity into
a component's feed.

## Before you start

- You need the component's ARI. Compass shows it on the component details page; there is no public
  REST operation that lists components — component lookup is GraphQL only
  (`compass.component`, `compass.searchComponents`, `compass.componentByExternalAlias`).
- You need the site `cloudId`.
- You need an Atlassian API token from https://id.atlassian.com/manage/api-tokens. The REST API
  authenticates with HTTP Basic using your account email as the username and the token as the
  password, and authorizes as *that user* — there is no service account.

## Call

`POST {base}/compass/v1/events` — operationId `createCompassEvent`.

Base is your own site gateway, `https://<your-site>.atlassian.net/gateway/api`.

Body is `CreateStreamlinedEventRequest`:

```json
{
  "cloudId": "<site cloud id>",
  "componentId": "ari:cloud:compass:<cloudId>:component/<id>",
  "event": { "deployment": { } }
}
```

`event` is `CompassEventInputDto`. **Exactly one** of its ten fields may be set:
`deployment`, `build`, `incident`, `alert`, `flag`, `lifecycle`, `push`, `pullRequest`,
`vulnerability`, `custom`. Setting two, or none, raises `ONE_OF_DIRECTIVE_VALIDATION_ERROR`.

Read the matching `Compass*EventPropertiesInputDto` schema in the spec for the fields each event
kind carries — deployment takes an environment and a pipeline, incident takes a severity,
vulnerability takes a severity, push takes an author.

## What you get back

`202 Accepted`. The event is processed **asynchronously**, so a 202 means accepted, not applied.
Do not treat the response as confirmation that the event is visible.

## Rules that will bite you

- **Rate limit: 100 requests per user per minute.** Exhaustion returns `429`. Atlassian publishes no
  `X-RateLimit-*`, `RateLimit-*` or `Retry-After` header for Compass, so you get no runtime budget
  signal — back off on the 429 itself, with jitter.
- **There is no idempotency.** No `Idempotency-Key` header exists. A retried POST writes a second
  event. `externalEventId` is a caller-supplied external identifier and is *not* documented as a
  replay guard — do not rely on it for deduplication.
- **This write cannot be undone.** There is no delete-event operation. An event in a component's
  activity feed stays there. Confirm the component ARI before you fire.
- **Errors** come back as `{"errors":[{"type":"...","message":"..."}]}`. Branch on `type`, never on
  `message` — Atlassian states the message text may change while the type will not. The full
  registry is in `errors/atlassian-compass-error-types.yml`.
- A `404` here means the *event source* was not found, not the component.

## Related

- `insertMetricValue` (`POST /compass/v1/metrics`) — same auth, same 100/min limit, same
  irreversibility; see the metrics skill.
- Event sources: if you need events grouped per connected tool, create one with
  `compass.createEventSource` and attach it with `compass.attachEventSource` (GraphQL).
