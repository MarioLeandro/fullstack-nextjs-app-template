# SURFACE.md

Commit base: f99141e
Date: 2025-11-03

## Summary

- Total pages: 12
- Total API endpoints: 17

## Pages (frontend)

| Page                   |                           URL |  Method  | Parameters (query/body)     | Authentication? | Notes / Entry points                                                                                                                    |
| ---------------------- | ----------------------------: | :------: | :-------------------------- | :-------------: | --------------------------------------------------------------------------------------------------------------------------------------- |
| Home                   |                             / |   GET    | -                           |       No        | `app/page.tsx` (also `@pages/index.tsx`)                                                                                                |
| About                  |                   /about.html |   GET    | -                           |       No        | `app/about.html/page.tsx`, `@pages/about.html.tsx`                                                                                      |
| Posts list             |                   /posts.html |   GET    | query: page?                |       No        | `app/posts.html/page.tsx`, `@pages/posts.html.tsx` — posts listing                                                                      |
| Post detail            |                   /posts/[id] |   GET    | params: id                  |       No        | `app/posts/[id]/page.tsx`, `@pages/posts/[id].tsx` — displays post by id                                                                |
| Pagination             |            /pagination/[page] |   GET    | params: page                |       No        | `app/pagination/[page]/page.tsx`, `@pages/pagination/[page].tsx`                                                                        |
| Markdown render        |               /md-render.html |   GET    | query or body (content/url) |       No        | `app/md-render.html/page.tsx`, `@pages/md-render.html.tsx` — Markdown renderer (possible XSS if not sanitized)                          |
| Nested routes list     |           /nested-routes.html |   GET    | -                           |       No        | `app/nested-routes.html/page.tsx`, `@pages/nested-routes.html.tsx`                                                                      |
| Nested route - item    |           /nested-routes/[id] |   GET    | params: id                  |       No        | `app/nested-routes/[id]/page.tsx`, `@pages/nested-routes/[id].tsx`                                                                      |
| Nested route - comment | /nested-routes/[id]/[comment] |   GET    | params: id, comment         |       No        | `app/nested-routes/[id]/[comment]/page.tsx`, `@pages/nested-routes/[id]/[comment].tsx`                                                  |
| AJAX demo              |            /request.ajax.html |   GET    | - (client-side AJAX)        |       No        | `app/request.ajax.html/page.tsx`, `@pages/request.ajax.html.tsx` — uses AJAX calls to internal endpoints                                |
| Sign-in                |                 /sign-in.html | GET/POST | body: email,password        |       No        | `app/sign-in.html/page.tsx`, `@pages/sign-in.html.tsx` — login form (see API auth routes if present)                                    |
| Dashboard              |         /dashboard/index.html |   GET    | -                           |  Possibly yes   | `app/dashboard/index.html/page.tsx`, `@pages/dashboard/index.html.tsx` (also `index.html.ServerSide.tsx`) — may display dynamic content |

> Note: the project contains both the `app/` folder (App Router) and a `@pages/` folder with legacy Pages Router files. They coexist and many routes are duplicated between `app` and `@pages`.

## API Endpoints

| ID  |  Method  |                  Endpoint | Body / Query          | Requires Auth? | Possible test vectors                                                   |
| --- | :------: | ------------------------: | :-------------------- | -------------: | ----------------------------------------------------------------------- |
| A1  |   GET    |                /api/posts | query: page, q        |             No | lists posts; test injection in `q` (XSS), pagination abuse              |
| A2  |   GET    |          /api/post-detail | query: id             |             No | returns post content — test XSS in HTML content if not sanitized        |
| A3  |   GET    |           /api/navigation | -                     |             No | used for navigation; minimal input — link enumeration                   |
| A4  | GET/POST |         /api/extract-file | query/body: url or id |             No | downloads/extracts files — test path traversal, SSRF depending on usage |
| A5  |   GET    |    /api/pagination/page-1 | -                     |             No | static pagination endpoints (page-1/2/3)                                |
| A6  |   GET    |    /api/pagination/page-2 | -                     |             No | same as above                                                           |
| A7  |   GET    |    /api/pagination/page-3 | -                     |             No | same as above                                                           |
| A8  |   GET    |         /api/video-stream | query: ...            |             No | streaming — test headers, range requests, header injection              |
| A9  |   GET    | /api/video-stream.generic | -                     |             No | alternative streaming endpoint                                          |
| A10 |   GET    |            /api/video-key | -                     |             No | may return key/URL — verify authorization                               |
| A11 | POST/GET |    /api/[...videoDecrypt] | body/query            |             No | dynamic decrypt route — review input handling (potential RCE/SSRF)      |
| A12 |   GET    | /api/dynamic-routes/[pid] | params: pid           |             No | dynamic route — test path handling                                      |

## Backend (express) routes / other server-side routes

| File                                              |                  Endpoint (base) |   Method | Notes                                                                          |
| ------------------------------------------------- | -------------------------------: | -------: | ------------------------------------------------------------------------------ |
| backend/routes/parse-htmlcssjs.js                 |      (likely internal endpoints) | GET/POST | HTML/CSS/JS parser — potential injection if user-supplied content is processed |
| backend/routes/example.js                         |                         /example | GET/POST | example route                                                                  |
| backend/routes/cache-static-assets.js             |             /cache-static-assets | GET/POST | manages asset cache                                                            |
| backend/routes/cache-delete-static-assets.js      |      /cache-delete-static-assets | GET/POST | cache removal — check permissions                                              |
| backend/routes/cache-delete-static-assets-spec.js | /cache-delete-static-assets-spec | GET/POST | spec for deletion                                                              |

## General notes and checks

- `server.js` exposes `/vars` as static (`plugins` directory) and handles `/a` and `/b` specially (renders pages `/a` and `/b`). Verify whether `/a` and `/b` exist or are special entry points.
- The app contains Next.js endpoints under `app/api/*` (App Router) and `@pages/api/*` (Pages Router). Many endpoints are duplicated between these folders.
- Authentication: there is no evidence of global protection in `server.js` (an auth middleware is present but commented out). Check `backend/core/auth` or `backend/server-auth.js` for auth/authorization logic.
- Environment variables: several scripts and routes depend on `process.env` (e.g., `NODE_ENV`, `EXPORT_ENABLED`, and possibly keys/API). See `package.json` (uses `cross-env`) and search for `process.env` in code to find sensitive values.
- Common attack surfaces to test: XSS when rendering posts/Markdown, SSRF/Path traversal in `extract-file`/file-download flows, unauthenticated access to endpoints that should require auth, insecure streaming endpoints, and upload/extract flows.
