# AdapterGit（agit）

Git for AI, not for editors — 让 Git 在 AI 时代不再卡死。

为自动化、脚本、CI/CD 和公共电脑环境设计的无 TUI Git 实现，完全从底层用 Rust 原生实现 Git 核心协议和算法，不依赖任何外部 Git 库（如 gitoxide）或系统 Git 命令。

---

## 痛点解决

- AI Agent 调用 Git 时被 TUI 编辑器卡死
- 在学校机房每次都要重新安装 Git
- 脚本中 Git 命令意外触发交互界面
- 原生 Git 在非 TTY 环境表现诡异

---

## 核心设计目标

1. **AI 优先** — 零 TUI 阻塞，结构化输出
2. **原生实现** — 从底层实现 Git 核心协议和算法
3. **便携性** — 单文件静态编译，无依赖
4. **安全性** — AI Agent 安全调用，防止误操作

---

## 核心特性

- 零 TUI 阻塞，AI Agent 的安全选择
- 结构化 JSON/YAML 输出，机器可读
- 自动添加 `[AI-committed]` 标记
- 危险操作防护，防止 AI 误操作
- 静态单文件编译，10MB 拷了就走
- 无需安装，无需 root 权限
- 兼容现有 Git 仓库和工作流

---

## 技术栈

Rust + zlib 压缩 + SHA-1 哈希 + Git 原生协议

---

## 扩展功能规划

### 多身份切换（user.json）
- 在仓库中添加 `user.json` 文件
- 支持一键切换用户名和邮箱
- 方便多人共用电脑场景

### 多仓库同步（source.json）
- 添加 `source.json` 配置文件
- 支持一键同步推送至多个远程仓库
- 解决 Gitee 强制同步无法同步 tags 和贡献记录的问题

### 提交者关联（connect 关键字）
- 允许将目录仓库与特定提交者关联
- 当 commit 对象非关联提交者时，提示用户确认是否继续
- 增强团队协作时的代码归属管理