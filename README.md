# Chain Hub 🔗

> **全链 AI Agent 服务入口**
>
> 一个 CLI，接入所有区块链服务。Agent 直接用，不管 SDK、API Key、认证。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: Design](https://img.shields.io/badge/Status-Design%20Phase-blue)]()

---

## 是什么

Chain Hub 是区块链版的 Stripe Projects——让 AI Agent 能用一行命令调用任何链上服务，而不需要持有任何 API Key 或管理复杂的 SDK。

```bash
chainhub add okx/dex              # 接入 OKX DEX
chainhub add aave/lend            # 接入 Aave 借贷
chainhub add chainlink/price      # 接入 Chainlink 预言机

chainhub use okx/swap --chain xlayer --from OKB --to USDC --amount 10
chainhub use aave/lend --chain eth --token USDC --amount 1000
chainhub use chainlink/price --token BTC
```

## 解决什么问题

每个 AI Agent 想使用链上服务，都要面对：注册账号、找 API Key、配置 SDK、处理认证……每个服务都不一样，全是为人设计的流程。

**Chain Hub 让 Agent 只需要有一个链上钱包，就能使用所有注册协议。**

```
现在：Agent → API Key A → 服务 A
             → API Key B → 服务 B
             → SDK C    → 服务 C

Chain Hub：Agent → 钱包签名 → Chain Hub → 服务 A / B / C / ...
```

## 核心设计

**双边平台**

```
协议开发者  ──→  提交 protocol.yaml  ──→  Chain Hub 注册表
                                              ↓
AI Agent    ←──  chainhub use/catalog  ←──  路由 + Key Vault
```

- **开发者**：提交 `protocol.yaml` 即完成接入，5 分钟上线，无需联系任何人
- **Agent**：`provider/service` 格式调用，换链只换 `--chain` 参数，命令永远不变
- **Key Vault**：API Key 加密托管，Agent 永不持有第三方凭证

**项目配置文件**

```yaml
# chainhub.yaml
name: "my-defi-agent"
chains: [xlayer, eth, solana]
services:
  - okx/dex
  - aave/lend
  - chainlink/price
  - thegraph/index
```

```bash
chainhub sync         # 一条命令同步所有协议访问凭证
chainhub status       # 查看调用量、余额、服务状态
chainhub catalog      # 浏览所有可用协议
```

## 多链支持

| Phase | 链 | 时间 |
|-------|-----|------|
| 1 | X-Layer、Ethereum、Base、Arbitrum、BSC | 0~3 个月 |
| 2 | Solana、Sui、Aptos | 3~9 个月 |
| 3 | Cosmos、TON + 跨链 Bridge | 9~18 个月 |

## 对比 Stripe Projects

| | Stripe Projects | Chain Hub |
|---|---|---|
| 目标服务 | Web2（Vercel、Neon、Supabase...） | 区块链协议（DEX、借贷、预言机...） |
| 服务接入 | 官方封闭 co-design | **任何协议开放注册** |
| 支付 | 法币，仅美/欧/英/加 | **链上，全球无限制** |
| 身份 | Stripe 账号 | 链上钱包，无需注册 |

## 与 Agent Arena 的关系

Chain Hub 是 **基础设施层**，Agent Arena 是 **协作层**：

```
Chain Hub（工具层）   Agent 用链上任何服务
     +
Agent Arena（市场层） Agent 接任务、竞争、自动结算
     =
完整的 Agent 经济网络基础设施
```

## 文档

完整产品设计见 [DESIGN.md](DESIGN.md)，包含：

- 协议注册表设计（链上 + IPFS 存储）
- Key Vault 三阶段路线（中心化 → Lit Protocol MPC → 完全去中心化）
- 信誉分算法（成功率 + 调用量 + 安全认证）
- 竞品分析（Stripe Projects 深度对比）
- 技术架构（Rust CLI + Go Server + 多链适配器）
- 盈利模式（订阅 + 路由费 + 接入费）

## 状态

**当前阶段：产品设计**

- [x] 产品定位与核心设计
- [x] 协议注册表设计
- [x] Key Vault 设计
- [x] 竞品分析
- [ ] Rust CLI 开发（Phase 1）
- [ ] 协议注册表合约
- [ ] Developer Portal

---

_Chain Hub 是 [Agent Arena](https://github.com/DaviRain-Su/agent-arena) 产品矩阵的基础设施层。_
