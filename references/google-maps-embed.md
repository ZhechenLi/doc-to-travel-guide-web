# Google 地图：文档链接 → 嵌入

若链接由 Agent 根据起终点/途经点**拼接生成**，规则见 [google-maps-url-builder.md](google-maps-url-builder.md)；嵌入方式与本页一致。

## 用户文档里常见的三种链接

1. **`https://www.google.com/maps/dir/?api=1&origin=...&destination=...&waypoints=...`**  
2. **`https://www.google.com/maps/search/?api=1&query=...`**  
3. **`https://maps.app.goo.gl/...`**（短链）

## 嵌入 URL 构造（无 API Key）

- **search**：转为 `https://maps.google.com/maps?q=<encode(query)>&output=embed&hl=zh-CN`
- **dir**：在 URL 上追加 `output=embed`、`hl=zh-CN`；将 host **`www.google.com` → `maps.google.com`** 再作为 iframe 地址（部分环境更稳定）
- **goo.gl / maps.app.goo.gl**：可直接作 iframe `src`（由 Google 跳转）；仍可能受策略限制

## 限制（必须在页面上说明）

- **完整驾车路线**在第三方 iframe 内常被 Google 限制，可能出现空白或仅概要。  
- **必须**保留与嵌入**完全相同**的「在 Google 地图中打开」按钮（`target="_blank"`）。  
- 若要**保证**内嵌路线：需 **Maps Embed API（Directions）+ API Key**，与用户单独约定，不要把 Key 写进仓库明文。

## HTML 属性约定

- 面板级：`data-gmap-src="文档完整 URL"`（属性里 `&` 写为 `&amp;`）  
- 仅地点关键词兜底：`data-gq="搜索词"` → `?q=...&output=embed`  
- 脚本：`gmapIframeSrcFromAttrs(dataGmapSrc, dataGq)`；`details` 展开时再 `iframe.src = iframe.dataset.src`
