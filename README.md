<p align="center">
  <img src="assets/screenshot.png" alt="free-code" width="720" />
</p>

<h1 align="center">free-code</h1>

<p align="center">
  Claude Code 的自由构建版本。<br>
  移除了遥测，上层提示护栏被尽量剥离，开放了更多实验特性。<br>
  现在额外支持 OpenAI 的标准 API Key + Base URL 接入。
</p>

<p align="center">
  <a href="./README.en.md">English README</a>
</p>

---

## 项目简介

`free-code` 是基于公开流出的 Claude Code 源快照整理出的一个可构建 CLI fork。  
项目保留了 Claude Code 的终端代理式交互体验，同时补充和解锁了多项能力：

- 移除或钝化大部分遥测/分析埋点
- 尽量去除上层安全提示注入与额外护栏
- 打开更多可编译的实验特性
- 支持多种模型提供方
- 在本仓库版本中，新增对 OpenAI 标准接口的支持：
  `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`gpt-5.4`

---

## 手动安装

```bash
cd /root/claude
git clone https://github.com/julioaaericksonaa/free-code-openai.git free-code
cd /root/claude/free-code

apt update
apt install -y curl unzip build-essential

curl -fsSL https://bun.sh/install | bash
source ~/.bashrc

bun install
bun run build
chmod +x ./cli
echo "alias code='/root/claude/free-code/cli'" >> ~/.bashrc
source ~/.bashrc
code
```

上面这套流程适合 WSL2 / Ubuntu 环境，会手动拉取仓库、安装 Bun、构建 CLI，并把 `./cli` 准备好供你直接运行。

---

## 运行要求

- 运行时：`Bun >= 1.3.11`
- 系统：macOS / Linux / Windows(WSL)
- 认证：取决于你使用的 provider

## 构建

```bash
cd /root/claude/free-code
source ~/.bashrc
bun install
bun run build
chmod +x ./cli
./cli
```

常用构建命令：

| 命令 | 输出 | 说明 |
|---|---|---|
| `bun run build` | `./cli` | 常规构建 |
| `bun run build:dev` | `./cli-dev` | 开发构建 |
| `bun run build:dev:full` | `./cli-dev` | 开启全部可工作的实验 flag |
| `bun run compile` | `./dist/cli` | 输出到 `dist` |

---

## Provider 支持

当前支持 5 类 provider：

| Provider | 启用方式 | 认证方式 |
|---|---|---|
| Anthropic | 默认 | `ANTHROPIC_API_KEY` 或 OAuth |
| OpenAI | `CLAUDE_CODE_USE_OPENAI=1` | `OPENAI_API_KEY` 或 OpenAI OAuth |
| AWS Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` | AWS 凭证 |
| Google Vertex AI | `CLAUDE_CODE_USE_VERTEX=1` | GCP ADC |
| Anthropic Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` | Foundry API Key |

### Anthropic 直连

```bash
./cli
```

常见模型：

- `claude-opus-4-6`
- `claude-sonnet-4-6`
- `claude-haiku-4-5`

### OpenAI

这个仓库在原有 Codex OAuth 接入基础上，额外补上了标准 OpenAI Responses API 兼容路径。

#### 方式 1：OpenAI API Key + Base URL

```bash
export CLAUDE_CODE_USE_OPENAI=1
export OPENAI_API_KEY="sk-..."
export OPENAI_BASE_URL="https://api.openai.com/v1"
./cli --model gpt-5.4
```

说明：

- `OPENAI_BASE_URL` 可选，默认是 `https://api.openai.com/v1`
- 使用 `OPENAI_API_KEY` 时，OpenAI provider 默认模型会走 `gpt-5.4`
- 也支持显式指定 `gpt-5.4-mini`

#### 方式 2：Codex OAuth

```bash
export CLAUDE_CODE_USE_OPENAI=1
./cli
```

然后在交互模式里运行：

```text
/login
```

当前 OpenAI 相关模型：

- `gpt-5.3-codex`
- `gpt-5.4`
- `gpt-5.4-mini`

### AWS Bedrock

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION="us-east-1"
./cli
```

### Google Vertex AI

```bash
export CLAUDE_CODE_USE_VERTEX=1
./cli
```

### Anthropic Foundry

```bash
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_API_KEY="..."
./cli
```

---

## 使用方式

```bash
# 交互模式
./cli

# 单次提问
./cli -p "what files are in this directory?"

# 指定模型
./cli --model gpt-5.4

# 直接从源码启动
bun run dev
```

常见环境变量：

| 变量 | 作用 |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API Key |
| `ANTHROPIC_BASE_URL` | 自定义 Anthropic 接口地址 |
| `CLAUDE_CODE_USE_OPENAI` | 启用 OpenAI provider |
| `OPENAI_API_KEY` | OpenAI API Key |
| `OPENAI_BASE_URL` | OpenAI 接口基地址，默认 `/v1` |
| `CLAUDE_CODE_USE_BEDROCK` | 启用 Bedrock |
| `CLAUDE_CODE_USE_VERTEX` | 启用 Vertex |
| `CLAUDE_CODE_USE_FOUNDRY` | 启用 Foundry |

---

## 本仓库新增的 OpenAI 改动

相对于上游 `free-code`，这个版本额外加入了以下能力：

1. OpenAI provider 支持直接读取 `OPENAI_API_KEY`
2. 支持通过 `OPENAI_BASE_URL` 对接标准 OpenAI 兼容网关
3. OpenAI API 路径使用 Responses API，而不是只依赖 Codex Web OAuth
4. OpenAI provider 默认模型已补到 `gpt-5.4`
5. `auth status`、状态页、顶部 billing 展示已补齐 OpenAI API 模式

---

## 项目结构

```text
scripts/
  build.ts                构建脚本与 feature flag 注入

src/
  entrypoints/cli.tsx     CLI 入口
  commands.ts             Slash 命令注册
  tools.ts                工具注册
  QueryEngine.ts          主查询引擎
  screens/REPL.tsx        Ink/React 终端界面

  commands/               命令实现
  tools/                  工具实现
  components/             终端 UI 组件
  hooks/                  React hooks
  services/               API / OAuth / MCP / 分析等服务
  state/                  状态存储
  utils/                  通用工具
  skills/                 Skill 系统
  plugins/                插件系统
  bridge/                 IDE 桥接
  voice/                  语音输入
  tasks/                  后台任务
```

---

## 技术栈

| 项 | 技术 |
|---|---|
| Runtime | Bun |
| 语言 | TypeScript |
| 终端 UI | React + Ink |
| CLI 解析 | Commander.js |
| 校验 | Zod v4 |
| 搜索 | ripgrep |
| 协议 | MCP、LSP |
| API | Anthropic Messages、OpenAI、Bedrock、Vertex |

---

## 实验特性

项目保留了大量通过 `bun:bundle` feature flag 控制的实验功能。  
如果你要全量打开可工作的实验特性，可以使用：

```bash
bun run build:dev:full
```

其中比较重要的包括：

- `ULTRAPLAN`
- `ULTRATHINK`
- `VOICE_MODE`
- `BRIDGE_MODE`
- `VERIFICATION_AGENT`
- `TEAMMEM`
- `TOKEN_BUDGET`

完整审计见 [FEATURES.md](./FEATURES.md)。

---

## 贡献

欢迎提交 issue 和 PR。  
如果你在修复 feature flag、补 provider、修构建链，建议先阅读：

- [FEATURES.md](./FEATURES.md)
- [changes.md](./changes.md)

---

## 许可证与风险说明

原始 Claude Code 源代码权利仍归 Anthropic 所有。  
本项目来自一次公开暴露的源码快照整理结果，法律与合规风险需要你自行评估。

请仅在你明确理解相关风险的前提下使用。
