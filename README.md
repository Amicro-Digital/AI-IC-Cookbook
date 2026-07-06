# AI-IC Cookbook 🍳

> 记录 AI Agent 辅助数字 IC 前端设计与验证时遇到的问题和解决方法

## 背景

本仓库用于记录在实际数字 IC 项目中，使用 Claude Code 等 AI Agent 辅助完成以下工作时踩过的坑、积累的经验和有效的实践模式：

- 设计文档撰写（Markdown/DrawIO）
- Verilog RTL 编码与 Review
- 寄存器配置分析
- 跨时钟域 CDC 分析
- 仿真验证环境搭建（ModelSim/UVM）
- 综合与时序收敛
- AI Agent 的提示词技巧

## 目录结构

```
ai-ic-cookbook/
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

## 相关资源

- Claude Code 项目配置：[claude-config](https://github.com/Amicro-Digital/claude-config)
- Claude Code 技能仓库：[FISHEYE 项目技能集](https://github.com/Amicro-Digital/FISHEYE0510)
