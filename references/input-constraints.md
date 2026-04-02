# 用户输入约束 — 便于稳定生成

用户不需要接触 YAML 模板。Agent 负责从用户输入中提取信息并内部归一化。

## 输入方式 A：Markdown 文件（最常见）

用户提供 `.md` 文件，格式自由。Agent 读取后按 [content-taxonomy.md](content-taxonomy.md) 归类，内部转为结构化数据（字段定义见 [trip-draft.template.yaml](trip-draft.template.yaml)）。

常见格式：按日分段（`## Day 1`）、列表、表格、甚至口语化散文。Agent 应能处理所有这些变体。

---

## 输入方式 B：Notion 页面链接

用户给出 Notion URL，Agent 通过 MCP 或浏览器读取。若为 Notion 数据库，建议列名如下（用户不一定严格遵守，Agent 应灵活匹配）：

**一行 = 一个「日历日」**（或「半天」若你明确拆分）。

| 建议列名 | 对应 Taxonomy | 内容 |
|----------|---------------|------|
| `date` | meta + 日锚点 | `2026-04-24` |
| `day_title` | meta | 当日标题，如「威尼斯半日」 |
| `intercity_transport` | 大交通 | 纯文本一行，如「🚄 米兰→威尼斯 09:35-11:52」 |
| `schedule_md` 或拆多列 | 时间表 | 建议：多行文本，每行 `HH:MM-HH:MM | 活动 | 子说明` |
| `route_primary_url` | 当日路线 | Google `dir` 或 `goo.gl`；**可空** |
| `route_origin` / `route_destination` / `route_waypoints` | 当日路线 | 若 URL 空：用分号或 JSON 列表提供起终点与途经点，**由 Agent 生成** `maps/dir` 链接（规则见 [google-maps-url-builder.md](google-maps-url-builder.md)） |
| `route_alt_url` | 当日路线 | 备选线（如奥地利版），可空 |
| `hotel_name` | 住宿 | 空表示当晚无单独酒店块 |
| `hotel_address` | 住宿 | |
| `hotel_map_url` | 住宿 | |
| `hotel_overnight_date` | 住宿 | **哪一晚**（与 `date` 可不同：如凌晨入住算前一日） |
| `lunch_json` / `dinner_json` | 餐饮 | 或用 `reservations_md` 多行文本约定格式 |
| `notes` | 杂项 | |

**餐饮多行文本约定示例**（若不用 JSON）：

```text
lunch | 12:00 | Bouillon Bruxelles | Rue des Dominicains 7 | reserved | https://...
dinner | 19:00 | 店名 | 地址 | walk-in | 
```

---

## 输入方式 C：对话粘贴

用户直接在对话里粘贴行程文字。若信息不完整或有歧义，Agent 列出待确认项（如「哪一晚住哪」「路线起终点」），用户确认后再生成。

---

## Agent 侧校验（生成前）

- 每个有 `hotel_name` 的日，必须有 `hotel_overnight_date` 或默认「等于当日 `date` 的当晚」— **需在 YAML 或对话里显式确认**。
- **`routes[].url` 非空**：原样进入 `data-gmap-src` / `href`，**不改写**。
- **`routes[].url` 空且存在 `route_spec`（或表格中等价字段）**：按 [google-maps-url-builder.md](google-maps-url-builder.md) 生成 `maps/dir` URL；页面上提示用户**打开核对**；用户日后可换成自己保存的 `goo.gl` 覆盖。
- `schedule` 内出现的餐厅若与 `dining` 重复，HTML 可合并展示，但源数据两类都要能区分。
