# 由路线描述生成 Google Maps 导航链接（无 API Key）

当用户**没有**从 Google Maps 复制链接，但提供了 **起点、终点、途经点**（地名或地址）时，Agent **可以**拼接官方「网页导航」URL，用于页面上的 `href` 与 `data-gmap-src`（嵌入逻辑同 [google-maps-embed.md](google-maps-embed.md)）。

---

## 基础格式（Directions URL）

```text
https://www.google.com/maps/dir/?api=1&origin=<ORIGIN>&destination=<DEST>&waypoints=<W1>%7C<W2>%7C...&travelmode=<MODE>
```

- 所有占位符均做 **`encodeURIComponent`**（空格 → `%20`，`|` 在 `waypoints` 里用 **`%7C`** 分隔多段，不要裸写 `|` 以免被浏览器截断）。
- **`travelmode`**：`driving`（默认）| `walking` | `transit` | `bicycling`。
- **途经点顺序**：与 `waypoints` 数组顺序一致。

### 示例（概念）

- `origin=Venice Mestre, Italy`
- `destination=Dobbiaco, Italy`
- `waypoints=Pieve di Cadore, Italy|San Candido, Italy`  
→ `waypoints` 参数值为编码后的 `Pieve...%7CSan...`（整串再作为单个 query 值）。

---

## 实现要点（Agent）

1. **优先用户链接**：`routes[].url` 非空时 **原样使用**，不覆盖。
2. **`url` 空且存在 `route_spec`**：根据 `origin`、`destination`、`waypoints[]`、`travelmode` 拼 URL；写入 YAML/HTML 时可在注释或页面小字标「链接由地点名生成，请出发前在 Google 地图中再确认」。
3. **地点写法**：尽量具体（城市+地标/街道/邮编），减少歧义；若有坐标可写 `lat,lng`（Google 一般能识别）。
4. **途经点数量**：URL 过长可能被截断；建议 **waypoints ≤ 8～10**；更多点拆成多条 `routes[]` 或只保留关键停靠。
5. **往返/环线**：`origin` 与 `destination` 可相同（如 `Florence` → `Florence`），途经点写 `waypoints`。
6. **不要编造**：用户未给出的停靠点不要擅自加入；不确定时留空 `url` 并只生成「起点→终点」或提示用户去 Maps 里保存短链。

---

## 单点搜索（非路线）

仅一个地点、无路线时，用：

```text
https://www.google.com/maps/search/?api=1&query=<encodeURIComponent(地点或地址)>
```

适用于：酒店、餐厅、停车场嵌入/按钮。

---

## 与用户沟通

- 生成链接后建议一句：**「请在 Google 地图中打开核对路线是否与你计划一致。」**
- 若用户后续在 Maps 里调整了路线，应让用户把 **`maps.app.goo.gl` 短链**或完整 `dir` URL 贴回，覆盖 Agent 生成的 URL。
