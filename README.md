# doc-to-travel-guide-web

Cursor **Agent Skill**：把结构化行程（YAML / Notion 表 / Markdown 表 / CSV / 粘贴文本）转成**单文件攻略 HTML**，并约定输出 **`*.sources.md` 信息源清单**、内容分类、Google 地图链接拼接与嵌入说明。

## 安装（Cursor）

本仓库根目录即 skill 根（含 `SKILL.md` 与 `references/`）。

**方式 A — 用户级（所有项目可用）**

```bash
git clone https://github.com/ZhechenLi/doc-to-travel-guide-web.git ~/.cursor/skills/doc-to-travel-guide-web
```

**方式 B — 仅当前项目**

```bash
mkdir -p .cursor/skills
git clone https://github.com/ZhechenLi/doc-to-travel-guide-web.git .cursor/skills/doc-to-travel-guide-web
```

重启 Cursor 或重新加载窗口后，在对话里提及「按 doc-to-travel-guide-web」「行程转攻略网页」等，应能匹配 skill 描述。

**方式 C — Submodule（大仓内固定版本）**

```bash
git submodule add https://github.com/ZhechenLi/doc-to-travel-guide-web.git .cursor/skills/doc-to-travel-guide-web
```

## 分发放哪里比较合适

| 场景 | 建议 |
|------|------|
| **公开分享** | [GitHub](https://github.com) / GitLab 公有仓库；README 写清安装路径；可打 `v1.0.0` tag 方便 submodule。 |
| **公司内** | 内部 Git（如自托管 GitLab、Bitbucket）；IT 若限制公网，用内网 clone URL。 |
| **不建远程** | `git bundle create doc-to-travel-guide-web.bundle HEAD` 发文件；对方 `git clone doc-to-travel-guide-web.bundle`。 |
| **极简** | 打包 zip 发布 Release；安装方解压到 `~/.cursor/skills/doc-to-travel-guide-web/`（需保证目录下有 `SKILL.md`）。 |

不推荐只靠 Gist：多文件 + `references/` 维护成本高。

## 仓库结构

```text
SKILL.md
README.md
LICENSE
references/
  content-taxonomy.md
  google-maps-embed.md
  google-maps-url-builder.md
  html-patterns.md
  input-constraints.md
  source-manifest.md
  trip-draft.template.yaml
  verification-checklist.md
```

## 许可

MIT（见 `LICENSE`）。
