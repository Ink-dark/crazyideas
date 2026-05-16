# AdapterGit（agit）

Git for AI, not for editors — 让 Git 在 AI 时代不再卡死。

为自动化、脚本、CI/CD 和公共电脑环境设计的无 TUI Git 实现，完全从底层用 Rust 原生实现 Git 核心协议和算法，不依赖任何外部 Git 库（如 gitoxide）或系统 Git 命令。

## 痛点解决

- AI Agent 调用 Git 时被 TUI 编辑器卡死
- 在学校机房每次都要重新安装 Git
- 脚本中 Git 命令意外触发交互界面
- 原生 Git 在非 TTY 环境表现诡异

## 核心设计目标

1. AI 优先 — 零 TUI 阻塞，结构化输出
2. 原生实现 — 从底层实现 Git 核心协议和算法
3. 便携性 — 单文件静态编译，无依赖
4. 安全性 — AI Agent 安全调用，防止误操作

## 核心特性

- 零 TUI 阻塞，AI Agent 的安全选择
- 结构化 JSON/YAML 输出，机器可读
- 自动添加 [AI-committed] 标记
- 危险操作防护，防止 AI 误操作
- 静态单文件编译，10MB 拷了就走
- 无需安装，无需 root 权限
- 兼容现有 Git 仓库和工作流

## 技术栈

Rust + zlib压缩 + SHA-1哈希 + Git原生协议
