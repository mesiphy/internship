# Codex 调用的终端环境

目标：理解 Codex 或其他 Agent 调用终端时，为什么经常和用户自己打开的终端环境不一致，并学会通过提示词要求 Agent 在正确的项目目录、虚拟环境或运行环境中执行命令。

## 一、对 Agent 调用终端环境的解释

### 1. 用户打开的终端环境是什么

用户自己打开的终端，例如 Terminal、iTerm2、VS Code 终端，通常是一个正在运行的 shell 会话。

这个会话里可能包含：

- 当前工作目录，例如 `pwd` 显示的路径。
- 当前 shell，例如 `zsh`、`bash`。
- 已经临时设置的环境变量，例如 `export API_KEY=xxx`。
- 已经激活的虚拟环境，例如 `source .venv/bin/activate`、`conda activate xxx`。
- 已经切换的 Node 版本，例如 `nvm use 20`。
- 当前 shell 进程继承到的系统环境变量。

这些状态通常只属于当前终端会话。也就是说，你在一个终端里执行过的 `export`、`source`、`conda activate`，不一定会影响另一个新开的终端，更不一定会影响 Agent 调用命令时使用的终端环境。

### 2. Codex 或 Agent 调用的终端环境是什么

严格来说，大模型本身没有终端。大模型只是根据上下文决定“应该执行什么命令”，真正执行命令的是宿主程序，例如 Codex、Cursor、Claude Code、Copilot Agent 等。

Agent 调用终端时，宿主程序通常会为它启动一个独立的命令执行环境。这个环境一般包括：

- 一个工作目录，即 `cwd`。
- 一个 shell，例如 `zsh`、`bash`、`sh`。
- 一组环境变量，即 `env`。
- 一个可见的文件系统范围。
- 一组权限限制。
- 可能存在的沙盒、容器、网络限制或读写限制。

因此，Agent 调用的终端环境不等于用户正在操作的那个终端窗口。它们可能共享同一个项目文件夹，但通常不共享“当前 shell 会话状态”。

### 3. 为什么 Agent 经常提示没有文档、环境变量或依赖

常见原因包括：

1. 工作目录不同  
   用户以为 Agent 在项目根目录执行命令，但 Agent 实际可能在另一个目录。此时它执行 `ls`、`pytest`、`npm run dev` 都可能找不到文件。

2. shell 会话不同  
   用户在自己的终端里执行过 `source .venv/bin/activate` 或 `export API_KEY=xxx`，但 Agent 新开的 shell 不一定继承这些状态。

3. 环境变量不是全局永久状态  
   `export FOO=bar` 只影响当前 shell 以及它启动的子进程。另一个终端或 Agent 的命令执行进程不一定能看到这个变量。

4. 虚拟环境没有被激活  
   用户本地终端已经进入 `.venv`、`conda` 或某个 Node 环境，但 Agent 使用的可能是系统默认 Python、系统默认 Node。

5. Agent 运行在沙盒或容器中  
   有些 Agent 不是直接运行在用户完整电脑环境中，而是在受限容器、远程环境或沙盒中运行。它可能只能访问项目目录，看不到用户桌面、下载目录、全局配置、浏览器缓存、SSH key 等。

6. 权限和网络受限  
   Agent 可能被限制只能读写 workspace，或者不能联网安装依赖。这时它会遇到“文件不存在”“权限不足”“无法访问环境变量”“依赖下载失败”等问题。

## 二、通过提示词告诉 Agent 在对应终端环境执行命令

### 1. 基本原则

给 Agent 的提示词应该明确说明：

- 在哪个项目目录执行命令。
- 使用哪个 Python、Node 或其他运行时。
- 是否需要激活虚拟环境。
- 执行主要任务前先运行哪些验证命令。
- 不要依赖系统默认环境。

提示词越明确，Agent 越不容易使用错误目录或错误解释器。

### 2. 推荐写法：直接指定项目目录和解释器

比起只说“激活虚拟环境”，更稳妥的方式是让 Agent 直接使用项目里的解释器。

示例：

```text
请始终在当前项目根目录执行命令：
/Users/mesiphy/需要同步的文件/internship

不要使用系统 Python。若项目中存在 `.venv/bin/python`，请优先使用：
./.venv/bin/python

安装依赖时使用：
./.venv/bin/python -m pip install ...

运行测试时使用：
./.venv/bin/python -m pytest

执行主要任务前，请先运行并汇报：
pwd
./.venv/bin/python --version
./.venv/bin/python -c "import sys; print(sys.executable)"
```

这种写法的好处是：即使 Agent 每次执行命令都会打开一个新的 shell，也不依赖 `source .venv/bin/activate` 的会话持续状态。

### 3. 如果希望 Agent 激活虚拟环境

可以这样提示：

```text
请在当前项目目录中执行命令。项目目录是：
/Users/mesiphy/需要同步的文件/internship

运行任何 Python 命令前，请先激活当前项目下的虚拟环境：
source .venv/bin/activate

激活后请先验证：
pwd
which python
python --version
python -c "import sys; print(sys.executable)"

确认 `which python` 指向当前项目的 `.venv/bin/python` 后，再继续执行后续命令。
```

需要注意：有些 Agent 的每次命令执行都是新的 shell。在这种情况下，`source .venv/bin/activate` 可能只在当前这一次命令中有效，下一次命令又会失效。因此更推荐直接使用 `./.venv/bin/python`。

### 4. Node 项目的提示词示例

```text
请始终在当前项目根目录执行命令：
/Users/mesiphy/需要同步的文件/internship

执行前先确认：
pwd
node -v
npm -v

如果项目中有 `package.json`，请优先使用其中定义的脚本，例如：
npm run dev
npm test
npm run build

不要假设全局安装的包一定存在。需要运行项目工具时，优先使用：
npm run ...
```

### 5. 一句话提示词模板

```text
请始终在 `/Users/mesiphy/需要同步的文件/internship` 下执行终端命令，并使用当前目录的 `.venv/bin/python`，不要使用系统 Python。执行前请用 `pwd` 和 `./.venv/bin/python -c "import sys; print(sys.executable)"` 验证环境。
```

## 三、建议 Agent 先执行的环境检查命令

当 Agent 说“找不到文件”“没有依赖”“环境变量不存在”时，可以要求它先执行：

```bash
pwd
ls
env | sort
which python
python --version
which node
node --version
```

Python 项目可以补充：

```bash
which pip
pip --version
python -c "import sys; print(sys.executable)"
python -m pip list
```

如果明确使用 `.venv`，则更推荐：

```bash
./.venv/bin/python --version
./.venv/bin/python -c "import sys; print(sys.executable)"
./.venv/bin/python -m pip list
```

Node 项目可以补充：

```bash
npm -v
npm config get prefix
npm list --depth=0
```

## 四、实用理解

核心区别可以总结为：

> 用户终端环境是用户当前 shell 会话的状态；Agent 终端环境是宿主程序为 Agent 启动的另一个命令执行进程。两者可能共享项目文件，但通常不共享临时环境变量、已激活的虚拟环境和当前 shell 状态。

因此，与其假设 Agent 知道“我现在的环境”，不如在提示词里明确告诉它：

- 项目根目录在哪里。
- 使用哪个解释器或运行时。
- 执行前如何验证环境。
- 不要使用系统默认环境。
- 如果需要环境变量，应说明变量从哪里读取，例如 `.env`、配置文件或 shell 环境。

## 五、补充：更稳的操作习惯

1. 尽量把项目依赖写进项目文件  
   Python 项目使用 `requirements.txt`、`pyproject.toml`、`uv.lock` 等；Node 项目使用 `package.json`、`package-lock.json` 等。

2. 尽量不要只依赖临时 `export`  
   如果某些变量必须存在，可以使用 `.env` 文件或明确告诉 Agent 变量名和加载方式。注意不要把真实密钥直接暴露给不可信环境。

3. 让 Agent 先验证再执行  
   对于安装依赖、运行测试、启动服务等操作，先让 Agent 执行 `pwd`、`which python`、`python --version` 这类检查命令，可以减少很多误判。

4. 使用项目内命令，而不是全局命令  
   Python 优先使用 `./.venv/bin/python -m ...`；Node 优先使用 `npm run ...`；这样可以降低环境漂移带来的问题。

