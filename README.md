# create-page

一个 AI Agent Skill：根据项目类型自动创建页面并注册路由。

## 支持的项目类型

| 项目类型 | 检测依据 | 输出文件 |
|---|---|---|
| 原生微信小程序 | `app.json` + `project.config.json`（或 `.wxml/.wxss` 文件） | `pages/{{PAGE_NAME}}/{{PAGE_NAME}}.*`（支持 JS / TS） |
| uniapp | `pages.json` + `manifest.json` + `@dcloudio` 依赖 | `pages/{{PAGE_NAME}}/{{PAGE_NAME}}.vue`（Vue2 选项式 / Vue3 组合式） |
| Vue | package.json 依赖 `vue` | `src/views/{{PAGE_NAME}}/index.vue`（Vue2 / Vue3） |
| React | package.json 依赖 `react` / `next` | `src/pages/{{PAGE_NAME_PASCAL}}/index.tsx`（TS / JS） |

## 功能特性

- 自动检测项目类型（含 TS/JS、Vue2/Vue3、命名风格、路由风格），检测不到会询问而不是瞎猜
- 自动识别小程序 / uniapp 分包，让用户选择页面归属
- 解析中英文页面名，按项目现有命名约定（kebab-case / PascalCase）转换
- 自动注册路由：小程序 `app.json`、uniapp `pages.json`、vue-router、react-router、Next.js 文件路由
- 目标文件已存在时不会覆盖，会先询问用户
- 完成后报告：项目类型、创建文件列表、路由注册位置（文件 + 行号）、页面访问路径

## 使用方法

这是一个 [Agent Skills](https://github.com/anthropics/skills) 格式的技能。对 AI Agent 说出类似下面的请求即可触发：

- "帮我创建一个用户列表页面"
- "新建一个商品详情页"
- "add a page: 订单管理"

## 目录结构

```
create-page/
├── SKILL.md                  # 技能定义（工作流说明）
└── templates/                # 页面模板
    ├── react/                # index.tsx / index.jsx
    ├── uniapp/               # page.vue (Vue3) / page-options.vue (Vue2)
    ├── vue/                  # index.vue (Vue3) / index-options.vue (Vue2)
    └── wx-mini/              # page.js / page.ts / page.wxml / page.wxss / page.json
```

模板中的占位符：

- `{{PAGE_NAME}}`：页面文件名（kebab-case）
- `{{PAGE_NAME_PASCAL}}`：PascalCase 页面名（如 `UserList`）
- `{{PAGE_TITLE}}`：页面标题
- `{{PAGE_PATH}}`：路由注册用的完整路径（如 `pages/user-list/user-list`）

## 安装

将本目录复制到你的 Agent 技能目录下，例如：

- Claude Code: `~/.claude/skills/create-page/`
- Kimi Code: `~/.agents/skills/create-page/`

然后重启会话或刷新技能列表即可。

## License

MIT
