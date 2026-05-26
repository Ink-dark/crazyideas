# HandleTokens

真实 provider key 永不出境。
HandleTokens 作为内网 AI 出口网关：
- 所有第三方模型调用统一路由到这里
- 真实 key 仅存 HT（env / vault / sealed file）
- 外部（IDE、CI、前端、开源 demo）只拿 HT 签发的「内网 key/token」
- 即便内网 token 泄漏，影响半径收敛为配额耗尽/被拒/可追溯，而非 provider key 裸奔

适合：开源项目、小团队、技术组织——
防止 .env 误提交、git history、CI log、打包泄漏导致真实 key 被盗用。

---

## Core（必须有）

### 1) 内网出口代理
- 仅允许来自内网/配置白名单的流量进入 HT
- HT 负责：鉴权 → 限流 → 路由 → 转发 → 记录 →（可选）预扣费

### 2) 内网 Key / Token 生命周期（可视化 CRUD）
每张 key 绑定：
- owner / project
- 限速（RPM/TPM/并发）
- 预算（次数 or 估算金额上限）
- 有效期（expireAt）
- 模型白名单/黑名单

### 3) 审计与经济（记准“发生了什么”）
每条调用记录至少：requestId、time、key、model、upstream、status、
promptTokens/completionTokens（优先 upstream usage）、估算金额、latency

### 4) 预扣费（预算护栏，非结算）
- 定义：用来拦超额，不保证等于 provider 最终账单
- 策略：保守上界（可能多扣）；定期对账修正差额
- 多模态：不计量（透传不计费 或 拒绝），避免“图片 token 黑洞”
- cache-awareness：HT 无法判定 upstream cache hit/miss → 按 miss 上界估（多扣是已知代价）

---

## Extensions（有用，但写清边界）

### 协议互转（OpenAI ↔ Anthropic）
- 先做 chat/completions 级兼容；images/files 标为 out-of-scope for metering
- 流式：尽量做 chunk 透传，避免强行反序列化 output 破坏厂商专有字段

### 余额自动查询
- 明确是“查 provider billing API”还是“查 HT 内部预算表”
- 若 provider 不给你 programmatic billing：就只做内部预算视图

---

## Explicitly not yet（by design）

- 不做 model 杂交（ensemble/vote/fuse）：HT 可做路由/降级/AB 测试，不做输出融合
- 不做 Tauri 控制台（运维/升级/审计链路断裂）：优先 Web console（只读 + 必要操作）
- 不做多-modal token 计量（image/audio/file）：要么透传不计量，要么拒

---

目标一句话：
**Real keys stay in HT; the outside only gets scoped, capped, auditable tokens.**
