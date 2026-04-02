---
name: doc-to-travel-guide-web
description: 将结构化行程文档转为单文件攻略 HTML；内置内容分类、YAML 草稿、Notion 列名、Google 链接拼接；每次生成后写入同名的信息源清单 sources.md。用于「行程变攻略网页」、itinerary、草稿再填。输出：.html + .sources.md（默认）。
---

# 文档 → 攻略 Web（单页 HTML）

**Announce:** 「使用 doc-to-travel-guide-web：从结构化行程文档生成攻略网页。」

把**已写好的行程事实**落到**一个可分享的 `.html`**，而不是再发明行程。生成前必须先做**内容分类**（见下），避免把住宿、餐厅、驾车路线混在同一自由文本里无法解析。

样式与交互可参考任意已完成的单页攻略 HTML（例如分屏 Leaflet、`data-gmap-src`、折叠 Google 嵌入、预订卡片）；本仓库不含范例文件，可与生成结果对照迭代。

---

## 何时触发

- 用户提供 Notion 表、YAML、Markdown 表格、CSV、或聊天内粘贴的「按日」行程
- 用户说：攻略页、行程网页、itinerary、旅行手册 web、打印友好行程单
- 用户只有模糊长文：先走 **草稿模板** 再生成

---

## 内容分类（必读）

文档里的信息要拆成不同类型，对应不同 UI 模块：

| 类型 | 典型内容 |
|------|----------|
| **当日时间表** | 同一天内几点做什么（不含整段城际票务全文） |
| **当日路线 / 导航** | Google `dir`、`goo.gl`、区域地图链接 |
| **大交通 / 城际** | 飞/铁/租车 A→B、班次时长价（若有） |
| **今晚住宿** | 酒店名、地址、地图链接、确认号、**指哪一晚**（`overnight_date`） |
| **餐饮** | 店名、订位时间、地址、是否已订、链接 |
| **景点科普** | 可选折叠文案 |
| **门票与准备** | 跨日博物馆票、通票等 |
| **杂项** | 提醒、Plan B |

完整定义与 HTML 映射见 **[references/content-taxonomy.md](references/content-taxonomy.md)**。

---

## 用户输入怎么约束

1. **推荐**：复制 **[references/trip-draft.template.yaml](references/trip-draft.template.yaml)** 到用户仓库路径（如 `.local/trips/<名>.source.yaml`），让用户按注释填写；Agent **以该 YAML 为唯一真源**生成 HTML。
2. **Notion / 表**：列名与含义见 **[references/input-constraints.md](references/input-constraints.md)**（含建议列名、餐饮多行格式）。
3. **散文 / 无表格**：**不要直接出成品页**；先输出填好的 YAML 草稿或表格，标出待用户确认的「哪一晚住哪」「路线 URL」，确认后再生成。

**原则**：缺字段就 `待填` 或省略模块；**不编造**预订、票价、时刻。

---

## 推荐工作流

0. **输入判断** — 若结构不清：先下发 `trip-draft.template.yaml` 或结构化大纲，再生成。
1. **归类** — 按 [content-taxonomy.md](references/content-taxonomy.md) 把每条信息标类型；冲突处问用户一句。
2. **归一数据模型** — 优先 YAML：`trip` + `days[]`（`schedule`、`routes[]`、`accommodation`、`dining[]` 等），与模板字段一致。
3. **生成单页 HTML** — 见 [references/html-patterns.md](references/html-patterns.md)；结构清晰、语义标签、`lang` 与标题反映真实路线。
4. **地图与嵌入** — Leaflet 小图按「城市尺度」拆屏（跨城日不要一条线 fit 全球）。Google：**优先**使用用户提供的 `routes[].url`；若为空但有 `route_spec`（起/终/途经点），按 [references/google-maps-url-builder.md](references/google-maps-url-builder.md) **生成** `maps/dir` 或 `maps/search` 链接，写入 `href` / `data-gmap-src`，并在页上提示用户**打开 Google 核对**。嵌入侧仍用 `buildGmapEmbedFromDocUrl` 处理 `dir` / `search` / `goo.gl`；驾车 iframe **可能不完整**，必须保留**同款**按钮。详见 [references/google-maps-embed.md](references/google-maps-embed.md)。
5. **外链图片** — 插入前用 `curl -sS -o /dev/null -w '%{http_code}' -L -A 'Mozilla/5.0' --max-time 20 "<url>"` 校验 **200**；优先稳定 CDN；不可用则换图或改本地资源路径。
6. **自检** — [references/verification-checklist.md](references/verification-checklist.md)；若用户要求，用浏览器 MCP 打开本地 HTTP 服务下的页面做冒烟检查。
7. **信息源清单（必做）** — 每次生成或重大更新 HTML 后，**另写** `*.sources.md`，把本次收集/使用的信息源集中归档：真源文件路径、Notion/Confluence URL、对话粘贴说明、进入页面的地图/图片链接、以及 **Agent 推断项**（待用户核对）。格式与模板见 **[references/source-manifest.md](references/source-manifest.md)**。默认与 HTML **同目录、同 basename**（如 `trip.html` → `trip.sources.md`）。

---

## 与用户约定（执行前可一句确认）

- 输出路径（例：`trip-name.html` 或 `docs/trips/xxx.html`）
- 信息源清单路径（默认：`同目录/<同名>.sources.md`）
- 是否需要 Leaflet + 是否允许外链瓦片
- 是否嵌入 Google（接受「路线 iframe 受限」说明）

---

## 范例与模式（无捆绑 HTML）

- 新攻略**不必**固定某一视觉，但应复用**可靠模式**：跨城地图拆分、文档链接驱动的 Google 嵌入、图片 URL 可访问性校验、iframe 懒加载。
- 若你另有「参考 `.html`」，在对话里把路径或链接交给 Agent 即可。

---

## 可选搭配（本仓库外）

以下非本 skill 依赖；按需在你自己的 Cursor / 规则里启用：

- **读需登录网页（Confluence 等）**：单独配置 MCP 或 Chrome CDP 类 skill。
- **与用户确认话术**：团队自有 `ai-interaction` 或项目规则。
- **从单页改为 React 等**：遵循项目内 `frontend-guidelines` 或框架规范。
