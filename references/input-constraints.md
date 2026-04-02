# 用户输入约束 — 便于稳定生成

## 策略 A：优先使用「草稿 YAML」（推荐）

1. Agent 复制 [trip-draft.template.yaml](trip-draft.template.yaml) 到用户指定路径，例如 `.local/trips/my-trip.source.yaml`。
2. 用户**只在该文件**内按注释填充；Agent **只认**该结构生成 HTML。
3. 优点：类型边界清晰（住宿 ≠ 路线）；缺字段可留空或 `null`。

若用户坚持 Notion/表格，Agent 应把表格**映射**到下列字段名（或先转成 YAML 再生成）。

---

## 策略 B：Notion 数据库 — 建议列名

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

## 策略 C：Markdown 表格

表头至少包含：`Date | Day title | Schedule | Routes | Hotel | Dining | Notes`

- `Schedule`：单元格内用 `<br>` 或分号分隔多条 `时间 — 活动`。
- `Routes`：每行一个 URL，或 `主:` / `备:` 前缀。

---

## 策略 D：用户只有散文

Agent **不得**直接生成最终 HTML；应先：

1. 输出**结构化草稿**（YAML 或表格），标出「推断/待确认」；
2. 用户确认 `overnight_for`、路线 URL、订位时间后再生成页。

---

## Agent 侧校验（生成前）

- 每个有 `hotel_name` 的日，必须有 `hotel_overnight_date` 或默认「等于当日 `date` 的当晚」— **需在 YAML 或对话里显式确认**。
- **`routes[].url` 非空**：原样进入 `data-gmap-src` / `href`，**不改写**。
- **`routes[].url` 空且存在 `route_spec`（或表格中等价字段）**：按 [google-maps-url-builder.md](google-maps-url-builder.md) 生成 `maps/dir` URL；页面上提示用户**打开核对**；用户日后可换成自己保存的 `goo.gl` 覆盖。
- `schedule` 内出现的餐厅若与 `dining` 重复，HTML 可合并展示，但源数据两类都要能区分。
