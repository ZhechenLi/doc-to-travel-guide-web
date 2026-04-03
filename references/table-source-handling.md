# 不同表格来源的读取与解析

用户的行程数据可能来自各种表格形式。不同来源的读取方式、数据结构、常见坑各不相同。Agent 应根据来源类型选择对应策略。

---

## Notion 表格（simple table）

用户给的 Notion 页面里常见的是 **simple table**（非数据库），一行一天。

### 数据结构

```
page
  └─ table (format.table_block_column_order 记录列顺序)
       ├─ table_row (properties 里按列 ID 存各列内容)
       ├─ table_row
       └─ ...
```

- **第一行通常是表头**（列名），但它和数据行一样都是 `table_row`，Agent 需要自行识别
- 列 ID 是随机短字符串（如 `=<y<`、`NDR[`），不是列名本身；需要把第一行的值当作列名映射
- 每个单元格是**富文本数组**：`[[" 正文 "], [" 链接文字 ", [["a", "https://..."]]]]`，提取时拼接第一个元素即可得到纯文本

### 读取方式

1. **浏览器 MCP**（首选）：打开页面，用 `scrollIntoView` 逐段截图提取。适合公开和私有页面
2. **内部 API**（公开页面 fallback）：`POST /api/v3/loadPageChunk`，一次返回全部 block，解析 `table_row.properties`。详见 [input-constraints.md](input-constraints.md)

### 常见坑

- 单元格内**换行**在 API 返回中是 `\n`，在浏览器截图中是视觉换行——两种方式都能拿到，但截图可能因行高截断
- 单元格内的**超链接**在 API 中是富文本第二层 `[["a", "url"]]`，浏览器 snapshot 里是 `role: link`
- Notion simple table **没有列名元数据**，列名就是第一行的内容

---

## Notion 数据库（database / collection）

比 simple table 更结构化，每列有明确的 type（text / date / select 等）。

### 数据结构

```
page
  └─ collection_view (collection_id + view_ids)
       └─ collection (schema 定义列名和类型)
            ├─ page (每行是一个子 page，properties 按 schema key 存)
            └─ ...
```

### 读取方式

1. **浏览器 MCP**：打开页面，看到的是渲染后的表格视图，逐段截图提取
2. **内部 API**：`loadPageChunk` 返回中会有 `collection` 和 `collection_view` 记录；需要额外调 `queryCollection` 获取行数据
3. **Notion 官方 API**（最佳）：若有 token，直接 `GET /v1/databases/{id}/query`，返回结构化的行数据

### 与 simple table 的区别

| | Simple Table | Database |
|---|---|---|
| 列名来源 | 第一行内容 | `collection.schema` |
| 行类型 | `table_row` | 子 `page` |
| API 解析 | `loadPageChunk` 一次拿完 | 需要 `queryCollection` |
| 列类型 | 全是文本 | 有 date / select / relation 等 |

---

## Markdown 表格

用户直接提供 `.md` 文件中的管道符表格。

### 格式

```markdown
| 日期 | 城市 | 行程 | 备注 |
|------|------|------|------|
| 04.24 | 米兰 | 抵达，吃晚饭 | 深圳-布鲁塞尔-米兰 |
```

### 解析要点

- 第一行是表头，第二行是分隔符 `|---|`，第三行起是数据
- 单元格内换行：标准 Markdown 不支持单元格内换行；用户可能用 `<br>` 或直接写多行文本（非标准但常见）
- 列对齐可能不整齐，按 `|` 分割即可

---

## HTML 表格

用户粘贴或提供来自网页的 HTML 表格。

### 解析要点

- `<thead>` / `<th>` 是列名，`<tbody>` / `<td>` 是数据
- 注意 `colspan` / `rowspan` — 行程表可能有跨天合并的住宿单元格
- 单元格内可能有 `<br>`、`<a>`、`<strong>` 等标签，需要提取纯文本 + 保留链接

---

## CSV / 电子表格粘贴

用户从 Excel / Google Sheets 复制粘贴，或提供 `.csv` 文件。

### 解析要点

- 分隔符通常是逗号或 Tab
- 第一行是列名
- 单元格内如果有逗号，整个值会被引号包裹：`"Bouillon Bruxelles, 午餐"`
- 从 Excel 粘贴到对话中时，列之间通常是 **Tab 分隔**，看起来像空格但不是

---

## 通用处理流程

无论来源，拿到表格数据后执行相同的归一化：

```
1. 识别列名 → 映射到 taxonomy 字段（date / schedule / dining / hotel / notes / food ...）
   - 列名不一定精确匹配，按语义灵活对应
   - 例：「美食」列 → 需区分餐厅和食物推荐（见 content-taxonomy.md §6）

2. 逐行提取 → 每行对应一个 day
   - 一行可能包含多种 taxonomy 类型的信息

3. 单元格内容拆分 → 一个单元格可能有多条信息
   - 用换行符、分号、编号等拆分
   - 链接单独提取，归入对应字段的 url / map_url

4. 归一化为内部数据模型 → trip + days[]
   - 字段定义见 trip-draft.template.yaml
```
