# 攻略页交付前自检

- [ ] 源数据已按 [content-taxonomy.md](content-taxonomy.md) 分类：住宿 / 餐饮 / 当日路线 / 大交通 / 时间表未混成一团
- [ ] 每晚住宿的「哪一夜」（`overnight_date` 或等价约定）与源 YAML/表格一致
- [ ] 所有日期与城市顺序与源文档一致，无臆造预订
- [ ] 每个 `maps/dir` / `goo.gl` 按钮与对应 `data-gmap-src`（若有）字符串一致
- [ ] 若路线链接为 Agent 根据 `route_spec` 生成：页面已提示用户**在 Google 地图中核对**；用户提供的官方短链优先于拼接 URL
- [ ] 已撰写 **信息源清单** `*.sources.md`（与 [source-manifest.md](source-manifest.md) 一致）：真源路径、外部 URL、地图/图片来源、推断项已列出
- [ ] 跨城日 Leaflet 已分屏或单图 `maxZoom` 合理，不出现「整条欧洲缩成一条线」
- [ ] 所有外链图片已 `curl` 校验 **HTTP 200**（或改成本地路径）
- [ ] Google iframe 均为懒加载；控制台无未捕获错误（本地 HTTP 打开测一遍）
- [ ] `alt` 与说明不声称「照片=精确打卡点」除非用户提供了对应图
- [ ] 打印预览：关键块不被裁切过多（可选）
