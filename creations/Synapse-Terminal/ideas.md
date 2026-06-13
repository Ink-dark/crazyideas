# Project Idea: **Synapse Terminal** (暂定名)

## 1. 核心概念 (The Pitch)
**Synapse** 是一个基于 **Windows Terminal (WT)** 深度魔改的 **AI-Native 终端**。

它不是简单的“终端里套个聊天框”，而是通过接管 WT 底层的 **PTY (伪终端)** 数据流，赋予 AI **真正的视觉** 和 **操作系统级的调度能力**。

## 2. 为什么要做？（痛点暴击）

### 2.1 现有 AI 工具的缺陷
目前的 AI 编程助手（Cursor, Claude Code, Warp）都存在“感知障碍”：
- **盲眼操作**：它们通过 `stdout/stderr` 获取信息，必须依赖 `2>&1` 或 `grep` 来猜发生了什么。一旦遇到交互式命令（Vim, Git Rebase, SSH），AI 就会卡死或失控。
- **文本解析脆弱**：AI 需要费力解析 Git 输出的文本来判断状态，一旦 Git 版本更新换了文案，AI 就瞎了。

### 2.2 我的解决方案
**让终端成为 AI 的“眼睛”和“手”。**
- **所见即所得**：AI 直接读取 PTY 缓冲区，屏幕上有啥它看啥，不需要 `2>&1`。
- **原生互操作性**：终端与我的另一个项目 **AdapterGit (Rust Git)** 深度绑定，AI 调用 Git 时直接走内部 API，无需解析文本。

## 3. 技术架构 (Architecture)

### 3.1 Fork Windows Terminal
- **Base**: Microsoft/Terminal (MIT License)。
- **Modification**: 在 C++/WinRT 层增加 IPC (命名管道/Named Pipe) 接口，用于向外部 AI 进程发送 PTY 数据流。

### 3.2 Rust AI Bridge (核心创新)
这是跑在后台的独立 Rust 进程，也是我的主战场：
- **UI Layer**: 接收来自 WT 的屏幕文本。
- **Logic Layer**: 运行 `AdapterGit` 核心库。当检测到当前目录是 Git 仓库时，直接调用 `agit::status()` 获取结构化数据，而不是解析 `git status` 的字符串。
- **Config Layer**: **Bring Your Own Key (BYOK)**。用户通过 `config.toml` 配置 API Endpoint (OpenAI, Anthropic, Ollama, 自定义)。

### 3.3 交互模式
- **Sidebar Mode**: 在 WT 右侧常驻一个 AI 面板。
- **Agent Mode**: 用户按快捷键，AI 接管输入框，直接生成并执行命令。

## 4. 杀手级应用场景 (Killer Features)

### 4.1 Git 冲突解决 (AdapterGit 专属)
1.  用户执行 `agit merge feature`。
2.  WT 检测到冲突，通知 AI Bridge。
3.  AI 直接读取 `agit` 内部的 Index 状态，精准定位冲突代码块。
4.  AI 给出修复建议，用户一键应用。

### 4.2 零配置自动化
- **传统**: `command > log.txt 2>&1 || exit 1`
- **Synapse**: AI 看着屏幕说：“这个命令报错了，原因是权限不足，需要 sudo 吗？” —— **完全不需要脚本处理 stderr。**

## 5. 为什么不用 Electron/Warp？
- **性能**: WT 是 GPU 加速的原生应用，Rust Bridge 零开销。
- **生态**: Windows 开发者缺乏好用的 AI 终端，Mac 有 Warp，Windows 是空白。
- **控制权**: 我不做 SaaS，我只做工具。用户自己付 API 费，自己掌控数据。

## 6. 当前状态 (Status)
- **AdapterGit**: P1 功能完成 (Merge/Squash/Stash/Tag)。
- **Synapse**: Idea Phase / 准备 Fork WT。

---

## 7. 给开发者的话 (Dev Notes)
- **不要训练 AI 识别 Git 文本**。要训练 AI 调用 `AdapterGit` 的 Rust API。
- **不要模拟终端**。要直接修改 Windows Terminal 源码。
- **不要硬编码 API**。要让用户自己填 Key。

---

**一句话总结：**
> 把 Windows Terminal 变成一个 **AI 操作系统**，并用 **Rust** 重写它的灵魂。
