## Summary

Add support for a `server` URL query parameter that rewrites the OpenAPI spec's `servers` section before rendering in Scalar. This allows a shared remote spec to be displayed as if it targets a specific server chosen in the page URL.

## Goals

- Preserve the current `url` parameter behavior for loading a remote OpenAPI spec.
- Support a `server` parameter that forces the rendered document to expose exactly one server.
- Keep the implementation inside the current static client page.
- Document the new query parameter in `README.md`.

## Non-Goals

- Adding any backend proxy or transformation service.
- Validating whether the `server` value is an absolute URL.
- Merging the requested server with existing servers from the source spec.

## Current State

The site is a single `index.html` page that:

- reads the `url` query parameter,
- fetches the remote OpenAPI document as text,
- passes that raw content directly to `Scalar.createApiReference`.

There is already a commented placeholder for reading a `server` query parameter, but no transformation is applied today.

## Proposed Design

### Query Parameters

- `url`: unchanged; points to the remote OpenAPI YAML or JSON document.
- `server`: optional; when present, overrides the parsed spec's `servers` field.

### Spec Loading Flow

1. Read `url` and `server` from `window.location`.
2. Fetch the remote spec as text.
3. Parse the text into a JavaScript object.
4. If `server` is present, replace `spec.servers` with `[{ url: serverParam }]`.
5. Pass the transformed spec object to Scalar for rendering.

### Parsing Strategy

The page already loads `js-yaml`, so parsing will work as follows:

1. Attempt `JSON.parse(specText)` first.
2. If JSON parsing fails, attempt `jsyaml.load(specText)`.
3. If both fail, log a clear parse error and stop rendering.

This keeps support for both JSON and YAML remote specs without adding dependencies.

### Server Override Rules

When `server` is present:

- `spec.servers` is always replaced, even if the source spec already defines one or more servers.
- the resulting value is always exactly `[{ url: serverParam }]`.
- the `server` value is written as-is without validation.

This matches the requested behavior that the rendered documentation should reflect calls against the specific server named in the page URL.

### Rendering Behavior

- When `server` is absent, rendering should remain effectively unchanged from the user's perspective.
- The internal implementation will still parse into an object before rendering, but no behavioral override should occur.
- Scalar will receive structured `content` instead of the raw fetched string.

## Error Handling

- If `url` is missing, existing browser/fetch failure behavior may remain unless a small explicit guard is already convenient during implementation.
- If fetch fails, the existing failure behavior remains acceptable.
- If parsing fails for both JSON and YAML, log a clear error in the console and do not call `Scalar.createApiReference`.

## Documentation Changes

Update `README.md` to:

- document the new `server` search parameter,
- show a link example that includes both `url` and `server`,
- update the helper snippet so callers can optionally include `server`.

## Testing Approach

Manual verification is sufficient for this change:

1. Load a JSON spec with no `server` parameter and confirm normal rendering.
2. Load a YAML spec with no `server` parameter and confirm normal rendering.
3. Load either spec with `server=https://example.com` and confirm the rendered document shows only that server.
4. Load with a non-URL string such as `server=/internal` and confirm it is still shown as the only server.
5. Load an invalid spec payload and confirm parsing failure is visible in the console and rendering does not proceed.

## Implementation Notes

- Keep the change localized to `index.html` and `README.md`.
- Prefer a tiny helper for parsing only if it improves readability; otherwise keep the logic inline in the existing IIFE.
- Avoid introducing compatibility branches or extra abstraction beyond what this page needs.
