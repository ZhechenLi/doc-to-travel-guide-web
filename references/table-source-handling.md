# 外部来源的读取与解析

用户给出一个 URL（Notion、Google Sheets、Confluence、任意网页）或文件时，Agent 不需要针对每种来源写专门的解析器。统一走**逐级降级**的导出管道，拿到一种可解析的中间格式后再进入归一化。

---

## 核心思路：逐级降级导出

```
URL / 文件
  │
  ▼
① 获取完整 HTML（浏览器打开页面）
  │
  ▼
② 尝试转 Markdown
  │  ├─ 结构清晰、信息完整 → 直接用 MD 进入归一化
  │  └─ 信息丢失（复杂布局、合并单元格、嵌入内容）↓
  ▼
③ 截图 / 导出 PDF
  │  ├─ 截图：逐屏截取，基于图片做 OCR 或视觉理解
  │  └─ PDF：如平台支持导出（Notion Export、Google Sheets 下载 PDF）
  │
  ▼
④ 平台专属 API（最后手段）
     Notion loadPageChunk / 官方 API、Google Sheets API 等
     只在前面几步都不够时才用，因为依赖平台且可能需要 token
```

**原则**：能用通用方式解决的不用专属方式。MD 够了就不截图，截图够了就不调 API。

---

## 第 ① 步：获取完整 HTML

通过浏览器 MCP 打开页面。关键操作：

- **等待渲染**：SPA 页面（Notion、Google Sheets）需要等待 JS 渲染完成
- **处理嵌套滚动**：用 `scrollIntoView` 跳转到页面各区域（见 browser-scroll-strategy skill）
- **处理登录**：如果页面需要登录，浏览器 MCP 使用的是用户已登录的会话

此步产出：浏览器中完整渲染的页面。

---

## 第 ② 步：尝试转 Markdown

从浏览器拿到的页面内容，尝试提取为 Markdown：

### 方法 a：从页面文本直接提取

- 通过 `browser_snapshot` 获取可交互元素树 + 逐段截图读取文本
- 手动组织为 Markdown 格式（标题、列表、表格）

### 方法 b：利用平台自带导出

| 平台 | 导出方式 |
|------|---------|
| Notion | 页面右上角 `···` → Export → Markdown & CSV |
| Google Docs | File → Download → Markdown (.md) |
| Confluence | 页面菜单 → Export → Word（再转 MD）或直接通过 MCP 读取 |

### 方法 c：HTML → Markdown 转换

如果能拿到页面 HTML 源码，用工具转换：

```bash
# 示例：用 pandoc 转换
curl -sS '<url>' | pandoc -f html -t markdown
```

### 判断 MD 是否「够用」

- 表格结构完整：列名 + 数据行都在
- 链接保留：Google Maps URL、票务链接没丢
- 多行内容没被截断：单元格内的换行、子条目都在
- 如果以上任何一条不满足 → 降级到第 ③ 步

---

## 第 ③ 步：截图 / 导出 PDF

当 Markdown 无法完整表达页面内容时：

### 截图方式

1. 用 `scrollIntoView` 定位到表格各区域
2. 逐段 `browser_take_screenshot`
3. 基于截图进行视觉理解，提取表格数据

**适用场景**：复杂布局、合并单元格、带背景色的分类标记、嵌入图片等。

### PDF 导出

部分平台支持导出 PDF，保留完整排版：

| 平台 | PDF 导出 |
|------|---------|
| Notion | Export → PDF |
| Google Sheets | File → Download → PDF |
| 网页 | 浏览器打印 → 另存为 PDF |

PDF 可交给支持 PDF 读取的工具解析。

---

## 第 ④ 步：平台专属 API（按需）

前面三步已经能覆盖绝大多数场景。只在以下情况才使用平台 API：

| 场景 | API |
|------|-----|
| Notion 公开页面需要精确的富文本和链接 | `POST /api/v3/loadPageChunk`（无需 token） |
| Notion 私有页面 + 有 token | 官方 API `GET /v1/pages/{id}` |
| Google Sheets 需要精确的单元格数据 | Google Sheets API（需 API key） |
| Confluence | 通过 Atlassian MCP 读取 |

详见 [input-constraints.md](input-constraints.md) 中的 Notion API 部分。

---

## 各来源的特殊注意事项

以下是在走通用管道时，各来源可能碰到的额外问题：

### Notion

- Simple table 的第一行是表头，和数据行格式一样，需自行识别
- 内部滚动容器：用 `scrollIntoView` 驱动（非 `scroll(direction=down)`）
- 富文本中的链接：截图中显示为蓝色文字，MD 提取中可能丢失 → 注意保留

### Google Sheets

- 默认视图可能只显示部分列/行，需要滚动或切换 sheet
- 冻结行/列在截图中始终可见，注意不要重复提取
- 单元格内的换行在 HTML 中是 `<br>`，在截图中是视觉换行

### Confluence

- 页面内容通常在 `#content` 区域内
- 表格可能使用 `<th>` 合并的复杂表头
- 通过 Atlassian MCP 读取是最稳的方式

### 普通网页 HTML 表格

- `colspan` / `rowspan` 合并单元格：截图能看到完整布局，但 MD 提取会丢失合并关系
- 嵌套表格：少见但存在，逐层解析

---

## 归一化（统一入口）

无论通过哪一步拿到数据，最终都进入同一个流程：

```
1. 识别列名 → 映射到 taxonomy 字段
   - 列名不一定精确匹配，按语义灵活对应
   - 「美食」列 → 区分餐厅和食物推荐（见 content-taxonomy.md §6）

2. 逐行提取 → 每行对应一个 day

3. 单元格内容拆分 → 一个单元格可能有多条信息
   - 换行符、分号、编号拆分
   - 链接单独提取

4. 归一化为 trip + days[]（字段定义见 trip-draft.template.yaml）
```
