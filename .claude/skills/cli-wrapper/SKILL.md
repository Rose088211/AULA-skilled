---
name: cli-wrapper
description: 把一个 Python/Node 函数或脚本封装成可安装、可发布的命令行工具——参数解析、--help、退出码、错误处理、打包。Use whenever the user wants to turn a function into a CLI, 把函数变成命令行工具, 写个命令行工具, cli tool, argparse, typer, click, commander, npm bin, executable script, or "make this runnable from the terminal". Also triggers when a script exists but needs proper argument parsing and packaging.
---

# 函数 → CLI 封装器

把一个「函数/脚本」变成能在终端里跑、能安装、能发布、能被别人用的命令行工具。核心不是写代码，而是把接口（参数、错误、帮助）做对。

## 何时使用

- 用户有个函数/脚本，想从命令行调用。
- 用户想「做个工具」「写个 CLI」「封装成命令」。
- 已有脚本只有裸 `sys.argv` 或硬编码输入，需要正规化。

## 先问三个问题

1. **输入从哪来**：命令行参数、配置文件、标准输入、还是环境变量？（多半是参数，但确认一下）
2. **输出到哪去**：打印到 stdout、写文件、还是生成报告？
3. **谁会用它**：自己用（够用就行）还是要发布（需要完善错误处理和打包）？问完再动手，避免过度设计。

## 选型

| 场景 | 推荐 |
|---|---|
| Python，新写 | `typer`（声明式、自带 --help，基于 click） |
| Python，不想加依赖 | 标准库 `argparse` |
| Python，已有函数 | 包一层，函数签名直接映射参数 |
| Node，新写 | `commander` 或 `meow`（轻量） |
| Node，已发布的 npm 包 | package.json `bin` 字段 |
| 跨语言/已有可执行 | 只需统一入口 + 退出码，不必重写 |

## 实现清单（无论语言）

1. **入口函数**：`main(argv)` 收参数、返回退出码；真正的逻辑函数独立、可测试。
2. **参数定义**：
   - 位置参数：必需输入
   - 选项：`--output`、`--verbose`
   - 标志：`--force`、`--dry-run`（布尔开关）
   - 类型校验 + 必填校验 + 默认值，错误信息要告诉用户「缺了什么、怎么写」。
3. **`--help`**：必须清晰列出所有参数和用途。typer/commander 自带，argparse 手写也别漏。
4. **退出码**：`0` 成功；`1` 运行时错误；`2` 参数错误（argparse 默认）；自定义错误码用于脚本化调用。
5. **错误处理**：捕获业务异常 → 打印人类可读的错误到 stderr → 返回非 0 退出码。**不要打印堆栈**给普通用户（`--verbose`/`--debug` 时才给）。
6. **stdout 干净**：正常输出走 stdout，日志/错误走 stderr——这样输出可以管道给别的命令。

## 打包

**Python（pyproject.toml）**：
```toml
[project]
name = "my-tool"
version = "0.1.0"
requires-python = ">=3.9"
dependencies = ["typer>=0.12"]

[project.scripts]
my-tool = "my_tool.cli:main"
```
发布：`pip install -e .` 即可得到 `my-tool` 命令。

**Node（package.json）**：
```json
{
  "name": "my-tool",
  "bin": { "my-tool": "./bin/my-tool.js" }
}
```
`bin/my-tool.js` 首行 `#!/usr/bin/env node` 且文件可执行。发布：`npm install -g .`。

## 验证

封装完必须跑一遍：
- `my-tool --help` 正常
- 正确参数跑通，输出符合预期
- 缺参数 → 友好报错 + 非 0 退出码
- 非法输入（类型错/文件不存在）→ 友好报错，不崩堆栈
- `echo $?` 检查退出码

## 规则

- 逻辑与 IO 分离：核心函数能单独 import 测试，CLI 只是薄薄一层。
- 别把所有东西都塞进 `main`，参数多了用子命令（`my-tool build`、`my-tool push`）。
- 保持向后兼容：新参数加默认值，别删旧参数。
- 交互式输入要谨慎：CLI 要能被脚本非交互调用（`--force`、stdin 读取）。

## 示例

**Python + typer 骨架**：
```python
import typer

app = typer.Typer()

@app.command()
def convert(input: str, output: str = "out.txt", force: bool = False):
    """把 input 处理成 output。"""
    if not force and Path(output).exists():
        typer.echo("文件已存在，加 --force 覆盖", err=True)
        raise typer.Exit(1)
    data = Path(input).read_text()
    # ... 真正的业务逻辑 ...
    Path(output).write_text(data.upper())

if __name__ == "__main__":
    app()
```
