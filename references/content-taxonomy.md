# 行程文档 — 内容分类（Taxonomy）

生成攻略页前，必须把用户文档里的信息**归类**到下列类型。不同类型在 HTML 里对应不同模块；**不要**把「住宿」写进「时间表」的小字而不打标签，否则无法稳定生成预订卡片与地图。

---

## 1. 元信息（Trip meta）

| 键 | 含义 | 示例 |
|----|------|------|
| 行程标题、副标题 | 整页 H1 / hero | 「五一托斯卡纳」 |
| 起止日期、天数 | hero / 页脚 | `2026-04-24` — `05-04` |
| 途经城市/国家列表 | chips、总览地图 | 米兰、威尼斯 |
| 语言/货币提示 | 可选 note | 欧元、意铁订票 |

---

## 2. 当日时间轴（Daily schedule）

**是什么**：按**同一天内**的时间顺序排列的活动（到达景点、吃饭、逛街、还车等）。

**不是什么**：不是跨城机票/火车的完整票务段落（可归到大交通）；不是酒店地址全文（归到住宿）。

| 子字段 | 说明 |
|--------|------|
| `time` | 时段或模糊标签（上午 / 📍） |
| `activity` | 主句，可加 **strong** |
| `detail` | 子说明（小字） |

---

## 3. 当日路线 / 导航（Daily route & maps）

**是什么**：用户在 Google Maps 里保存的**驾车/步行路线**、或**单日活动地理范围**对应的链接。

| 子类型 | 典型 URL | 页面表现 |
|--------|----------|----------|
| 多点驾车路线 | `google.com/maps/dir/?api=1&...` | `map-link` 按钮 + 可选 `data-gmap-src` 嵌入 |
| 短链路线 | `maps.app.goo.gl/...` | 同上 |
| 单点搜索/地点 | `maps/search/?api=1&query=...` | 嵌入或按钮；可与住宿/餐厅共用 |

**约束**：一条路线一个 URL；若同一天有「意大利版 / 奥地利版」两条线，拆成两个 `routes[]` 项。用户未提供链接时，可由 Agent 根据 `origin` / `destination` / `waypoints`（见 `trip-draft.template.yaml` 的 `route_spec`）按 [google-maps-url-builder.md](google-maps-url-builder.md) 生成 `maps/dir` URL，并提示用户打开 Google 核对。

---

## 4. 大交通 / 城际 Leg（Intercity transport）

**是什么**：**城市 A → 城市 B** 的航班、高铁、租车起点终点、渡轮等（通常写在日头或单独一行）。

**与「当日路线」区别**：城际 leg 强调**交通工具+班次/时长/价格**；当日路线强调**市内/区域内**动线。

---

## 5. 住宿（Accommodation）

**是什么**：**某一晚**睡在哪里（酒店 / Airbnb / 朋友家）。

| 子字段 | 说明 |
|--------|------|
| `overnight_for` | **关键**：指「哪一夜」。建议用日期表示「当晚入住日」如 `2026-04-26`（26 号晚上睡这家） |
| `name` | 酒店名 |
| `address` | 可复制给司机/导航 |
| `map_url` | Google 搜索或 place 链接 |
| `booking_ref` | 确认号、平台 |
| `check_in_out` | 若已知 |

**常见用户错误**：把酒店只写在 Day 3 正文里但没标「今晚住这」— 生成时应归入 `accommodation` 模块。

---

## 6. 餐饮（Dining / reservations）

用户的输入中，「美食」信息常常**混杂两类内容**，Agent 必须先区分：

### 6a. 餐厅（Restaurant）— 可纳入行程

有具体店名，通常附带地址、订位信息，是**可执行的行程节点**。

| 子字段 | 说明 |
|--------|------|
| `meal` | `breakfast` / `lunch` / `dinner` / `cafe` / `snack` |
| `name` | 店名 |
| `time` | 订位时间（如有） |
| `address` | |
| `map_url` | |
| `reserved` | `true` / `false` |
| `links` | 官网等 |

**处理**：归入当日 `dining[]`，在时间表中体现，生成预订卡片。可与「时间表」里同一顿饭**互链**：时间表一行写时间线，预订卡片写详情。

### 6b. 特色菜 / 美食推荐（Local food tips）— 参考信息

只有菜名或食物类型，没有具体餐厅，是**当地饮食指南**。

示例：「米兰烩饭 Risotto alla Milanese」「卡邦尼意面 Carbonara」「找 bacaro（小酒馆）」

**处理**：不放进 `dining[]`，而是归入当日的 `food_tips`（或写进当日 `notes` / 独立的「当地美食」折叠块）。在 HTML 中用 `food-tips` 区域呈现（如侧栏卡片、折叠列表），与正式餐饮预订视觉区分。

### 如何判断

| 线索 | → 归为 |
|------|--------|
| 有店名 + 地址 / 地图链接 / 订位 | 餐厅 → `dining[]` |
| 有店名但无地址，上下文暗示「推荐去」 | 餐厅 → `dining[]`（`reserved: false`） |
| 只有菜名 / 食物类型 / 「当地必吃」 | 美食推荐 → `food_tips` |
| 「找 bacaro」「路边摊薯条」等泛指 | 美食推荐 → `food_tips` |
| 混合：「Bouillon Bruxelles 点 moules-frites」 | 餐厅 → `dining[]`，菜名写入 `notes` 字段 |

---

## 7. 景点与背景（Attractions / editorial）

**是什么**：可选的科普、历史、拍照提示（非硬性「事实」）。

单独用折叠块 `spotlight-fold` 呈现，与时间表区分。

---

## 8. 门票 / 准备事项（Tickets & prep）

**是什么**：跨多日的博物馆票、通票、签证、租车材料等。

通常放在全页「门票」区块或首日 `notes`，不强行塞进某一天。

---

## 9. 杂项备注（Misc notes）

**是什么**：提醒、备选方案、天气、着装。

用 `note-box` 或当日 `raw_notes`。

---

## 映射到页面模块（速查）

| Taxonomy | HTML 区域（参考） |
|----------|-------------------|
| meta | hero、footer、总览 |
| intercity_transport | `day-transport`、卡片头 |
| daily schedule | `ul.schedule` |
| daily route | `map-link`、`gmap-route-from-doc`、Leaflet `dayMap*` |
| accommodation | `rsv-card` 或独立 `accommodation` 块（可扩展） |
| dining (餐厅) | `rsv-card`、`food-section` |
| food_tips (美食推荐) | `food-tips`（折叠卡片 / 侧栏 / 当日底部列表） |
| attractions | `spotlight-fold` |
| tickets | `tickets-section` |
| misc | `note-box` |
