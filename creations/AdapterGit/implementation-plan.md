# 实现计划

## 当前状态

Phase 1 核心已完成，包含底层对象系统和存储基础设施，CLI 命令实现中。

## 依赖库

| 库 | 用途 |
|----|------|
| `sha1` | SHA-1 哈希 |
| `flate2` | zlib 压缩/解压 |
| `clap` | CLI 参数解析（derive 模式） |
| `serde` + `serde_json` | 序列化与 JSON 输出 |
| `anyhow` | 通用错误处理 |

明确不使用 gitoxide 或任何外部 Git 库，完全原生实现。

## 开发路线

### Phase 1 — 项目初始化（✅ 已完成）

- Rust 项目框架搭建
- 目录结构创建
- 基础 CLI 框架（clap derive）
- 错误处理系统

### Phase 2 — 核心对象系统（✅ 已完成）

- SHA-1 哈希实现
- zlib 压缩/解压
- Blob 对象（含序列化/反序列化）
- Tree 对象（含序列化/反序列化）
- Commit 对象（含序列化/反序列化、多父提交支持）
- 对象存储（loose objects 读写）
- 引用系统（HEAD、分支、标签）
- 索引文件（index 序列化/反序列化）

### Phase 3 — 基础命令实现（🚧 进行中）

P0 命令：
- `init` — 初始化仓库
- `add` — 添加文件到暂存区
- `commit` — 提交更改

P1 命令：
- `status` — 查看状态
- `log` — 查看提交历史

P2 命令：
- `cat-file` — 查看对象内容
- `ls-tree` — 列出树对象
- `diff` — 比较差异

### Phase 4 — AI 模式与输出

- AI 模式 `--ai` 参数
- 自动标记 `[AI-committed]`
- JSON 输出 `--json`
- YAML 输出 `--yaml`
- 危险操作防护
- 命令自动转换

### Phase 5 — 网络功能

- `clone` — 克隆仓库
- `push` — 推送到远程
- `pull` / `fetch` — 拉取更新
- Git HTTP(S) Smart Protocol 实现

### Phase 6 — 配置与扩展

- 环境变量支持（`AGIT_AI_MODE`、`AGIT_OUTPUT_FORMAT`）
- 配置文件（`~/.config/agit/config.toml`）
- 颜色控制（`--no-color`）

### Phase 7 — 测试与发布

- 单元测试全覆盖
- 集成测试
- 与原生 Git 一致性测试
- 跨平台编译（Linux/macOS/Windows）
- 静态 musl 编译
- Release 构建

## 里程碑

- v0.1.0 — MVP：init, add, commit, status, log + 基础 AI 模式 + JSON 输出
- v0.2.0 — 本地完整：所有本地命令 + AI 安全防护 + 配置文件
- v0.3.0 — 网络功能：clone, push, pull, fetch
- v1.0.0 — 稳定版：完整 Git 子集 + 跨平台 + 完善文档
