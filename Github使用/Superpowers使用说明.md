# Superpowers 使用说明

Superpowers 是一套面向 Codex、Claude Code 等编程 Agent 的开发方法论和技能集合。它不会替换模型，也不会单独运行，而是通过一组可自动触发的技能，约束 AI 在需求分析、方案设计、编码、测试、调试和交付时的行为。

本文对应当前 Codex 官方插件市场中的 Superpowers `5.1.3`。

## 1. 当前安装状态

- 插件：`superpowers@openai-curated`
- 插件版本：`5.1.3`
- 安装状态：已安装、已启用
- 安装方式：Codex 官方插件市场
- 本地缓存目录：`C:\Users\zhou\.codex\plugins\cache\openai-curated\superpowers\11c74d6b`

安装或重新安装：

```powershell
codex plugin add superpowers@openai-curated
```

查看插件状态：

```powershell
codex plugin list --json
```

安装完成后，需要重新打开 Codex，或者新建一个任务，让新增技能进入任务上下文。

## 2. Superpowers 解决什么问题

普通的 AI 编程流程经常是：收到一句需求后立即修改代码，等编译失败或结果不符合预期时再返工。需求越复杂，这种方式越容易出现理解偏差、修改范围失控、测试不足和“看起来完成了但实际上没有验证”等问题。

Superpowers 把流程调整为：

```text
需求讨论 → 设计确认 → 隔离工作区 → 任务规划
→ 测试先行 → 分任务实现 → 代码审查 → 完成前验证
```

它强调四个基本原则：

1. 编码前先把问题和设计说清楚。
2. 使用系统化流程代替临时猜测。
3. 尽量降低复杂度，避免提前实现不需要的功能。
4. 使用测试、构建结果和运行证据证明任务完成。

## 3. 它是如何工作的

Superpowers 安装后会向 Codex 提供一组技能。Codex接到开发任务时，会先检查是否存在适用技能，再按照技能规定的流程行动。

典型过程如下：

1. `brainstorming` 通过提问澄清需求，并比较不同方案。
2. 设计得到确认后，`using-git-worktrees` 创建隔离开发环境。
3. `writing-plans` 把设计拆成包含文件路径和验证步骤的小任务。
4. `subagent-driven-development` 或 `executing-plans` 逐项实施计划。
5. `test-driven-development` 要求先看到测试失败，再写最小实现使其通过。
6. `requesting-code-review` 检查规格符合性和代码质量。
7. `verification-before-completion` 运行测试、构建或其他验证。
8. `finishing-a-development-branch` 处理合并、Pull Request、保留或清理分支。

这些流程不是单纯的提示词建议。技能触发后，Codex会把它们当成当前任务的执行约束。

## 4. 主要技能

### 需求和计划

- `brainstorming`：澄清需求、比较方案并形成设计。
- `writing-plans`：把设计拆成可以逐项执行的计划。
- `executing-plans`：按批次执行计划并设置人工检查点。

### 开发和协作

- `using-git-worktrees`：在独立 Worktree 中开发，避免污染当前工作区。
- `subagent-driven-development`：为不同任务分配独立子代理，并进行两阶段审查。
- `dispatching-parallel-agents`：并行处理互不依赖的任务。
- `requesting-code-review`：主动发起代码审查。
- `receiving-code-review`：根据证据处理审查意见。

### 测试和调试

- `test-driven-development`：执行红—绿—重构循环。
- `systematic-debugging`：先复现和定位根因，再实施修复。
- `verification-before-completion`：没有新鲜验证证据时，不宣称任务已经完成。

### 收尾

- `finishing-a-development-branch`：测试完成后决定合并、创建 PR、保留或清理分支。
- `using-superpowers`：说明如何发现和使用整套技能。
- `writing-skills`：指导编写或修改 Agent 技能。

## 5. 在 Codex 中使用

### 自动触发

大部分情况下不需要输入特殊命令。重新打开 Codex 后，直接描述开发任务即可：

```text
我想给现有项目增加用户登录功能。
先分析需求和现有结构，不要立即修改代码。
```

如果任务适合 Superpowers，Codex会先进入需求澄清和设计阶段，而不是马上写代码。

### 显式调用某个技能

如果希望明确使用某项能力，可以在 Codex 对话中写：

```text
$brainstorming

我想给现有项目增加用户登录功能，请先帮助我澄清需求并比较设计方案。
```

也可以调用：

```text
$writing-plans        根据已经确认的设计生成实施计划
$systematic-debugging 系统化诊断当前故障
$requesting-code-review 审查已经完成的修改
$verification-before-completion 在交付前执行完整验证
```

这些内容发送到 Codex 对话框，不是在 PowerShell 中执行。

## 6. 一个完整的使用示例

假设要为 STM32 红外遥控项目增加“长按连续调节”功能，可以先发送：

```text
$brainstorming

我想为现有 STM32 红外遥控项目增加长按连续调节功能。
请先读取当前按键解析、消息传递和控制任务的调用链，
向我确认长按判定时间、重复间隔、松开行为和边界条件，
暂时不要修改代码。
```

确认设计后，让 Codex 生成计划：

```text
$writing-plans

根据刚才确认的设计生成实施计划。
计划必须写明修改文件、调用链影响、测试方法、编译验证和硬件验证步骤。
不要修改 CubeMX 自动生成区以外的配置，涉及引脚或 .ioc 修改时先列出人工操作步骤。
```

开始实施前，可以要求：

```text
按照计划执行。纯逻辑部分使用测试驱动开发；
硬件相关部分使用编译、串口日志和实际按键测试验证，
不要用无法运行的伪测试代替硬件证据。
```

完成后执行审查和验证：

```text
$requesting-code-review

对照设计和实施计划审查本次修改，重点检查状态机边界、
重复触发、任务间通信和中断上下文安全性。
```

```text
$verification-before-completion

运行能够执行的测试和完整构建，列出实际执行的命令、
结果以及仍需人工完成的烧录和硬件验证项目。
```

## 7. 与 Spec Kit 配合使用

Spec Kit 主要维护项目规格、技术方案和任务文档；Superpowers 主要约束 Codex 如何完成分析、编码、测试和审查。

一种比较清晰的组合方式是：

```text
Spec Kit：constitution → specify → clarify → plan → tasks
Superpowers：执行计划 → TDD → 调试 → 审查 → 完成验证
```

已经生成 Spec Kit 文档时，可以告诉 Codex：

```text
项目已经通过 Spec Kit 生成规格、技术方案和任务清单。
请先检查这些材料是否足够明确，然后以它们为实施依据，
使用 Superpowers 的测试驱动、系统化调试、代码审查和完成前验证流程。
```

两套工具都可能进行需求澄清和任务规划。遇到重复时，应明确以已经批准的 Spec Kit 文档为需求基准，让 Superpowers 重点负责执行质量。

## 8. 在嵌入式项目中使用时的边界

Superpowers 对测试驱动开发要求较严格，但嵌入式项目并不是所有行为都能在电脑上自动测试。

- 状态机、协议解析、滤波和控制算法：适合主机端单元测试。
- HAL 调用和任务通信：可以通过 Mock、Stub 或测试替身验证。
- 中断时序、引脚电平、PWM 波形和真实传感器行为：需要编译、烧录、串口日志、示波器或逻辑分析仪验证。
- CubeMX、引脚和 `.ioc` 配置：应先列出准确的人工操作步骤，确认后再继续修改相关代码。

不要为了满足“TDD”形式而编写无法运行的伪测试。测试的目的仍然是提供可信证据。

## 9. 常见误区

1. 认为安装后必须记住所有技能名称。多数技能会自动触发，显式调用只是为了加强意图。
2. 在需求尚未确认时催促 Codex 立即编码，这会绕回原来的低质量工作方式。
3. 在有未提交修改的仓库中直接创建 Worktree。开始前应检查 `git status`。
4. 把“命令执行成功”当成“功能验证完成”。构建成功、自动测试和真实硬件验证是不同层次的证据。
5. 同时让 Spec Kit 和 Superpowers 重复生成多套互相矛盾的计划。

## 10. 更新、隐私与卸载

查看当前插件：

```powershell
codex plugin list --json
```

卸载：

```powershell
codex plugin remove superpowers@openai-curated
```

Superpowers 的可选视觉组件会从 Prime Radiant 网站加载图标，并携带插件版本号。仓库说明该请求不包含项目内容、提示词或点击信息。如果不需要这一功能，可以设置：

```powershell
$env:SUPERPOWERS_DISABLE_TELEMETRY = 'true'
```

该命令只影响当前 PowerShell 会话。

## 相关笔记

- [[Spec Kit使用说明]]

## 参考资料

- [Superpowers GitHub 仓库](https://github.com/obra/superpowers)
- [Superpowers Releases](https://github.com/obra/superpowers/releases)
