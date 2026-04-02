# 攻略单页 HTML — 结构模式

## 页面骨架

- 单文件：`DOCTYPE`、`<meta viewport>`、`<title>` 含路线关键词
- **Hero**：行程名、日期区间、城市 chips、可选背景图（渐变+照片）
- **总览区**：一条路线叙事 + 大地图（Leaflet 或静态图）+ 图例
- **按日时间线**：`.timeline` + `.day-card`；左侧色点区分阶段（城市主题色 CSS 变量）
- **阶段分隔**：大区块切换时用 `.phase-divider`（可加背景图+遮罩）
- **页脚**：一句引用 + 行程摘要 + 配图版权/免责声明

## 单日卡片内容顺序（建议）

1. 标题区：日期、Day 序号、标题、交通一行（若有）
2. **当日地图**（可选）：`.day-map-wrap` → 跨城用 `.day-map-panels--split` 两格 Leaflet
3. **时间表**：`<ul class="schedule">`，`<li>` 内 `.time` + `.activity`（`.sub` 小字）
4. **Google 导航按钮**：`<a class="map-link" href="文档原链">` — 与嵌入同源
5. **路线嵌入**（可选）：`<details class="gmap-details gmap-route-from-doc" data-gmap-src="...">` + `.gmap-frame-wrap`；由脚本写入 `iframe[data-src]`，首次展开再设 `src`
6. **预订/餐厅**：`.rsv-section` / `.rsv-card`
7. **景点速览**：`<details class="spotlight-fold">` 折叠科普

## CSS 变量（示例）

```text
--cream, --navy, --terracotta, --shadow
各城市主题色：--brussels, --milan, --venice …
```

## 动画与性能

- 日程卡片 `IntersectionObserver` 进视口再加 `.visible`（可选）
- Google iframe：**不要**首屏一次性加载；用 `<details>` + `toggle` 监听设 `src`
- 图片：`loading="lazy"`；外链先校验 HTTP 200

## 打印

- `@media print`：隐藏浮动导航、固定 `day-card` 不透明
