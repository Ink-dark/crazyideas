# 架构设计

## 系统架构

```
┌─────────────────────────────────────────┐
│             AI Agent / Script           │
└─────────────────┬───────────────────────┘
                  │ JSON / 结构化输出
┌─────────────────▼───────────────────────┐
│              agit (适配层)               │
│  ┌───────────────────────────────────┐  │
│  │  TUI 消除  │ 便携封装 │ AI 安全    │  │
│  └─────────────────┬─────────────────┘  │
└─────────────────┬───────────────────────┘
                  │ 纯 Rust 原生实现
┌─────────────────▼───────────────────────┐
│         原生 Git 核心实现 (Pure Rust)    │
└─────────────────────────────────────────┘
```

## 目录结构

```
src/
├── core/              # 核心 Git 算法
│   ├── objects/       # Git 对象 (blob, tree, commit, tag)
│   ├── storage/       # 对象存储 (.git/objects)
│   ├── pack/          # Pack 文件读写
│   ├── index/         # 索引文件操作
│   ├── diff/          # Diff 和 Patch 算法
│   └── hash/          # SHA-1 哈希实现
├── refs/              # 引用管理
│   ├── heads/         # 分支 refs/heads/*
│   ├── tags/          # 标签 refs/tags/*
│   └── packed_refs/   # 打包引用
├── protocol/          # Git 协议
│   ├── client/        # fetch/push 客户端
│   └── server/        # receive-pack/upload-pack
├── cli/               # 命令行解析
│   ├── commands/      # 命令定义
│   ├── parser/        # 参数解析
│   └── help/         # 帮助系统
├── ai/                # AI 模式
│   ├── tagger/       # 自动标记 [AI-committed]
│   ├── safety/       # 危险操作防护
│   └── converter/    # 命令转换
├── output/            # 格式化输出
│   ├── json/         # JSON 输出
│   ├── yaml/         # YAML 输出
│   └── text/         # 文本输出
├── config/            # 配置管理
│   ├── env/          # 环境变量
│   ├── file/         # 配置文件
│   └── loader/       # 配置加载
└── utils/            # 工具函数
    ├── error/        # 错误处理
    ├── path/         # 路径操作
    └── crypto/       # 加密工具
```

## 核心技术挑战

| 层级 | 内容 | 复杂度 |
|------|------|--------|
| **对象模型** | blob, tree, commit, tag 对象 | ⭐⭐ |
| **存储系统** | loose objects, pack files, index | ⭐⭐⭐ |
| **引用系统** | refs, HEAD, branches, tags | ⭐⭐ |
| **差异算法** | diff, patch generation | ⭐⭐⭐ |
| **压缩算法** | zlib 压缩, delta 压缩 | ⭐⭐⭐ |
| **传输协议** | HTTP(S) smart protocol, SSH | ⭐⭐⭐⭐ |

## 核心模块

### 对象系统 (core/objects)

基于内容寻址的对象存储，四种对象类型：

- Blob — 原始文件内容
- Tree — 目录快照，条目指向 blob 或子树
- Commit — 提交记录，指向 tree，包含作者和父提交
- Tag — 符号引用，指向任意 Git 对象

### 存储系统 (core/storage)

Loose Objects 按 SHA-1 前两位分目录：

```
.git/objects/
├── ab/
│   └── cdef1234...  # 后38位为文件名
```

Pack Files 使用 delta 压缩减少空间，创建 .idx 索引文件加速查找。

### 引用系统 (refs)

```
.git/
├── HEAD              # 当前分支指针（symbolic 或 detached）
├── refs/
│   ├── heads/        # 本地分支
│   └── tags/         # 标签
├── packed-refs       # 打包引用
└── ORIG_HEAD         # 操作前 HEAD
```

HEAD 支持两种模式：符号引用（`ref: refs/heads/main`）和 detached 模式（直接指向 commit SHA-1）。

### 压缩算法

Git 对象使用 zlib 压缩存储：

```
写入: 原始内容 → zlib 压缩 → 写入 .git/objects
读取: .git/objects → zlib 解压 → 原始内容
```

### 索引文件 (.git/index)

二进制格式，头部为 DIRC 签名，每个条目记录文件模式、SHA-1、路径等信息。条目按 8 字节对齐，末尾包含整个索引的 SHA-1 校验和。

## 错误处理

统一错误类型，覆盖 Git 操作的所有异常场景：

- Io — IO 操作错误
- ObjectNotFound — SHA-1 对象未找到
- InvalidObject — 对象格式无效
- InvalidRef — 引用格式无效
- CompressionError — 压缩/解压错误
- RepoNotFound — 仓库路径不存在
- NotAGitRepo — 路径不是 Git 仓库

## 性能策略

1. 延迟加载 — 不一次性加载所有对象
2. 缓存 — LRU 缓存频繁访问的对象
3. 批量操作 — 合并小操作为批量操作
4. 增量索引 — 避免全量重建索引

## 测试策略

1. 单元测试 — 每个模块独立测试（含序列化/反序列化往返测试）
2. 集成测试 — 完整的 Git 工作流测试
3. 一致性测试 — 与原生 Git 输出对比
4. 模糊测试 — 随机输入测试鲁棒性
