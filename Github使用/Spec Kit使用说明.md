# Spec Kit 简短使用说明

## 1. 当前安装状态

- Spec Kit CLI：`0.16.5`
- 安装方式：`uv tool`
- 可执行文件：`C:\Users\zhou\.local\bin\specify.exe`
- 当前版本检查结果：已是最新版本

安装后需要重新打开 PowerShell，新的终端才能直接识别 `specify` 命令。

验证命令：

```powershell
specify version
specify self check
```

## 2. 为新项目启用 Codex 集成

在 PowerShell 中执行：

```powershell
specify init my-project --integration codex --script ps
Set-Location .\my-project
```

初始化后，Spec Kit 会在项目中生成 `.specify` 配置和 `.agents\skills` 下的 Codex 技能。

## 3. 为已有项目启用

先进入项目并检查 Git 状态：

```powershell
Set-Location '项目路径'
git status
specify init --here --integration codex --script ps
```

如果非空目录被拒绝，应先提交或备份已有文件，再根据提示考虑加入 `--force`。不要未经检查直接使用 `--force`。

## 4. 在 Codex 对话中使用

以下命令发送到 Codex 对话框，不是在 PowerShell 中运行：

```text
$speckit-constitution  制定项目原则和质量要求
$speckit-specify       描述要实现什么、为什么实现
$speckit-clarify       澄清模糊或缺失的需求
$speckit-plan          确定技术栈和实现方案
$speckit-tasks         把方案拆成可执行任务
$speckit-analyze       检查规格、方案和任务是否一致
$speckit-implement     按任务实施代码
$speckit-converge      对照规格检查遗漏并追加任务
```

推荐顺序：

```text
constitution → specify → clarify → plan
→ tasks → analyze → implement → converge
```

## 5. 最小示例

先发送：

```text
$speckit-specify

开发一个 GitHub 仓库分析工具。用户输入关键词后，程序搜索相关仓库，
显示 Star、Fork、最近更新时间和仓库链接，并导出 Markdown 报告。
```

需求澄清完成后再发送：

```text
$speckit-plan

使用 Python 3.13，通过 GitHub API 获取数据，输出 UTF-8 Markdown，
并为筛选和排序逻辑编写自动化测试。
```

然后依次执行 `$speckit-tasks`、`$speckit-analyze` 和 `$speckit-implement`。

## 6. 更新与卸载

```powershell
specify self upgrade
uv tool uninstall specify-cli
```

官方仓库：<https://github.com/github/spec-kit>
