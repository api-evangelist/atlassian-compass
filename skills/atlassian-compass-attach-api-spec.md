---
name: atlassian-compass-attach-api-spec
description: Attach an OpenAPI (or other) API specification file to a Compass component so the
  component page renders the API it exposes, and remove it again.
api: Atlassian Compass REST API
version: v1
operations:
  - uploadAPISpec
  - deleteAPISpec
spec: openapi/atlassian-compass-compass-rest-api-openapi.json
generated: '2026-09-06'
method: generated
source: openapi/atlassian-compass-compass-rest-api-openapi.json
---

# Attach an API spec to a Compass component

This is the one Compass write that is cleanly reversible: upload and delete are a matched pair on
the same path.

## Upload

`PUT {base}/compass/v1/component/{componentId}/api_specs` — operationId `uploadAPISpec`.

- `componentId` is the component ARI.
- Accepted file types are **JSON and YML**. Anything else returns `422`.
- Authentication is HTTP Basic (account email + API token).

The uploaded spec surfaces on the component as `CompassComponent.api`
(`CompassComponentApi` in the GraphQL schema).

## Remove

`DELETE {base}/compass/v1/component/{componentId}/api_specs` — operationId `deleteAPISpec`.

There is **no stated window** — the delete works whenever you call it, and Atlassian publishes no
retention or restore path for the removed file. Treat removal as permanent; keep your own copy.

## Errors

| Status | Meaning |
|---|---|
| `400` | The API spec file is not valid |
| `403` | The user is not authorized to upload (or delete) API specs |
| `404` | The component is not found (delete only) |
| `422` | File type not accepted, or content could not be parsed |

Errors return `{"errors":[{"type":"...","message":"..."}]}`. The stable codes are in
`errors/atlassian-compass-error-types.yml`.

## Related

`uploadAttachment` / `getAttachment` / `deleteAttachment` on
`/compass/v1/component/{componentId}/app/{forgeAppId}/attachment/{key}` are the same shape for Forge
app attachments — but they `403` unless the Forge app has attachment upload enabled, which requires
contacting Atlassian support.
