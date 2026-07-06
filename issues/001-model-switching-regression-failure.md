# 001 — 切换大模型回归仿真时输出图像花屏

## 基本信息

| 项目 | 内容 |
|:--|:--|
| **编号** | 001 |
| **日期** | 2026-07-06 |
| **项目** | AGDC |
| **阶段** | 仿真验证 |
| **严重程度** | P1 重要 |
| **AI Agent** | Claude Opus 4.8 → DeepSeek-v4 |
| **状态** | 🟢已解决 |

## 问题描述

### 现象

用 **Claude Opus 4.8** 调试仿真的 RTL 代码跑通后，输出图像正确。切换到 **DeepSeek-v4** 对同一套代码做回归验证时，仿真虽然能跑完（没报 error），但输出的图像直接变成花屏——数据完全乱了。

### 预期行为

不同模型对同一套 RTL + testbench 做回归仿真，结果应该一致（至少仿真可正常跑通、输出数据 bit-exact 匹配）。

### 复现步骤

1. Claude Opus 4.8 完成某模块的仿真调通（含 testbench 修改、脚本调整、图像数据灌入等）
2. 将 Opus 4.8 调出来的代码/脚本原封不动交给 DeepSeek-v4
3. DeepSeek-v4 执行仿真，出现花屏输出

## 根因分析

Opus 4.8 和 DeepSeek-v4 的上下文理解、指令遵循风格差异较大。Opus 调通过程中隐含了大量"只存在于对话历史中"的上下文决策：

- 某次仿真失败后临时改了什么参数
- 某个 warning 为什么可以忽略
- 某个文件路径为什么选这个版本
- 数据喂入的顺序/格式约定

这些**隐性上下文**没有被固化为可执行文档。DeepSeek-v4 面对相同的最终代码时，由于缺乏这些历史决策信息，只能自行推断，导致仿真流程走了不同的分支，输出花屏。

## 解决方法

**核心原则：主力模型调通后，立即让它把流程固化为"精确到命令的可复现操作流程"。**

具体做法：

1. Opus 4.8 跑通仿真后，立即让它总结一份 **可复现流程文档**，要求：
   - 每一步写出**确切的可执行命令**（含参数）
   - 标注关键约束（文件路径、依赖版本、环境变量）
   - 记录曾踩过、现已绕过的坑（为什么这套参数是对的）
2. 该文档与代码一起入库，回归时直接交给其他模型**按流程执行**，而非让它自行推断
3. 理想情况下，这份流程可以做成 script / Makefile，减少 AI 自由发挥的空间

## AI Agent 使用反思

### AI 做得好的地方

- Opus 4.8 在复杂仿真调试中能自主探索并最终跑通
- 能根据要求输出结构化的可复现流程文档

### AI 做得差的地方 / 踩的坑

- 不同模型对同一代码仓的理解路径完全不同，不能假设"代码一样结果就一样"
- 对话历史中积累的决策信息跨模型不可见，是沉默的信息断层
- DeepSeek 在遇到未明确指定的参数时，会给出"合理但不正确"的默认值

### 经验教训

1. **隐性上下文是跨模型协作的杀手**——跑通之后必须立即文档化
2. **"可复现"应该精确到命令级**，不能停留在"大概这样做"的描述
3. 如果一个流程不能被 script 自动化，就至少应该有一份 step-by-step 命令清单
4. 回归验证应该尽量用同一模型，或至少用同一份固化流程文档驱动不同模型

## 可复现流程示例（模板）

```bash
# === 环境 ===
# ModelSim/Questa 版本: 20xx.x
# 工作目录: D:/project/sim/

# Step 1: 编译 RTL
vlog -work work -f flist.txt +define+SIM

# Step 2: 编译 testbench
vlog -work work -sv tb_top.sv

# Step 3: 运行仿真 + 波形
vsim -c -do "run -all; quit" work.tb_top \
  -gINPUT_IMG=../data/input.yuv \
  -gOUTPUT_IMG=../data/output.yuv

# Step 4: 校验输出
python ../scripts/compare_yuv.py ../data/output.yuv ../data/golden.yuv
```

## 相关链接

- 仓库: [AGDC (Advanced GDC)](https://github.com/Amicro-Digital/Advanced_GDC)
- 关联仓库: [ai-ic-cookbook](https://github.com/Amicro-Digital/ai-ic-cookbook)
