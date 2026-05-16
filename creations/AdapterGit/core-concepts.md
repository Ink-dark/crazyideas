# 核心概念

## Git 对象模型

agit 原生实现了 Git 的四种核心对象类型。

### Blob（文件内容）

最基础的对象类型，存储文件的原始字节内容。通过 SHA-1 内容寻址。

序列化格式：`blob <size>\0<content>`

```rust
let blob = Blob::new(b"hello world\n".to_vec());
blob.hash() // "3b18e512dba79e4c8300dd08aeb37f8e728b8dad"
```

### Tree（目录快照）

记录目录结构，每个条目包含文件名、权限模式（100644/100755/40000）和指向 blob 或子树的 SHA-1。

序列化格式：`tree <size>\0<mode> <name>\0<20-byte sha1>...`

### Commit（提交记录）

指向一个 tree 对象，记录作者、提交者、时间戳和提交消息。可包含多个父提交（普通提交一个，合并提交多个）。

序列化格式：
```
commit <size>\0tree <sha1>
parent <sha1>
author <name> <email> <timestamp> <timezone>
committer <name> <email> <timestamp> <timezone>

<message>
```

### Tag（标签引用）

指向任意 Git 对象的带注解符号引用。

## SHA-1 哈希

每个 Git 对象通过 SHA-1 唯一标识。哈希计算方式为：

```
sha1 = SHA1("<type> <size>\0" + content)
```

实现了 `hash_bytes`（通用哈希）和 `hash_git_object`（带类型头的 Git 对象专用哈希）。

## 对象存储

### Loose Objects

未打包的独立对象，按 SHA-1 前两字符分目录存储在 `.git/objects/` 下。写入时自动 zlib 压缩，读取时自动解压。

```rust
// 写入 blob 到 .git/objects/
let sha1 = write_object(repo_path, "blob", content)?;

// 读取对象，返回 (类型, 内容)
let (obj_type, data) = read_object(repo_path, &sha1)?;
```

### Pack Files（计划中）

将多个对象打包并做 delta 压缩，配合 .idx 索引文件实现高效查找。

## 引用系统

### HEAD

仓库的当前状态指针，支持两种模式：

- 符号引用：`ref: refs/heads/main` — 指向某个分支
- Detached HEAD：`<commit-sha1>` — 直接指向某次提交

```rust
let head = read_head(repo)?; // 自动解析符号引用
write_head(repo, "ref: refs/heads/main")?;
```

### 分支

存储在 `refs/heads/<name>`，内容为指向该分支最新 commit 的 SHA-1。

```rust
create_branch(repo, "feature", &commit_sha1)?;
let branches = list_branches(repo)?; // ["main", "feature"]
```

## 索引文件 (.git/index)

暂存区的实现，二进制格式记录待提交的文件快照。agit 完整实现了 index 的序列化和反序列化，包括 DIRC 头部、条目对齐和 SHA-1 尾部校验。

## AI 安全层

### 自动标记

AI 模式下自动为提交添加 `[AI-committed]` 标记，明确区分人工和 AI 提交。

### 命令转换

自动将交互式命令转换为非交互式：
- `git commit` → `agit commit -m "auto-commit"`
- `git rebase -i` → `agit rebase --no-edit`
- `git add -p` → `agit add -A`

### 危险操作防护

明确拒绝执行以下高危交互操作：
- `git mergetool`
- 交互式 rebase
- 交互式 add

## 输出格式

支持三种输出模式，通过 `--json`、`--yaml` 或环境变量切换，适配不同自动化场景。
