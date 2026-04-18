# Server Param Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add support for a `server` query parameter that rewrites the loaded OpenAPI spec's `servers` array before rendering in Scalar.

**Architecture:** Keep the implementation inside the existing `index.html` bootstrap script. Fetch the remote spec as text, parse it as JSON or YAML, optionally replace `spec.servers` with a single entry derived from the `server` query parameter, and pass the resulting object to Scalar. Update `README.md` so callers know how to build links with the new parameter.

**Tech Stack:** Static HTML, browser Fetch API, `@scalar/api-reference`, `js-yaml`, Markdown

---

## File Structure

- Modify: `index.html`
  Responsibility: fetch the remote spec, parse it, apply the optional `server` override, and render the API reference.
- Modify: `README.md`
  Responsibility: document the `server` query parameter and show updated link-generation examples.
- Create: `docs/superpowers/plans/2026-04-18-server-param-implementation.md`
  Responsibility: implementation handoff for this feature.

### Task 1: Parse Spec Content Into an Object

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace raw string rendering with a parse helper**

Update the inline script in `index.html` so it can parse either JSON or YAML before rendering.

```html
<script>
  function parseSpec(specText) {
    try {
      return JSON.parse(specText);
    } catch (jsonError) {
      try {
        return jsyaml.load(specText);
      } catch (yamlError) {
        console.error('Failed to parse OpenAPI spec as JSON or YAML.', {
          jsonError,
          yamlError,
        });
        return null;
      }
    }
  }

  (async function () {
    const url = new URL(document.location.href);
    const specUrl = url.searchParams.get('url');
    const specText = await fetch(specUrl).then(x => x.text());
    const spec = parseSpec(specText);

    if (!spec) {
      return;
    }

    Scalar.createApiReference('#app', {
      content: spec,
      theme: 'default',
      layout: 'modern',
      showSidebar: true,
    });
  })();
</script>
```

- [ ] **Step 2: Load the page with a known-good JSON spec**

Run a local static server from the repository root and open the page with a JSON spec URL.

Run: `python3 -m http.server 8000`

Open:

```text
http://localhost:8000/?url=https%3A%2F%2Fpetstore3.swagger.io%2Fapi%2Fv3%2Fopenapi.json
```

Expected: the API reference renders normally, showing the same operations as before the change.

- [ ] **Step 3: Load the page with a known-good YAML spec**

Open:

```text
http://localhost:8000/?url=https%3A%2F%2Fraw.githubusercontent.com%2FOAI%2FOpenAPI-Specification%2Fmain%2Ftests%2Fv3.0%2Fpetstore.yaml
```

Expected: the API reference renders successfully from YAML input.

- [ ] **Step 4: Commit the parsing change**

```bash
git add index.html
git commit -m "feat: parse remote specs before rendering"
```

### Task 2: Apply the `server` Query Parameter Override

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Read the `server` parameter and override `spec.servers`**

Extend the same inline script so the parsed spec can be rewritten before rendering.

```html
<script>
  function parseSpec(specText) {
    try {
      return JSON.parse(specText);
    } catch (jsonError) {
      try {
        return jsyaml.load(specText);
      } catch (yamlError) {
        console.error('Failed to parse OpenAPI spec as JSON or YAML.', {
          jsonError,
          yamlError,
        });
        return null;
      }
    }
  }

  (async function () {
    const url = new URL(document.location.href);
    const specUrl = url.searchParams.get('url');
    const serverUrl = url.searchParams.get('server');
    const specText = await fetch(specUrl).then(x => x.text());
    const spec = parseSpec(specText);

    if (!spec) {
      return;
    }

    if (serverUrl) {
      spec.servers = [{ url: serverUrl }];
    }

    Scalar.createApiReference('#app', {
      content: spec,
      theme: 'default',
      layout: 'modern',
      showSidebar: true,
    });
  })();
</script>
```

- [ ] **Step 2: Verify the server override with an absolute URL**

Open:

```text
http://localhost:8000/?url=https%3A%2F%2Fpetstore3.swagger.io%2Fapi%2Fv3%2Fopenapi.json&server=https%3A%2F%2Fexample.com
```

Expected: the rendered documentation shows exactly one server, `https://example.com`.

- [ ] **Step 3: Verify the server override with a non-URL value**

Open:

```text
http://localhost:8000/?url=https%3A%2F%2Fpetstore3.swagger.io%2Fapi%2Fv3%2Fopenapi.json&server=%2Finternal
```

Expected: the rendered documentation shows exactly one server, `/internal`.

- [ ] **Step 4: Commit the server override change**

```bash
git add index.html
git commit -m "feat: support server query overrides"
```

### Task 3: Document the New Query Parameter

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update the README examples and search parameter list**

Replace the current helper snippet and parameter docs with this content.

````md
# OpenAPI-DocSite

A single scalar documentation site for arbitrage URL of your OpenAPI Spec YAML/JSON.

```text
https://openapi.zccz14.com?url=<your-openapi-yaml-or-json-remote-url-encoded>
```

Search Params:

- `url`: SHOULD be url-encoded. RECOMMEND to use the following script to generate.
- `server`: optional. If provided, the rendered document will replace the source spec's `servers` with exactly this value.

```js
function getLink(url, server) {
  const target = new URL('https://openapi.zccz14.com');
  target.searchParams.set('url', url);

  if (server) {
    target.searchParams.set('server', server);
  }

  return target.href;
}
```

Example:

```text
https://openapi.zccz14.com?url=https%3A%2F%2Fpetstore3.swagger.io%2Fapi%2Fv3%2Fopenapi.json&server=https%3A%2F%2Fexample.com
```
````

- [ ] **Step 2: Read the rendered Markdown and verify the code fences are balanced**

Run: `sed -n '1,200p' README.md`

Expected: the Markdown renders with one JavaScript snippet and one plain-text example, with no broken code fence.

- [ ] **Step 3: Commit the documentation update**

```bash
git add README.md
git commit -m "docs: describe server query parameter"
```

### Task 4: Final Verification

**Files:**
- Modify: `index.html`
- Modify: `README.md`

- [ ] **Step 1: Review the final `index.html` script**

Confirm the final inline script contains all of the following logic together:

```html
<script>
  function parseSpec(specText) {
    try {
      return JSON.parse(specText);
    } catch (jsonError) {
      try {
        return jsyaml.load(specText);
      } catch (yamlError) {
        console.error('Failed to parse OpenAPI spec as JSON or YAML.', {
          jsonError,
          yamlError,
        });
        return null;
      }
    }
  }

  (async function () {
    const url = new URL(document.location.href);
    const specUrl = url.searchParams.get('url');
    const serverUrl = url.searchParams.get('server');
    const specText = await fetch(specUrl).then(x => x.text());
    const spec = parseSpec(specText);

    if (!spec) {
      return;
    }

    if (serverUrl) {
      spec.servers = [{ url: serverUrl }];
    }

    Scalar.createApiReference('#app', {
      content: spec,
      theme: 'default',
      layout: 'modern',
      showSidebar: true,
    });
  })();
</script>
```

- [ ] **Step 2: Verify parse failure behavior**

Open:

```text
http://localhost:8000/?url=https%3A%2F%2Fexample.com
```

Expected: the page does not render an API reference, and the browser console contains `Failed to parse OpenAPI spec as JSON or YAML.`

- [ ] **Step 3: Check git status before handoff**

Run: `git status --short`

Expected: only the intended files are modified or the working tree is clean if all commits were created.

- [ ] **Step 4: Create the final feature commit if the work is still uncommitted**

```bash
git add index.html README.md
git commit -m "feat: support overriding rendered spec servers"
```

Expected: skip this step if the earlier task-level commits already captured the final state.
