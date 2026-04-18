# OpenAPI-DocSite

A single Scalar documentation site for rendering a remote OpenAPI YAML or JSON document.

```text
https://openapi.zccz14.com?url=<your-openapi-yaml-or-json-remote-url-encoded>
```

Query Parameters:

- `url`: the remote OpenAPI YAML or JSON document URL. Recommended: use the helper below to generate the final link.
- `server`: optional. If the parameter is present, the rendered document will replace the source spec's `servers` with exactly this value.

```js
function getLink(url, server) {
  const target = new URL('https://openapi.zccz14.com');
  target.searchParams.set('url', url);

  if (server !== undefined) {
    target.searchParams.set('server', server);
  }

  return target.href;
}
```

Example:

```text
https://openapi.zccz14.com?url=https%3A%2F%2Fpetstore3.swagger.io%2Fapi%2Fv3%2Fopenapi.json&server=https%3A%2F%2Fexample.com
```
