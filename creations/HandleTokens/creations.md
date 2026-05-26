# HandleTokens

## 核心原则：纯文本网关（Text-Only Gateway）

HandleTokens 是一个**严格意义上的纯文本代理**。
- **不支持任何形式的多模态输入**（图片、音频、视频、PDF 解析等）。
- **不支持多模态的理由**：无法离线计算 Token 消耗，无法准确预扣费，无法审计缓存命中率。
- **行为**：检测到非文本模态（如 `image_url`, `image` 类型）时，请求将被立即拒绝。

> 设计目标：做最稳的"文本 Token 防火墙"，而不是什么都包的瑞士军刀。

真实 provider key 永不出境。
HandleTokens 作为内网 AI 出口网关：
- 所有第三方模型调用统一路由到这里
- 真实 key 仅存 HT（env / vault / sealed file）
- 外部（IDE、CI、前端、开源 demo）只拿 HT 签发的「内网 key/token」
- 即便内网 token 泄漏，影响半径收敛为配额耗尽/被拒/可追溯，而非 provider key 裸奔

适合：开源项目、小团队、技术组织——
防止 .env 误提交、git history、CI log、打包泄漏导致真实 key 被盗用。

---

## 功能

### 1. 协议伪装（IDE 兼容）
- 对外暴露 **100% 兼容 OpenAI / Anthropic** 的 API Endpoint。
- 用户只需将 IDE（Cursor/Continue/Cline）的 API Base 指向 HT，无需修改请求逻辑。
- 鉴权头：`Authorization: Bearer sk-ht_xxxxxxxx`（HT 内网 Key）。

### 2. 密钥与权限
- 内网 Key 管理（CRUD、过期、禁用）。
- 细粒度权限：
  - RPM / TPM 限制
  - 总预算限制（金额/Token 数）
  - 模型白名单（如仅允许 `gpt-4o-mini`）

### 3. 计量与预扣费（保守策略）
- **数据源**：优先采信 Provider 返回的 `usage`（若可用）。
- **离线计算**：仅在 Provider 未返回时，使用官方 Tokenizer 进行**纯文本**估算。
- **缓存盲区**：因无法感知 Cache Hit/Miss，预扣费按 **Cache Miss（最高费率）** 计算，必然多扣。
- **多模态**：已全局禁用，不参与计量。

### 4. 审计与回溯
- 记录文本 Prompt 与 Completion（需配置开关，默认建议关闭以保隐私）。
- 记录 Request ID、耗时、模型、Key 归属。

---

## 明确不做（By Design）
- ❌ 模型杂交（Ensemble/Voting）。
- ❌ 多模态 Token 计量。
- ❌ Tauri 桌面端（仅 Web Console）。
- ❌ 试图精确匹配厂商最终账单（仅做预算护栏）。

---
---

目标一句话：
**Real keys stay in HT; the outside only gets scoped, capped, auditable tokens.**