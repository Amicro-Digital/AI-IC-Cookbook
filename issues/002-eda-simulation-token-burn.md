# 002 — EDA 仿真消耗大量 Token：根因与优化

## 基本信息

| 日期 | 阶段 | 状态 |
|:--|:--|:--|
| 2026-07-06 | 仿真验证 / 综合 / 全流程 | 🟡待观察 |

## 问题描述

### 现象

用 AI Agent 跑 VCS / DC 等 EDA 工具时，一个看似简单的编译-仿真-报错-修复循环，动辄消耗几万到几十万 token。尤其是仿真日志较大的设计，token 消耗速度远超预期。

### 预期行为

仿真调试的 token 消耗应该与编辑 RTL 代码处于同一量级，而非指数膨胀。

## 根因分析

Token 被大量消耗的四个核心来源：

**1. EDA 工具日志天生冗长**

VCS 编译一个中等模块会产生几千行 log（`vcs -f flist`），仿真运行又会输出 `$display`/`$monitor`、assertion 报错、时序违例等。这些全部涌入 AI 上下文时，真正的有用信息（error/warning）占比不足 5%。

**2. 调试循环中的重复读取**

```
编译失败 → AI 读取全部 log → 修复 RTL → 再次编译 → AI 再次读取全部 log → ...
```

每轮迭代都把已经读过的编译信息重新加载一遍，且 log 中无关部分（INFO、进度条、时间戳）被反复 tokenize。

**3. 源文件重复加载**

AI 需要同时看到 RTL、testbench、脚本、flist 才能定位问题，这些文件加上 EDA log 同时进入上下文，单轮对话就突破 50k token。

**4. 波形数据误入上下文**

波形文本 dump（`$dumpvars` 输出、VCD 文本段）如果被 AI 读取，一行 `#10 a=0 b=1` 的密度极低，但 token 开销等同于同长度的代码。

## 解决方法

### 策略一：不让 AI 读原始 Log

```bash
# AI 写好了编译命令后，用户执行时用 grep 只提取关键信息
vcs -f flist 2>&1 | grep -E "Error|Warning|Lint"

# 仿真只提取失败信息
./simv 2>&1 | grep -E "FAIL|ERROR|Mismatch|assert"
```

规则：**只把 grep 后的结果喂给 AI，原始 log 最多保留在后端可按需查阅。**

### 策略二：脚本驱动，AI 不参与运行时

> 这是 issue #001 中"可复现流程"思路的直接延伸。

```
AI 的职责：编写 Makefile / run脚本，不是逐行盯着仿真输出
人的职责： 执行脚本，只将 exit code + 错误摘要回报 AI
```

```makefile
# AI 一次性产出的 Makefile
sim:
	vcs -f flist -o simv
	./simv +vcs+flush+log | tee sim.log
	@grep -E "FAIL|ERROR" sim.log && exit 1 || echo "PASS"
```

执行后只需告诉 AI "PASS" 或贴出 FAIL 行，不需要传整个 `sim.log`。

### 策略三：减少仿真本身的日志量

- 用 `+define+NODISPLAY` 关掉调试期 `$display`
- `$monitor` 只在单元验证阶段打开，系统级关闭
- VCS 编译加 `-quiet` 或 `+vcs+quiet`
- UVM 控制 verbosity：`+UVM_VERBOSITY=LOW`

### 策略四：分阶段、分文件喂入

| 阶段 | AI 需要的文件 | 不需要的 |
|:--|:--|:--|
| 编译报错 | 报错文件 + error log 片段 | 全部 flist、完整 log |
| 仿真失败 | TB 顶层 + 关键波形信号值 | VCD 文本 dump |
| 综合违例 | 关键路径报告 + SDC | 完整面积/timing 报告 |

不要让 AI "全仓读一遍再干活"——按问题精确喂入。

### 策略五：固化上下文，避免重复加载

- 模块接口、参数、时钟方案写入 `CLAUDE.md` 或项目 spec，让 AI 从摘要理解而非反复读源码
- 编译环境变量（`$VCS_HOME`、flist 路径）一次性写入项目级配置

## AI Agent 使用反思

### 踩的坑

- 早期直接把完整仿真 log 贴给 AI，一个简单 typo 修复就烧掉 30k token
- 让 AI "帮我跑仿真" 等同于让它读完所有无关的系统路径、license 信息、INFO 进度条
- 波形 dump 文件被不小心喂入过一次，token 直接爆炸

### 经验教训

1. **AI 应该写脚本，不是盯日志**——它是架构师，不是操作工
2. **grep 是你最好的 token 守门员**——5% 的关键行比 100% 的 log 价值高 10 倍
3. **每个 EDA 工具都默认冗长，默认关闭 verbose**——编译加 `-quiet`，仿真加 `+define+NODISPLAY`
4. **跨迭代共享的上下文应该固化**——不要每轮都重新解释一遍"这个模块的时钟结构是什么"
5. **token 优化本质是对 AI 的最大尊重**——喂得越精准，回复越可靠；噪音越多，幻觉越多

## 相关链接

- [#001 — 跨模型回归](001-model-switching-regression-failure.md) → 可复现流程策略
- 关联仓库: [AI-IC-Cookbook](https://github.com/Amicro-Digital/AI-IC-Cookbook)
