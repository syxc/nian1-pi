# nian1-pi-preset

nian1 的 [Pi](https://pi.dev) coding agent preset。

通过 Pi Package 打包并分发个人常用的 **extensions / skills / prompts / themes**，一次 `pi install` 即加载全部 22 个常用插件。

> **Security:** Pi packages 拥有完整系统权限。Extensions 可执行任意代码，skills 可指示模型执行任意操作。安装第三方包前请先审阅源码。

## 安装

```bash
# 从本地路径安装（开发中）
pi install /absolute/path/to/nian1-pi
pi install ./relative/path/to/nian1-pi

# 从 git 安装
pi install git:github.com/syxc/nian1-pi
pi install https://github.com/syxc/nian1-pi
```

仅当前会话试用（不写入 settings）：

```bash
pi -e /path/to/nian1-pi
pi -e git:github.com/syxc/nian1-pi
```

安装到项目级（写入 `.pi/settings.json`，可团队共享）：

```bash
pi install -l /path/to/nian1-pi
```

## 卸载 / 管理

```bash
pi remove git:github.com/syxc/nian1-pi
pi list
pi update --extensions
pi config                       # 启用/禁用具体资源
```

## 包结构

```
nian1-pi/
├── package.json          # pi manifest + pi-package keyword
├── README.md
├── extensions/           # .ts / .js 扩展
├── skills/               # SKILL.md 技能目录
├── prompts/              # .md 提示词模板
└── themes/               # .json 主题
```

`package.json` 中的 `pi` 字段声明资源路径：

```json
{
  "name": "nian1-pi-preset",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"]
  }
}
```

路径相对于包根目录；数组支持 glob 与 `!exclusions`。

若省略 `pi` manifest，Pi 会从同名约定目录自动发现资源。

## 开发

1. 在对应目录添加资源：
   - `extensions/*.ts` — 扩展
   - `skills/<name>/SKILL.md` — 技能
   - `prompts/*.md` — 提示词
   - `themes/*.json` — 主题
2. 本地加载验证：

```bash
pi -e .
# 或
pi install .
```

3. 用 `pi config` 检查资源是否被加载。

### 依赖约定

- 运行时第三方依赖放在 `dependencies`（`pi install` 会执行 `npm install`）
- 若 import Pi 核心包，放入 `peerDependencies` 且版本为 `"*"`，不要打包：
  - `@earendil-works/pi-ai`
  - `@earendil-works/pi-agent-core`
  - `@earendil-works/pi-coding-agent`
  - `@earendil-works/pi-tui`
  - `typebox`
- 捆绑包对 Pi 核心包的 peer range 经常已过期（例如多个包要求
  `@earendil-works/pi-tui >=0.74.0`，而运行时已在 0.85.x）。仓库根目录 `.npmrc`
  因此设置 `force=true`，让 `npm ci` 容忍这类冲突但不丢弃 peer 树。
  不要改用 `legacy-peer-deps`（会从 lock 中移除整个 `@earendil-works/*` peer 子树）。
- 本 preset 包含 5 个 git 来源依赖；npm 12 默认 `allow-git=none`，安装时需加
  `--allow-git=root`。

## 内置依赖（bundled pi packages）

安装本 preset 时会一并带上下列 22 个 Pi 包，并自动加载其 extensions / skills / prompts / themes。

| 包 | 来源 | 提供 |
|----|------|------|
| [pi-mcp-adapter](https://www.npmjs.com/package/pi-mcp-adapter) | npm | MCP 协议适配扩展，把 MCP server 工具接入 Pi |
| [pi-web-access](https://www.npmjs.com/package/pi-web-access) | npm | Web 搜索 / URL 抓取 / GitHub clone / PDF / YouTube 等 |
| [@ff-labs/pi-fff](https://www.npmjs.com/package/@ff-labs/pi-fff) | npm | FFF 驱动的模糊文件与内容搜索 |
| [pi-fff-non-ascii-guard](https://github.com/eiei114/pi-fff-non-ascii-guard) | git | fff 搜索前检测并重命名非 ASCII 文件名，避免 UTF-8 边界 panic |
| [pi-subagents](https://www.npmjs.com/package/pi-subagents) | npm | 单 agent 委派与脚本化多 agent 工作流 |
| [pi-messenger](https://www.npmjs.com/package/pi-messenger) | npm | agent 间消息传递与文件占用协调 |
| [pi-btw](https://www.npmjs.com/package/pi-btw) | npm | `/btw` 并行支线对话 |
| [@juicesharp/rpiv-todo](https://www.npmjs.com/package/@juicesharp/rpiv-todo) | npm | 模型驱动的 todo 列表，存活于 `/reload` 与会话压缩 |
| [pi-goals](https://www.npmjs.com/package/pi-goals) | npm | 持久化目标跟踪，带预算、可复用 prompt 与 churn 监控 |
| [context-mode](https://www.npmjs.com/package/context-mode) | npm | 沙箱代码执行 + FTS5 知识索引，节省上下文窗口 |
| [pi-hermes-memory](https://www.npmjs.com/package/pi-hermes-memory) | npm | 持久记忆 / 会话搜索 / 密钥扫描 |
| [@sting8k/pi-vcc](https://www.npmjs.com/package/@sting8k/pi-vcc) | npm | 无 LLM 调用的结构化会话压缩，保留原文转录 |
| [sol-pi](https://github.com/NVlabs/SoL-Pi) | git | 上下文与工具效率扩展（NVlabs） |
| [pi-cache-optimizer](https://www.npmjs.com/package/pi-cache-optimizer) | npm | Prompt/KV cache 命中率优化 |
| [visual-explainer](https://www.npmjs.com/package/visual-explainer) | npm | 生成图表 / diff review / plan review / slides 的 HTML 页面 |
| [pi-warden](https://www.npmjs.com/package/pi-warden) | npm | 按 `pi-warden.md` 对每次写入做规则校验 |
| [pi-cc-extensions](https://www.npmjs.com/package/pi-cc-extensions) | npm | Claude Code 风格 UI、上下文检视 + cc-dark/cc-light 主题 |
| [pi-autoresearch](https://github.com/davebcn87/pi-autoresearch) | git | 自主实验循环：运行、测量、保留或丢弃 |
| [@joelhooks/pi-until](https://github.com/syxc/pi-until) | git | shell 条件监视与周期性 agent 跟进（含个人 fork） |
| [pi-rewind-hook](https://www.npmjs.com/package/pi-rewind-hook) | npm | 自动 git checkpoint，支持文件/会话回退 |
| [@juicesharp/rpiv-ask-user-question](https://www.npmjs.com/package/@juicesharp/rpiv-ask-user-question) | npm | 结构化问询卡片，避免模型替用户猜测 |
| [@dietrichgebert/ponytail](https://github.com/DietrichGebert/ponytail) | git | 懒惰资深工程师模式：最好的代码是没写的代码 |

它们声明在 `dependencies` + `bundleDependencies` 中，资源通过 `pi.extensions` /
`pi.skills` / `pi.prompts` / `pi.themes` 的 `node_modules/...` 路径引用。

## 依赖更新

本 preset 通过 git 分发，**不发布到 npm**。依赖刷新由手动 workflow 完成：

1. GitHub → **Actions** → **Update dependencies** → **Run workflow**
2. Workflow 用 `npm-check-updates` 更新直接 npm 依赖、刷新 lockfile，并校验 `pi` manifest 路径仍可解析
3. 有变更则提交到 `chore/update-dependencies` 分支并开 PR；无变更则跳过

前置条件：仓库 Settings → Actions → Workflow permissions 为 **Read and write**（需推送分支并创建 PR）。
Workflow 文件：`.github/workflows/update-dependencies.yml`。

## 当前本地资源

| 类型 | 状态 |
|------|------|
| Extensions | 待添加（`./extensions`） |
| Skills | 待添加（`./skills`） |
| Prompts | 待添加（`./prompts`） |
| Themes | 待添加（`./themes`） |

## License

MIT
