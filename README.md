# AI-IC Cookbook 🍳

> 记录 AI Agent 辅助数字 IC 前端设计与验证时遇到的问题和解决方法

## 背景

本仓库用于记录在实际数字 IC 项目中，使用 Claude Code 等 AI Agent 辅助设计验证时踩过的坑、积累的经验和有效的实践模式。

## 问题速览

| # | 标题 | 阶段 | 状态 |
|:--|:--|:--|:--|
| [001](issues/001-model-switching-regression-failure.md) | 跨模型回归：主力模型调通的仿真切换模型后结果不一致 | 仿真验证 | 🟡待验证 |
| [002](issues/002-eda-simulation-token-burn.md) | EDA 仿真 Token 消耗过大：根因与优化 | 仿真验证 | 🟡待验证 |

## 目录结构

```
AI-IC-Cookbook/
├── README.md
├── issues/              # 问题与解决记录
│   └── template.md      # 记录模板
├── tips/                # 技巧与最佳实践
└── .gitignore
```

## 快速开始

1. 遇到 AI Agent 辅助设计/验证相关问题时，按模板记录在 `issues/` 下
2. 解决后补充根因分析和经验教训
3. 通用技巧沉淀到 `tips/` 目录
