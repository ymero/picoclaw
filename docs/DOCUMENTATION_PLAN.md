# 文档目录规划方案

## 当前文档结构分析

### 已有目录结构

```
docs/
├── channels/          # 渠道配置文档
│   ├── dingtalk/
│   ├── discord/
│   ├── feishu/
│   ├── line/
│   ├── maixcam/
│   ├── onebot/
│   ├── qq/
│   ├── slack/
│   ├── telegram/
│   └── wecom/
├── design/           # 设计文档
│   ├── issue-783-investigation-and-fix-plan.zh.md
│   ├── provider-refactoring-tests.md
│   └── provider-refactoring.md
├── migration/         # 迁移文档
│   └── model-list-migration.md
├── ANTIGRAVITY_AUTH.md
├── ANTIGRAVITY_USAGE.md
├── tools_configuration.md
├── troubleshooting.md
└── wecom-app-configuration.md
```

### 新增文档列表

| 文档 | 当前路径 | 建议分类 |
|------|----------|----------|
| architecture.md | docs/ | docs/architecture/overview.md |
| architecture-diagrams.mmd | docs/ | docs/architecture/diagrams.mmd |
| architecture-design.md | docs/ | docs/architecture/design.md |
| development-history.md | docs/ | docs/history/development.md |
| product-requirements.md | docs/ | docs/product/requirements.md |
| insights-report.md | docs/ | docs/analysis/insights.md |
| analysis-sop.md | docs/ | docs/analysis/sop.md |

---

## 建议的文档目录结构

```
docs/
├── # 架构文档
├── architecture/
│   ├── overview.md          # 架构概览 (原 architecture.md)
│   ├── design.md           # 架构设计 (原 architecture-design.md)
│   └── diagrams.mmd        # 架构图表 (原 architecture-diagrams.mmd)
│
├── # 产品文档
├── product/
│   ├── requirements.md     # 产品需求文档 (原 product-requirements.md)
│   ├── roadmap.md          # 产品路线图 (待创建)
│   └── changelog.md        # 版本变更日志 (待创建)
│
├── # 分析文档
├── analysis/
│   ├── insights.md         # 洞察报告 (原 insights-report.md)
│   ├── sop.md              # 分析标准操作流程 (原 analysis-sop.md)
│   └── methodology.md      # 分析方法论 (待创建)
│
├── # 历史文档
├── history/
│   ├── development.md      # 开发历史 (原 development-history.md)
│   └── version.md          # 版本历史 (待创建)
│
├── # 渠道文档 (已有)
├── channels/
│   ├── dingtalk/
│   ├── discord/
│   ├── feishu/
│   ├── line/
│   ├── maixcam/
│   ├── onebot/
│   ├── qq/
│   ├── slack/
│   ├── telegram/
│   └── wecom/
│
├── # 设计文档 (已有)
├── design/
│   ├── issue-783-investigation-and-fix-plan.zh.md
│   ├── provider-refactoring-tests.md
│   └── provider-refactoring.md
│
├── # 迁移文档 (已有)
├── migration/
│   └── model-list-migration.md
│
├── # 用户指南
├── guides/
│   ├── getting-started.md  # 快速入门
│   ├── configuration.md    # 配置指南
│   └── troubleshooting.md   # 故障排除 (现有)
│
├── # 参考文档
├── reference/
│   ├── tools.md            # 工具配置 (现有 tools_configuration.md)
│   ├── api.md              # API 参考 (待创建)
│   └── cli.md              # CLI 参考 (待创建)
│
├── # 供应商文档
├── providers/
│   ├── ANTIGRAVITY_AUTH.md
│   └── ANTIGRAVITY_USAGE.md
│
├── # 第三方配置
├── integrations/
│   ├── wecom-app-configuration.md
│   └── (其他第三方集成)
│
└── README.md               # 文档索引
```

---

## 文档分类说明

| 分类 | 包含内容 | 目标读者 |
|------|----------|----------|
| **architecture/** | 系统架构、设计原则、模块关系 | 开发者、架构师 |
| **product/** | 需求文档、路线图、版本说明 | 产品经理、项目经理 |
| **analysis/** | 分析报告、方法论、SOP | 分析师、审查者 |
| **history/** | 开发历史、版本变更 | 所有贡献者 |
| **guides/** | 快速入门、配置教程 | 最终用户 |
| **reference/** | API、CLI、工具说明 | 开发者 |
| **channels/** | 各渠道配置指南 | 运维、开发者 |
| **design/** | 内部设计文档 | 核心开发者 |
| **migration/** | 迁移指南 | 运维、用户 |

---

## 实施计划

### 阶段一：创建目录结构

```bash
mkdir -p docs/architecture
mkdir -p docs/product
mkdir -p docs/analysis
mkdir -p docs/history
mkdir -p docs/guides
mkdir -p docs/reference
mkdir -p docs/integrations
```

### 阶段二：移动文档

```bash
# 架构文档
mv docs/architecture.md docs/architecture/overview.md
mv docs/architecture-design.md docs/architecture/design.md
mv docs/architecture-diagrams.mmd docs/architecture/diagrams.mmd

# 产品文档
mv docs/product-requirements.md docs/product/requirements.md

# 分析文档
mv docs/insights-report.md docs/analysis/insights.md
mv docs/analysis-sop.md docs/analysis/sop.md

# 历史文档
mv docs/development-history.md docs/history/development.md

# 迁移文档到正确位置
mkdir -p docs/integrations
mv docs/wecom-app-configuration.md docs/integrations/

# 工具配置移动到参考文档
mv docs/tools_configuration.md docs/reference/tools.md
```

### 阶段三：创建文档索引

创建 `docs/README.md` 作为文档入口

---

## 待补充文档

| 文档 | 分类 | 优先级 |
|------|------|--------|
| README.md | 文档索引 | 高 |
| getting-started.md | guides | 高 |
| roadmap.md | product | 中 |
| changelog.md | history | 中 |
| api.md | reference | 中 |
| cli.md | reference | 中 |
| methodology.md | analysis | 低 |

---

*规划版本: v1.0*
*创建时间: 2026-02-28*
