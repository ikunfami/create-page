---
name: create-page
description: Detect the project type (native WeChat Mini Program, uniapp, Vue, React) and automatically create one or more pages from the user's page names, registering the routes. Supports batch creation — separate multiple pages with 、 , or newlines. Use when the user asks to "create a page / add pages / new page / 批量创建页面".
---

# create-page: create pages matching the project type

Supports **single page** and **batch creation** (multiple pages in one request). Steps 1–2 (detection) run once per request; steps 3–5 loop per page.

## Workflow

### 1. Detect the project type

Check in the following order; stop at the first match and record the evidence (report it to the user):

| What to check | Result |
|---|---|
| `pages.json` + `manifest.json` exist, and package.json contains `@dcloudio` | uniapp |
| `app.json` and `project.config.json` exist (or page files use `.wxml/.wxss`) | Native WeChat Mini Program |
| package.json dependencies contain `react` / `next` | React |
| package.json dependencies contain `vue` | Vue |

If none match, list the files you actually found and ask the user to confirm the project type — do not guess.

Also collect the following context (it determines template details):

- **TS or JS**: check whether `tsconfig.json` exists and the ratio of `.ts/.tsx` files in the project.
- **Vue2 or Vue3**: check the vue version in package.json (>= 3 means Vue3).
- **Page directory and naming conventions**: check where existing pages live (`src/views` vs `src/pages`, etc.) and whether file names are kebab-case or PascalCase. **The new page must follow the project's existing conventions.**
- **React routing style**: look for a react-router config file; if it is Next.js (has `pages/` or `app/` directory), use file-based routing.

### 2. Native Mini Program / uniapp: subpackage detection (asked once per request)

Read `app.json` (native mini program) or `pages.json` (uniapp):

- If a non-empty `subPackages` field exists (also accept the legacy `subpackages` key) → use AskUserQuestion to list every subpackage (`root` path + name) plus a "main package" option, and let the user choose which package the page(s) go into. **With multiple pages, all pages default to the chosen package** — state this in the question (e.g. "以下 N 个页面全部放入所选包"); only ask again if the user explicitly wants different packages for different pages.
- If there is no subpackage → proceed with the main package normally.

### 3. Parse the user input (single or multiple pages)

First split the input into page entries — separators: `、` `，` `,` or newlines. Examples:

- `创建页面 用户列表 user-list` → 1 entry
- `创建页面 用户列表 user-list、订单详情 order-detail、设置 settings` → 3 entries
- A bare word like `批量创建页面` with no entry following → ask for the page list.

For **each** entry, parse:

- The Chinese part → page title (e.g. "用户列表")
- The English part → page name (e.g. `user-list`)
- If only Chinese is given: confirm the English name with the user when there is just **one** page; for **batch** requests, auto-convert each to a pinyin-style kebab-case name and present the full mapping list for one-shot confirmation instead of asking per page.
- Convert the page name following the project's existing naming convention (kebab-case or PascalCase)

Placeholder conventions (replaced inside templates):

- `{{PAGE_NAME}}`: page file/directory name (kebab-case)
- `{{PAGE_NAME_PASCAL}}`: PascalCase (e.g. `UserList`)
- `{{PAGE_TITLE}}`: page title
- `{{PAGE_PATH}}`: full path used for route registration (e.g. `pages/user-list/user-list`)

### 4. Generate the page files (loop over every page)

Templates live in the `templates/` directory of this skill:

| Project type | Template | Output location |
|---|---|---|
| Native mini program (JS) | `wx-mini/page.js` + `page.wxml` + `page.wxss` + `page.json` | `pages/{{PAGE_NAME}}/{{PAGE_NAME}}.*`; under a subpackage: `{subpackageRoot}/{{PAGE_NAME}}/{{PAGE_NAME}}.*` |
| Native mini program (TS) | Same as above, but use `page.ts` instead of page.js | Same |
| uniapp (Vue3) | `uniapp/page.vue` | `pages/{{PAGE_NAME}}/{{PAGE_NAME}}.vue`; under a subpackage: `{subpackageRoot}/...` |
| uniapp (Vue2) | `uniapp/page-options.vue` | Same |
| Vue (Vue3) | `vue/index.vue` | `src/views/{{PAGE_NAME}}/index.vue` (follow the project's existing directory) |
| Vue (Vue2) | `vue/index-options.vue` | Same |
| React (TS) | `react/index.tsx` | `src/pages/{{PAGE_NAME_PASCAL}}/index.tsx` (follow the project's existing directory) |
| React (JS) | `react/index.jsx` | Same |

Additional rules:

- Apply the table above **per page**, looping until every parsed page has been generated.
- If the project uses scss/less, change the style file extension accordingly (content unchanged).
- For uniapp/Vue projects using TS, change `<script setup>` to `<script setup lang="ts">`.
- Do not add `usingComponents` (mini program) or easycom config (uniapp); keep the templates as-is.
- **If a target file already exists: do not overwrite it.** Skip that page, record it as "skipped (already exists)", and continue with the remaining pages. At the end, ask the user whether to rename or cancel the skipped ones.

### 5. Register the routes (append all at once)

Collect the route entries of **all successfully generated pages** first, then register them in a single edit per file:

- **Native mini program**: append every path to the end of the main `pages` array in `app.json`, or the `pages` array of the chosen subpackage (subpackage paths omit the root prefix, e.g. `"user-list/user-list"`). **Never insert at index 0** — the first entry is the mini program home page. Edit the file once, adding all new entries in one operation.
- **uniapp**: same as above, edit `pages.json` once with all new entries.
- **Vue (vue-router)**: append one object per page to the router config file:
  ```js
  {
    path: '/{{PAGE_NAME}}',
    name: '{{PAGE_NAME_PASCAL}}',
    component: () => import('@/views/{{PAGE_NAME}}/index.vue'),
    meta: { title: '{{PAGE_TITLE}}' },
  }
  ```
  Follow the router file's existing style (skip `meta` if the project does not use it). For file-based routing projects (unplugin-vue-router, Nuxt), only create the files — the routes are generated automatically.
- **React (react-router)**: append every path and import at the route config in one edit, following the project's existing style (`element` for v6, `component` for v5).
- **Next.js**: for the pages directory, placing each file at `pages/{{PAGE_NAME}}.tsx` is enough; for the app directory, create `app/{{PAGE_NAME}}/page.tsx` per page.

### 6. Report

When done, report: the detected project type, then a summary table with one row per page:

| 页面 | 创建的文件 | 路由注册位置 | 访问路径 | 状态 |
|---|---|---|---|---|
| UserList 用户列表 | `src/views/user-list/index.vue` | `src/router/index.ts:42` | `/user-list` | 成功 |
| OrderDetail 订单详情 | — | — | — | 已存在，跳过 |

End with the overall result: N created, M skipped, and remind the user of any skipped pages awaiting their decision (rename or cancel).
