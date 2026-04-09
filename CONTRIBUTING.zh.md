# 参与贡献 OpenCode

我们希望让您能够轻松地为 OpenCode 做出贡献。以下是最常被合并的更改类型：

- Bug 修复
- 额外的 LSP / 格式化工具
- LLM 性能的改进
- 对新提供商的支持
- 针对特定环境问题的修复
- 缺失的标准行为
- 文档改进

但是，任何 UI 或核心产品功能在实施之前，都必须经过核心团队的设计审查。

如果您不确定 PR 是否会被接受，请随时询问维护者，或者查找带有以下标签的 issue：

- [`help wanted`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3Ahelp-wanted)
- [`good first issue`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22good%20first%20issue%22)
- [`bug`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3Abug)
- [`perf`](https://github.com/anomalyco/opencode/issues?q=is%3Aopen%20is%3Aissue%20label%3A%22perf%22)

> [!NOTE]
> 无视这些指导原则的 PR 很可能会被关闭。

想接手一个 issue 吗？请留言评论，除非我们已经在处理它，否则维护者可能会将其分配给您。

## 添加新的提供商

新的提供商不应该需要太多（甚至不需要任何）代码更改，但如果您想添加对新提供商的支持，请首先向以下地址提交 PR：
https://github.com/anomalyco/models.dev

## 开发 OpenCode

- 要求：Bun 1.3+
- 在项目根目录下安装依赖并启动开发服务器：

  ```bash
  bun install
  bun dev
  ```

### 在不同目录下运行

默认情况下，`bun dev` 会在 `packages/opencode` 目录下运行 OpenCode。要使其在不同的目录或存储库下运行：

```bash
bun dev <directory>
```

要在 opencode 仓库本身的根目录下运行 OpenCode：

```bash
bun dev .
```

### 构建 "localcode"

要编译一个独立的可执行文件：

```bash
./packages/opencode/script/build.ts --single
```

然后通过以下命令运行它：

```bash
./packages/opencode/dist/opencode-<platform>/bin/opencode
```

将 `<platform>` 替换为您的平台（例如，`darwin-arm64`, `linux-x64`）。

- 核心部分：
  - `packages/opencode`: OpenCode 核心业务逻辑和服务器。
  - `packages/opencode/src/cli/cmd/tui/`: TUI 代码，使用 SolidJS 和 [opentui](https://github.com/sst/opentui) 编写。
  - `packages/app`: 共享的 Web UI 组件，使用 SolidJS 编写。
  - `packages/desktop`: 原生桌面应用，使用 Tauri 构建（包装了 `packages/app`）。
  - `packages/plugin`: `@opencode-ai/plugin` 的源代码。

### 理解 bun dev 与 opencode 的区别

在开发过程中，`bun dev` 相当于构建后的 `opencode` 命令的本地版本。两者运行相同的 CLI 界面：

```bash
# 开发模式 (在项目根目录运行)
bun dev --help           # 显示所有可用命令
bun dev serve            # 启动无头 API 服务器
bun dev web              # 启动服务器 + 打开 Web 界面
bun dev <directory>      # 在特定目录启动 TUI

# 生产模式
opencode --help          # 显示所有可用命令
opencode serve           # 启动无头 API 服务器
opencode web             # 启动服务器 + 打开 Web 界面
opencode <directory>     # 在特定目录启动 TUI
```

### 运行 API 服务器

要启动 OpenCode 无头 API 服务器：

```bash
bun dev serve
```

默认情况下会在 4096 端口启动无头服务器。您可以指定其他端口：

```bash
bun dev serve --port 8080
```

### 运行 Web 应用

要在开发期间测试 UI 更改：

1. **首先，启动 OpenCode 服务器**（参见上方[运行 API 服务器](#运行-api-服务器)部分）
2. **然后运行 Web 应用：**

```bash
bun run --cwd packages/app dev
```

这将在 http://localhost:5173 （或输出中显示的类似端口）启动本地开发服务器。大多数 UI 更改可以在此处进行测试，但必须运行服务器以获得完整功能。

### 运行桌面应用

桌面应用是一个封装了 Web UI 的原生 Tauri 应用程序。

要运行原生桌面应用：

```bash
bun run --cwd packages/desktop tauri dev
```

这将在 http://localhost:1420 启动 Web 开发服务器并打开原生窗口。

如果您只想要 Web 开发服务器（无需原生外壳）：

```bash
bun run --cwd packages/desktop dev
```

要创建生产环境的 `dist/` 并构建原生应用程序包：

```bash
bun run --cwd packages/desktop tauri build
```

这会通过 Tauri 的 `beforeBuildCommand` 自动运行 `bun run --cwd packages/desktop build`。

> [!NOTE]
> 运行桌面应用需要额外的 Tauri 依赖项（Rust 工具链，特定于平台的库）。有关设置说明，请参阅 [Tauri 先决条件](https://v2.tauri.app/start/prerequisites/)。

> [!NOTE]
> 如果您对 API 或 SDK（例如 `packages/opencode/src/server/server.ts`）进行了更改，请运行 `./script/generate.ts` 来重新生成 SDK 及相关文件。

请尽量遵循[代码风格指南](./AGENTS.md)

### 设置调试器

目前 Bun 的调试功能还有待完善。我们希望本指南能帮助您完成设置并避免一些痛点。

调试 OpenCode 最可靠的方法是通过 `bun run --inspect=<url> dev ...` 在终端中手动运行它，并通过该 URL 附加您的调试器。其他方法可能会导致断点映射错误，至少在 VSCode 中是这样（因人而异）。

注意事项：

- 如果您想运行 OpenCode TUI 并希望在服务器代码中触发断点，您可能需要运行 `bun dev spawn` 而不是通常的 `bun dev`。这是因为 `bun dev` 在工作线程中运行服务器，而断点在那里可能不起作用。
- 如果 `spawn` 对您不起作用，您可以单独调试服务器：
  - 调试服务器：`bun run --inspect=ws://localhost:6499/ --cwd packages/opencode ./src/index.ts serve --port 4096`，然后使用 `opencode attach http://localhost:4096` 附加 TUI
  - 调试 TUI：`bun run --inspect=ws://localhost:6499/ --cwd packages/opencode --conditions=browser ./src/index.ts`

其他提示和技巧：

- 您可能需要使用 `--inspect-wait` 或 `--inspect-brk` 而不是 `--inspect`，具体取决于您的工作流程
- 每次调用都指定 `--inspect=ws://localhost:6499/` 可能会很繁琐，您可能希望改用 `export BUN_OPTIONS=--inspect=ws://localhost:6499/`

#### VSCode 设置

如果您使用 VSCode，您可以使用我们的示例配置 [.vscode/settings.example.json](.vscode/settings.example.json) 和 [.vscode/launch.example.json](.vscode/launch.example.json)。

一些可能有问题的调试方法：

- 带有 `"request": "launch"` 的调试配置可能会导致断点映射不正确，从而无法使用
- 在 VSCode `JavaScript Debug Terminal` 中运行 OpenCode 时也会出现同样的问题

话虽如此，您可能仍想尝试这些方法，因为它们可能对您有效。

## Pull Request 期望

### Issue 优先策略

**所有 PR 必须引用现有的 issue。** 在开启 PR 之前，请打开一个描述 bug 或功能的 issue。这有助于维护者进行分类并防止重复工作。没有链接 issue 的 PR 可能会在没有审查的情况下被关闭。

- 在您的 PR 描述中使用 `Fixes #123` 或 `Closes #123` 来链接 issue
- 对于小修复，一个简短的 issue 就可以了——只要能让维护者理解问题所在的上下文即可

### 一般要求

- 保持 PR 小而专注
- 解释 issue 以及为什么您的更改能修复它
- 在添加新功能之前，确保它在代码库的其他地方还不存在

### UI 更改

如果您的 PR 包含 UI 更改，请附上展示前后对比的屏幕截图或视频。这有助于维护者更快地进行审查，并为您提供更快的反馈。

### 逻辑更改

对于非 UI 更改（Bug 修复、新功能、重构），请解释**您是如何验证它是否有效的**：

- 您测试了什么？
- 审查者如何重现/确认该修复？

### 拒绝 AI 生成的长篇大论

冗长的、AI 生成的 PR 描述和 issue 是不可接受的，可能会被忽略。请尊重维护者的时间：

- 编写简短、重点突出的描述
- 用您自己的话解释更改了什么以及为什么更改
- 如果您无法简短地解释，您的 PR 可能太大了

### PR 标题

PR 标题应遵循约定式提交标准（Conventional Commits）：

- `feat:` 新功能
- `fix:` Bug 修复
- `docs:` 文档或 README 更改
- `chore:` 维护任务、依赖更新等
- `refactor:` 代码重构（不改变行为）
- `test:` 添加或更新测试

您可以选择包含一个作用域（scope）来指示受影响的包：

- `feat(app):` app 包中的新功能
- `fix(desktop):` desktop 包中的 bug 修复
- `chore(opencode):` opencode 包中的维护

示例：

- `docs: update contributing guidelines`
- `fix: resolve crash on startup`
- `feat: add dark mode support`
- `feat(app): add dark mode support`
- `fix(desktop): resolve crash on startup`
- `chore: bump dependency versions`

### 代码风格偏好

这些不是严格执行的，它们只是一般的指导原则：

- **函数 (Functions):** 除非拆分能带来明显的复用或组合优势，否则请将逻辑保留在单个函数中。
- **解构 (Destructuring):** 不要对变量进行不必要的解构。
- **控制流 (Control flow):** 避免使用 `else` 语句。
- **错误处理 (Error handling):** 在可能的情况下，优先使用 `.catch(...)` 而不是 `try`/`catch`。
- **类型 (Types):** 尽可能使用精确的类型，避免使用 `any`。
- **变量 (Variables):** 坚持不可变模式，避免使用 `let`。
- **命名 (Naming):** 在保持描述性的前提下，选择简洁的单字标识符。
- **运行时 API (Runtime APIs):** 在适用的情况下，使用 Bun 的辅助函数，例如 `Bun.file()`。

## 功能请求

对于全新的功能，请从设计讨论开始。开启一个 issue，描述问题、您的拟议方法（可选），以及为什么它属于 OpenCode。核心团队将帮助决定是否应该推进；请等待该批准，而不是直接开启功能 PR。

## 信任与担保 (Vouch) 系统

本项目使用 [vouch](https://github.com/mitchellh/vouch) 来管理贡献者的信任。担保列表维护在 [`.github/VOUCHED.td`](.github/VOUCHED.td) 中。

### 工作原理

- **被担保的用户 (Vouched users)** 是被明确信任的贡献者。
- **被谴责的用户 (Denounced users)** 是被明确屏蔽的。来自被谴责用户的 Issue 和 PR 将自动关闭。如果您被谴责了，可以通过在 [Discord](https://opencode.ai/discord) 上联系维护者来请求取消担保。
- **其他所有人** 都可以正常参与 —— 您无需被担保即可提交 Issue 或 PR。

### 给维护者的指南

具有写入权限的协作者可以通过在任何 issue 上评论来管理担保列表：

- `vouch` — 为 issue 作者担保
- `vouch @username` — 为特定用户担保
- `denounce` — 谴责 issue 作者
- `denounce @username` — 谴责特定用户
- `denounce @username <reason>` — 附带理由谴责
- `unvouch` / `unvouch @username` — 从列表中移除某人

更改会自动提交到 `.github/VOUCHED.td`。

### 谴责策略

谴责仅保留给那些反复提交低质量的 AI 生成的贡献、发送垃圾信息或以其他恶意方式行事的用户。它不用于分歧或无心之过。

## Issue 要求

所有 issue **必须** 使用我们的 issue 模板之一：

- **Bug report** (Bug 报告) — 用于报告 Bug（需要描述）
- **Feature request** (功能请求) — 用于建议增强功能（需要勾选验证复选框并提供描述）
- **Question** (问题) — 用于提问（需要填写问题）

不允许提交空白 issue。开启新 issue 时，自动检查程序会验证其是否遵循了模板并符合我们的贡献准则。如果 issue 不符合要求，您将收到一条评论解释需要修改的地方，并且有 **2 小时** 的时间来编辑该 issue。超时后它将自动关闭。

Issue 可能会因以下原因被标记：

- 没有使用模板
- 必填字段留空或填充了占位符文本
- AI 生成的长篇大论
- 缺乏有意义的内容

如果您认为您的 issue 被错误标记，请告知维护者。