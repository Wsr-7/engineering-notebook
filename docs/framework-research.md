# 个人工程门户：框架适配研究

- 研究日期：2026-10-20
- 状态：官方资料评估；未创建站点、安装依赖或构建实测。
- 范围：单人维护、内网静态 Git Pages、Markdown 知识库、交互展示、既有 HTML/SVG、浏览器本地搜索、阶段记录，以及 AI 辅助整理。

后续决定：作者已接受 Astro + Starlight 路线，并选择完全在公司环境开发和发布。当前开工范围见 [开工计划](kickoff-plan.md)；以下保留选型时的事实依据与取舍。

## 判断

**如果从零建设，并且文档与自由展示同样重要，优先考虑 Astro + Starlight。** 用 Starlight 承载常规文档；用 Astro 页面承载项目门户、业务演示、工作总结；既有独立 HTML 可以保留。Starlight 自带 Pagefind 搜索，Astro 页面也可以直接用浏览器脚本实现交互，无需为了每个演示引入 React 或 Vue。[Starlight 页面](https://starlight.astro.build/guides/pages/)、[默认搜索](https://starlight.astro.build/guides/site-search/)、[Astro 浏览器脚本](https://docs.astro.build/en/guides/client-side-scripts/)

这是基于当前需求的工程判断，**不是性能测试或维护成本实测排名**。如果公司已经有可复用的 Docusaurus 模板、发布流程或维护经验，继续用它也很合理。VitePress 则是文档为主、搭配少量定制页面时的有效备选。用户尚未表达 React/Vue 偏好，本次不据此作选择。

## 核心能力对照

| 需求 | Astro + Starlight | Docusaurus | VitePress |
| --- | --- | --- | --- |
| 常规知识库 | 文档用 Markdown/MDX。[来源](https://starlight.astro.build/guides/pages/) | 文档用 Markdown/MDX，可嵌入 React。[来源](https://docusaurus.io/docs/markdown-features/) | Markdown 可嵌入 Vue 组件和 HTML。[来源](https://vuejs.github.io/vitepress/v1/guide/using-vue) |
| 自由展示页面 | `src/pages` 可放 Astro、HTML 页面；可选择复用 Starlight 布局。[来源](https://starlight.astro.build/guides/pages/) | 可创建独立 React/TSX 页面，自行决定是否使用主题布局。[来源](https://docusaurus.io/docs/creating-pages) | 支持 `page`、`layout: false` 和自定义 Vue 布局。[来源](https://vuejs.github.io/vitepress/v1/reference/default-theme-layout) |
| 浏览器本地搜索 | 默认集成 Pagefind。[来源](https://starlight.astro.build/guides/site-search/) | 官方直接支持 Algolia；本地搜索采用社区维护插件。[来源](https://docusaurus.io/docs/search) | 内置 MiniSearch，可配置 `provider: 'local'`。[来源](https://vuejs.github.io/vitepress/v1/reference/default-theme-search) |
| 静态发布 | Astro 默认静态生成，可部署 GitHub Pages。[来源](https://docs.astro.build/en/guides/on-demand-rendering/)、[部署](https://docs.astro.build/en/guides/deploy/github/) | 生成 `build` 静态目录。[来源](https://docusaurus.io/docs/deployment) | 生成 `.vitepress/dist`，支持 Pages 部署。[来源](https://vuejs.github.io/vitepress/v1/guide/deploy) |

三者都能改变布局；参考图的三栏结构只是具体主题的呈现。真正需要权衡的是：多少页面复用文档布局、多少页面独立设计，以及这些定制以后由谁维护。

## 会影响选择的边界

### 1. 自由设计可以实现，但不同方式的维护成本不同

Astro + Starlight 可以把普通文档和自定义页面放在同一项目。自定义页面也可使用 `StarlightPage`，但不会自动出现在自动生成的侧栏里，需要安排入口。Starlight 支持覆盖主题组件；官方提醒，深改 `PageFrame`、`TwoColumnContent` 等结构需要理解相关组件关系。[页面与侧栏](https://starlight.astro.build/guides/pages/)、[组件覆盖](https://starlight.astro.build/guides/overriding-components/)、[覆盖接口](https://starlight.astro.build/reference/overrides/)

Docusaurus 的独立页面可以完全自定义。若要重写文档主题内部结构，则可使用 swizzling。官方明确区分稳定接口与内部组件：复制并改写 unsafe 组件可能增加升级工作，能通过包装实现时，优先包装。独立 Pages 默认没有 Docs 侧栏，不能把二者当成同一种页面。[独立页面](https://docusaurus.io/docs/creating-pages)、[swizzling 维护边界](https://docusaurus.io/docs/swizzling)

VitePress 可替换根 `Layout`，也可保留默认主题，只定制个别页面。定制组件仍需兼容构建时的服务端渲染；使用 `window`、DOM 或浏览器专用库时，要安排在客户端生命周期执行。[自定义主题](https://vuejs.github.io/vitepress/v1/guide/custom-theme)、[SSR 兼容](https://vuejs.github.io/vitepress/v1/guide/ssr-compat)

**建议：**常规文档复用主题；需要明显不同表达的业务演示使用独立页面。避免为了一个演示重写整个文档主题。

### 2. “能放 HTML”与“自动融入知识库”是两个验收点

Docusaurus 的 `static` 和 VitePress 的 `public` 会将文件按原样复制到构建产物，因此能保留已有 HTML、SVG 及其附件。完整 HTML 文件可作为独立页面链接；它不会因此自动套用文档布局。[Docusaurus 静态文件](https://docusaurus.io/docs/static-assets)、[VitePress 静态文件](https://vuejs.github.io/vitepress/v1/guide/asset-handling)

Astro + Starlight 官方示例直接展示了自定义 Astro 页面与独立 HTML 页面。搜索收录仍需单独处理：Pagefind 使用页面内容标记决定索引范围，不能承诺复制进去的所有 HTML 自动完整收录。[自定义页面](https://starlight.astro.build/guides/pages/)、[Pagefind 索引范围](https://pagefind.app/docs/indexing/)

VitePress 稳定版的本地搜索构建流程索引 Markdown 页面；独立 HTML 不在该默认页面集合中。这个边界由官方源码中的页面解析和索引循环确认。[页面集合源码](https://github.com/vuejs/vitepress/blob/v1.6.4/src/node/plugins/dynamicRoutesPlugin.ts)、[本地搜索源码](https://github.com/vuejs/vitepress/blob/v1.6.4/src/node/plugins/localSearchPlugin.ts)

**建议：**每个独立演示配一篇短 Markdown 说明，保存标题、用途、适用阶段、关键结论和演示入口。即使 HTML 正文未被索引，也能从知识库找到它。若以后需要搜索演示内所有文字，再扩展索引。

不能把任意完整 HTML 直接粘入 MDX 或 Vue Markdown 后就视为完成迁移：Docusaurus 默认按 MDX 语法处理 `.md`；VitePress 会把 Markdown 编译为 Vue，标签、脚本与样式会进入对应编译规则。[MDX 解析](https://docusaurus.io/docs/markdown-features/)、[Vue Markdown 规则](https://vuejs.github.io/vitepress/v1/guide/using-vue)

### 3. 内网搜索优先确认索引留在哪里

Starlight 的 Pagefind 和 VitePress 的内置本地搜索符合“构建索引、浏览器检索”的静态站方向。Docusaurus 也能采用本地搜索，但需挑选并维护社区插件兼容性；不能把它当作 classic 主题默认拥有的本地能力。[Starlight 搜索](https://starlight.astro.build/guides/site-search/)、[VitePress 搜索](https://vuejs.github.io/vitepress/v1/reference/default-theme-search)、[Docusaurus 搜索](https://docusaurus.io/docs/search)

Algolia 免费 DocSearch 面向公开技术文档及技术博客。Docusaurus 提到的“自己运行爬虫”链接属于旧版文档，其流程仍把记录发送到 Algolia；这不等于整套搜索都部署在内网。[申请条件](https://docsearch.algolia.com/docs/who-can-apply/)、[旧版自运行爬虫](https://docsearch.algolia.com/docs/legacy/run-your-own/)

中文、英文 API 标识和混合词检索需要拿真实内容验收。Pagefind 对中文分词有 extended release 条件，文档说明 `npx` 默认提供这一版本；VitePress 提供分词和检索参数定制。以上资料证明存在能力，不证明用户实际词库的召回效果。[Pagefind 多语言](https://pagefind.app/docs/multilingual/)、[MiniSearch 配置入口](https://vuejs.github.io/vitepress/v1/reference/default-theme-search)

### 4. 阶段快照不必升级为全站多版本文档系统

Docusaurus 自带文档版本机制，会复制整套 Docs 和相应侧栏。官方同时指出，它会增加构建时间与复杂度，多数网站不必启用。[版本机制](https://docusaurus.io/docs/versioning)

**建议：**把“某次评审、联调或交付时的说明”作为有日期的内容条目保存，记录适用版本和来源 commit/ticket；当前说明与历史说明分别给出清楚入口。只有同一 API 或产品需要长期并行维护多套完整文档时，再评估框架级版本功能。Git 历史能保存修改轨迹，但同事要直接访问旧说明时，仍需可见链接或独立快照页面。

### 5. AI 整理能力主要来自内容流程

**设计建议，非框架原生能力承诺：**AI 根据代码、草稿或 ticket 生成 Markdown、图示和演示草稿，再由作者确认业务含义和发布范围。三者的文本文件输入都可以接入这种流程；本次资料不能证明其中某个框架会更准确理解业务。

建议为内容保留少量一致字段：标题、项目、内容类型、状态、适用阶段、更新日期、来源链接。状态至少能表达“讨论草稿”“已确认说明”“历史快照”。这些字段有助于检索和生成工作总结，也能让读者判断一篇 API 说明是否适用于当前联调。

## 落地前的最小验收样例

这不是要求先建设完整系统，而是建议用以下五个真实样例检查选型：

1. 一篇含中文、request/response、错误码的 API 说明。
2. 一篇业务流程解释，包含图示和可复制示例。
3. 一页自由布局的交互演示，以及一个未经重写的现有 HTML/SVG。
4. 搜索中文业务词、英文 API 名、字段名，核对结果与跳转。
5. 在实际 Pages 子路径下打开、刷新、复制链接，检查资源路径、内网访问和历史快照入口。

落地时锁定相互兼容的已发布版本，避免把深层主题覆盖当作默认做法。本研究对 VitePress 采用官方 **v1.6.4 文档与对应源码作为稳定能力基线**；官网当前主文档已标注 2.0 alpha，因此没有把主文档新增能力用于稳定版承诺。[稳定版文档](https://vuejs.github.io/vitepress/v1/)、[当前主文档](https://vitepress.dev/guide/deploy)

尚未核验：公司实际 Pages 权限和发布流程、内网构建依赖来源、真实中文/API 搜索效果、现有 HTML 的脚本兼容性，以及内容规模下的加载与构建表现。
