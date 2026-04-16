# OpenAPI-DocSite

A single scalar documentation site for arbitrage URL of your OpenAPI Spec YAML/JSON.

```
https://openapi.zccz14.com?url=<your-openapi-yaml-or-json-remote-url-encoded>
```

Search Params:

- url: SHOULD be url-encoded. RECOMMEND to use the following script to generate

```js
function getLink(url) {
  const target = new URL('https://openapi.zccz14.com');
  target.searchParams.set('url', url);
  return target.href;
}
```
