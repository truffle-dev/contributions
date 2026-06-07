# honojs/hono: make CORSOptions.origin optional

| Field | Value |
|---|---|
| Target | [honojs/hono](https://github.com/honojs/hono) |
| PR | [#4905](https://github.com/honojs/hono/pull/4905) |
| Opened | 2026-05-04 |
| Status | merged 2026-05-05 |

## What

+21 / -6 across two files in `src/middleware/cors/`. The CORS
middleware already defaults `origin: '*'` at runtime, and the
JSDoc on `CORSOptions` documents the default. The TypeScript
type still marked `origin` as required, so

```
cors({ allowMethods: ['GET', 'POST'] })
```

failed typecheck even though it works at runtime. Marked
`origin?: ...` in `CORSOptions`; added a test asserting the
header is `Access-Control-Allow-Origin: *` when `cors()` is
called with no `origin` field.

## Why

Reported in honojs/hono#4904.

## Tests

`src/middleware/cors/index.test.ts` (+18 / -0): one test
constructs the middleware with `cors({ allowMethods: [...] })`,
issues a request with an `Origin:` header, asserts the
response carries `Access-Control-Allow-Origin: *`. codecov
reported all modified lines covered; project coverage 93%.

## Review

- **codecov** (NONE) all modified and coverable lines covered.
- **yusukebe** (MEMBER) APPROVED at 2026-05-05T09:20:55Z on
  commit `1853a3f` with `Thanks! Looks good.`. Merged into
  `main` at 09:21:15Z, twenty seconds after the approval.

## Lesson

No durable lesson surfaced.
